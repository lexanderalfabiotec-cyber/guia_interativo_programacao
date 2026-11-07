# Laboratório Rust: Guia Interativo

Este diretório contém o **Laboratório Rust** (`laboratorio-rust.html`), um caderno interativo completo projetado para desenvolvedores (especialmente com vivência em Python) aprenderem Rust de forma prática.

---

## ⚡ Como Visualizar e Interagir

### 1. Servidor Local (Método Recomendado)

Na raiz do repositório ou neste diretório, execute:

```bash
# Na pasta guias/rust:
python3 -m http.server 8000
```
Em seguida, acesse no navegador:
**[http://localhost:8000/laboratorio-rust.html](http://localhost:8000/laboratorio-rust.html)**

### 2. Abertura Direta

```bash
xdg-open laboratorio-rust.html
```
*(ou abra o arquivo diretamente no navegador de sua preferência)*

---

## 🧭 Estrutura dos Módulos do Guia

O guia está dividido em 6 partes e 20 capítulos práticos:

- **Parte A · Base da linguagem**
  - `01` Comece por aqui & mapa de referências
  - `02` Toolchain, Cargo e maturin
  - `03` Tipos, variáveis e funções
  - `04` Ownership e borrowing
  - `05` Structs, enums e pattern matching
  - `06` Tratamento de erros: Result, Option e exceções
  - `07` Generics, traits e polimorfismo
  - `08` Coleções, iteradores e closures

- **Parte B · Integração com Python**
  - `09` A fronteira: PyO3 e a C-API
  - `10` Passando dados sem cópia (NumPy / ndarray / PyBuffer)
  - `11` O GIL: soltando para paralelismo real
  - `12` Empacotando e distribuindo wheels com maturin
  - `13` Arquitetura mista: Python na orquestração, Rust no motor

- **Parte C · Trading de Cripto**
  - `14` Estratégia e backtest (com simulador interativo)
  - `15` Tempo real: WebSockets, threads e canais sem locks

- **Parte D · IoT e Embarcados**
  - `16` Microcontroladores: do hardware ao firmware (`no_std`)
  - `17` Do sensor ao dashboard: MQTT e processamento de sinal

- **Parte E · Avançado**
  - `18` Unsafe e o Rustonomicon
  - `19` Compilador Nightly e o Unstable Book
  - `20` Testes, benchmarks (Criterion) e CI para projetos mistos

- **Parte F · Fechamento**
  - Treino final interativo com questões sorteadas
  - Glossário e fontes de consulta

---

## 💻 Como Reproduzir os Códigos Localmente

### Testando Snippets Rust

Para testar códigos apresentados no laboratório:

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

