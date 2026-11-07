# Guia Completo de Rust

Nov 7, 2026 · @Alexandre Santos

## 0. Como usar este guia

Este guia leva você do zero até concorrência, async e integração com Python, em 23 partes que seguem a ordem em que eu ensino Rust em sala. Cada parte segue o mesmo molde: **o que é**, **para que serve**, **termos e siglas** da parte, **exemplos reais** (no mínimo dois) e, quando o fluxo não é óbvio, um **diagrama**.

Como estudar: digite os exemplos em vez de copiar, rode `cargo run` e leia as mensagens de erro do compilador com calma. Em Rust, o compilador é o seu monitor de laboratório: os erros dele quase sempre dizem o que corrigir.

### Glossário geral de siglas e termos

| Sigla / termo | Significado | Em uma frase |
| --- | --- | --- |
| ABI | Application Binary Interface (Interface Binária de Aplicação) | Contrato de como funções são chamadas no nível de máquina; usado em FFI. |
| API | Application Programming Interface | Conjunto de funções que um programa expõe para outro usar. |
| Arc | Atomic Reference Counted | Ponteiro com contagem de referências segura entre threads. |
| Borrow checker | Verificador de empréstimos | Parte do compilador que garante que referências nunca ficam inválidas. |
| CLI | Command Line Interface | Programa usado pelo terminal. |
| Crate | Caixote / pacote | Unidade de compilação do Rust: uma biblioteca ou um executável. |
| FFI | Foreign Function Interface | Mecanismo para chamar código de outra linguagem (C, Python). |
| GC | Garbage Collector (coletor de lixo) | Libera memória em tempo de execução; Rust **não** tem. |
| Heap | Monte | Região de memória para dados de tamanho dinâmico. |
| HIR / MIR | High-level / Mid-level Intermediate Representation | Representações internas do código dentro do compilador. |
| IO / E/S | Input/Output (entrada e saída) | Leitura e escrita em arquivo, rede, terminal. |
| LLVM | Antes Low Level Virtual Machine; hoje só o nome do projeto | Backend que transforma o código em instruções de máquina. |
| Monomorfização | Gerar uma cópia da função para cada tipo concreto | É como generics ficam com custo zero. |
| RAII | Resource Acquisition Is Initialization | O recurso é liberado quando a variável dona sai de escopo. |
| Rc | Reference Counted | Ponteiro com contagem de referências para uma única thread. |
| Stack | Pilha | Região de memória rápida para dados de tamanho fixo. |
| std | Standard library | Biblioteca padrão do Rust. |
| TOML | Tom's Obvious, Minimal Language | Formato do arquivo de configuração `Cargo.toml`. |
| UB | Undefined Behavior (comportamento indefinido) | Situação em que o programa pode fazer qualquer coisa; Rust seguro impede. |
| UTF-8 | Unicode Transformation Format, 8 bits | Codificação de texto usada por toda `String` em Rust. |
| WASM | WebAssembly | Formato binário que roda no navegador; Rust compila para ele. |
| Zero-cost abstraction | Abstração de custo zero | Código de alto nível que compila para algo tão rápido quanto o escrito à mão. |

## 1. Introdução: o que é Rust

**O que é:** Rust é uma linguagem de programação compilada, de tipagem estática, voltada a desempenho e segurança. Nasceu como projeto pessoal de Graydon Hoare em 2006, foi adotada pela Mozilla e chegou à versão 1.0 em 2015. Hoje é mantida pela Rust Foundation.

**Para que serve:** escrever programas tão rápidos quanto C e C++, mas sem os erros clássicos de memória (ponteiro solto, uso depois de liberar, condição de corrida). Ela faz isso **sem GC**: as regras de *ownership* são checadas na compilação, então não há pausa de coleta de lixo em execução.

Onde Rust aparece no mundo real:

- **Ferramentas de Python e dados:** `ruff` (linter), `uv` (gerenciador de pacotes), Polars (DataFrames) e os `tokenizers` da Hugging Face são escritos em Rust.
- **Infraestrutura:** Cloudflare e AWS (Firecracker, a base do Lambda) usam Rust em serviços críticos.
- **Sistemas operacionais:** o kernel Linux aceita drivers em Rust desde a versão 6.1.
- **Web e WASM:** backends com Axum/Actix e código que roda no navegador via WebAssembly.

**Termos e siglas:**

- **Compilada:** o código vira um binário nativo antes de rodar (diferente do Python, que é interpretado).
- **Tipagem estática:** o tipo de cada variável é conhecido na compilação.
- **rustup:** instalador e gerenciador de versões do Rust.
- **rustc:** o compilador (Rust Compiler).
- **cargo:** gerenciador de projetos e pacotes (equivalente a pip + venv + make).
- **Toolchain:** conjunto rustc + cargo + std de uma versão. Canais: `stable`, `beta`, `nightly`.
- **Edition:** "edição" da linguagem (2015, 2018, 2021, 2024). Muda regras de sintaxe sem quebrar código antigo.

### Instalação

```bash
# Linux / macOS / WSL
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Conferir
rustc --version
cargo --version

# Atualizar a toolchain
rustup update

# Ferramentas que todo projeto usa
rustup component add rustfmt clippy
```

No Windows, baixe o `rustup-init.exe` em rustup.rs. No editor, instale a extensão **rust-analyzer** (autocompletar, erros em tempo real, tipos inferidos na tela).

### Exemplo 1 — Hello world compilado à mão

```rust
// arquivo: ola.rs
fn main() {
    println!("Olá, turma de Rust!");
}
```

```bash
rustc ola.rs   # gera o binário ./ola (ola.exe no Windows)
./ola
```

`fn main()` é o ponto de entrada. `println!` termina com `!` porque é uma **macro**, não uma função (parte 20).

### Exemplo 2 — Uma CLI de saudação que lê argumentos

```rust
use std::env;

fn main() {
    // env::args() devolve os argumentos; o primeiro é o nome do programa
    let args: Vec<String> = env::args().collect();

    if args.len() < 2 {
        eprintln!("Uso: saudacao <nome>");
        std::process::exit(1);
    }

    let nome = &args[1];
    println!("Bem-vindo, {nome}! Seu nome tem {} letras.", nome.chars().count());
}
```

```bash
cargo run -- Alexandre
# Bem-vindo, Alexandre! Seu nome tem 9 letras.
```

`eprintln!` escreve na saída de erro (stderr), o lugar certo para mensagens de uso. `chars().count()` conta caracteres Unicode, não bytes.

### Como o código vira binário

O diagrama abaixo mostra o caminho de um arquivo `.rs` até o executável. A etapa que diferencia Rust é a checagem de empréstimos sobre a MIR.

&#91;embedded content: pipeline de compilação do rustc · 6 etapas, 1 saída de erro\]

Se a MIR violar uma regra de ownership, a compilação para ali com uma mensagem explicativa; só código aprovado chega ao LLVM e vira executável.

## 2. Cargo e estrutura de projeto

**O que é:** Cargo é a ferramenta oficial de build e gerenciamento de pacotes do Rust. Ele cria projetos, baixa dependências do crates.io, compila, roda testes, gera documentação e formata o código.

**Para que serve:** padronizar todo projeto Rust. Qualquer pessoa clona um repositório e roda `cargo build`, sem script de instalação próprio.

**Termos e siglas:**

- **crates.io:** repositório público de pacotes Rust (o "PyPI do Rust").
- **Dependência:** crate de terceiros que o seu projeto usa.
- **Cargo.lock:** trava as versões exatas baixadas, para builds reproduzíveis.
- **SemVer (Semantic Versioning):** versão no formato MAIOR.MENOR.CORREÇÃO. `"1.4"` aceita qualquer 1.x ≥ 1.4.
- **Profile:** perfil de compilação. `dev` compila rápido e com checagens; `release` otimiza para velocidade.
- **Feature:** parte opcional de um crate que você liga quando precisa.
- **Workspace:** vários crates no mesmo repositório, compartilhando um `Cargo.lock` e uma pasta `target/`.

### Estrutura criada por `cargo new`

```text
meu_app/
├── Cargo.toml      # manifesto: nome, versão, edition, dependências
├── Cargo.lock      # versões travadas (criado no primeiro build)
├── src/
│   └── main.rs     # ponto de entrada de um binário
│   └── lib.rs      # (opcional) código de biblioteca
├── tests/          # (opcional) testes de integração
└── target/         # saída da compilação (não vai para o git)
```

### Comandos do dia a dia

| Comando | O que faz |
| --- | --- |
| `cargo new nome` | Cria projeto binário (`--lib` para biblioteca) |
| `cargo check` | Só verifica erros, sem gerar binário. O mais rápido |
| `cargo build` | Compila em modo dev para `target/debug/` |
| `cargo build --release` | Compila otimizado para `target/release/` |
| `cargo run` | Compila e executa |
| `cargo test` | Roda todos os testes |
| `cargo add crate` | Adiciona dependência ao `Cargo.toml` |
| `cargo fmt` | Formata o código no padrão oficial |
| `cargo clippy` | Linter com centenas de sugestões de boas práticas |
| `cargo doc --open` | Gera e abre a documentação HTML do projeto |

### Exemplo 1 — Sorteador de amigo secreto com dependência externa

```bash
cargo new amigo_secreto
cd amigo_secreto
cargo add rand@0.8
```

```toml
# Cargo.toml
[package]
name = "amigo_secreto"
version = "0.1.0"
edition = "2024"

[dependencies]
rand = "0.8"
```

```rust
// src/main.rs
use rand::seq::SliceRandom;

fn main() {
    let mut nomes = vec!["Ana", "Bruno", "Carla", "Diego", "Eva"];
    let mut rng = rand::thread_rng();
    nomes.shuffle(&mut rng);

    // Cada pessoa tira a próxima da lista embaralhada; a última tira a primeira
    for i in 0..nomes.len() {
        let tirou = nomes[(i + 1) % nomes.len()];
        println!("{} tirou {}", nomes[i], tirou);
    }
}
```

O truque do `% nomes.len()` (resto da divisão) fecha o ciclo, garantindo que ninguém tira a si mesmo.

### Exemplo 2 — Workspace com API e núcleo separados

Em empresas, é comum separar a regra de negócio (biblioteca) da interface (binário). Assim a mesma regra serve a uma API web e a uma CLI.

```text
banco/
├── Cargo.toml          # o workspace
├── nucleo/             # crate de biblioteca: regras de negócio
│   ├── Cargo.toml
│   └── src/lib.rs
└── api/                # crate binário: servidor web
    ├── Cargo.toml
    └── src/main.rs
```

```toml
# banco/Cargo.toml
[workspace]
resolver = "3"
members = ["nucleo", "api"]

# banco/api/Cargo.toml
[package]
name = "api"
version = "0.1.0"
edition = "2024"

[dependencies]
nucleo = { path = "../nucleo" }   # dependência local
```

```bash
cargo build              # compila os dois crates
cargo run -p api         # roda só o binário api
cargo test -p nucleo     # testa só a biblioteca
```

### Exemplo 3 — Perfil release ajustado para produção

```toml
[profile.release]
opt-level = 3        # otimização máxima
lto = true           # LTO: Link Time Optimization, otimiza entre crates
codegen-units = 1    # compila mais devagar, gera binário mais rápido
strip = true         # remove símbolos de depuração, binário menor
```

Use isso em contêineres Docker: o binário fica menor e sobe mais rápido.

## 3. Variáveis, mutabilidade, constantes e shadowing

**O que é:** em Rust, `let` cria uma variável **imutável por padrão**. Para poder alterar, você declara com `let mut`. Constantes (`const`) e estáticas (`static`) são valores fixos conhecidos na compilação. *Shadowing* é declarar de novo uma variável com o mesmo nome.

**Para que serve:** a imutabilidade padrão evita alterações acidentais e deixa claro, só lendo a declaração, quais valores mudam ao longo do código. Isso é ainda mais importante com várias threads.

**Termos e siglas:**

- **Imutável:** não pode mudar depois de criado.
- **Inferência de tipo:** o compilador descobre o tipo sozinho (`let x = 5;` vira `i32`).
- **Anotação de tipo:** você escreve o tipo explicitamente: `let x: u64 = 5;`.
- **Shadowing (sombreamento):** um novo `let` com o mesmo nome "esconde" o anterior; pode até mudar o tipo.
- **SCREAMING\_SNAKE\_CASE:** convenção de nome para constantes (`MAX_TENTATIVAS`).

| Declaração | Muda o valor? | Tipo obrigatório? | Quando usar |
| --- | --- | --- | --- |
| `let x = 1;` | Não | Não (inferido) | Padrão para quase tudo |
| `let mut x = 1;` | Sim | Não | Contadores, acumuladores, buffers |
| `const X: u32 = 1;` | Nunca | Sim | Limites e configurações fixas |
| `static X: &str = "..";` | Nunca (em código seguro) | Sim | Valor com endereço fixo na memória durante todo o programa |

### Exemplo 1 — Total de um carrinho de compras

```rust
fn main() {
    let precos = [1990, 4590, 899];   // em centavos, imutável
    let mut total = 0;                 // vai mudar, então precisa de mut

    for preco in precos {
        total += preco;
    }

    println!("Total: R$ {},{:02}", total / 100, total % 100);
    // Total: R$ 74,79
}
```

Se você tirar o `mut`, o compilador recusa com o erro E0384 (*cannot assign twice to immutable variable*) e sugere onde colocar o `mut`.

### Exemplo 2 — Shadowing ao tratar entrada do usuário

```rust
use std::io;

fn main() {
    println!("Quantos anos você tem?");

    let mut idade = String::new();
    io::stdin().read_line(&mut idade).expect("falha ao ler");

    // Mesmo nome, novo tipo: de String para u8
    let idade: u8 = idade.trim().parse().expect("digite um número");

    if idade >= 18 {
        println!("Pode tirar a CNH.");
    } else {
        println!("Faltam {} anos.", 18 - idade);
    }
}
```

Sem shadowing, você precisaria de dois nomes (`idade_texto` e `idade_num`). O shadowing deixa claro que é o mesmo dado, só convertido.

### Exemplo 3 — Constantes de configuração de login

```rust
const MAX_TENTATIVAS: u32 = 3;
const TIMEOUT_SEGUNDOS: u64 = 30;
static NOME_SISTEMA: &str = "Portal do Aluno";

fn main() {
    println!("{NOME_SISTEMA}: até {MAX_TENTATIVAS} tentativas, timeout de {TIMEOUT_SEGUNDOS}s");
}
```

Diferença prática: `const` é copiada em cada lugar onde é usada (como um "copiar e colar" do compilador); `static` tem um único endereço na memória.

> **Dica do professor:** `mut` é para a variável, não para o valor. Shadowing cria uma variável nova, então funciona mesmo sem `mut`.

## 4. Tipos de dados

**O que é:** todo valor em Rust tem um tipo conhecido na compilação. Os **escalares** guardam um único valor (inteiro, decimal, booleano, caractere). Os **compostos** agrupam vários valores (tupla e array).

