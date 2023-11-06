---
layout: post
title: "Escrevendo um sistema operacional com rust part 3"
date: 2023-11-06 11:30:00 -0300
categories: ciencia programacao os rust
---

No final teremos o json assim.

```json
{
  "llvm-target": "x86_64-unknown-none",
  "data-layout": "e-m:e-i64:64-f80:128-n8:16:32:64-S128",
  "arch": "x86_64",
  "target-endian": "little",
  "target-pointer-width": "64",
  "target-c-int-width": "32",
  "os": "none",
  "executables": true,
  "linker-flavor": "ld.lld",
  "linker": "rust-lld",
  "panic-strategy": "abort",
  "disable-redzone": true,
  "features": "-mmx,-sse,+soft-float"
}
```
Mesmo assim se rodarmos `cargo build --target x86_64-blog_os.json` vai dar error:

        [italo@italo blog_os]$ cargo build --target x86_64-blog_os.json
        Compiling blog_os v0.1.0 (/home/italo/Documentos/api/blog_os)
        error[E0463]: can't find crate for `core`

Falha! O erro nos informa que o compilador Rust não encontra mais a core biblioteca . Esta biblioteca contém tipos básicos de Rust, como Result, Option e iteradores, e está implicitamente vinculada a todas as no_std caixas.

Vamos espeficicar no `.cargo/config.toml`
```toml
[unstable]
build-std = ["core", "compiler_builtins"]
```

Isso falar para o compilar recompilar essas duas libs

rodando o build novamente tava dando erro na minha maquina. Rodei esses dois comandos.

```bash
rustup override set nightly
rustup component add rust-src --toolchain nightly-x86_64-unknown-linux-gnu
```

E depois buildou

## Interferencias com relação a memoria

O compilador Rust assume que existe um conjunto de funcões integradas e diponiveis a todo o sistema. A marioria das funções é fornecida em `compiler_builtins`. No entanto algumas funções não são habilitadas por padrão pq normalmente são fornecidads pela biblioteca C do sistema. Como `memset`, `memcpy`, `memcmp`. 

Como não podemos vincula elas na lib C do sistema precisa de uma forma alternatica de fornecer essas funcões para o compilador.
Felizmente o `compiler_builtins` ja tem implementações para todas essas funções e apenas estão desabilitadas para não colidirem com as do lib C.

`.cargo/config.toml`
```toml
[build]
target = "x86_64-blog_os.json"

[unstable]
build-std-features = ["compiler-builtins-mem"]
build-std = ["core", "compiler_builtins"]
```

Adicione tbm o build para evitar de usar --target no build

## Escrevendo na tela

A maneira mais facil de imprimir texto na tela neste estagio é o buffer de texto VGA. É uma area de memoria especial mapeada para o hardware VGA que contem o conteudo exibido na tela. Normalmente consiste em 25 linhas, cada uma conteudo celulas de 80 chars e cada celula exibe um ASCII

A implementação fica assim:

```rs
static HELLO: &[u8] = b"Hello World!";

#[no_mangle]
pub extern "C" fn _start() -> ! {
    let vga_buffer = 0xb8000 as *mut u8;

    for (i, &byte) in HELLO.iter().enumerate() {
        unsafe {
            *vga_buffer.offset(i as isize * 2) = byte;
            *vga_buffer.offset(i as isize * 2 + 1) = 0xb;
        }
    }

    loop {}
}
```

Primeiro convertemos o numero `0xb8000` para um ponteiro bruto. Em seguida iteramos sobre os bytes da string estatica. Dentro do for usamos o metodo para escrever byte na string e a cor dela.

## Executando nosso Kernel

Agora que temos um executável que faz algo perceptível, é hora de executá-lo. Primeiro, precisamos transformar nosso kernel compilado em uma imagem de disco inicializável, vinculando-o a um gerenciador de inicialização. Então podemos executar a imagem do disco na máquina virtual QEMU ou inicializá-la em hardware real usando um pendrive.

Em vez de escrever nosso próprio bootloader, que é um projeto por si só, usamos o bootloader crate. Esta caixa implementa um bootloader BIOS básico sem nenhuma dependência C, apenas Rust e montagem embutida. Para usá-lo para inicializar nosso kernel, precisamos adicionar uma dependência nele:

`Cargo.toml`
```toml
[dependencies]
bootloader = "0.9.23"
```
Run:
```bash
cargo install bootimage
cargo bootimage
```

Execulte `qemu-system-x86_64 -drive format=raw,file=target/x86_64-blog_os/debug/bootimage-blog_os.bin` e vc vera seu os rodar.

para pendrive:
`dd if=target/x86_64-blog_os/debug/bootimage-blog_os.bin of=/dev/sdX && sync`
