+++
title = "C Projects"
date = "2026-08-28"
author = "berzrk"
description = "Some projects I did in C"
+++

# Projects

## Philips Hue Controller + HTTP Client

This is an interesting project because it could have been easily two projects.
It includes a simple **HTTP client** capable of `GET` and `PUT` requests,
and a client to interact with Philips Hue API.

My focus on this project was to have no dependency, except for GnuTLS because the v2 API
required HTTPS.

But the craft of HTTP requests and handling of responses and also the manipulation of JSON was
done in pure C. The hardest part was definitely handling **chunked transfer** in HTTP/1.1.

Sadly I couldn't improve it anymore because my lights broke :(, so I had no reason to use.

<a href="https://github.com/berzrk1/Philips-HUE-in-C" class="button inline">GitHub</a>

## Chip 8 Emulator

**Chip 8 Emulator** which is the Hello World of emulator development.

It is very interesting to write **CPU instructions** as _code_
and also a practical and interesting use (at least for me) for
**bit manipulation**.

There are still improvements to be made. Replace the `switch`
case with a table of **function pointers**, so the lookup
is faster and more organized.

<a href="https://github.com/berzrk1/chip8emulator" class="button inline">GitHub</a>

## Shell

I implemented a simple shell that is capable of **piping**, **I/O redirection**
and a small selection of built-in commands: `exit`, `cd`, `history`.

It was my first attempt at **systems programming** where I had to make
use of a lot of **system calls** like `fork`, `exec`, `dup`.

<a href="https://github.com/berzrk1/Mini-Shell-in-C" class="button inline">GitHub</a>