**Para que serve:** o tipo define quanta memória o valor ocupa e quais operações são permitidas. Escolher bem o tipo evita bugs como estouro de inteiro ou erro de arredondamento com dinheiro.

**Termos e siglas:**

- **Signed / unsigned:** com sinal (`i`, aceita negativos) e sem sinal (`u`, só zero e positivos).
- **Overflow (estouro):** quando o resultado não cabe no tipo. Em modo dev o programa entra em *panic*; em release o valor "dá a volta".
- **IEEE 754:** padrão de números de ponto flutuante usado por `f32` e `f64`.
- **Unicode / scalar value:** `char` guarda qualquer caractere Unicode em 4 bytes, inclusive emoji e acentos.
- **usize / isize:** inteiro do tamanho da palavra da CPU (64 bits em máquinas de 64 bits). É o tipo dos índices.
- **Casting:** conversão explícita com `as` (`x as f64`).

### Tipos escalares

| Tipo | Tamanho | Faixa / uso típico |
| --- | --- | --- |
| `i8` / `u8` | 1 byte | −128 a 127 / 0 a 255 (bytes, cor RGB) |
| `i16` / `u16` | 2 bytes | Portas de rede (`u16`: 0 a 65 535) |
| `i32` / `u32` | 4 bytes | Inteiro padrão (`i32`) |
| `i64` / `u64` | 8 bytes | Valores monetários em centavos, timestamps |
| `i128` / `u128` | 16 bytes | Identificadores enormes, criptografia |
| `isize` / `usize` | 4 ou 8 bytes | Índices e tamanhos de coleções |
| `f32` / `f64` | 4 / 8 bytes | Medidas, ciência, ML (`f64` é o padrão) |
| `bool` | 1 byte | `true` / `false` |
| `char` | 4 bytes | Um caractere Unicode: `'a'`, `'ç'`, `'🦀'` |

### Tipos compostos

- **Tupla** `(i32, f64, char)`: valores de tipos diferentes, tamanho fixo. Acesso por `t.0`, `t.1`.
- **Array** `[i32; 5]`: valores do mesmo tipo, tamanho fixo, na stack. Para tamanho variável, use `Vec` (parte 12).

### Exemplo 1 — Saldo bancário em centavos (nunca use float para dinheiro)

```rust
fn main() {
    // Errado: float acumula erro de arredondamento
    let a: f64 = 0.1 + 0.2;
    println!("{a}"); // 0.30000000000000004

    // Certo: guardar em centavos com inteiro
    let saldo_centavos: i64 = 150_075; // R$ 1.500,75 (o _ só ajuda a ler)
    let saque: i64 = 20_000;

    // checked_sub devolve None se estourar, em vez de quebrar
    match saldo_centavos.checked_sub(saque) {
        Some(novo) => println!("Novo saldo: R$ {:.2}", novo as f64 / 100.0),
        None => println!("Erro de cálculo"),
    }
}
```

Bancos e sistemas de pagamento guardam valores como inteiros em centavos (ou usam um tipo decimal, como o crate `rust_decimal`). O `as f64` aqui é só para exibir.

### Exemplo 2 — Pixel de imagem com `u8` e tupla

```rust
fn escurecer(pixel: (u8, u8, u8), fator: f32) -> (u8, u8, u8) {
    let (r, g, b) = pixel; // desestruturação da tupla
    (
        (r as f32 * fator) as u8,
        (g as f32 * fator) as u8,
        (b as f32 * fator) as u8,
    )
}

fn main() {
    let laranja = (255, 165, 0);
    let escuro = escurecer(laranja, 0.5);
    println!("{:?}", escuro); // (127, 82, 0)

    // Uma linha de 4 pixels em array de tamanho fixo
    let linha: [(u8, u8, u8); 4] = [laranja; 4];
    println!("pixels na linha: {}", linha.len());
}
```

`u8` é perfeito para cor porque cada canal vai de 0 a 255, exatamente a faixa do tipo.

### Exemplo 3 — Controlando overflow em contador de requisições

```rust
fn main() {
    let contador: u8 = 250;

    println!("{:?}", contador.checked_add(10));    // None (não cabe)
    println!("{}", contador.saturating_add(10));   // 255 (trava no máximo)
    println!("{}", contador.wrapping_add(10));     // 4 (dá a volta)
}
```

Escolha conforme a regra de negócio: `saturating` para limites ("no máximo 255"), `checked` quando o estouro é um erro, `wrapping` para hashes e contadores circulares.

> **Dica do professor:** `{:?}` imprime no formato de depuração (trait `Debug`); `{}` usa o formato para usuário (trait `Display`). Veremos traits na parte 13.

## 5. Funções, expressões e controle de fluxo

**O que é:** funções são declaradas com `fn`, com tipos obrigatórios nos parâmetros e no retorno. Em Rust quase tudo é **expressão** (produz um valor): `if`, `match`, `loop` e blocos `{ }`. O controle de fluxo usa `if`, `loop`, `while` e `for`.

**Para que serve:** organizar o código em partes reutilizáveis e decidir o caminho da execução. Como `if` e `loop` devolvem valor, você escreve menos variáveis temporárias.

**Termos e siglas:**

- **Expressão:** trecho que resulta em um valor (`2 + 3`, `if x { 1 } else { 2 }`).
- **Instrução (statement):** trecho que executa algo e não devolve valor (`let x = 5;`).
- **Retorno implícito:** a última expressão de um bloco, **sem ponto e vírgula**, é o valor devolvido.
- **Range:** intervalo. `0..5` vai de 0 a 4; `0..=5` vai de 0 a 5.
- **Label (rótulo):** nome para um laço (`'externo: loop`), usado para sair de laços aninhados.
- **`()` (unit):** o "tipo vazio", devolvido por funções que não retornam nada.

### Exemplo 1 — Situação acadêmica do aluno (if como expressão)

```rust
fn media(notas: &[f64]) -> f64 {
    let soma: f64 = notas.iter().sum();
    soma / notas.len() as f64       // sem ; = valor de retorno
}

fn situacao(media: f64, frequencia: u32) -> &'static str {
    if frequencia < 75 {
        "Reprovado por falta"
    } else if media >= 7.0 {
        "Aprovado"
    } else if media >= 5.0 {
        "Exame final"
    } else {
        "Reprovado por nota"
    }
}

fn main() {
    let notas = [8.0, 6.5, 7.5];
    let m = media(&notas);
    println!("Média {:.1}: {}", m, situacao(m, 90));
    // Média 7.3: Aprovado
}
```

Repare que `situacao` não tem `return`: o `if` inteiro é a expressão de retorno. Use `return` só para sair cedo da função.

### Exemplo 2 — Tentar conectar com retry (loop que devolve valor)

```rust
fn tentar_conectar(tentativa: u32) -> bool {
    // Simulação: o servidor só responde na 3ª tentativa
    tentativa == 3
}

fn main() {
    let mut tentativa = 0;

    let resultado = loop {
        tentativa += 1;
        println!("Tentativa {tentativa}...");

        if tentar_conectar(tentativa) {
            break Ok(tentativa);           // sai do loop devolvendo valor
        }
        if tentativa == 5 {
            break Err("servidor indisponível");
        }
    };

    match resultado {
        Ok(n) => println!("Conectado na tentativa {n}"),
        Err(e) => println!("Falhou: {e}"),
    }
}
```

Esse padrão é a base de clientes HTTP com *retry*. Em produção, você soma um intervalo crescente entre tentativas (*exponential backoff*, espera exponencial).

### Exemplo 3 — Laços `for`, `while` e rótulos

```rust
fn main() {
    // for com range e passo: parcelas de 3 em 3 meses
    for mes in (3..=12).step_by(3) {
        println!("Revisão trimestral no mês {mes}");
    }

    // while: contagem regressiva
    let mut n = 3;
    while n > 0 {
        println!("{n}...");
        n -= 1;
    }

    // Rótulo: procurar a primeira poltrona livre numa sala
    let sala = [[true, true], [true, false], [false, false]]; // true = ocupada
    'fileiras: for (f, fileira) in sala.iter().enumerate() {
        for (c, ocupada) in fileira.iter().enumerate() {
            if !ocupada {
                println!("Livre: fileira {f}, cadeira {c}");
                break 'fileiras; // sai dos dois laços de uma vez
            }
        }
    }
}
```

> **Dica do professor:** o erro mais comum de iniciante é colocar `;` na última linha de uma função que deveria retornar valor. O compilador avisa: *expected `f64`, found `()`*.

## 6. Ownership (posse) e memória stack/heap

**O que é:** ownership é o sistema que Rust usa para gerenciar memória sem GC. Cada valor tem **um único dono** (a variável que o guarda). Quando o dono sai de escopo, o valor é liberado automaticamente. Atribuir ou passar o valor para outra variável **move** a posse.

**Para que serve:** eliminar, já na compilação, três classes de bug que em C/C++ causam falhas de segurança: *use after free* (usar memória já liberada), *double free* (liberar duas vezes) e vazamento de memória por esquecimento.

**As 3 regras:**

1. Cada valor tem uma variável dona.
2. Só pode existir um dono por vez.
3. Quando o dono sai de escopo, o valor é descartado (*drop*).

**Termos e siglas:**

- **Stack (pilha):** memória rápida, organizada em ordem de chegada e saída. Guarda dados de tamanho fixo conhecido na compilação (`i32`, `bool`, arrays).
- **Heap (monte):** memória para dados que crescem ou cujo tamanho só se sabe em execução (`String`, `Vec`). Acesso via ponteiro guardado na stack.
- **Move (mover):** transferir a posse. A variável antiga fica inválida.
- **Copy:** tipos simples da stack (inteiros, `bool`, `char`, `f64`) são copiados bit a bit em vez de movidos.
- **Clone:** cópia profunda explícita, inclusive dos dados na heap (`s.clone()`). Tem custo.
- **Drop:** função chamada automaticamente quando o valor sai de escopo. É o RAII.
- **Escopo:** trecho delimitado por `{ }` onde a variável existe.

### Exemplo 1 — Move de `String` e o erro E0382

```rust
fn main() {
    let s1 = String::from("relatório.pdf");
    let s2 = s1;                 // a posse MOVE para s2

    // println!("{s1}");         // ERRO E0382: borrow of moved value: `s1`
    println!("{s2}");            // ok

    let s3 = s2.clone();         // cópia profunda: duas strings na heap
    println!("{s2} e {s3}");     // ambas válidas

    let x = 10;
    let y = x;                   // i32 é Copy: x continua válido
    println!("{x} {y}");
}
```

Por que mover em vez de copiar? Se `s1` e `s2` apontassem para o mesmo texto na heap, as duas tentariam liberá-lo ao sair de escopo (*double free*). Ao invalidar `s1`, só existe um responsável.

### Exemplo 2 — Função que consome um pedido vs função que devolve

```rust
fn enviar_para_cozinha(pedido: String) {
    println!("Cozinha recebeu: {pedido}");
} // `pedido` é liberado aqui

fn adicionar_bebida(mut pedido: String) -> String {
    pedido.push_str(" + refrigerante");
    pedido                      // devolve a posse para quem chamou
}

fn main() {
    let pedido = String::from("X-burguer");
    let pedido = adicionar_bebida(pedido);   // posse vai e volta
    enviar_para_cozinha(pedido);             // posse vai e não volta
    // println!("{pedido}");                 // ERRO: o pedido foi para a cozinha
}
```

Passar a posse e depois devolver funciona, mas é trabalhoso. A solução elegante é **emprestar** em vez de mover, assunto da parte 7.

### Exemplo 3 — Liberação automática de recursos com `Drop` (RAII)

```rust
struct ConexaoBanco {
    nome: String,
}

impl Drop for ConexaoBanco {
    fn drop(&mut self) {
        println!("Conexão '{}' fechada e devolvida ao pool", self.nome);
    }
}

fn main() {
    let _principal = ConexaoBanco { nome: "principal".into() };
    {
        let _relatorio = ConexaoBanco { nome: "relatorio".into() };
        println!("Gerando relatório...");
    } // _relatorio sai de escopo aqui: drop automático
    println!("Fim do main");
} // _principal liberada aqui
```

Saída:

```text
Gerando relatório...
Conexão 'relatorio' fechada e devolvida ao pool
Fim do main
Conexão 'principal' fechada e devolvida ao pool
```

É assim que arquivos (`File`), travas (`MutexGuard`) e conexões são fechados em Rust: nunca é preciso lembrar de chamar `close()`.

### Como fica a memória num move

O diagrama mostra o que acontece com `let s2 = s1;`: só o "cabeçalho" da stack é copiado, o texto na heap não se move, e `s1` deixa de valer.

&#91;embedded content: memória durante let s2 = s1 · stack e heap\]

Como só `s2` aponta para os bytes, a heap é liberada exatamente uma vez. Já `s1.clone()` copiaria também os 14 bytes da heap.

## 7. Borrowing, referências e slices

**O que é:** *borrowing* (empréstimo) é usar um valor sem tomar a posse dele, por meio de uma **referência**. `&T` é uma referência de leitura (compartilhada); `&mut T` é uma referência de escrita (exclusiva). Uma **slice** é uma referência a um pedaço contínuo de uma coleção (`&str`, `&[T]`).

**Para que serve:** permitir que funções leiam ou alterem dados sem copiá-los e sem precisar devolver a posse. O borrow checker garante que nenhuma referência aponte para memória já liberada e que não haja duas escritas simultâneas.

**As regras de empréstimo:**

1. Em um dado momento você pode ter **muitas** referências `&T` **ou uma única** `&mut T`, nunca as duas ao mesmo tempo.
2. Referências precisam ser sempre válidas: o dado não pode morrer antes da referência.

**Termos e siglas:**

- **Referência:** um ponteiro seguro, que o compilador garante ser válido.
- **Desreferenciar:** acessar o valor apontado, com `*` (`*contador += 1`).
- **Data race (condição de corrida):** dois acessos simultâneos à mesma memória, pelo menos um de escrita, sem sincronização. A regra 1 impede isso na compilação.
- **Dangling reference (referência pendente):** referência para memória já liberada. A regra 2 impede.
- **NLL (Non-Lexical Lifetimes):** o empréstimo termina no **último uso** da referência, não no fim do bloco. Isso deixa o compilador menos rígido.
- **`&str` vs `String`:** `String` é dona do texto (heap); `&str` é uma "janela" emprestada para um texto.

### Exemplo 1 — Validar um e-mail sem tomar posse

```rust
fn email_valido(email: &str) -> bool {
    email.contains('@') && email.contains('.') && !email.starts_with('@')
}

fn main() {
    let email = String::from("aluno@unicid.edu.br");

    if email_valido(&email) {           // empresta, não move
        println!("{email} é válido");   // email continua nosso
    }
}
```

