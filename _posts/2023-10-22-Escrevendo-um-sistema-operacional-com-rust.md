---
layout: post
title: "Escrevendo um sistema operacional com rust"
date: 2023-10-22 22:10:00 -0300
categories: ciencia programacao os rust
---

Olá amates de low level, hoje vamos começar um longa jornada onde vou apresentar pra voces minha trajetoria escrevendo um sistema operacional simples em rust. Vou utilizar o tutorial do [os.phil-opp.com](https://os.phil-opp.com/freestanding-rust-binary/) para seguir essa jornada vou tentar melhorar o possivel no codigo dele lembrando que não sou um programador muito experiente em rust e nem sei tanto sobre criação de OS isso vai ser apenas um experimento meu que quero compartilhar com vcs.

# 1. Por onde começar

A primeira coisa que precisamos fazer para criar nosso sistema operacional e construir nosso kernel ou seja um executavel rust que não está vinculado a nenhuma biblioteca padrão. Isso fazendo possivel rodar o executavel no [barel metal](https://en.wikipedia.org/wiki/Bare_machine)

Vamos começar criando o projeto:

```
  cargo new os --bin --edition 2018
```

## Atributo `no_std`

Esse é um atributo no rust que desabilita o uso da biblioteca padrão do rust.

```rs
// main.rs
#![no_std]

fn main() {
    println!("Hello, world!");
}
```

Se executarmos `cargo build` e esperado que tenhamos o seguinte erro.

```bash
italo@italo:~/os$ cargo build
   Compiling os v0.1.0 (/home/italo/os)
error: cannot find macro `println` in this scope
 --> src/main.rs:4:5
  |
4 |     println!("Hello, world!");
  |     ^^^^^^^

error: `#[panic_handler]` function required, but not found

error: language item required, but not found: `eh_personality`
  |
  = note: this can occur when a binary crate with `#![no_std]` is compiled for a target where `eh_personality` is defined in the standard library
  = help: you may be able to compile for a target that doesn't need `eh_personality`, specify a target with `--target` or in `.cargo/config`

error: could not compile `os` due to 3 previous errors
```

A razao disso é que println é um macro da lib padrão.

```rs
// main.rs
#![no_std]

fn main() {

}
```

Se tentarmos buildar novamente teremos o erro.

```bash
italo@italo:~/os$ cargo build
   Compiling os v0.1.0 (/home/italo/os)
error: `#[panic_handler]` function required, but not found

error: language item required, but not found: `eh_personality`
  |
  = note: this can occur when a binary crate with `#![no_std]` is compiled for a target where `eh_personality` is defined in the standard library
  = help: you may be able to compile for a target that doesn't need `eh_personality`, specify a target with `--target` or in `.cargo/config`

error: could not compile `os` due to 2 previous errors
```

O panic_handler atributo define a função que o compilador deve invocar quando ocorrer um pânico . A biblioteca padrão fornece sua própria função de tratamento de pânico, mas em um no_stdambiente precisamos defini-la nós mesmos:

```rs
#![no_std]

fn main() {
}

use core::panic::PanicInfo;

/// This function is called on panic.
#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}
}
```

Alem disso temos que desabilitar o Unwinding.

```toml
# Cargo.toml

[profile.dev]
panic = "abort"

[profile.release]
panic = "abort"
```

Buildando novamente:

```bash
italo@italo:~/os$ cargo build
   Compiling os v0.1.0 (/home/italo/os)
error: requires `start` lang_item

error: could not compile `os` due to previous error
```

## Atributo start

Em rust pensamos que a função `main` e a primeira função que é execultada. Porem a maioria das linguagens possui um [Runtime system](https://en.wikipedia.org/wiki/Runtime_system) utilizado para fazer varias coisas como coleta de lixo como no Java ou threads de software como o goroutine em Go. Esse Runtime System é executado antes do main.
Em um binario Rust que esta vinculado a lib padrão a execução começa em um lib de runtime system C chamada `crt0`(“C runtime zero”) que configura o ambiente para o aplicativo C isso inclui criação de pilha e correções de argumentos nos registradores. O Runtime system invoca o [ponto de entrada em tempo de execução](https://github.com/rust-lang/rust/blob/bb4d1491466d8239a7a5fd68bd605e3276e97afb/src/libstd/rt.rs#L32-L73) que esta marcado como `start`

Como não temos acesso a crt0 pois nosso binario rust é independente vamos ter criar nosso ponto de entrada.

Adicione o atributo `#![no_main]`.

```rust
#![no_std]
#![no_main]

use core::panic::PanicInfo;


#[no_mangle]
pub extern "C" fn _start() -> ! {
    loop {

    }
}

/// This function is called on panic.
#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}
}
```

Ao usar o #[no_mangle] atributo, desabilitamos a manipulação de nomes para garantir que o compilador Rust realmente produza uma função com o nome `\_start`. Sem o atributo, o compilador geraria algum `\_ZN3blog_os4_start7hb173fedf945531caE` símbolo enigmático para dar a cada função um nome exclusivo. O atributo é obrigatório porque precisamos informar o nome da função do ponto de entrada ao vinculador na próxima etapa.

Tambem usar [Calling convention](https://en.wikipedia.org/wiki/Calling_convention) com `extern "C"` para informar ou compilador que ele deve usar essa funcão como `start`

Quando executamos cargo buildagora, obtemos um erro feio no vinculador .

### Erros do vinculador

O vinculador é um programa que combina o código gerado em um executável. Como o formato executável difere entre Linux, Windows e macOS, cada sistema possui seu próprio vinculador que gera um erro diferente. A causa fundamental dos erros é a mesma: a configuração padrão do vinculador assume que nosso programa depende do tempo de execução C, o que não acontece.

Para resolver os erros, precisamos informar ao vinculador que ele não deve incluir o tempo de execução C. Podemos fazer isso passando um determinado conjunto de argumentos para o vinculador ou construindo para um alvo bare metal.

### 🔗Construindo para um alvo Bare Metal

Por padrão, o Rust tenta construir um executável que seja capaz de ser executado no ambiente atual do sistema. Por exemplo, se você estiver usando o Windows no x86_64, o Rust tentará criar um .exeexecutável do Windows que use x86_64instruções. Este ambiente é chamado de sistema “host”.

Para descrever diferentes ambientes, Rust usa uma string chamada target triple . Você pode ver o triplo alvo do seu sistema host executando rustc --version --verbose:

```
italo@italo:~/os$ rustc --version --verbose
rustc 1.66.1 (90743e729 2023-01-10) (built from a source tarball)
binary: rustc
commit-hash: 90743e7298aca107ddaa0c202a4d3604e29bfeb6
commit-date: 2023-01-10
host: x86_64-unknown-linux-gnu
release: 1.66.1
LLVM version: 15.0.7
```

Ao compilar para nosso host triplo, o compilador Rust e o vinculador assumem que existe um sistema operacional subjacente, como Linux ou Windows, que usa o tempo de execução C por padrão, o que causa erros no vinculador. Portanto, para evitar erros do vinculador, podemos compilar para um ambiente diferente sem sistema operacional subjacente.

Um exemplo de ambiente bare metal é o thumbv7em-none-eabihfalvo triplo, que descreve um sistema ARM embarcado . Os detalhes não são importantes, tudo o que importa é que o triplo alvo não tenha nenhum sistema operacional subjacente, o que é indicado pelo no triplo alvo. Para poder compilar para este alvo, precisamos adicioná-lo no Rustup:none

`rustup target add thumbv7em-none-eabihf`

Isso baixa uma cópia da biblioteca padrão (e principal) do sistema. Agora podemos construir nosso executável independente para este alvo:

`cargo build --target thumbv7em-none-eabihf`

Agora sim roda de boa.

Como ficou tudo ?

`src/main`:

```rs
#![no_std]
#![no_main]

use core::panic::PanicInfo;

#[no_mangle]
pub extern "C" fn _start() -> ! {
    loop {}
}

/// This function is called on panic.
#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}
}
```

`Cargo.toml`:

```t
[package]
name = "blog_os"
version = "0.1.0"
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html


[profile.dev]
panic = "abort"

[profile.release]
panic = "abort"

[dependencies]

```

Para buildar use `cargo build --target thumbv7em-none-eabihf`

para compilar para host especificos use:

```bash
# Linux
cargo rustc -- -C link-arg=-nostartfiles
# Windows
cargo rustc -- -C link-args="/ENTRY:_start /SUBSYSTEM:console"
# macOS
cargo rustc -- -C link-args="-e __start -static -nostartfiles"
```

Esse foi o primeira etapa fique atento a novas postagens.
