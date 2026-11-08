# Laboratório Rust: Guia Interativo

Este diretório contém o **Laboratório Rust** (`laboratorio_rust.html`), um caderno interativo completo projetado para desenvolvedores (especialmente com vivência em Python) aprenderem Rust de forma prática.

---

## ⚡ Como Visualizar e Interagir

### 1. Servidor Local (Método Recomendado)

Dentro de `guias/rust`, execute:

```bash
# Na pasta guias/rust:
python3 -m http.server 8000
```
Em seguida, acesse no navegador:
**[http://localhost:8000/laboratorio_rust.html](http://localhost:8000/laboratorio_rust.html)**

### 2. Abertura Direta

```bash
xdg-open laboratorio_rust.html
```
*(ou abra o arquivo diretamente no navegador de sua preferência)*

---

## 🧭 Estrutura dos Módulos do Guia

Capítulos concluídos, respostas dos quizzes, placar do treino, tema e linguagem ficam salvos automaticamente no `localStorage` do navegador, com prefixo `labrs:`. O ponto da leitura fica em `labrs:reading` e é restaurado ao reabrir sem uma âncora no endereço. Os dados permanecem após reiniciar o navegador; um aviso aparece no índice se o armazenamento estiver indisponível. Consulte [onde os dados ficam salvos e como consultá-los](../../README.md#onde-o-progresso-fica-salvo).

Ao rolar para baixo até o final de um capítulo aparecer na tela, ele é marcado como concluído automaticamente. Você também pode marcar ou desmarcar a caixa "Já estudei este capítulo" manualmente.

O guia está dividido em 6 partes e 35 capítulos práticos. A Parte A cobre todos os 23 temas do Guia Completo de Rust, com explicações descontraídas, exemplos, comparações com Python e quizzes. Os exemplos comparativos que já existiam continuam disponíveis em caixas expansíveis nos respectivos capítulos.

- **Parte A · Base da linguagem**
  - `01` Introdução: o que é Rust e como usar o guia
  - `02` Cargo e estrutura de projeto
  - `03` Variáveis, mutabilidade, constantes e shadowing
  - `04` Tipos de dados
  - `05` Funções, expressões e controle de fluxo
  - `06` Ownership e memória stack/heap
  - `07` Borrowing, referências e slices
  - `08` Lifetimes
  - `09` Structs e métodos
  - `10` Enums e pattern matching
  - `11` Tratamento de erros: Option, Result e `?`
  - `12` Coleções: Vec, String, HashMap e companhia
  - `13` Traits
  - `14` Generics
  - `15` Closures e iteradores
  - `16` Smart pointers: Box, Rc, RefCell e Arc
  - `17` Módulos, crates e visibilidade
  - `18` Concorrência: threads, channels e Mutex
  - `19` Async/await com Tokio
  - `20` Macros
  - `21` Testes unitários, mocks, integração e doc tests
  - `22` Unsafe e FFI com C e Python
  - `23` Projeto final: CLI de tarefas com JSON e roteiro de oito semanas

- **Parte B · Integração com Python**
  - `24` PyO3 e maturin na prática, incluindo NumPy sem cópia
  - `25` Outras pontes: ctypes, subprocess e Rust chamando Python
  - `26` Concorrência e async sem GIL
  - `27` CLIs em Rust

- **Parte C · Trading de Cripto**
  - `28` Dados de mercado e indicadores
  - `29` Estratégia e backtest (com simulador interativo)
  - `30` Tempo real: WebSocket, ordens e risco

- **Parte D · IoT e Embarcados**
  - `31` Microcontroladores: do hardware ao firmware (`no_std`)
  - `32` Do sensor ao dashboard: MQTT e processamento de sinal

- **Parte E · Avançado**
  - `33` Unsafe e o Rustonomicon
  - `34` Compilador Nightly e o Unstable Book
  - `35` Testes, benchmarks (Criterion) e CI para projetos mistos

- **Parte F · Fechamento**
  - Treino final interativo com questões sorteadas
  - Glossário e fontes de consulta

---

## 💻 Como Reproduzir os Códigos Localmente

### Testando Snippets Rust

Para testar códigos apresentados no laboratório, use Rust 1.85 ou posterior para os exemplos com edição 2024. Alguns exemplos requerem as dependências indicadas ao lado do código:

```bash
# Crie uma sandbox temporária
cargo new /tmp/teste_rust
cd /tmp/teste_rust

# Edite src/main.rs com o exemplo desejado e rode:
cargo run --release
```

### Testando Extensões PyO3 (Python + Rust)

Para os capítulos da Parte B:

1. Tenha Python 3.9+ e `maturin` instalados:
   ```bash
   pip install maturin
   ```
2. Crie um projeto com `maturin`:
   ```bash
   maturin new -b pyo3 meu_modulo
   cd meu_modulo
   ```
3. Compile no ambiente virtual:
   ```bash
   maturin develop --release
   ```
4. Teste a importação no Python:
   ```bash
   python3 -c "import meu_modulo; print(dir(meu_modulo))"
   ```