Receber `&str` em vez de `&String` é a boa prática: a função aceita tanto `String` (via `&email`) quanto texto literal (`"a@b.com"`).

### Exemplo 2 — Carrinho alterado por referência mutável

```rust
fn adicionar_item(carrinho: &mut Vec<String>, item: &str) {
    carrinho.push(item.to_string());
}

fn total_itens(carrinho: &Vec<String>) -> usize {
    carrinho.len()
}

fn main() {
    let mut carrinho = Vec::new();

    adicionar_item(&mut carrinho, "Teclado");
    adicionar_item(&mut carrinho, "Mouse");

    println!("{} itens: {:?}", total_itens(&carrinho), carrinho);
}
```

O erro clássico que a regra 1 impede:

```rust
let mut v = vec![1, 2, 3];
let primeiro = &v[0];      // empréstimo de leitura
v.push(4);                 // ERRO E0502: precisa de &mut enquanto `primeiro` existe
println!("{primeiro}");
```

Por que é um erro real? O `push` pode realocar o vetor para outro lugar da heap, e `primeiro` passaria a apontar para memória liberada. Em C++ isso compila e quebra em produção.

### Exemplo 3 — Slices: extrair o domínio e calcular média de parte das notas

```rust
fn dominio(email: &str) -> &str {
    match email.find('@') {
        Some(pos) => &email[pos + 1..],   // slice do @ até o fim
        None => "",
    }
}

fn media(notas: &[f64]) -> f64 {
    notas.iter().sum::<f64>() / notas.len() as f64
}

fn main() {
    println!("{}", dominio("prof@empresa.com.br")); // empresa.com.br

    let notas = [5.0, 7.0, 9.0, 10.0];
    println!("Média geral: {}", media(&notas));          // 7.75
    println!("Média das 2 últimas: {}", media(&notas[2..])); // 9.5
}
```

A slice não copia nada: ela guarda só um ponteiro para o início e o tamanho. Por isso é barata.

### O borrow checker decide assim

O diagrama resume a pergunta que o compilador faz a cada empréstimo.

&#91;embedded content: decisão de empréstimo · 3 perguntas, 4 resultados\]

Na dúvida, comece por `&T`; troque por `&mut T` só onde precisar alterar, e por posse (`T`) só onde o valor precisar sobreviver à chamada.

## 8. Lifetimes (tempos de vida)

**O que é:** *lifetime* é o trecho do código durante o qual uma referência é válida. Na maioria das vezes o compilador deduz sozinho. Quando há ambiguidade, você escreve uma **anotação de lifetime**, como `'a`, que liga o tempo de vida de uma referência ao de outra.

**Para que serve:** provar ao compilador que uma referência devolvida por uma função, ou guardada dentro de uma struct, nunca vai sobreviver ao dado de onde veio. A anotação **não muda** quanto tempo nada vive; ela só **descreve** a relação.

**Termos e siglas:**

- **`'a`:** nome genérico de um lifetime (lê-se "lifetime a"). Pode ser qualquer nome: `'texto`, `'req`.
- **Elision (omissão):** regras que deixam você não escrever lifetimes nos casos óbvios. Ex.: uma função com um único parâmetro de referência devolve algo com o mesmo lifetime dele.
- **`'static`:** a referência vale durante todo o programa. Textos literais (`"olá"`) são `&'static str`, pois ficam gravados no binário.
- **Dangling:** referência para algo que já morreu. É o que os lifetimes impedem.

### Exemplo 1 — Função que devolve a maior de duas strings

```rust
// Sem 'a o compilador não sabe se o retorno vem de `a` ou de `b`
fn mais_longo<'a>(a: &'a str, b: &'a str) -> &'a str {
    if a.len() >= b.len() { a } else { b }
}

fn main() {
    let titulo = String::from("Introdução ao Rust");
    let resultado;
    {
        let subtitulo = String::from("Ownership");
        resultado = mais_longo(&titulo, &subtitulo);
        println!("{resultado}"); // ok: os dois ainda vivem aqui
    }
    // println!("{resultado}"); // ERRO E0597: `subtitulo` não vive o suficiente
}
```

Leia a assinatura assim: "o retorno vive no máximo tanto quanto o **menor** dos dois parâmetros". Por isso o uso fora do bloco interno é recusado.

### Exemplo 2 — Struct que guarda uma referência: parser de linha de log

```rust
struct LinhaLog<'a> {
    nivel: &'a str,
    mensagem: &'a str,
}

impl<'a> LinhaLog<'a> {
    fn parse(linha: &'a str) -> Option<LinhaLog<'a>> {
        // formato: "[ERRO] falha ao conectar"
        let fim = linha.find(']')?;
        Some(LinhaLog {
            nivel: &linha[1..fim],
            mensagem: linha[fim + 1..].trim(),
        })
    }
}

fn main() {
    let conteudo = String::from("[ERRO] timeout no serviço de pagamento");
    if let Some(log) = LinhaLog::parse(&conteudo) {
        println!("nível={} msg={}", log.nivel, log.mensagem);
    }
}
```

A struct não copia o texto, só aponta para partes dele. Em um arquivo de log de vários GB, isso evita milhões de alocações. O `'a` garante que `LinhaLog` não sobrevive a `conteudo`.

### Exemplo 3 — `'static` em mensagens de erro fixas

```rust
fn mensagem_http(codigo: u16) -> &'static str {
    match codigo {
        200 => "OK",
        404 => "Não encontrado",
        500 => "Erro interno do servidor",
        _ => "Código desconhecido",
    }
}

fn main() {
    println!("{}", mensagem_http(404));
}
```

Como os textos estão embutidos no binário, a função pode devolvê-los sem nenhum parâmetro de referência.

> **Dica do professor:** se você está lutando com lifetimes em uma struct, pergunte se ela precisa mesmo guardar referência. Muitas vezes guardar `String` (dona do dado) é mais simples e o custo é irrelevante.

## 9. Structs e métodos

**O que é:** `struct` é um tipo que você cria agrupando campos com nome. Com um bloco `impl` você associa **métodos** (funções que operam sobre a struct) e **funções associadas** (como construtores). Rust não tem classes nem herança; struct + impl + traits cumprem esse papel.

**Para que serve:** modelar entidades do domínio (cliente, conta, pedido) com dados e comportamento juntos, e criar tipos que impedem erros de lógica, como somar reais com dólares.

**Termos e siglas:**

- **Campo:** cada dado dentro da struct (`nome`, `saldo`).
- **`impl`:** bloco de implementação, onde ficam os métodos.
- **`self` / `&self` / `&mut self`:** o método consome a struct / lê / altera.
- **`Self`:** apelido para o próprio tipo dentro do `impl`.
- **Função associada:** função no `impl` sem `self`, chamada com `Tipo::funcao()`. `new` é a convenção para construtor.
- **`#[derive(...)]`:** pede ao compilador para gerar implementações prontas de traits (`Debug`, `Clone`, `PartialEq`).
- **Newtype:** struct de um campo só que embrulha outro tipo para dar significado (`struct Cpf(String)`).
- **Encapsulamento:** campos privados (padrão) só podem ser mexidos pelos métodos do próprio módulo.

| Forma | Sintaxe | Uso |
| --- | --- | --- |
| Struct com campos nomeados | `struct Conta { titular: String, saldo: i64 }` | Entidades com vários dados |
| Tuple struct (newtype) | `struct Centavos(i64);` | Dar significado a um valor simples |
| Unit struct | `struct Marcador;` | Tipos sem dados, usados com traits |

### Exemplo 1 — Conta bancária com regras de negócio

```rust
#[derive(Debug)]
struct Conta {
    titular: String,
    saldo_centavos: i64,
    limite_centavos: i64,
}

impl Conta {
    // Função associada: construtor
    fn new(titular: &str, limite_centavos: i64) -> Self {
        Self { titular: titular.to_string(), saldo_centavos: 0, limite_centavos }
    }

    fn depositar(&mut self, valor: i64) {
        self.saldo_centavos += valor;
    }

    fn sacar(&mut self, valor: i64) -> Result<(), String> {
        if self.saldo_centavos + self.limite_centavos < valor {
            return Err(format!("Saldo insuficiente para {}", self.titular));
        }
        self.saldo_centavos -= valor;
        Ok(())
    }

    fn saldo(&self) -> f64 {
        self.saldo_centavos as f64 / 100.0
    }
}

fn main() {
    let mut conta = Conta::new("Maria", 50_000);   // limite de R$ 500
    conta.depositar(100_000);

    match conta.sacar(140_000) {
        Ok(()) => println!("Saque ok. Saldo: R$ {:.2}", conta.saldo()), // -400.00
        Err(e) => println!("{e}"),
    }
    println!("{:?}", conta);
}
```

### Exemplo 2 — Newtypes para não misturar unidades

```rust
#[derive(Debug, Clone, Copy, PartialEq, PartialOrd)]
struct Reais(f64);

#[derive(Debug, Clone, Copy)]
struct Dolares(f64);

impl Dolares {
    fn para_reais(self, cotacao: f64) -> Reais {
        Reais(self.0 * cotacao)
    }
}

fn pagar_fornecedor(valor: Reais) {
    println!("Pagando R$ {:.2}", valor.0);
}

fn main() {
    let fatura = Dolares(120.0);
    // pagar_fornecedor(fatura);           // ERRO: esperado Reais, recebido Dolares
    pagar_fornecedor(fatura.para_reais(5.40));
}
```

A NASA perdeu a sonda *Mars Climate Orbiter* em 1999 porque um sistema enviava dados em libras-força e outro esperava newtons. Newtypes transformam esse tipo de erro em erro de compilação, sem custo em execução.

### Exemplo 3 — Atualização de struct e métodos encadeados (padrão builder)

```rust
#[derive(Debug, Clone)]
struct ConfigServidor {
    host: String,
    porta: u16,
    tls: bool,
}

impl ConfigServidor {
    fn padrao() -> Self {
        Self { host: "127.0.0.1".into(), porta: 8080, tls: false }
    }
    fn porta(mut self, p: u16) -> Self { self.porta = p; self }
    fn com_tls(mut self) -> Self { self.tls = true; self }
}

fn main() {
    let prod = ConfigServidor::padrao().porta(443).com_tls();

    // Sintaxe de atualização: copia os demais campos de `prod`
    let homolog = ConfigServidor { host: "homolog.local".into(), ..prod.clone() };

    println!("{:?}\n{:?}", prod, homolog);
}
```

O padrão *builder* (construtor encadeado) aparece em quase todo crate popular, como `reqwest::Client::builder()`.

## 10. Enums e pattern matching

**O que é:** `enum` define um tipo que pode ser **uma entre várias variantes**, e cada variante pode carregar dados diferentes. `match` compara um valor contra padrões e executa o braço que casar. O compilador obriga a tratar **todas** as variantes.

**Para que serve:** modelar estados e alternativas de forma que estados inválidos sejam impossíveis de representar. Se amanhã alguém adicionar uma variante nova, todo `match` que não a trata vira erro de compilação, e você descobre cada lugar a ajustar.

**Termos e siglas:**

- **Variante:** cada opção do enum (`Pix`, `Cartao`, `Boleto`).
- **Tipo soma / ADT (Algebraic Data Type, tipo algébrico de dados):** nome acadêmico para enums que carregam dados. Vem de linguagens funcionais como Haskell.
- **Exaustividade:** o `match` precisa cobrir todos os casos possíveis.
- **`_` (curinga):** casa com qualquer coisa que não foi tratada antes.
- **Guard (guarda):** condição extra em um braço: `x if x > 10 =>`.
- **Desestruturação:** extrair os campos de dentro do padrão (`Cartao { parcelas, .. }`).
- **`if let` / `let else`:** atalhos para quando só um padrão interessa.

### Exemplo 1 — Formas de pagamento com dados diferentes

```rust
enum Pagamento {
    Pix { chave: String },
    Cartao { final_numero: String, parcelas: u8 },
    Boleto(String), // linha digitável
}

fn taxa_percentual(p: &Pagamento) -> f64 {
    match p {
        Pagamento::Pix { .. } => 0.0,
        Pagamento::Cartao { parcelas: 1, .. } => 2.5,
        Pagamento::Cartao { parcelas, .. } if *parcelas <= 6 => 3.5,
        Pagamento::Cartao { .. } => 4.9,
        Pagamento::Boleto(_) => 1.0,
    }
}

fn descrever(p: &Pagamento) -> String {
    match p {
        Pagamento::Pix { chave } => format!("Pix para {chave}"),
        Pagamento::Cartao { final_numero, parcelas } => {
            format!("Cartão final {final_numero} em {parcelas}x")
        }
        Pagamento::Boleto(linha) => format!("Boleto {}...", &linha[..10]),
    }
}

fn main() {
    let pagamentos = vec![
        Pagamento::Pix { chave: "loja@exemplo.com".into() },
        Pagamento::Cartao { final_numero: "4242".into(), parcelas: 10 },
        Pagamento::Boleto("23790.12345 60000.000003".into()),
    ];

    for p in &pagamentos {
        println!("{} (taxa {}%)", descrever(p), taxa_percentual(p));
    }
}
```

Na orientação a objetos, isso exigiria uma classe base, três subclasses e campos que ficariam nulos em metade dos casos. Aqui cada variante carrega exatamente o que precisa.

### Exemplo 2 — Máquina de estados de um pedido

```rust
#[derive(Debug)]
enum StatusPedido {
    Criado,
    Pago { transacao: String },
    Enviado { rastreio: String },
    Entregue,
    Cancelado { motivo: String },
}

impl StatusPedido {
    fn avancar(self, info: &str) -> Result<StatusPedido, String> {
        use StatusPedido::*;
        match self {
            Criado => Ok(Pago { transacao: info.into() }),
            Pago { .. } => Ok(Enviado { rastreio: info.into() }),
            Enviado { .. } => Ok(Entregue),
            Entregue | Cancelado { .. } => Err(format!("Pedido finalizado: {:?}", self)),
        }
    }
}

fn main() -> Result<(), String> {
    let s = StatusPedido::Criado;
    let s = s.avancar("TX-981")?;
    let s = s.avancar("BR123456789")?;
    let s = s.avancar("")?;
    println!("{:?}", s);              // Entregue
    println!("{:?}", s.avancar(""));   // Err("Pedido finalizado: Entregue")
    Ok(())
}
```

Não existe como um pedido ter código de rastreio sem estar `Enviado`: o dado só existe dentro da variante certa.

### Exemplo 3 — `if let`, `let else` e `matches!`

```rust
fn porta_do_ambiente(valor: Option<&str>) -> u16 {
    // let else: se não casar, precisa sair da função (return, panic, continue)
    let Some(texto) = valor else {
        return 8080;
    };
    texto.parse().unwrap_or(8080)
}

fn main() {
    let config = Some("3000");

    if let Some(p) = config {
        println!("Porta configurada: {p}");
    }

    println!("{}", porta_do_ambiente(None));         // 8080
    println!("{}", porta_do_ambiente(Some("9000"))); // 9000

    let codigo = 404;
    let erro_cliente = matches!(codigo, 400..=499);
    println!("Erro do cliente? {erro_cliente}");     // true
}
```

