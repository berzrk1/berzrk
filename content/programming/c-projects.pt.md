+++
title = "C Projects"
date = "2026-08-28"
author = "berzrk"
description = "Alguns projetos que fiz em C"
+++

# Projetos

## Philips Hue Controller + HTTP Client

Esse é um projeto interessante porque poderia facilmente ter sido dois projetos.
Ele inclui um **HTTP client** simples capaz de fazer requisições `GET` e `PUT`,
e um client para interagir com a API do Philips Hue.

Meu foco nesse projeto foi não ter nenhuma dependência, exceto o GnuTLS, porque a
API v2 exige HTTPS.

Mas a montagem das requisições HTTP e o tratamento das respostas, além da
manipulação de JSON, foram feitos em C puro. A parte mais difícil foi com certeza
lidar com **chunked transfer** no HTTP/1.1.

Infelizmente não consegui melhorar mais porque minhas lâmpadas quebraram :(, então
não tinha mais motivo para usar.

<a href="https://github.com/berzrk1/Philips-HUE-in-C" class="button inline">GitHub</a>

## Chip 8 Emulator

**Chip 8 Emulator**, que é o Hello World do desenvolvimento de emuladores.

É bem interessante escrever as **instruções de CPU** como _código_, e também é um
uso prático e interessante (pelo menos para mim) de **bit manipulation**.

Ainda há melhorias para fazer. Substituir o `switch` case por uma tabela de
**function pointers**, para que o lookup seja mais rápido e mais organizado.

<a href="https://github.com/berzrk1/chip8emulator" class="button inline">GitHub</a>

## Shell

Implementei uma shell simples capaz de fazer **piping**, **redirecionamento de I/O**
e uma pequena seleção de comandos built-in: `exit`, `cd`, `history`.

Foi minha primeira tentativa de **systems programming**, onde tive que usar
bastante **system calls** como `fork`, `exec`, `dup`.

<a href="https://github.com/berzrk1/Mini-Shell-in-C" class="button inline">GitHub</a>
