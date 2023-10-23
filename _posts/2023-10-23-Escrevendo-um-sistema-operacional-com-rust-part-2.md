---
layout: post
title: "Escrevendo um sistema operacional com rust part 2"
date: 2023-10-23 11:30:00 -0300
categories: ciencia programacao os rust
---

Olá amigos vamos começar o segundo passo do nosso sistema operacional. Na parte inicial
ja temos um programa rust independente com ponto de entrada e funcao de panic configurados alem de 
podermos compilar para qualquer arquitetura.

Nesse post vamos criar uma kernel de no minimo 64 bits para arquitetura x86 baseado no nosso binario independente.

## O processo de inicialização

Quando ligamos o computador, ele começa a execultar codigo do firmware armazenado na [ROM](https://en.wikipedia.org/wiki/Read-only_memory) da placa mãe. Este codigo executa um [Power-on self-test](https://en.wikipedia.org/wiki/Power-on_self-test), detectar a RAM disponivel e pre-inicializa a CPU e o hardware. Depois, procura um disco inicializável e inicia a kernel do sistema operacional.

No x86 existem dois padrões de firmware: o "Basic Input/Output System" [BIOS](https://en.wikipedia.org/wiki/BIOS) e o mais recente "Unified Extensible Firmware Interface" [UEFI](https://en.wikipedia.org/wiki/UEFI). O padrão BIOS é antigo e desatualizado, mas simples e bem suportado em qualquer máquina x86 desde a década de 1980. O UEFI, por outro lado, é mais moderno e tem muito mais recurso, mas é mais complexo de configurar.

Atualmente, fornecemos apenas suporte para BIOS.

## Inicialização do BIOS

Quase todas os sistemas x86 tem suporte para inicialização BIOS, incluindo maquinas mais recentes baseadas em UEFI que usam BIOS emulado. Isso é otimo por que vc pode usar a mesma logica de inicialização em todas as maquinas do seculo passado. Mas essa ampla compatibilidade é ao mesmo tempo a maior desvantagem da inicialização BIOS, por que significa que a CPU é colocada em um modo de compatibilidade de 16 bits chamado modo [real](https://en.wikipedia.org/wiki/Real_mode) antes da inicialização, para que os carregadores de inicialização arcaicos da decada de 1980 ainda funcionem.

Vamos começar do inicio:

Quando vc liga o computador ele carrega a BIOS de alguma memoria flash especial localizada na placa mae. O BIOS executa rotinas de autoteste e inicialização do hardware e, em seguida, procura disco inicializavel. Se encontrar um, o controler é transferido para o seu bootloader, que é uma porção de 512 bytes de codigo executavel armazenado no inicio do disco. A maioria dos bootloaders tem mais de 512 bytes, entt os bootloaders são comumente divididos em um pequeno primeiro estagio, que cabe em 512 bytes e um segundo estagio que é posteriormente carregado pelo primeiro.

O bootloader deve determinar a localização da imagem do kernel no disco e carrega-la na memoria. ELe tambem precisar mudar a CPU do [modo real](https://en.wikipedia.org/wiki/Real_mode) para o [modo protegido](https://en.wikipedia.org/wiki/Protected_mode) de 32 bits e depois para o [modo longo](https://en.wikipedia.org/wiki/Long_mode) de 64 bits, onde os registros de 64 bits e a memoria principal completa estao disponiveis. Sua terceira tarefa é consultar certas informacoes como (um mapa de memoria) do BIOS e passa-la para o kernel do sistema operacional.

Escrever um bootloader é um pouco complicado pois requer linguaguem como assembly e muitas etapas não esclarecedoras, como "escrever este valor magico neste registro do processador". Portanto não abordaremos a criacao de bootloader nesse post, em vez disso, forneceremos uma ferramenta chamada [bootimage](https://github.com/rust-osdev/bootimage) que acrecenta automaticamente um bootloader ao seu kernel.

### O padrão de inicialização múltipla

Para evitar que cada sistema operacional implemente seu proprio bootloader, que so é compativel com um unico sistema, a [Free Software Foundation](https://en.wikipedia.org/wiki/Free_Software_Foundation) criou um padrão de bootloader aberto chamado [Multiboot](https://wiki.osdev.org/Multiboot) em 1995. O padrão define uma interface entre o bootloader e o sistema operacional, para que qualquer bootloader compativel com MultiBoot possa carregar possa carregar qualquer sistema operacional com multiboot.

Para tornar um kernel compatível com Multiboot, basta inserir o chamado [cabeçalho Multiboot](https://www.gnu.org/software/grub/manual/multiboot/multiboot.html#OS-image-format) no início do arquivo do kernel. Isso torna muito fácil inicializar um sistema operacional a partir do GRUB. No entanto, o GRUB e o padrão Multiboot também apresentam alguns problemas:

- Eles suportam apenas o modo protegido de 32 bits. Isso significa que você ainda precisa fazer a configuração da CPU para mudar para o modo longo de 64 bits.

- Eles são projetados para tornar o bootloader simples em vez do kernel. Por exemplo, o kernel precisa estar vinculado a um tamanho de página padrão ajustado , porque o GRUB não consegue encontrar o cabeçalho Multiboot de outra forma. Outro exemplo é que as informações de inicialização , que são passadas para o kernel, contêm muitas estruturas dependentes da arquitetura em vez de fornecer abstrações limpas.

- Tanto o GRUB quanto o padrão Multiboot são pouco documentados.

- O GRUB precisa ser instalado no sistema host para criar uma imagem de disco inicializável a partir do arquivo do kernel. Isso torna o desenvolvimento em Windows ou Mac mais difícil.

Devido a essas desvantagens, decidimos não usar o GRUB ou o padrão Multiboot. No entanto, planejamos adicionar suporte Multiboot à nossa ferramenta bootimage , para que seja possível carregar seu kernel também em um sistema GRUB.

### UEFI
Não oferecemos suporte UEFI no momento.

## Um kernel mínimo

Agora que sabemos aproximadamente como um computador inicia, é hora de criar nosso proprio kernel minimo, Nosso objetivo é criar uma imagem que imprima um "Hello world" para a tela quando inicializar. Fazemos isso estendendo o binario rust independente

Como você deve se lembrar, construímos o binário independente por meio do cargo, mas dependendo do sistema operacional, precisávamos de diferentes nomes de pontos de entrada e sinalizadores de compilação. Isso ocorre porque cargoas compilações para o sistema host por padrão, ou seja, o sistema em que você está executando. Isso não é algo que queremos para o nosso kernel, porque um kernel que roda sobre, por exemplo, o Windows, não faz muito sentido. Em vez disso, queremos compilar para um sistema de destino claramente definido .

## Instalando Rust Nightly

RUst tem três canais de lançamento: stable, beta e nightly. O Rust Book explica muito bem a diferença entre esses canais, então reselve um minuto [e de uma olhada](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#choo-choo-release-channels-and-riding-the-trains). Para contruir um sistema operacional, precisamos de alguns recursos experimentais que estão disponiveis apenas no canal nightly, portanto, precisamos instalar uma versão nightly do rust

Para gerenciar instalações Rust, recomendo enfaticamente o rustup . Ele permite que você instale compiladores noturnos, beta e estáveis ​​lado a lado e facilita sua atualização. Com o Rustup, você pode usar um compilador noturno para o diretório atual executando `rustup override set nightly`. Alternativamente, você pode adicionar um arquivo chamado `rust-toolchain` com o conteúdo `nightly` ao diretório raiz do projeto. Você pode verificar se possui uma versão noturna instalada executando `rustc --version`: O número da versão deve conter -nightly no final.


O compilador noturno nos permite ativar vários recursos experimentais usando os chamados sinalizadores de recurso na parte superior do nosso arquivo. Por exemplo, poderíamos ativar a `asm!` macro experimental para montagem inline adicionando `#![feature(asm)]` ao topo do nosso arquivo main.rs. Observe que esses recursos experimentais são completamente instáveis, o que significa que versões futuras do Rust podem alterá-los ou removê-los sem aviso prévio. Por este motivo, só os utilizaremos se for absolutamente necessário.

### Especificação de destino

Cargo oferece suporte a diferentes sistemas de destino por meio do --targetparâmetro. O alvo é descrito pelo chamado [alvo triplo](https://clang.llvm.org/docs/CrossCompilation.html#target-triple) que descreve a arquitetura da CPU, o fornecedor, o sistema operacional e a [ABI](https://stackoverflow.com/questions/2171177/what-is-an-application-binary-interface-abi/2456882#2456882). Por exemplo, o `x86_64-unknown-linux-gnu` triplo alvo descreve um sistema com `x86_64` CPU, sem fornecedor claro e um sistema operacional Linux com GNU ABI. Rust oferece suporte a [muitos triplos de destino diferentes](https://doc.rust-lang.org/nightly/rustc/platform-support.html), inclusive `arm-linux-androideabi` para Android ou `wasm32-unknown-unknown` para WebAssembly.

Para o nosso sistema alvo, no entanto, necessitamos de alguns parâmetros de configuração especiais (por exemplo, nenhum sistema operacional subjacente), de modo que nenhum dos triplos alvo existentes se ajuste. Felizmente, Rust nos permite definir nosso próprio alvo através de um arquivo JSON. Por exemplo, um arquivo JSON que descreve o `x86_64-unknown-linux-gnu` destino tem esta aparência:

```json
{
    "llvm-target": "x86_64-unknown-linux-gnu",
    "data-layout": "e-m:e-i64:64-f80:128-n8:16:32:64-S128",
    "arch": "x86_64",
    "target-endian": "little",
    "target-pointer-width": "64",
    "target-c-int-width": "32",
    "os": "linux",
    "executables": true,
    "linker-flavor": "gcc",
    "pre-link-args": ["-m64"],
    "morestack": false
}
```

A maioria dos campos é exigida pelo LLVM para gerar código para essa plataforma. Por exemplo, o [data-layout](https://llvm.org/docs/LangRef.html#data-layout) campo define o tamanho de vários tipos inteiros, de ponto flutuante e de ponteiro. Depois, há campos que Rust usa para compilação condicional, como target-pointer-width. O terceiro tipo de campo define como a caixa deve ser construída. Por exemplo, o pre-link-argscampo especifica argumentos passados ​​ao [vinculador](https://en.wikipedia.org/wiki/Linker_(computing)).

Também visamos `x86_64` sistemas com nosso kernel, então nossa especificação de destino será muito semelhante à acima. Vamos começar criando um `x86_64-blog_os.json` arquivo (escolha o nome que desejar) com o conteúdo comum:

```json 
{
    "llvm-target": "x86_64-unknown-none",
    "data-layout": "e-m:e-i64:64-f80:128-n8:16:32:64-S128",
    "arch": "x86_64",
    "target-endian": "little",
    "target-pointer-width": "64",
    "target-c-int-width": "32",
    "os": "none",
    "executables": true
}
```