## 11. Tratamento de erros: Option, Result e o operador ?

**O que é:** Rust não tem exceções nem `null`. Ausência de valor é representada por `Option<T>` (`Some(valor)` ou `None`). Operações que podem falhar devolvem `Result<T, E>` (`Ok(valor)` ou `Err(erro)`). O operador `?` repassa o erro para quem chamou, encerrando a função cedo. Para falhas irrecuperáveis existe `panic!`.

**Para que serve:** tornar o erro **visível na assinatura** da função. Quem chama é obrigado a decidir o que fazer, e não existe *NullPointerException* esquecida em produção.

**Termos e siglas:**

- **`Option<T>`:** "pode ter um T ou não ter nada".
- **`Result<T, E>`:** "deu certo com um T ou falhou com um E".
- **`panic!`:** encerra a thread com mensagem. Para bugs, não para erros esperados.
- **`unwrap()` / `expect("msg")`:** extrai o valor ou causa panic. Aceitável em protótipos e testes.
- **Propagação:** devolver o erro para a função de cima, em vez de tratá-lo ali.
- **`From`:** trait de conversão. O `?` usa `From::from` para converter o erro de um tipo para o tipo de erro da sua função.
- **`thiserror` / `anyhow`:** crates populares. `thiserror` para bibliotecas (erros tipados); `anyhow` para aplicações (qualquer erro, com contexto).

| Situação | Ferramenta |
| --- | --- |
| Valor pode não existir (busca, campo opcional) | `Option<T>` |
| Operação pode falhar (IO, parse, rede) | `Result<T, E>` |
| Repassar erro para cima | `?` |
| Valor padrão se faltar | `unwrap_or`, `unwrap_or_default` |
| Transformar o valor de dentro | `map`, `and_then` |
| Bug: estado que nunca deveria acontecer | `panic!`, `unreachable!` |

### Exemplo 1 — Buscar usuário por id com `Option`

```rust
struct Usuario { id: u32, nome: String, email: Option<String> }

fn buscar(usuarios: &[Usuario], id: u32) -> Option<&Usuario> {
    usuarios.iter().find(|u| u.id == id)
}

fn main() {
    let base = vec![
        Usuario { id: 1, nome: "Ana".into(), email: Some("ana@ex.com".into()) },
        Usuario { id: 2, nome: "Rui".into(), email: None },
    ];

    // Encadeando: buscar o usuário e depois o e-mail, que também é opcional
    let contato = buscar(&base, 2)
        .and_then(|u| u.email.as_deref())
        .unwrap_or("sem e-mail cadastrado");
    println!("{contato}");

    match buscar(&base, 99) {
        Some(u) => println!("Achei {}", u.nome),
        None => println!("Usuário 99 não existe"),
    }
}
```

### Exemplo 2 — Ler a porta de um arquivo de configuração com erro próprio e `?`

```rust
use std::{fmt, fs, num::ParseIntError, io};

#[derive(Debug)]
enum ErroConfig {
    Leitura(io::Error),
    PortaInvalida(ParseIntError),
    PortaReservada(u16),
}

impl fmt::Display for ErroConfig {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            ErroConfig::Leitura(e) => write!(f, "não consegui ler o arquivo: {e}"),
            ErroConfig::PortaInvalida(e) => write!(f, "porta não é número: {e}"),
            ErroConfig::PortaReservada(p) => write!(f, "porta {p} é reservada (< 1024)"),
        }
    }
}

// Estas conversões permitem que o ? transforme os erros automaticamente
impl From<io::Error> for ErroConfig {
    fn from(e: io::Error) -> Self { ErroConfig::Leitura(e) }
}
impl From<ParseIntError> for ErroConfig {
    fn from(e: ParseIntError) -> Self { ErroConfig::PortaInvalida(e) }
}

fn ler_porta(caminho: &str) -> Result<u16, ErroConfig> {
    let texto = fs::read_to_string(caminho)?;   // io::Error -> ErroConfig
    let porta: u16 = texto.trim().parse()?;     // ParseIntError -> ErroConfig
    if porta < 1024 {
        return Err(ErroConfig::PortaReservada(porta));
    }
    Ok(porta)
}

fn main() {
    match ler_porta("porta.txt") {
        Ok(p) => println!("Subindo na porta {p}"),
        Err(e) => eprintln!("Erro de configuração: {e}"),
    }
}
```

Com o crate `thiserror`, o `Display` e os `From` acima viram atributos: `#[error("porta {0} é reservada")]` e `#[from]`. Vale aprender à mão primeiro para entender o que ele gera.

### Exemplo 3 — Aplicação com `anyhow` e contexto

```rust
// Cargo.toml: anyhow = "1"
use anyhow::{Context, Result, bail};

fn carregar_modelo(caminho: &str) -> Result<Vec<u8>> {
    let bytes = std::fs::read(caminho)
        .with_context(|| format!("falha ao abrir o modelo em {caminho}"))?;

    if bytes.is_empty() {
        bail!("arquivo de modelo vazio"); // atalho para return Err(...)
    }
    Ok(bytes)
}

fn main() -> Result<()> {
    let modelo = carregar_modelo("modelo.onnx")?;
    println!("{} bytes carregados", modelo.len());
    Ok(())
}
```

Se o arquivo não existir, a saída mostra a cadeia completa: *falha ao abrir o modelo em modelo.onnx* → *No such file or directory*. ONNX (Open Neural Network Exchange) é um formato aberto de modelos de ML.

### O que o `?` faz por baixo

O diagrama mostra o caminho de um valor ao passar por `?`.

&#91;embedded content: operador ? · caminho do Ok e do Err\]

Por isso o `?` só compila dentro de funções que devolvem `Result` ou `Option`: ele precisa de um tipo de retorno para onde mandar o erro.

## 12. Coleções: Vec, String, HashMap e companhia

**O que é:** coleções são estruturas da biblioteca padrão que guardam vários valores **na heap**, com tamanho que cresce em execução. As principais são `Vec<T>` (lista), `String` (texto UTF-8) e `HashMap<K, V>` (dicionário).

**Para que serve:** armazenar e processar conjuntos de dados cujo tamanho você não sabe na hora de compilar: linhas de um CSV, usuários de um sistema, contagem de palavras.

**Termos e siglas:**

- **Capacidade vs tamanho:** `len()` é quantos itens existem; `capacity()` é quanto espaço já está reservado. `Vec::with_capacity(n)` evita realocações.
- **Hash:** função que transforma uma chave em número para achar o valor rápido. Busca média em O(1).
- **O(1), O(n), O(log n):** notação Big-O, que diz como o custo cresce com a quantidade de dados. O(1) = constante; O(n) = proporcional.
- **Entry API:** forma do `HashMap` de "inserir se não existir, senão alterar" em uma só busca.
- **Byte vs char:** `String` é UTF-8, então `"ção"` tem 3 caracteres e 5 bytes. Por isso não existe `texto[0]`.
- **FIFO (First In, First Out):** primeiro a entrar, primeiro a sair. Comportamento de fila.

| Coleção | Equivalente em Python | Quando usar |
| --- | --- | --- |
| `Vec<T>` | `list` | Lista ordenada, acesso por índice. A mais usada |
| `VecDeque<T>` | `collections.deque` | Fila: inserir/remover nas duas pontas em O(1) |
| `HashMap<K, V>` | `dict` | Busca por chave, sem ordem |
| `BTreeMap<K, V>` | `dict` ordenado por chave | Precisa iterar em ordem de chave |
| `HashSet<T>` | `set` | Itens únicos, teste de pertença |
| `String` / `&str` | `str` | Texto dono / texto emprestado |

### Exemplo 1 — Notas da turma com `Vec`

```rust
fn main() {
    let mut notas: Vec<f64> = Vec::new();
    notas.push(7.5);
    notas.extend([9.0, 4.5, 6.0, 8.0]);

    // Ordenar f64 exige total_cmp, porque NaN não tem ordem
    notas.sort_by(|a, b| b.total_cmp(a)); // decrescente

    let media = notas.iter().sum::<f64>() / notas.len() as f64;
    let aprovados = notas.iter().filter(|&&n| n >= 6.0).count();

    println!("Ranking: {:?}", notas);   // [9.0, 8.0, 7.5, 6.0, 4.5]
    println!("Média: {media:.2}");       // 7.00
    println!("Aprovados: {aprovados}");  // 4

    // Acesso seguro: get devolve Option em vez de dar panic
    match notas.get(10) {
        Some(n) => println!("{n}"),
        None => println!("Não existe 11º aluno"),
    }
}
```

### Exemplo 2 — Contar palavras de um texto com `HashMap` (entry API)

```rust
use std::collections::HashMap;

fn main() {
    let texto = "o rato roeu a roupa do rei de roma e o rei riu";
    let mut contagem: HashMap<&str, u32> = HashMap::new();

    for palavra in texto.split_whitespace() {
        *contagem.entry(palavra).or_insert(0) += 1;
    }

    // HashMap não tem ordem: copiamos para Vec e ordenamos
    let mut ranking: Vec<(&str, u32)> = contagem.into_iter().collect();
    ranking.sort_by(|a, b| b.1.cmp(&a.1).then(a.0.cmp(b.0)));

    for (palavra, n) in ranking.iter().take(3) {
        println!("{palavra}: {n}");
    }
    // o: 2
    // rei: 2
    // a: 1
}
```

Esse é o "hello world" de processamento de texto e a base de um *bag of words* em NLP (Natural Language Processing, processamento de linguagem natural).

### Exemplo 3 — Fila de atendimento com `VecDeque` e strings UTF-8

```rust
use std::collections::VecDeque;

fn main() {
    let mut fila: VecDeque<String> = VecDeque::new();
    fila.push_back("João".to_string());
    fila.push_back("Conceição".to_string());
    fila.push_front("Dona Maria (prioridade)".to_string());

    while let Some(pessoa) = fila.pop_front() {
        println!("Atendendo: {pessoa}");
    }

    // Strings: bytes vs caracteres
    let nome = String::from("Conceição");
    println!("{} bytes, {} caracteres", nome.len(), nome.chars().count()); // 11 bytes, 9 caracteres
    let inicial: String = nome.chars().take(3).collect();
    println!("{inicial}"); // Con
    // let x = &nome[0..7]; // panic: o byte 7 cai no meio do 'ç'
}
```

## 13. Traits

**O que é:** uma *trait* define um conjunto de métodos que um tipo promete oferecer, como uma interface em Java/C# ou um protocolo em Python. Qualquer tipo pode implementar uma trait com `impl Trait for Tipo`. Traits podem ter métodos com implementação padrão.

**Para que serve:** escrever código que funciona para **qualquer tipo** que tenha um certo comportamento (polimorfismo), sem herança. Também é como Rust integra seus tipos à linguagem: imprimir (`Display`), comparar (`PartialEq`), iterar (`Iterator`), converter (`From`).

**Termos e siglas:**

- **Polimorfismo:** o mesmo código opera sobre tipos diferentes.
- **Trait bound (restrição):** exige que um tipo genérico implemente uma trait: `T: Display`.
- **`impl Trait`:** "algum tipo que implementa Trait". Resolvido na compilação (despacho estático).
- **`dyn Trait` (trait object):** "qualquer tipo que implementa Trait", escolhido em execução (despacho dinâmico).
- **vtable (virtual method table):** tabela de ponteiros para os métodos, usada pelo `dyn Trait` para saber qual função chamar.
- **Orphan rule (regra do órfão):** você só pode implementar uma trait para um tipo se a trait **ou** o tipo for do seu crate.

| Trait da std | Para que serve | Como obter |
| --- | --- | --- |
| `Debug` | Imprimir com `{:?}` | `#[derive(Debug)]` |
| `Display` | Imprimir com `{}` para o usuário | Implementar à mão |
| `Clone` / `Copy` | Duplicar valores | `derive` |
| `PartialEq` / `Eq` | Comparar com `==` | `derive` |
| `PartialOrd` / `Ord` | Comparar com `<`, ordenar | `derive` |
| `Hash` | Usar como chave de `HashMap` | `derive` |
| `Default` | Valor padrão (`Tipo::default()`) | `derive` ou à mão |
| `From` / `Into` | Conversão entre tipos | Implementar `From` (ganha `Into` grátis) |
| `Iterator` | Ser percorrido em `for` | Implementar `next()` |

### Exemplo 1 — Sistema de notificações com vários canais

```rust
trait Notificador {
    fn enviar(&self, destino: &str, msg: &str) -> Result<(), String>;

    // Método com implementação padrão
    fn nome_canal(&self) -> &str { "genérico" }
}

struct Email { remetente: String }
struct Sms;
struct Slack { webhook: String }

impl Notificador for Email {
    fn enviar(&self, destino: &str, msg: &str) -> Result<(), String> {
        println!("[email de {}] para {destino}: {msg}", self.remetente);
        Ok(())
    }
    fn nome_canal(&self) -> &str { "e-mail" }
}

impl Notificador for Sms {
    fn enviar(&self, destino: &str, msg: &str) -> Result<(), String> {
        if msg.len() > 160 {
            return Err("SMS passa de 160 caracteres".into());
        }
        println!("[sms] para {destino}: {msg}");
        Ok(())
    }
}

impl Notificador for Slack {
    fn enviar(&self, destino: &str, msg: &str) -> Result<(), String> {
        println!("[slack via {}] #{destino}: {msg}", self.webhook);
        Ok(())
    }
    fn nome_canal(&self) -> &str { "slack" }
}

fn main() {
    // Vec de trait objects: tipos diferentes na mesma lista
    let canais: Vec<Box<dyn Notificador>> = vec![
        Box::new(Email { remetente: "noreply@banco.com".into() }),
        Box::new(Sms),
        Box::new(Slack { webhook: "hooks/abc".into() }),
    ];

    for c in &canais {
        if let Err(e) = c.enviar("time-ops", "Deploy concluído") {
            eprintln!("Falha no canal {}: {e}", c.nome_canal());
        }
    }
}
```

Para adicionar WhatsApp amanhã, basta criar a struct e implementar a trait. Nenhuma linha do `main` muda.

### Exemplo 2 — `Display` e `From` para um tipo monetário

```rust
use std::fmt;

#[derive(Debug, Clone, Copy, PartialEq, PartialOrd, Default)]
struct Dinheiro { centavos: i64 }

impl fmt::Display for Dinheiro {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        let reais = self.centavos / 100;
        let cent = (self.centavos % 100).abs();
        write!(f, "R$ {reais},{cent:02}")
    }
}

impl From<f64> for Dinheiro {
    fn from(valor: f64) -> Self {
        Dinheiro { centavos: (valor * 100.0).round() as i64 }
    }
}

impl std::ops::Add for Dinheiro {
    type Output = Dinheiro;
    fn add(self, outro: Dinheiro) -> Dinheiro {
        Dinheiro { centavos: self.centavos + outro.centavos }
    }
}

fn main() {
    let a = Dinheiro::from(19.90);
    let b: Dinheiro = 5.5.into();          // Into vem de graça com From
    println!("{}", a + b);                 // R$ 25,40 (operador + via trait Add)
    println!("{}", Dinheiro::default());   // R$ 0,00
}
```

