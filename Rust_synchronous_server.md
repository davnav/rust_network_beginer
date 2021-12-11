Below example of Rust Code can accept multiple connects and respond to it

```
use std::error::Error;
use std::net::{TcpListener, TcpStream};
use std::thread;
use std::io::prelude::*;



fn main() {
    let listener = TcpListener::bind("127.0.0.1:8080").unwrap();
    
    for stream in listener.incoming(){
        println!("connection establised {:?}",stream);
        let stream = stream.unwrap();
        handle_connection(stream);
    }

}

fn handle_connection(mut stream:TcpStream ){

    let get = "";
    let mut buffer = [0;1024];

    thread::spawn( move ||{
        loop{
            let req = stream.read(&mut buffer).unwrap();
            println!("Request:{}",String::from_utf8_lossy(&buffer) );
        }

    });
    
}
```
