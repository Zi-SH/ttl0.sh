---
title: "xkpass-io"
status: "Active"
weight: 101
---
An XKCD-inspired passphrase generator, written in Rust and intented for command line. 

https://xkpass.io

<!--more-->

Originally created xkpasswd.net was going through a rewrite, xkpass-io was my first attempt at a project in Rust. 

It was inspired by sites such as canihazip and cheat.sh, and offers a default generation of acceptable strength as well as customization via parameters. 

Given that it's working with passwords, the backend is stateless and there's no query logging for requests. 