Repare que até o operador `+` é uma trait (`std::ops::Add`). Isso se chama sobrecarga de operadores.

### Exemplo 3 — `impl Trait` (estático) vs `dyn Trait` (dinâmico)

```rust
use std::fmt::Display;

// Estático: o compilador gera uma versão para cada tipo usado. Mais rápido.
fn log_estatico(item: impl Display) {
    println!("[LOG] {item}");
}

// Dinâmico: uma só versão, que consulta a vtable em execução. Mais flexível.
fn log_dinamico(item: &dyn Display) {
    println!("[LOG] {item}");
}

fn main() {
    log_estatico(42);
    log_estatico("texto");
    log_dinamico(&3.14);
}
```

Regra prática: use `impl Trait` ou generics por padrão; use `dyn Trait` quando precisar guardar tipos diferentes juntos (como o `Vec` do Exemplo 1) ou quando o tipo só é conhecido em execução (plugins).

## 14. Generics (tipos genéricos)

**O que é:** generics permitem escrever funções, structs e enums com **parâmetros de tipo** (`T`, `K`, `V`), que são preenchidos com tipos concretos no uso. `Vec<T>`, `Option<T>` e `Result<T, E>` são genéricos.

**Para que serve:** reaproveitar a mesma lógica para vários tipos sem duplicar código e sem perder a checagem de tipos. Combinados com trait bounds, você diz exatamente o que o tipo precisa saber fazer.

**Termos e siglas:**

- **Parâmetro de tipo:** o `T` em `fn f<T>(x: T)`. Por convenção, letras maiúsculas.
- **Trait bound:** restrição sobre `T`: `T: PartialOrd + Copy`.
- **Cláusula `where`:** lugar alternativo para escrever bounds longos, deixando a assinatura legível.
- **Monomorfização:** o compilador gera uma cópia da função para cada tipo concreto usado. Resultado: **custo zero** em execução, ao preço de binário um pouco maior e compilação mais lenta.
- **Turbofish `::<>`:** sintaxe para informar o tipo explicitamente: `"5".parse::<i32>()`.
- **Const generics:** parâmetro que é um valor constante, como o tamanho de um array: `[T; N]`.

### Exemplo 1 — Pilha genérica para desfazer/refazer de um editor

```rust
#[derive(Debug)]
struct Pilha<T> {
    itens: Vec<T>,
}

impl<T> Pilha<T> {
    fn new() -> Self { Pilha { itens: Vec::new() } }
    fn empilhar(&mut self, item: T) { self.itens.push(item); }
    fn desempilhar(&mut self) -> Option<T> { self.itens.pop() }
    fn topo(&self) -> Option<&T> { self.itens.last() }
}

#[derive(Debug, Clone)]
enum Acao { Digitar(String), Apagar(usize) }

fn main() {
    let mut desfazer: Pilha<Acao> = Pilha::new();
    let mut refazer: Pilha<Acao> = Pilha::new();

    desfazer.empilhar(Acao::Digitar("Olá".into()));
    desfazer.empilhar(Acao::Apagar(2));

    // Ctrl+Z
    if let Some(a) = desfazer.desempilhar() {
        println!("Desfazendo {:?}", a);
        refazer.empilhar(a);
    }
    println!("Topo agora: {:?}", desfazer.topo());

    // A mesma Pilha serve para números, sem reescrever nada
    let mut numeros = Pilha::new();
    numeros.empilhar(10);
    println!("{:?}", numeros);
}
```

### Exemplo 2 — Resposta padrão de API, genérica no conteúdo

```rust
#[derive(Debug)]
struct Resposta<T> {
    status: u16,
    dados: Option<T>,
    erro: Option<String>,
}

impl<T> Resposta<T> {
    fn ok(dados: T) -> Self { Resposta { status: 200, dados: Some(dados), erro: None } }
    fn falha(status: u16, msg: &str) -> Self {
        Resposta { status, dados: None, erro: Some(msg.into()) }
    }
}

#[derive(Debug)]
struct Produto { nome: String, preco: f64 }

fn main() {
    let r1 = Resposta::ok(Produto { nome: "Notebook".into(), preco: 4500.0 });
    let r2: Resposta<Vec<u32>> = Resposta::ok(vec![1, 2, 3]);
    let r3: Resposta<Produto> = Resposta::falha(404, "produto não encontrado");

    println!("{:?}\n{:?}\n{:?}", r1, r2, r3);
}
```

Com o crate `serde`, basta adicionar `#[derive(Serialize)]` e essa mesma struct vira o JSON de resposta de qualquer endpoint.

### Exemplo 3 — Função genérica com bounds e `where`

```rust
use std::fmt::Display;

// Devolve o maior elemento de qualquer slice de itens comparáveis
fn maior<T: PartialOrd>(itens: &[T]) -> Option<&T> {
    let mut maior = itens.first()?;
    for item in itens {
        if item > maior { maior = item; }
    }
    Some(maior)
}

// Bounds longos ficam mais legíveis com where
fn relatorio<K, V>(titulo: &str, pares: &[(K, V)])
where
    K: Display,
    V: Display + PartialOrd,
{
    println!("== {titulo} ==");
    for (k, v) in pares {
        println!("{k:<10} {v}");
    }
}

fn main() {
    println!("{:?}", maior(&[3, 9, 2]));             // Some(9)
    println!("{:?}", maior(&["pera", "uva", "abacaxi"])); // Some("uva")
    println!("{:?}", maior::<f64>(&[]));             // None

    relatorio("Vendas", &[("Janeiro", 1200), ("Fevereiro", 980)]);
}
```

Por causa da monomorfização, `maior::<i32>` e `maior::<&str>` viram duas funções separadas no binário, cada uma tão rápida quanto se você as tivesse escrito à mão.

## 15. Closures e iteradores

**O que é:** uma *closure* é uma função anônima que pode **capturar variáveis** do ambiente onde foi criada: `|x| x * fator`. Um *iterador* é qualquer tipo que implementa a trait `Iterator`, produzindo valores um por vez com `next()`. Os dois juntos formam pipelines como `filter → map → sum`.

**Para que serve:** processar coleções de forma declarativa, parecida com pandas ou list comprehensions do Python, mas com o desempenho de um laço escrito à mão. Closures também servem como callbacks: ordenação, tratamento de eventos, threads.

**Termos e siglas:**

- **Captura:** a closure usa uma variável de fora. Pode ser por referência, por referência mutável ou por posse (`move`).
- **`Fn` / `FnMut` / `FnOnce`:** as três traits de closure. `Fn` só lê o que capturou; `FnMut` altera; `FnOnce` consome e só pode ser chamada uma vez.
- **`move`:** força a closure a tomar posse do que captura. Obrigatório ao enviar para outra thread.
- **Lazy (preguiçoso):** adaptadores como `map` e `filter` não fazem nada até um **consumidor** (`collect`, `sum`, `for`) puxar os valores.
- **Adaptador:** método que transforma um iterador em outro (`map`, `filter`, `take`, `zip`, `enumerate`).
- **Consumidor:** método que esgota o iterador e produz um resultado (`collect`, `sum`, `count`, `fold`, `max`).

| Método | Retorna | Exemplo |
| --- | --- | --- |
| `iter()` | Itera por `&T` | lê sem consumir a coleção |
| `iter_mut()` | Itera por `&mut T` | altera cada item no lugar |
| `into_iter()` | Itera por `T` | consome a coleção |

### Exemplo 1 — Pipeline de transações financeiras

```rust
#[derive(Debug)]
struct Transacao { conta: String, valor: f64, tipo: Tipo }

#[derive(Debug, PartialEq)]
enum Tipo { Credito, Debito }

fn main() {
    let transacoes = vec![
        Transacao { conta: "A".into(), valor: 1500.0, tipo: Tipo::Debito },
        Transacao { conta: "B".into(), valor: 80.0, tipo: Tipo::Debito },
        Transacao { conta: "A".into(), valor: 3000.0, tipo: Tipo::Credito },
        Transacao { conta: "C".into(), valor: 12000.0, tipo: Tipo::Debito },
    ];

    let limite_alerta = 1000.0; // capturado pela closure abaixo

    // Débitos acima do limite, para análise antifraude
    let suspeitas: Vec<&Transacao> = transacoes
        .iter()
        .filter(|t| t.tipo == Tipo::Debito && t.valor > limite_alerta)
        .collect();

    let total_debitos: f64 = transacoes
        .iter()
        .filter(|t| t.tipo == Tipo::Debito)
        .map(|t| t.valor)
        .sum();

    let maior = transacoes
        .iter()
        .max_by(|a, b| a.valor.total_cmp(&b.valor))
        .map(|t| &t.conta);

    println!("{} suspeitas", suspeitas.len());      // 2
    println!("Total de débitos: {total_debitos}");  // 13580
    println!("Conta com maior valor: {:?}", maior); // Some("C")
}
```

Equivalente em pandas: `df[(df.tipo == "D") & (df.valor > 1000)]`. A diferença é que aqui o compilador funde tudo em um único laço, sem criar listas intermediárias.

### Exemplo 2 — Ordenar funcionários e capturar estado com `FnMut`

```rust
struct Funcionario { nome: String, salario: u32, setor: String }

fn main() {
    let mut equipe = vec![
        Funcionario { nome: "Ana".into(), salario: 9000, setor: "TI".into() },
        Funcionario { nome: "Bia".into(), salario: 7000, setor: "RH".into() },
        Funcionario { nome: "Caio".into(), salario: 12000, setor: "TI".into() },
    ];

    // Closure como critério de ordenação
    equipe.sort_by_key(|f| std::cmp::Reverse(f.salario));

    // FnMut: a closure altera uma variável capturada
    let mut folha_ti = 0;
    equipe.iter()
        .filter(|f| f.setor == "TI")
        .for_each(|f| folha_ti += f.salario);

    let nomes: Vec<&str> = equipe.iter().map(|f| f.nome.as_str()).collect();
    println!("{:?} | folha TI: {folha_ti}", nomes); // ["Caio", "Ana", "Bia"] | folha TI: 21000

    // move: a closure leva uma cópia de `taxa` consigo
    let taxa = 0.1;
    let aplicar_bonus = move |s: u32| (s as f64 * (1.0 + taxa)) as u32;
    println!("{}", aplicar_bonus(9000)); // 9900
}
```

### Exemplo 3 — Criando seu próprio iterador: paginação de API

```rust
struct Paginas { total_itens: u32, por_pagina: u32, atual: u32 }

impl Iterator for Paginas {
    type Item = (u32, u32); // (offset, limit)

    fn next(&mut self) -> Option<Self::Item> {
        let offset = self.atual * self.por_pagina;
        if offset >= self.total_itens {
            return None; // fim da iteração
        }
        self.atual += 1;
        let limite = self.por_pagina.min(self.total_itens - offset);
        Some((offset, limite))
    }
}

fn main() {
    let paginas = Paginas { total_itens: 250, por_pagina: 100, atual: 0 };

    // Como implementamos Iterator, ganhamos for, map, enumerate... de graça
    for (n, (offset, limit)) in paginas.enumerate() {
        println!("Página {}: GET /clientes?offset={offset}&limit={limit}", n + 1);
    }
}
```

Saída: três páginas, com `limit` 100, 100 e 50. Implementando só `next()`, o tipo herda mais de 70 métodos prontos da trait `Iterator`.

## 16. Smart pointers: Box, Rc, RefCell, Arc

**O que é:** *smart pointers* (ponteiros inteligentes) são structs que se comportam como referências, mas **são donas** do dado e trazem regras extras: guardar na heap, permitir vários donos, ou permitir mutação checada em execução.

**Para que serve:** resolver os casos em que as regras básicas de ownership não bastam: tipos recursivos (árvores), dados com vários donos (grafos, caches compartilhados) e dados compartilhados entre threads.

**Termos e siglas:**

- **`Box<T>`:** coloca `T` na heap, com um único dono. Custo mínimo.
- **`Rc<T>` (Reference Counted):** vários donos na mesma thread. Conta quantos donos existem; libera quando chega a zero.
- **`Arc<T>` (Atomic Reference Counted):** como `Rc`, mas a contagem usa operações **atômicas**, seguras entre threads.
- **Atômica:** operação que a CPU executa de forma indivisível, sem que outra thread veja um estado intermediário.
- **`RefCell<T>`:** permite alterar o conteúdo mesmo com referência imutável (*interior mutability*, mutabilidade interior). As regras de empréstimo são checadas **em execução**: violar causa panic.
- **`Mutex<T>` (mutual exclusion, exclusão mútua):** versão de `RefCell` para threads: só uma thread por vez acessa o dado.
- **`Weak<T>`:** referência que não conta como dono. Evita ciclos que vazariam memória.
- **`Deref`:** trait que faz o smart pointer se comportar como referência (`*box`, chamada de métodos direto).

| Ponteiro | Donos | Muta o conteúdo? | Threads? | Uso típico |
| --- | --- | --- | --- | --- |
| `Box<T>` | 1 | Se o Box for `mut` | Sim, se `T` permitir | Árvores, `dyn Trait`, dados grandes |
| `Rc<T>` | Vários | Não | Não | Grafos, dados compartilhados de leitura |
| `Rc<RefCell<T>>` | Vários | Sim (checagem em execução) | Não | Estado compartilhado em uma thread |
| `Arc<T>` | Vários | Não | Sim | Config compartilhada entre threads |
| `Arc<Mutex<T>>` | Vários | Sim (com trava) | Sim | Contadores, caches entre threads |

### Exemplo 1 — `Box` para árvore de expressão de uma calculadora

```rust
// Sem Box, Expr teria tamanho infinito (contém a si mesma)
enum Expr {
    Num(f64),
    Soma(Box<Expr>, Box<Expr>),
    Mult(Box<Expr>, Box<Expr>),
}

fn avaliar(e: &Expr) -> f64 {
    match e {
        Expr::Num(n) => *n,
        Expr::Soma(a, b) => avaliar(a) + avaliar(b),
        Expr::Mult(a, b) => avaliar(a) * avaliar(b),
    }
}

fn main() {
    // (2 + 3) * 4
    let expr = Expr::Mult(
        Box::new(Expr::Soma(Box::new(Expr::Num(2.0)), Box::new(Expr::Num(3.0)))),
        Box::new(Expr::Num(4.0)),
    );
    println!("{}", avaliar(&expr)); // 20
}
```

Compiladores, interpretadores e parsers de JSON usam exatamente essa estrutura, chamada AST (Abstract Syntax Tree, árvore sintática abstrata).

