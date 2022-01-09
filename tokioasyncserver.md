Below example shows how to make a tokio async server. The program can accept multiple connections and process them through tokio::spawn .

```
use tokio::net::TcpListener ;
use tokio::io::{AsyncReadExt, AsyncWriteExt};
use tokio::fs::File;

#[tokio::main]
async fn main() -> Result<(),Box<dyn std::error::Error>> {

    let listener = TcpListener::bind("127.0.0.1:6000").await?;

    loop{

            let (mut stream,addr) =  listener.accept().await?;
            println!("Tcp stream = {:?},addr = {:?}",stream,addr);

            let mut file = File::open("hello.html").await?;
            let mut contents = String::new();
            file.read_to_string(&mut contents).await?;
            
            let response = format!("HTTP/1.1 200 OK\r\n\r\n{}", contents);

            tokio::spawn(async move {

                let mut buf = [0;1024];

                loop{
                    match stream.read(&mut buf).await{
                        Ok(n) if n ==0 => return,
                        Ok(n) => n,
                        Err(e) => {
                            eprintln!("{}",e);
                            return
                        }

                    };

                    match stream.write_all(response.as_bytes()).await{

                        Ok(()) => return,
                        Err(e) => {
                            eprintln!("{}",e);
                            return
                        },
                    };                
                }
            });

    }

}


```
