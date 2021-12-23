
This is example for how to build a chat server in Rust language with synchronus programming .

```
use std::error::Error;
use std::io::prelude::*;
use std::net::{TcpListener, TcpStream};
use std::sync::{Arc, Mutex};
use std::thread;
use crossbeam_channel::{select, unbounded, Receiver, Sender};



fn sleep() {
    thread::sleep(::std::time::Duration::from_millis(100));
}

fn main() {
    let listener = TcpListener::bind("127.0.0.1:6000").unwrap();

    let (s, r) = crossbeam_channel::unbounded::<String>();
    let mut clients = vec![];

    let mut clientArc = Arc::new(Mutex::new(clients));
    for mut stream in listener.incoming() {
        println!("connection establised {:?}", stream);
        let stream = stream.unwrap();
        let clientArc1 = Arc::clone(&clientArc);
        let mut clientm = &mut *clientArc1.lock().unwrap();
        clientm.push(stream.try_clone().expect("err cloning stream"));
        let s1 = s.clone();
        let r1 = r.clone();
        let clientArc2 = Arc::clone(&clientArc);
        handle_connection(s1, r1, stream, clientArc2);


    }
}

fn handle_connection(
    s: Sender<String>,
    r: Receiver<String>,
    mut stream: TcpStream,
    clientArc: Arc<Mutex<Vec<TcpStream>>>,
) {
    
    thread::spawn(move || loop {
        let mut buffer = [0; 1024];
        let bytes_read = stream.read(&mut buffer).unwrap();
        // if bytes_read == 0{
        //     break;
        // }
        let msg = String::from_utf8_lossy(&buffer);

        s.send(msg.to_string());
     

        if let Ok(msg) = r.recv() {
            println!("{}", msg);
            let mut buff = msg.clone().into_bytes();
            for client_stream in clientArc.lock().unwrap().iter_mut() {
                client_stream.write_all(&buff);
            }
        }
    });
}
```
