# Guia Interativo de Programação

Repositório de cadernos e laboratórios interativos focados no aprendizado prático de programação, arquitetura de sistemas e integração entre linguagens.

---

## 📚 Guias Disponíveis

| Guia | Arquivo Principal | Descrição | Documentação |
| :--- | :--- | :--- | :--- |
| **Laboratório Rust** | [`guias/rust/laboratorio-rust.html`](guias/rust/laboratorio-rust.html) | Caderno interativo com 35 capítulos: os 23 temas do Guia Completo de Rust na Parte A, integração com Python (PyO3/maturin), trading algorítmico, sistemas IoT embarcados e simuladores em tempo real. | [Guia de uso](guias/rust/README.md) |
| **Flutter com IA no bolso** | [`guias/flutter/guia-flutter-ia.html`](guias/flutter/guia-flutter-ia.html) | Guia de Flutter, Dart e IA: módulo 0 disponível com softmax em Dart/Python/Go, demo de concorrência, quizzes e exportações; módulos 1 a 13 planejados. | [Guia de uso](guias/flutter/README.md) · [Validação](guias/flutter/VALIDACAO.md) |

---

## 🚀 Como Visualizar os Guias

Cada guia é uma aplicação web interativa independente, contida em um arquivo HTML. Não é necessário instalar Rust ou Flutter para visualizar o conteúdo.

### Opção 1: Servidor Local HTTP (Recomendado)

Iniciar um servidor HTTP local garante que fontes, scripts e recursos externos (CDN) funcionem sem restrições de segurança do protocolo `file://`.

#### Com Python 3
No terminal, execute na raiz do projeto:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Abra o navegador no endereço:

- **Rust:** [http://localhost:8000/guias/rust/laboratorio-rust.html](http://localhost:8000/guias/rust/laboratorio-rust.html)
- **Flutter:** [http://localhost:8000/guias/flutter/guia-flutter-ia.html](http://localhost:8000/guias/flutter/guia-flutter-ia.html)

#### Com Node.js / npx
```bash
npx serve .
```

#### Com Live Server (VS Code / Antigravity IDE)
1. Clique com o botão direito no arquivo HTML do guia desejado.
2. Selecione **Open with Live Server**.

---

### Opção 2: Abrir Diretamente no Navegador

Você pode abrir o arquivo diretamente com o navegador padrão do seu sistema:

#### No Linux
```bash
xdg-open guias/rust/laboratorio-rust.html
```

#### No macOS
```bash
open guias/rust/laboratorio-rust.html
```

#### No Windows (PowerShell)
```powershell
Start-Process "guias\rust\laboratorio-rust.html"
```

Ou simplesmente dê duplo-clique no arquivo através do gerenciador de arquivos.

Os comandos acima abrem o laboratório Rust. Para Flutter, use o caminho `guias/flutter/guia-flutter-ia.html`; consulte também as [instruções específicas do guia](guias/flutter/README.md).

---

## 🛠️ Executando os Exemplos Práticos de Rust

Para compilar e testar os exemplos de código contidos no guia no seu próprio ambiente:

### 1. Pré-requisitos
- **Rust e Cargo** (versão 1.85+ para os exemplos com edição 2024):
  ```bash
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  source $HOME/.cargo/env
  ```
- **Python 3.9+** (para os módulos com PyO3):
  ```bash
  python3 --version
  ```

### 2. Comandos Essenciais do Cargo
```bash
# Criar novo projeto binário
cargo new meu_projeto
cd meu_projeto

# Validar compilação rapidamente sem gerar executável
cargo check

# Executar em modo desenvolvimento
cargo run

# Executar em modo otimizado (essencial para medições de desempenho/benchmarks)
cargo run --release

# Executar testes unitários
cargo test
```

### 3. Projetos Mistos Python + Rust (PyO3 & maturin)
Conforme abordado nos capítulos de integração do guia:

```bash
# Instalar o empacotador maturin no seu ambiente Python
pip install maturin

# Dentro do diretório do projeto misto, compilar e disponibilizar a biblioteca para o Python
maturin develop --release
```

---

## 📱 Exemplos e Validação do Guia de Flutter

O módulo 0 contém exemplos executáveis de softmax e temperatura em Dart puro, Python com NumPy e Go. O HTML também oferece comparação entre thread principal e Web Worker, busca, glossário, tema, progresso, três quizzes, download de notebook e exportação Markdown/PDF.

Veja os pré-requisitos e os comandos na [documentação de Flutter](guias/flutter/README.md). O [relatório de validação](guias/flutter/VALIDACAO.md) registra as correções e limitações. Para repetir os testes com `uv` e Chrome/Chromium instalado:

```bash
uv run guias/flutter/validar_guia.py
```

Os módulos 1 a 13 ainda são planejamentos, incluindo a criação do aplicativo Flutter e do proxy de IA em Go.

---

## 🎯 Recursos do Laboratório Rust

- **Abas Comparativas**: Alterne entre implementações em **Python** e **Rust** com um clique.
- **Simuladores Embutidos**:
  - Simulador de decisão de integração (FFI vs PyO3 vs IPC).
  - Simulador de backtest quantitativo (estratégia RSI + Williams %R).
  - Simulador de pipeline IoT com protocolo MQTT e filtro EMA.
- **Quizzes Interativos**: Questões com diagnóstico instantâneo e explicação de erros de compilação.
- **Progresso e Tema**: Armazenamento automático do seu progresso de estudo e preferência de tema (claro/escuro) no navegador (`localStorage`).
