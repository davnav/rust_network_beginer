Below example shows how to make a tokio async server. The program can accept multiple connections and process them through tokio::spawn .

```

use tokio::net::TcpListener ;
use tokio::io::{AsyncReadExt, AsyncWriteExt};


#[tokio::main]
async fn main() -> Result<(),Box<dyn std::error::Error>> {

    let listener = TcpListener::bind("127.0.0.1:6000").await?;

    loop{

            let (mut stream,addr) =  listener.accept().await?;
            println!("Tcp stream = {:?},addr = {:?}",stream,addr);

            tokio::spawn(async move {

                let mut buf = [0;1024];

                loop{
                    let n = match stream.read(&mut buf).await{
                        Ok(n) if n ==0 => return,
                        Ok(n) => n,
                        Err(e) => {
                            eprintln!("{}",e);
                            return
                        }

                    };
                }
            });

    }

}


```
