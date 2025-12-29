---
title: "Quote of the Day: The easiest protocol"
author: Phineas Jensen
date: 2025-09-08
description: "Port List—QOTD—RFC 865—Rust implementation"
tags:
  - rust
  - networking
---

I recently found myself browsing the [List of TCP and UDP Port Numbers](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers) on Wikipedia. Knowing what a few ports are reserved or commonly used for (such as 80 for HTTP, 443 for HTTPS, and 22 for SSH), I was wondering what the other ~1000 well-known ports are used for. There are a lot of interesting protocols listed on that page, but one that quickly jumped out to me was the [Quote of the Day](https://en.wikipedia.org/wiki/QOTD) (QOTD) protocol, which uses port 17.

The Wikipedia page for QOTD is short and sweet and it made me wonder how quickly I could implement my own QOTD server program based on [RFC 865](https://www.rfc-editor.org/rfc/rfc865), which standardized the protocol, in Rust. The RFC is delightfully short and easy to read, so I'll quote it here in its entirety:

> Quote of the Day Protocol
> 
> This RFC specifies a standard for the ARPA Internet community.  Hosts on
> the ARPA Internet that choose to implement a Quote of the Day Protocol
> are expected to adopt and implement this standard.
> 
> A useful debugging and measurement tool is a quote of the day service.
> A quote of the day service simply sends a short message without regard
> to the input.
> 
> TCP Based Character Generator Service
> 
>    One quote of the day service is defined as a connection based
>    application on TCP.  A server listens for TCP connections on TCP port
>    17.  Once a connection is established a short message is sent out the
>    connection (and any data received is thrown away).  The service
>    closes the connection after sending the quote.
> 
> UDP Based Character Generator Service
> 
>    Another quote of the day service is defined as a datagram based
>    application on UDP.  A server listens for UDP datagrams on UDP port
>    17.  When a datagram is received, an answering datagram is sent
>    containing a quote (the data in the received datagram is ignored).
> 
> Quote Syntax
> 
>    There is no specific syntax for the quote.  It is recommended that it
>    be limited to the ASCII printing characters, space, carriage return,
>    and line feed.  The quote may be just one or up to several lines, but
>    it should be less than 512 characters.

So I used `cargo new` to start a new Rust project, opened up `main.rs` in my editor and the Rust standard library documentation in my browser, and started looking at the [`std::net::TcpListener`](https://doc.rust-lang.org/std/net/struct.TcpListener.html) struct documentation to learn how to do basic TCP networking in Rust. After copying the example program on that page and adding a `writeln!` call to the `handle_client` function, I realized I had created a fully compliant QOTD server:

```rust
use std::{
    io::Write,
    net::{TcpListener, TcpStream},
};

fn handle_client(mut stream: TcpStream) -> std::io::Result<()> {
    writeln!(&mut stream, "Heyo!")
}

fn main() -> std::io::Result<()> {
    let listener = TcpListener::bind("0.0.0.0:17")?;
    for stream in listener.incoming() {
        handle_client(stream?)?;
    }
    Ok(())
}
```

Let's review the RFC and make a checklist of what a QOTD service needs:

- [x] A TCP or UDP service listening on port 17
- [x] For TCP, once a connection is established, a short message is sent out and the service closes the connection
- [x] The quote is less than 512 characters

That's it! It never says the quote actually has to change daily, or be a quote of anything at all; just that it's a "short message" sent on port 17. "Heyo!" doesn't communicate much, but I'd call it a short message. 

I can run this server with `cargo run` and communicate with it using `nc` and get this deeply impactful quote whenever I desire:

```bash
$ cargo run &
$ nc localhost 17
Heyo!
```
