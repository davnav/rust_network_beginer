

How to set up Oauth using Rust

```
use anyhow;
use oauth2::basic::BasicClient;
use oauth2::reqwest::http_client;
use oauth2::{
    AuthUrl, AuthorizationCode, ClientId, ClientSecret, CsrfToken, PkceCodeChallenge, RedirectUrl,
    Scope, TokenResponse, TokenUrl,
};
use url::Url;
use std::env;

use std::net::TcpListener;
use std::io::{BufReader, BufRead, Write};

fn main() {

    let client_id = env::var("Client").unwrap();
    // let client_id = client_id.unwrap();
    let client_sec = env::var("CLIENTSEC").unwrap();
    let client = BasicClient::new(
        ClientId::new(client_id),
        Some(ClientSecret::new(client_sec.to_string())),
        AuthUrl::new("https://hackaday.io/authorize".to_string()).unwrap(),
        Some(TokenUrl::new("https://hackaday.io/access_token".to_string()).unwrap())
    )
    // Set the URL the user will be redirected to after the authorization process.
    .set_redirect_uri(RedirectUrl::new("http://localhost:8080".to_string()).unwrap());
    
    println!("{:?}",client);
    // Generate a PKCE challenge.
    let (pkce_challenge, pkce_verifier) = PkceCodeChallenge::new_random_sha256();

    // Generate the full authorization URL.
    let (auth_url, csrf_token) = client
        .authorize_url(CsrfToken::new_random)
        .set_pkce_challenge(pkce_challenge)
        .url();
    println!("{:?}",auth_url);

        let (authorize_url, csrf_state) = client
        .authorize_url(CsrfToken::new_random)
        .url();
        
    println!("open the url in browser {}",authorize_url.to_string());
    let listener = TcpListener::bind("127.0.0.1:8080").unwrap();

    println!("connected to 127.0.0.1:8080");
    println!("{:?}",listener);
    for stream in listener.incoming() {

        println!("stream came = {:?}",stream);

        if let Ok(mut stream) = stream {
            let code;
            {
                let mut reader = BufReader::new(&stream);

                let mut request_line = String::new();
                reader.read_line(&mut request_line).expect("reading failed from stream");

                let redirect_url = request_line.split_whitespace().nth(1).unwrap();

                println!("{}",redirect_url);

                let url = Url::parse(&("http://localhost".to_string() + redirect_url)).unwrap();

                

                println!("{}",url);                

                let code_pair = url
                    .query_pairs()
                    .find(|pair| {
                        let &(ref key, _) = pair;
                        key == "code"
                    })
                    .unwrap();

                let (_, value) = code_pair;

                println!("{}",value);
                code = AuthorizationCode::new(value.into_owned());

            }

            let message = "Go back to your terminal :)";
            let response = format!(
                "HTTP/1.1 200 OK\r\ncontent-length: {}\r\n\r\n{}",
                message.len(),
                message
            );
            stream.write_all(response.as_bytes()).unwrap();
            
                        // Exchange the code with a token.
                        let token_res = client.exchange_code(code).request(http_client);

                        if let Ok(token) = token_res {
                            let scopes = if let Some(scopes_vec) = token.scopes() {
                                scopes_vec
                                    .iter()
                                    .map(|comma_separated| comma_separated.split(','))
                                    .flatten()
                                    .collect::<Vec<_>>()
                            } else {
                                Vec::new()
                            };
                        }
            
                        break;
                    }




        }    
}
```
