
How to make a api request from wasm.

Below is the example code following below examples documentaion.
https://rustwasm.github.io/docs/book/game-of-life/hello-world.html
https://rustwasm.github.io/wasm-bindgen/examples/fetch.html

Rust program:
```
mod utils;

use wasm_bindgen::prelude::*;
use serde::{ Deserialize, Serialize};
use wasm_bindgen::JsCast;
use wasm_bindgen_futures::JsFuture;
use web_sys::{Request,RequestInit,RequestMode,Response};



// When the `wee_alloc` feature is enabled, use `wee_alloc` as the global
// allocator.
#[cfg(feature = "wee_alloc")]
#[global_allocator]
static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;


#[derive(Debug,Serialize,Deserialize)]
pub struct Branch{
    pub name:String,
    pub commit: Commit,
}


#[derive(Debug,Serialize,Deserialize)]
pub struct Commit{
    pub sha:String,
    pub commit:CommitDetails,
}

#[derive(Debug,Serialize,Deserialize)]
pub struct CommitDetails{
    pub author:Signature,
    pub committer:Signature,
}


#[derive(Debug,Serialize,Deserialize)]
pub struct Signature{
    pub name:String,
    pub email:String,
}

#[wasm_bindgen]
pub async fn run() -> Result<JsValue,JsValue>{

    let mut opts = RequestInit::new();
    opts.method("GET");
    opts.mode(RequestMode::Cors);

    let url = format!("https://api.github.com/repos/{}/branches/master", "rustwasm/wasm-bindgen");
    let request = Request::new_with_str_and_init(&url, &opts)?;

    request
        .headers()
        .set("Accept", "application/vnd.github.v3+json")?;

    let window = web_sys::window().unwrap();
    let resp_value = JsFuture::from(window.fetch_with_request(&request)).await?;

    assert!(resp_value.is_instance_of::<Response>());

    let resp: Response = resp_value.dyn_into().unwrap();

    let json = JsFuture::from(resp.json()?).await?;


    let branch_info: Branch = json.into_serde().unwrap();

    println!("{:?}",branch_info);

    Ok(JsValue::from_serde(&branch_info).unwrap())
}




#[wasm_bindgen]
extern {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet() {
    alert("Hello, wasmexp!");
}





```


index.js

```
import * as wasm from "wasmexp";

wasm.greet();
wasm.run().then(
    (data) => { 
        console.log(data.name); 
        console.log(data.commit.commit.author.name); 
    
    
    }
).catch(console.error);

```