### Exemplo 2 — `Rc<RefCell>` para um carrinho compartilhado por vários componentes

```rust
use std::cell::RefCell;
use std::rc::Rc;

struct Carrinho { itens: Vec<String> }

struct BotaoComprar { carrinho: Rc<RefCell<Carrinho>> }
struct ResumoLateral { carrinho: Rc<RefCell<Carrinho>> }

impl BotaoComprar {
    fn clicar(&self, produto: &str) {
        self.carrinho.borrow_mut().itens.push(produto.into()); // empréstimo mutável em execução
    }
}

impl ResumoLateral {
    fn mostrar(&self) {
        let c = self.carrinho.borrow(); // empréstimo de leitura em execução
        println!("Carrinho: {} itens {:?}", c.itens.len(), c.itens);
    }
}

fn main() {
    let carrinho = Rc::new(RefCell::new(Carrinho { itens: vec![] }));

    let botao = BotaoComprar { carrinho: Rc::clone(&carrinho) };
    let resumo = ResumoLateral { carrinho: Rc::clone(&carrinho) };

    botao.clicar("Fone");
    botao.clicar("Capa");
    resumo.mostrar();

    println!("Donos do carrinho: {}", Rc::strong_count(&carrinho)); // 3
}
```

`Rc::clone` não copia o carrinho: só incrementa o contador de donos. Os dois componentes enxergam o mesmo dado, padrão comum em interfaces gráficas e jogos.

### Exemplo 3 — `Arc` para configuração lida por várias threads

```rust
use std::sync::Arc;
use std::thread;

struct Config { url_banco: String, max_conexoes: u32 }

fn main() {
    let config = Arc::new(Config {
        url_banco: "postgres://db:5432/app".into(),
        max_conexoes: 20,
    });

    let mut handles = vec![];
    for id in 0..3 {
        let cfg = Arc::clone(&config); // cada thread ganha um dono
        handles.push(thread::spawn(move || {
            println!("Worker {id} usa {} (max {})", cfg.url_banco, cfg.max_conexoes);
        }));
    }
    for h in handles { h.join().unwrap(); }
}
```

Trocar `Arc` por `Rc` aqui dá erro de compilação: *`Rc<Config>` cannot be sent between threads safely*. O compilador sabe que a contagem do `Rc` não é atômica.

### Qual ponteiro usar?

O diagrama abaixo é a árvore de decisão que eu uso em revisão de código.

&#91;embedded content: escolha de smart pointer · 5 perguntas, 6 resultados\]

Comece sempre pelo ramo mais simples (`T` ou `&T`); suba para `Rc`, `Arc` ou travas só quando o compilador mostrar que a posse precisa ser compartilhada.

## 17. Módulos, crates e visibilidade

**O que é:** o sistema de módulos organiza o código em uma árvore. Um **crate** é a unidade de compilação (biblioteca ou binário). Dentro dele, **módulos** (`mod`) agrupam funções, structs e outros módulos. Tudo é **privado por padrão**; `pub` torna público. `use` traz nomes para o escopo atual.

**Para que serve:** dividir projetos grandes em partes coesas, esconder detalhes de implementação e expor uma API pública pequena e estável.

**Termos e siglas:**

- **Crate raiz:** `src/main.rs` (binário) ou `src/lib.rs` (biblioteca). É a raiz da árvore de módulos.
- **Package:** o que o `Cargo.toml` descreve. Contém no máximo uma biblioteca e quantos binários quiser.
- **Caminho (path):** endereço de um item: `crate::modelos::Cliente`.
- **`crate::` / `super::` / `self::`:** a partir da raiz / do módulo pai / do módulo atual.
- **`pub(crate)`:** visível em todo o crate, mas não para quem usa a biblioteca.
- **Re-export (`pub use`):** expõe um item em outro caminho, para simplificar a API pública.
- **Prelude:** itens importados automaticamente em todo arquivo (`Vec`, `String`, `Option`...).

| Visibilidade | Quem enxerga |
| --- | --- |
| (nada) | Só o próprio módulo e seus filhos |
| `pub(super)` | O módulo pai |
| `pub(crate)` | Todo o crate |
| `pub` | Todos, inclusive outros crates |

### Exemplo 1 — Organização de uma API de clientes em arquivos

```text
src/
├── main.rs
├── modelos.rs          # mod modelos
├── repositorio.rs      # mod repositorio
└── servicos/
    ├── mod.rs          # mod servicos
    └── cadastro.rs     # mod servicos::cadastro
```

```rust
// src/modelos.rs
#[derive(Debug, Clone)]
pub struct Cliente {
    pub id: u32,
    pub nome: String,
    cpf: String,              // privado: ninguém de fora lê direto
}

impl Cliente {
    pub fn new(id: u32, nome: &str, cpf: &str) -> Self {
        Self { id, nome: nome.into(), cpf: cpf.into() }
    }
    pub fn cpf_mascarado(&self) -> String {
        format!("***.***.***-{}", &self.cpf[self.cpf.len() - 2..])
    }
}
```

```rust
// src/repositorio.rs
use crate::modelos::Cliente;

pub struct Repositorio { clientes: Vec<Cliente> }

impl Repositorio {
    pub fn new() -> Self { Self { clientes: vec![] } }
    pub fn salvar(&mut self, c: Cliente) { self.clientes.push(c); }
    pub fn buscar(&self, id: u32) -> Option<&Cliente> {
        self.clientes.iter().find(|c| c.id == id)
    }
}
```

```rust
// src/servicos/mod.rs
pub mod cadastro;

// src/servicos/cadastro.rs
use crate::modelos::Cliente;
use crate::repositorio::Repositorio;

pub fn cadastrar(repo: &mut Repositorio, id: u32, nome: &str, cpf: &str) -> Result<(), String> {
    if cpf.len() != 11 {
        return Err("CPF deve ter 11 dígitos".into());
    }
    repo.salvar(Cliente::new(id, nome, cpf));
    Ok(())
}
```

```rust
// src/main.rs
mod modelos;
mod repositorio;
mod servicos;

use repositorio::Repositorio;
use servicos::cadastro;

fn main() {
    let mut repo = Repositorio::new();
    cadastro::cadastrar(&mut repo, 1, "Ana", "12345678901").unwrap();

    if let Some(c) = repo.buscar(1) {
        println!("{} - {}", c.nome, c.cpf_mascarado()); // Ana - ***.***.***-01
        // println!("{}", c.cpf); // ERRO E0616: campo privado
    }
}
```

O CPF (Cadastro de Pessoas Físicas) fica privado: só o método `cpf_mascarado` decide como ele aparece. Isso ajuda a cumprir a LGPD (Lei Geral de Proteção de Dados) por construção.

### Exemplo 2 — Biblioteca com fachada via `pub use`

```rust
// src/lib.rs de um crate "pagamentos"
mod interno {
    pub mod gateways {
        pub mod pix {
            pub fn gerar_qrcode(valor: f64) -> String {
                format!("PIX|{valor:.2}|{}", super::super::util::checksum())
            }
        }
    }
    pub(crate) mod util {
        pub(crate) fn checksum() -> u32 { 4242 }
    }
}

// Fachada: quem usa a lib escreve pagamentos::gerar_qrcode
// em vez de pagamentos::interno::gateways::pix::gerar_qrcode
pub use interno::gateways::pix::gerar_qrcode;
```

```rust
// Em outro crate que depende de "pagamentos"
fn main() {
    println!("{}", pagamentos::gerar_qrcode(99.9));
    // pagamentos::interno::util::checksum(); // ERRO: módulo privado
}
```

Crates populares fazem isso o tempo todo. Por exemplo, `tokio::spawn` é um re-export de `tokio::task::spawn`. Você pode reorganizar o interno sem quebrar quem usa a biblioteca.

## 18. Concorrência: threads, channels e Mutex

**O que é:** concorrência é executar várias tarefas ao mesmo tempo. Rust usa **threads do sistema operacional** (`std::thread`), **canais** para trocar mensagens entre elas (`mpsc`) e **travas** para compartilhar estado (`Mutex`, `RwLock`). As traits `Send` e `Sync` dizem ao compilador o que pode cruzar a fronteira entre threads.

**Para que serve:** aproveitar todos os núcleos da CPU em tarefas pesadas (processar imagens, treinar modelos, comprimir arquivos). O lema da linguagem é **fearless concurrency** (concorrência sem medo): condições de corrida viram erro de compilação.

**Termos e siglas:**

- **Thread:** linha de execução independente dentro do mesmo processo, que compartilha a memória dele.
- **OS (Operating System):** sistema operacional, quem cria e agenda as threads.
- **`spawn`:** cria uma thread. Devolve um `JoinHandle`.
- **`join`:** espera a thread terminar e pega o resultado.
- **mpsc (multiple producer, single consumer):** canal com vários produtores e um consumidor.
- **Deadlock (impasse):** duas threads esperando, cada uma, a trava da outra. Rust **não** impede deadlock; mantenha travas curtas.
- **`Send`:** o tipo pode ser **movido** para outra thread. **`Sync`:** o tipo pode ser **compartilhado por referência** entre threads.
- **CPU-bound vs IO-bound:** limitado por cálculo (use threads) vs limitado por espera de rede/disco (use async, parte 19).

### Exemplo 1 — Processar arquivos em paralelo com workers e canal

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn processar(arquivo: &str) -> usize {
    thread::sleep(Duration::from_millis(200)); // simula trabalho pesado
    arquivo.len() * 1000                         // "linhas processadas"
}

fn main() {
    let arquivos = vec!["vendas_jan.csv", "vendas_fev.csv", "vendas_mar.csv", "vendas_abr.csv"];
    let (tx, rx) = mpsc::channel();

    for arquivo in arquivos {
        let tx = tx.clone(); // cada produtor tem seu transmissor
        thread::spawn(move || {
            let linhas = processar(arquivo);
            tx.send((arquivo, linhas)).unwrap();
        });
    }
    drop(tx); // fecha o transmissor original: o laço abaixo termina quando todos acabarem

    let mut total = 0;
    for (arquivo, linhas) in rx {         // consumidor recebe na ordem em que terminam
        println!("{arquivo}: {linhas} linhas");
        total += linhas;
    }
    println!("Total: {total}");
}
```

Os quatro arquivos são processados em cerca de 200 ms, e não 800 ms, porque cada um roda em sua thread.

### Exemplo 2 — Contador de acessos compartilhado com `Arc<Mutex>`

```rust
use std::collections::HashMap;
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let acessos: Arc<Mutex<HashMap<String, u32>>> = Arc::new(Mutex::new(HashMap::new()));
    let rotas = ["/login", "/home", "/login", "/extrato", "/home", "/login"];

    let handles: Vec<_> = rotas
        .iter()
        .map(|&rota| {
            let acessos = Arc::clone(&acessos);
            thread::spawn(move || {
                let mut mapa = acessos.lock().unwrap(); // trava
                *mapa.entry(rota.to_string()).or_insert(0) += 1;
            }) // a trava é liberada aqui, quando `mapa` sai de escopo (RAII)
        })
        .collect();

    for h in handles { h.join().unwrap(); }

    println!("{:?}", acessos.lock().unwrap());
    // {"/login": 3, "/home": 2, "/extrato": 1} (ordem pode variar)
}
```

Tente tirar o `Mutex` e usar só `Arc<HashMap>`: o compilador recusa a alteração, porque `Arc` só dá acesso de leitura. Em Java ou Go, o mesmo erro compilaria e daria resultado errado de vez em quando.

### Exemplo 3 — Paralelismo de dados com `rayon` e threads com escopo

```rust
// Cargo.toml: rayon = "1"
use rayon::prelude::*;

fn eh_primo(n: u64) -> bool {
    n >= 2 && (2..=((n as f64).sqrt() as u64)).all(|d| n % d != 0)
}

fn main() {
    // Trocar iter() por par_iter() distribui o trabalho entre todos os núcleos
    let quantidade = (1..5_000_000u64).into_par_iter().filter(|&n| eh_primo(n)).count();
    println!("{quantidade} primos");

    // thread::scope: threads que podem pegar emprestado dados da stack
    let mut metade_a = vec![1, 2, 3];
    let mut metade_b = vec![4, 5, 6];
    std::thread::scope(|s| {
        s.spawn(|| metade_a.iter_mut().for_each(|x| *x *= 10));
        s.spawn(|| metade_b.iter_mut().for_each(|x| *x *= 10));
    }); // todas as threads do escopo terminam aqui
    println!("{:?} {:?}", metade_a, metade_b); // [10, 20, 30] [40, 50, 60]
}
```

`rayon` é o equivalente a um `multiprocessing.Pool` do Python, sem o custo de copiar dados entre processos, porque threads compartilham memória.

### Fluxo do Exemplo 1

O diagrama mostra vários produtores enviando resultados por um único canal para o consumidor.

&#91;embedded content: produtores e consumidor com mpsc · 4 threads, 1 canal\]

Como cada thread só envia mensagens e nunca toca a memória das outras, não há trava nem condição de corrida: o dado muda de dono ao passar pelo canal.

## 19. Async/await com Tokio

**O que é:** programação assíncrona permite que **uma thread atenda milhares de tarefas** que passam a maior parte do tempo esperando (rede, banco, disco). Uma `async fn` devolve um **Future**, um valor que representa um resultado que ainda vai ficar pronto. `.await` pausa a tarefa até o Future concluir, liberando a thread para outras tarefas. Rust não traz um executor na std; o mais usado é o **Tokio**.

**Para que serve:** servidores web, clientes HTTP, bots, gateways de API, consumidores de fila. Qualquer coisa IO-bound com muitas conexões simultâneas. Para cálculo pesado (CPU-bound), threads ou `rayon` continuam sendo a escolha.

**Termos e siglas:**

- **Future:** trait que representa um cálculo assíncrono. É **preguiçoso**: não faz nada até ser aguardado com `.await` ou entregue ao executor.
- **Runtime / executor:** o "motor" que agenda e executa os Futures. Tokio usa um pool de threads com *work stealing* (threads ociosas roubam tarefas das ocupadas).
- **Task (tarefa):** um Future entregue ao runtime com `tokio::spawn`. É uma "thread verde": custa poucos KB, contra cerca de 2 MB de pilha de uma thread do OS por padrão.
- **Poll:** o executor pergunta ao Future "já terminou?". A resposta é `Ready(valor)` ou `Pending`.
- **Waker:** mecanismo que o Future usa para avisar o executor "agora pode me perguntar de novo".
- **Event loop / reactor:** parte do runtime que monitora sockets e timers (via epoll no Linux, kqueue no macOS, IOCP no Windows) e acorda as tarefas certas.
- **`join!` / `select!`:** aguardar vários Futures juntos / ficar com o primeiro que terminar.
- **HTTP (HyperText Transfer Protocol)** e **JSON (JavaScript Object Notation):** protocolo e formato usados nos exemplos.

### Exemplo 1 — Chamar três serviços ao mesmo tempo

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```

