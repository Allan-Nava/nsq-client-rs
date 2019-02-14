# NSQ Rust client [![Build Status](https://travis-ci.com/alex179ohm/nsq-client-rs.svg?branch=master)](https://travis-ci.com/alex179ohm/nsq-client-rs) [![Build status](https://ci.appveyor.com/api/projects/status/ov5ryj2r4iy2v7rp/branch/master?svg=true)](https://ci.appveyor.com/project/alex179ohm/nsq-client-rs/branch/master)

A [Actix](https://actix.rs/) based client implementation for the [NSQ](https://nsq.io) realtime message processing system.
Nsq-client it's designed to support by default multiple Readers for Multiple Connections, readers are routed per single connection by a round robin algorithm.

## Examples
- [Simple Processing Message](https://github.com/alex179ohm/nsqueue/example/reader)
- [Simple Consumer](https://github.com/alex179ohm/nsqueue/example/consumer)
- [wesocket tuning Consumer](https://github.com/alex179ohm/example/ws-consumer)


### Simple Reader (SUB)
```rust
extern crate nsqueue;
extern crate actix;

use std::sync::Arc;

use actix::prelude::*;

use nsqueue::{Connection, Msg, Fin, Subscribe, Config};

struct MyReader {
	pub conn: Arc<Addr<Connection>>,
}

impl Actor for MyReader {
    type Context = Context<Self>;
    fn started(&mut self, ctx: &mut Self::Context) {
        self.subscribe::<Msg>(ctx, self.conn.clone());
    }
}

impl Handler<Msg> for MyReader {
    fn handle(&mut self, msg: Msg, _: &mut Self::Context) {
        println!("MyReader received {:?}", msg);
        self.conn.do_send(Fin(msg.id));
    }
}

fn main() {
    let sys = System::new("consumer");
    let config = Config::default().client_id("consumer");
    let c = Supervisor::start(|_| Connection::new(
        "test", // <- topic
        "test", // <- channel
        "0.0.0.0:4150", // <- nsqd tcp address
        Some(config), // <- config (Optional)
	Some(2) // <- RDY (Optional default: 1)
    ));
    let conn = Arc::new(c);
    let _ = MyReader{ conn: conn.clone() }.start(); // <- Same thread reader
    let _ = Arbiter::start(|_| MyReader{ conn: conn }); // <- start another reader in different thread
    sys.run();
}
```
### launch the reader
```bash
$ RUST_LOG=nsq-client=debug cargo run
```

### Current features and work in progress
- [X] PUB
- [X] SUB
- [ ] Discovery
- [X] Backoff
- [ ] TLS
- [ ] Snappy
- [X] Auth
- [ ] First-ready-first-served readers routing algorithm.

## License

Licensed under
* MIT license (see [LICENSE](LICENSE) or <http://opensource.org/licenses/MIT>)