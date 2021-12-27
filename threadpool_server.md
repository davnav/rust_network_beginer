
How to use a threadpool to serve files from server. 

```
use std::fs::File;
use std::hash::Hasher;
use std::io::{Read, Write};
use std::net::{TcpListener, TcpStream};
use std::sync::{mpsc, Arc, Mutex};
use std::thread::{self, *};
use std::time;

fn handle_connection(mut connection: TcpStream) {
    let mut buff = [0; 1024];

    let mut file = File::open("hello.html").unwrap();
    let mut contents = String::new();
    file.read_to_string(&mut contents).unwrap();
    let response = format!("HTTP/1.1 200 OK\r\n\r\n{}", contents);

    //intentionally creating some delay using sleep function.
    let minutes = time::Duration::from_secs(10);
    let minutes = thread::sleep(minutes);

    connection.read(&mut buff).unwrap();
    connection.write(response.as_bytes()).unwrap();
    println!("{}", String::from_utf8_lossy(&buff[..]));
}

// a type which is executable function and send across threads
type Job = Box<dyn FnBox + Send + 'static>;


// special trait( a trick) require to deference a box value
// Box values needs to be owned first , before making changes or executing it
trait FnBox {
    fn call(self: Box<Self>);
}
impl<F: FnOnce()> FnBox for F {
    fn call(self: Box<Self>) {
        (*self)();
    }
}
struct ThreadPool {
    worker: Vec<thread::JoinHandle<()>>,
    sender: mpsc::Sender<Job>,
}

impl ThreadPool {
    //creating a new threadpool
    //while creating a threadpool, we spawn 4 threads which keep looping to recv end of the channel
    fn new() -> Self {
        let (tx, rx) = mpsc::channel::<Job>();
        let mut workers = vec![];
        /// make the reciever a thread safe shared reference so that we can use it in mutiple threads
        let mut rx_arc = Arc::new(Mutex::new(rx));
        let tx = tx.clone();

        // Four threads handles stored  in a threadpool
        for i in 1..4 {
            let rx_arc = Arc::clone(&rx_arc);

            let join_handle: JoinHandle<_> = thread::spawn(move || loop {
                let job = rx_arc.lock().unwrap().recv().unwrap();

                job.call();
            });

            workers.push(join_handle);
        }

        ThreadPool {
            worker: workers,
            sender: tx,
        }
    }

    //execute functionality for threadpool
    fn execute<F>(&self, f: F)
    where
        F: FnBox + Send + 'static,
    {
        let job = Box::new(f);

        // we are just sending the handle connection from main thread which will be received 
        // in any of the four thread stored in thread pool as we are using recv only on those four threads
        
        self.sender.send(job).unwrap();
    }
}

fn main() {
    let listener = TcpListener::bind("127.0.0.1:6000").expect("binding to server failed");
    println!("listening on port 6000");
    
    //create threadpool which has the capability to store four threads
    let pool = ThreadPool::new();

    //wheneven a clients connects , below listener logic detect it and serve
    //using the threadpool execute logic 
    for stream in listener.incoming() {
        let connection = stream.unwrap();
        println!("connectoin established");
        pool.execute(|| {
            handle_connection(connection);
        });
    }
}

```