```rust
use std::time::{Duration, Instant};
use tokio::time::sleep;

async fn buscar_saldo(conta: u32) -> f64 {
    sleep(Duration::from_millis(300)).await; // simula chamada de rede
    1520.75 + conta as f64
}

async fn buscar_limite(conta: u32) -> f64 {
    sleep(Duration::from_millis(500)).await;
    5000.0 + conta as f64
}

async fn buscar_score(cpf: &str) -> u32 {
    sleep(Duration::from_millis(400)).await;
    if cpf.ends_with('0') { 820 } else { 650 }
}

#[tokio::main] // macro que cria o runtime e roda o main assíncrono
async fn main() {
    let inicio = Instant::now();

    // Em sequência levaria 300 + 500 + 400 = 1200 ms.
    // Com join!, as três esperas acontecem juntas: ~500 ms (a mais lenta).
    let (saldo, limite, score) = tokio::join!(
        buscar_saldo(42),
        buscar_limite(42),
        buscar_score("12345678900"),
    );

    println!("saldo={saldo} limite={limite} score={score}");
    println!("tempo: {:?}", inicio.elapsed());
}
```

Esse é o padrão de um BFF (Backend For Frontend): uma tela de app que precisa de dados de vários microsserviços e não pode esperar um de cada vez.

### Exemplo 2 — API REST com Axum

REST (Representational State Transfer) é o estilo de API baseado em recursos e verbos HTTP (GET, POST...).

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
axum = "0.8"
serde = { version = "1", features = ["derive"] }
```

```rust
use axum::{extract::{Path, State}, http::StatusCode, routing::get, Json, Router};
use serde::{Deserialize, Serialize};
use std::{collections::HashMap, sync::Arc};
use tokio::sync::RwLock;

#[derive(Clone, Serialize, Deserialize)]
struct Produto { id: u32, nome: String, preco: f64 }

type Banco = Arc<RwLock<HashMap<u32, Produto>>>;

async fn listar(State(db): State<Banco>) -> Json<Vec<Produto>> {
    Json(db.read().await.values().cloned().collect())
}

async fn obter(State(db): State<Banco>, Path(id): Path<u32>) -> Result<Json<Produto>, StatusCode> {
    db.read().await.get(&id).cloned().map(Json).ok_or(StatusCode::NOT_FOUND)
}

async fn criar(State(db): State<Banco>, Json(p): Json<Produto>) -> StatusCode {
    db.write().await.insert(p.id, p);
    StatusCode::CREATED
}

#[tokio::main]
async fn main() {
    let db: Banco = Arc::new(RwLock::new(HashMap::new()));

    let app = Router::new()
        .route("/produtos", get(listar).post(criar))
        .route("/produtos/{id}", get(obter))
        .with_state(db);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("Ouvindo em http://localhost:3000");
    axum::serve(listener, app).await.unwrap();
}
```

```bash
curl -X POST localhost:3000/produtos -H 'content-type: application/json' \
     -d '{"id":1,"nome":"Teclado","preco":199.9}'
curl localhost:3000/produtos/1
```

Cada requisição vira uma task. Um servidor desses atende dezenas de milhares de conexões simultâneas com poucos MB de memória. Note o `tokio::sync::RwLock`: dentro de código async, prefira as travas do Tokio quando a trava precisar atravessar um `.await`.

### Exemplo 3 — Timeout e cancelamento com `select!`

```rust
use std::time::Duration;
use tokio::time::{sleep, timeout};

async fn consultar_modelo_ia(pergunta: &str) -> String {
    sleep(Duration::from_secs(3)).await; // modelo lento
    format!("resposta para: {pergunta}")
}

#[tokio::main]
async fn main() {
    // Forma 1: timeout direto
    match timeout(Duration::from_secs(1), consultar_modelo_ia("resuma o contrato")).await {
        Ok(r) => println!("{r}"),
        Err(_) => println!("Modelo demorou demais, usando resposta em cache"),
    }

    // Forma 2: select! fica com o primeiro que terminar e cancela o outro
    tokio::select! {
        r = consultar_modelo_ia("classifique o ticket") => println!("{r}"),
        _ = tokio::signal::ctrl_c() => println!("Encerrando a pedido do usuário"),
    }
}
```

Ao perder o `select!`, o Future do modelo é simplesmente descartado (*drop*), e o trabalho pendente para. Esse é o modelo de cancelamento do Rust async.

### Como o runtime executa as tarefas

O diagrama mostra o ciclo de poll: o executor só volta a uma tarefa quando o reactor avisa que o dado dela chegou.

&#91;embedded content: ciclo de poll do runtime Tokio · executor, reactor e waker\]

Enquanto uma task espera a rede, a thread atende outras. Por isso código bloqueante (como `std::thread::sleep` ou cálculo pesado) dentro de `async` trava o runtime: use `tokio::time::sleep` ou `tokio::task::spawn_blocking`.

## 20. Macros

**O que é:** macros são código que **gera código** durante a compilação. Há dois tipos: **declarativas** (`macro_rules!`), que funcionam por casamento de padrões sobre a sintaxe, e **procedurais**, que são programas Rust que recebem e devolvem trechos de código (`#[derive(...)]`, `#[tokio::main]`, `sql!(...)`). Você reconhece uma macro pelo `!` no nome ou pelo `#[...]`.

**Para que serve:** eliminar código repetitivo que funções não conseguem resolver, como aceitar número variável de argumentos (`println!`, `vec!`), gerar implementações de traits automaticamente (`derive`) ou validar SQL e formatos em tempo de compilação.

**Termos e siglas:**

- **Metaprogramação:** escrever código que escreve código.
- **Token:** menor unidade da sintaxe (um nome, um número, um `+`).
- **TokenStream:** sequência de tokens que uma macro procedural recebe e devolve.
- **Fragment specifier (designador):** tipo de trecho que a macro aceita: `$x:expr` (expressão), `$t:ty` (tipo), `$i:ident` (identificador), `$l:literal`.
- **Repetição `$( ... ),*`:** casa zero ou mais itens separados por vírgula (`+` para um ou mais).
- **Higiene:** variáveis criadas dentro da macro não colidem com as de quem a usa.
- **`cargo expand`:** ferramenta que mostra o código depois que as macros são expandidas. Ótima para aprender.

| Macro | Tipo | O que gera |
| --- | --- | --- |
| `println!`, `format!` | Declarativa (embutida) | Formatação checada na compilação |
| `vec![1, 2, 3]` | Declarativa | Criação e preenchimento de um `Vec` |
| `#[derive(Debug)]` | Procedural (derive) | `impl Debug for Tipo` |
| `#[derive(Serialize)]` | Procedural (serde) | Conversão para JSON, YAML, etc. |
| `#[tokio::main]` | Procedural (atributo) | Cria o runtime e envolve o `main` |
| `sqlx::query!` | Procedural (função) | Valida o SQL contra o banco real ao compilar |

### Exemplo 1 — `hashmap!`: criar um mapa em uma linha

```rust
macro_rules! hashmap {
    // Casa: chave => valor, repetido, com vírgula final opcional
    ( $( $chave:expr => $valor:expr ),* $(,)? ) => {{
        let mut mapa = std::collections::HashMap::new();
        $( mapa.insert($chave, $valor); )*
        mapa
    }};
}

fn main() {
    let ddd = hashmap! {
        "São Paulo" => 11,
        "Rio de Janeiro" => 21,
        "Belo Horizonte" => 31,
    };
    println!("{:?}", ddd.get("São Paulo")); // Some(11)
}
```

A expansão vira três chamadas `insert`. Nenhuma função comum aceitaria um número livre de pares com essa sintaxe.

### Exemplo 2 — Macro de log com nível e local do código

```rust
macro_rules! log {
    ($nivel:literal, $($arg:tt)*) => {
        println!(
            "[{}] {}:{} - {}",
            $nivel,
            file!(),   // arquivo onde a macro foi CHAMADA
            line!(),   // linha onde foi chamada
            format!($($arg)*)
        )
    };
}

fn main() {
    let usuario = "ana";
    log!("INFO", "usuário {} fez login", usuario);
    log!("ERRO", "falha ao gravar pedido {}", 1042);
}
// [INFO] src/main.rs:15 - usuário ana fez login
```

Uma função não conseguiria descobrir a linha de quem a chamou dessa forma; a macro consegue porque é expandida no local da chamada. É por isso que crates de log como `tracing` usam macros (`info!`, `error!`).

### Exemplo 3 — Macros procedurais na prática: `serde`

```toml
[dependencies]
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
#[serde(rename_all = "camelCase")]   // nomes em camelCase no JSON
struct Aluno {
    nome_completo: String,
    matricula: u32,
    #[serde(default)]                // se faltar no JSON, usa o padrão
    bolsista: bool,
}

fn main() -> Result<(), serde_json::Error> {
    let json = r#"{ "nomeCompleto": "Carla Souza", "matricula": 20261 }"#;

    let aluno: Aluno = serde_json::from_str(json)?;  // JSON -> struct
    println!("{:?}", aluno);

    let saida = serde_json::to_string_pretty(&aluno)?; // struct -> JSON
    println!("{saida}");
    Ok(())
}
```

O `derive` gera centenas de linhas de código de conversão, otimizadas para o seu tipo exato. Escrever uma macro procedural própria exige um crate separado com `proc-macro = true` e os crates `syn` e `quote`: assunto avançado, mas é como `serde`, `clap` e `sqlx` são feitos.

> **Dica do professor:** prefira funções e generics. Use macro só quando a função não resolver: argumentos variáveis, sintaxe própria ou geração de código para muitos tipos.

## 21. Testes

**O que é:** Rust traz um framework de testes embutido. Uma função marcada com `#[test]` vira um teste, e `cargo test` encontra e executa todos eles em paralelo. Existem três tipos: **unitários** (no mesmo arquivo do código), **de integração** (pasta `tests/`) e **doc tests** (exemplos dentro da documentação, que também são executados).

**Para que serve:** garantir que o código faz o que deveria e continua fazendo depois de mudanças. Em Rust, o compilador já pega erros de tipo e memória; os testes cuidam da **lógica de negócio**.

**Termos e siglas:**

- **Asserção:** verificação que faz o teste falhar se for falsa: `assert!`, `assert_eq!`, `assert_ne!`.
- **`#[cfg(test)]`:** compila o módulo só quando rodando testes (não vai para o binário final).
- **`#[should_panic]`:** o teste passa se o código entrar em panic.
- **`#[ignore]`:** pula o teste por padrão (testes lentos); rode com `cargo test -- --ignored`.
- **TDD (Test-Driven Development):** escrever o teste antes do código.
- **CI (Continuous Integration, integração contínua):** rodar os testes automaticamente a cada push, por exemplo no GitHub Actions.
- **Mock:** objeto falso que substitui uma dependência externa (banco, API) no teste. Em Rust, geralmente feito com traits.

### Exemplo 1 — Testes unitários de uma regra de desconto

```rust
// src/lib.rs
pub fn preco_final(preco: f64, cupom: Option<&str>) -> Result<f64, String> {
    if preco < 0.0 {
        return Err("preço negativo".into());
    }
    let desconto = match cupom {
        None => 0.0,
        Some("ALUNO10") => 0.10,
        Some("BLACK30") => 0.30,
        Some(c) => return Err(format!("cupom inválido: {c}")),
    };
    Ok((preco * (1.0 - desconto) * 100.0).round() / 100.0)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn sem_cupom_mantem_preco() {
        assert_eq!(preco_final(100.0, None), Ok(100.0));
    }

    #[test]
    fn cupom_aluno_da_10_por_cento() {
        assert_eq!(preco_final(59.90, Some("ALUNO10")).unwrap(), 53.91);
    }

    #[test]
    fn cupom_invalido_retorna_erro() {
        let r = preco_final(10.0, Some("XPTO"));
        assert!(r.is_err());
        assert!(r.unwrap_err().contains("XPTO"));
    }

    #[test]
    fn preco_negativo_e_erro() -> Result<(), String> {
        // Testes também podem devolver Result
        match preco_final(-1.0, None) {
            Err(_) => Ok(()),
            Ok(v) => Err(format!("esperava erro, veio {v}")),
        }
    }
}
```

```bash
cargo test
# running 4 tests
# test tests::sem_cupom_mantem_preco ... ok
# test tests::cupom_aluno_da_10_por_cento ... ok
# ...
# test result: ok. 4 passed; 0 failed

cargo test cupom      # roda só os testes com "cupom" no nome
```

### Exemplo 2 — Mock com trait para testar sem banco de dados

```rust
pub trait RepositorioUsuarios {
    fn existe_email(&self, email: &str) -> bool;
}

pub fn pode_cadastrar(repo: &impl RepositorioUsuarios, email: &str) -> Result<(), String> {
    if !email.contains('@') {
        return Err("e-mail inválido".into());
    }
    if repo.existe_email(email) {
        return Err("e-mail já cadastrado".into());
    }
    Ok(())
}

#[cfg(test)]
mod tests {
    use super::*;

    // Implementação falsa: guarda e-mails em memória
    struct RepoFake { emails: Vec<String> }

    impl RepositorioUsuarios for RepoFake {
        fn existe_email(&self, email: &str) -> bool {
            self.emails.iter().any(|e| e == email)
        }
    }

    #[test]
    fn recusa_email_duplicado() {
        let repo = RepoFake { emails: vec!["ana@ex.com".into()] };
        assert_eq!(pode_cadastrar(&repo, "ana@ex.com"), Err("e-mail já cadastrado".into()));
    }

    #[test]
    fn aceita_email_novo() {
        let repo = RepoFake { emails: vec![] };
        assert!(pode_cadastrar(&repo, "bia@ex.com").is_ok());
    }
}
```

Em produção, outra struct implementa a mesma trait consultando o PostgreSQL. A regra de negócio é testada em milissegundos, sem subir banco.

### Exemplo 3 — Teste de integração e doc test

````rust
// src/lib.rs
/// Converte graus Celsius para Fahrenheit.
///
/// # Exemplo
///
/// ```
/// let f = loja::celsius_para_fahrenheit(100.0);
/// assert_eq!(f, 212.0);
/// ```
pub fn celsius_para_fahrenheit(c: f64) -> f64 {
    c * 9.0 / 5.0 + 32.0
}
````

```rust
// tests/api_publica.rs — enxerga o crate como um usuário externo
use loja::{celsius_para_fahrenheit, preco_final};

#[test]
fn fluxo_de_compra_com_cupom() {
    let total = preco_final(200.0, Some("BLACK30")).unwrap();
    assert_eq!(total, 140.0);
}

#[test]
fn ponto_de_congelamento() {
    assert_eq!(celsius_para_fahrenheit(0.0), 32.0);
}
```

O bloco de código no comentário `///` aparece na documentação gerada por `cargo doc` **e** é executado por `cargo test`. Documentação desatualizada vira teste falhando.

> **Dica do professor:** rode `cargo clippy` e `cargo test` no CI de todo projeto. Para cobertura de testes, use `cargo llvm-cov` ou `cargo tarpaulin`.

## 22. Unsafe e FFI

**O que é:** `unsafe` é um bloco onde você assume para si algumas garantias que o compilador não consegue provar. Ele libera cinco "superpoderes": desreferenciar ponteiros crus, chamar funções `unsafe`, acessar `static mut`, implementar traits `unsafe` e ler campos de `union`. **FFI** é a ponte para chamar código C (ou ser chamado por C, Python, Node...).

**Para que serve:** conversar com o sistema operacional, com bibliotecas em C (OpenSSL, SQLite, CUDA), com hardware em sistemas embarcados, e escrever as estruturas de baixo nível que a própria std usa por baixo (`Vec`, `Mutex`). A regra de ouro: **isole o `unsafe` em uma função pequena e exponha uma API segura**.

**Termos e siglas:**

- **Raw pointer (ponteiro cru):** `*const T` / `*mut T`. Como ponteiros de C: podem ser nulos ou inválidos.
- **`unsafe` não desliga o borrow checker:** só libera os cinco superpoderes. O resto continua checado.
- **FFI (Foreign Function Interface):** chamada entre linguagens.
- **ABI (Application Binary Interface):** convenção de chamada em nível de máquina. `extern "C"` usa a ABI de C, que todas as linguagens entendem.
- **`#[unsafe(no_mangle)]`:** mantém o nome da função exatamente como escrito no binário, para outra linguagem encontrá-la. (Na edition 2021 escrevia-se `#[no_mangle]`.)
- **`cdylib`:** tipo de crate que gera uma biblioteca dinâmica (`.so`, `.dll`, `.dylib`) para outras linguagens.
- **PyO3:** crate para escrever módulos Python em Rust. **maturin:** ferramenta que compila e empacota esses módulos como wheel (pacote instalável via pip).
- **GIL (Global Interpreter Lock):** trava do CPython que impede threads Python de rodarem em paralelo. Código Rust pode liberá-la e usar todos os núcleos.

### Exemplo 1 — Chamar funções da biblioteca C do sistema

```rust
use std::ffi::{CStr, CString};
use std::os::raw::{c_char, c_int};

// Edition 2024: blocos extern precisam ser marcados como unsafe
unsafe extern "C" {
    fn abs(x: c_int) -> c_int;
    fn strlen(s: *const c_char) -> usize;
}

// API segura que esconde o unsafe
fn tamanho_c(texto: &str) -> usize {
    let c = CString::new(texto).expect("texto com byte nulo"); // C exige terminador \0
    // SAFETY: `c` é uma string C válida e vive até o fim desta função
    unsafe { strlen(c.as_ptr()) }
}

fn main() {
    let v = unsafe { abs(-42) };
    println!("abs(-42) = {v}");
    println!("strlen = {}", tamanho_c("ferrugem"));

    // Ponteiro cru: criar é seguro, desreferenciar exige unsafe
    let numero = 7;
    let p = &numero as *const i32;
    unsafe { println!("valor via ponteiro: {}", *p); }

    // Ler uma string que veio do C
    let bruto = b"vindo do C\0";
    let s = unsafe { CStr::from_ptr(bruto.as_ptr() as *const c_char) };
    println!("{}", s.to_str().unwrap());
}
```

Em projetos reais, crates como `bindgen` geram essas declarações automaticamente a partir dos arquivos `.h` da biblioteca C.

### Exemplo 2 — Acelerar Python com um módulo em Rust (PyO3 + maturin)

```bash
pip install maturin
maturin new --bindings pyo3 rapido
cd rapido
```

```toml
# Cargo.toml
[package]
name = "rapido"
version = "0.1.0"
edition = "2021"

[lib]
name = "rapido"
crate-type = ["cdylib"]

[dependencies]
pyo3 = { version = "0.23", features = ["extension-module"] }
```

```rust
// src/lib.rs
use pyo3::prelude::*;

/// Distância euclidiana entre dois vetores (ex.: embeddings).
#[pyfunction]
fn distancia(a: Vec<f64>, b: Vec<f64>) -> PyResult<f64> {
    if a.len() != b.len() {
        return Err(pyo3::exceptions::PyValueError::new_err("vetores de tamanhos diferentes"));
    }
    Ok(a.iter().zip(&b).map(|(x, y)| (x - y).powi(2)).sum::<f64>().sqrt())
}

/// Conta palavras de vários textos, liberando o GIL durante o cálculo.
#[pyfunction]
fn contar_palavras(py: Python<'_>, textos: Vec<String>) -> usize {
    py.allow_threads(|| textos.iter().map(|t| t.split_whitespace().count()).sum())
}

#[pymodule]
fn rapido(m: &Bound<'_, PyModule>) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(distancia, m)?)?;
    m.add_function(wrap_pyfunction!(contar_palavras, m)?)?;
    Ok(())
}
```

```bash
maturin develop --release   # compila e instala no virtualenv atual
```

```python
import rapido

print(rapido.distancia([1.0, 2.0], [4.0, 6.0]))       # 5.0
print(rapido.contar_palavras(["olá mundo", "rust é rápido"]))  # 5

try:
    rapido.distancia([1.0], [1.0, 2.0])
except ValueError as e:
    print(e)  # vetores de tamanhos diferentes
```

Repare que um `Err` do Rust vira uma exceção Python normal. É assim que Polars, `tokenizers` e `pydantic-core` funcionam. A API do PyO3 muda entre versões; confira a versão do seu `Cargo.toml` na documentação em pyo3.rs.

### Exemplo 3 — Expor uma função Rust para C

```rust
// Cargo.toml: [lib] crate-type = ["cdylib"]

#[unsafe(no_mangle)]
pub extern "C" fn calcular_juros(principal: f64, taxa_mensal: f64, meses: u32) -> f64 {
    principal * (1.0 + taxa_mensal).powi(meses as i32)
}
```

```c
// main.c
#include <stdio.h>
double calcular_juros(double principal, double taxa_mensal, unsigned int meses);

int main(void) {
    printf("%.2f\n", calcular_juros(1000.0, 0.01, 12)); // 1126.83
    return 0;
}
```

Com isso, uma regra de cálculo escrita uma vez em Rust pode ser usada por sistemas legados em C, C++, Go (cgo) ou .NET (P/Invoke).

> **Dica do professor:** todo bloco `unsafe` deve ter um comentário `// SAFETY:` explicando por que aquilo é seguro. O `cargo clippy` pode exigir isso com a lint `undocumented_unsafe_blocks`.

## 23. Projeto final integrador e roteiro de estudo

O projeto final é uma CLI de tarefas que salva em JSON e junta, em cerca de 130 linhas, quase tudo deste guia. Monte, rode, e depois implemente os desafios.

### Onde cada conceito aparece

| Conceito | Parte | Onde no projeto |
| --- | --- | --- |
| Structs e métodos | 9 | `Tarefa`, `Lista` e seu `impl` |
| Enums e `match` | 10 | `Prioridade`, `Comando`, `interpretar` |
| Pattern matching em slices | 10 | `[cmd, titulo, prio] if cmd == "add"` |
| `Result`, `?`, `anyhow` | 11 | Todas as funções que podem falhar |
| `Vec` e iteradores | 12, 15 | `pendentes()`, `max()`, `sort_by_key` |
| Traits (`FromStr`, `Default`) | 13 | `"alta".parse::<Prioridade>()` |
| Borrowing | 7 | `&self` para ler, `&mut self` para alterar |
| Macros procedurais (serde) | 20 | `#[derive(Serialize, Deserialize)]` |
| Testes | 21 | Módulo `tests` no fim do arquivo |

### Código completo

```toml
# Cargo.toml
[package]
name = "tarefas"
version = "0.1.0"
edition = "2024"

[dependencies]
anyhow = "1"
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

```rust
// src/main.rs
use anyhow::{bail, Context, Result};
use serde::{Deserialize, Serialize};
use std::{env, fs, path::Path, str::FromStr};

const ARQUIVO: &str = "tarefas.json";

#[derive(Debug, Clone, Copy, PartialEq, Serialize, Deserialize)]
enum Prioridade { Baixa, Media, Alta }

impl FromStr for Prioridade {
    type Err = anyhow::Error;
    fn from_str(s: &str) -> Result<Self> {
        match s.to_lowercase().as_str() {
            "baixa" => Ok(Prioridade::Baixa),
            "media" | "média" => Ok(Prioridade::Media),
            "alta" => Ok(Prioridade::Alta),
            outro => bail!("prioridade inválida: {outro}"),
        }
    }
}

#[derive(Debug, Serialize, Deserialize)]
struct Tarefa { id: u32, titulo: String, prioridade: Prioridade, feita: bool }

#[derive(Default, Serialize, Deserialize)]
struct Lista { tarefas: Vec<Tarefa> }

impl Lista {
    fn carregar(caminho: &Path) -> Result<Self> {
        if !caminho.exists() {
            return Ok(Lista::default());
        }
        let texto = fs::read_to_string(caminho).context("lendo arquivo de tarefas")?;
        serde_json::from_str(&texto).context("JSON de tarefas corrompido")
    }

    fn salvar(&self, caminho: &Path) -> Result<()> {
        fs::write(caminho, serde_json::to_string_pretty(self)?)?;
        Ok(())
    }

    fn adicionar(&mut self, titulo: &str, prioridade: Prioridade) -> u32 {
        let id = self.tarefas.iter().map(|t| t.id).max().unwrap_or(0) + 1;
        self.tarefas.push(Tarefa { id, titulo: titulo.into(), prioridade, feita: false });
        id
    }

    fn concluir(&mut self, id: u32) -> Result<()> {
        let tarefa = self
            .tarefas
            .iter_mut()
            .find(|t| t.id == id)
            .with_context(|| format!("tarefa {id} não existe"))?;
        tarefa.feita = true;
        Ok(())
    }

    fn pendentes(&self) -> impl Iterator<Item = &Tarefa> {
        self.tarefas.iter().filter(|t| !t.feita)
    }
}

enum Comando {
    Add { titulo: String, prioridade: Prioridade },
    Feito(u32),
    Listar,
}

fn interpretar(args: &[String]) -> Result<Comando> {
    match args {
        [cmd, titulo, prio] if cmd == "add" => Ok(Comando::Add {
            titulo: titulo.clone(),
            prioridade: prio.parse()?,
        }),
        [cmd, titulo] if cmd == "add" => Ok(Comando::Add {
            titulo: titulo.clone(),
            prioridade: Prioridade::Media,
        }),
        [cmd, id] if cmd == "feito" => Ok(Comando::Feito(id.parse().context("id deve ser número")?)),
        [cmd] if cmd == "listar" => Ok(Comando::Listar),
        _ => bail!("uso: tarefas add <titulo> [baixa|media|alta] | feito <id> | listar"),
    }
}

fn main() -> Result<()> {
    let args: Vec<String> = env::args().skip(1).collect();
    let caminho = Path::new(ARQUIVO);
    let mut lista = Lista::carregar(caminho)?;

    match interpretar(&args)? {
        Comando::Add { titulo, prioridade } => {
            let id = lista.adicionar(&titulo, prioridade);
            println!("Tarefa {id} criada");
        }
        Comando::Feito(id) => {
            lista.concluir(id)?;
            println!("Tarefa {id} concluída");
        }
        Comando::Listar => {
            let mut pendentes: Vec<&Tarefa> = lista.pendentes().collect();
            pendentes.sort_by_key(|t| std::cmp::Reverse(t.prioridade as u8));
            for t in pendentes {
                println!("#{} [{:?}] {}", t.id, t.prioridade, t.titulo);
            }
        }
    }

    lista.salvar(caminho)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn ids_sao_sequenciais() {
        let mut l = Lista::default();
        assert_eq!(l.adicionar("estudar ownership", Prioridade::Alta), 1);
        assert_eq!(l.adicionar("fazer exercícios", Prioridade::Baixa), 2);
    }

    #[test]
    fn concluir_inexistente_falha() {
        let mut l = Lista::default();
        assert!(l.concluir(99).is_err());
    }

    #[test]
    fn prioridade_invalida_falha() {
        let args = vec!["add".to_string(), "x".into(), "urgente".into()];
        assert!(interpretar(&args).is_err());
    }
}
```

```bash
cargo run -- add "Ler o capítulo de traits" alta
cargo run -- add "Revisar lifetimes"
cargo run -- listar
# #1 [Alta] Ler o capítulo de traits
# #2 [Media] Revisar lifetimes
cargo run -- feito 1
cargo test
```

### Desafios para estender

- [ ] Trocar o parse manual de argumentos pelo crate `clap` com `#[derive(Parser)]`.
- [ ] Adicionar data de vencimento com o crate `chrono` e listar as atrasadas primeiro.
- [ ] Trocar o JSON por SQLite com `rusqlite`, mantendo a mesma API de `Lista` (dica: extraia uma trait `Armazenamento`).
- [ ] Expor as tarefas como API REST com Axum (parte 19), reaproveitando `Lista` como biblioteca.
- [ ] Configurar CI no GitHub Actions rodando `cargo fmt --check`, `cargo clippy -- -D warnings` e `cargo test`.

### Roteiro de estudo sugerido (8 semanas)

1. **Semana 1:** partes 1 a 5. Faça os exercícios iniciais do Rustlings.
2. **Semana 2:** partes 6 a 8 (ownership, borrowing, lifetimes). É a semana mais difícil; releia os erros do compilador com calma.
3. **Semana 3:** partes 9 a 11. Reescreva em Rust um script pequeno que você já tem em Python.
4. **Semana 4:** partes 12 a 15. Resolva 10 exercícios da trilha de Rust do Exercism.
5. **Semana 5:** partes 16 e 17. Monte o projeto final desta parte.
6. **Semana 6:** parte 18. Paralelize o processamento de um CSV grande com `rayon`.
7. **Semana 7:** parte 19. Siga o tutorial oficial do Tokio e suba uma API com Axum.
8. **Semana 8:** partes 20 a 22. Escreva um módulo Python em Rust com PyO3 e compare o tempo com a versão em Python puro.

### Recursos recomendados

- [The Rust Programming Language ("o livro")](https://doc.rust-lang.org/book/) — referência oficial, base deste guia.
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/) — conceitos explicados por exemplos executáveis.
- [Rustlings](https://github.com/rust-lang/rustlings) — exercícios curtos para corrigir no terminal.
- [Comprehensive Rust](https://google.github.io/comprehensive-rust/) — curso do Google usado no treinamento interno do Android.
- [Exercism — trilha Rust](https://exercism.org/tracks/rust) — exercícios com mentoria gratuita.
- [Tutorial do Tokio](https://tokio.rs/tokio/tutorial) — async na prática.
- [Guia do PyO3](https://pyo3.rs) — integração Rust e Python.
