# Guia Interativo de Programação

Repositório de cadernos e laboratórios interativos focados no aprendizado prático de programação, arquitetura de sistemas e integração entre linguagens.

---

## 📚 Guias Disponíveis

| Guia | Arquivo Principal | Descrição | Documentação |
| :--- | :--- | :--- | :--- |
| **Laboratório Rust** | [`guias/rust/laboratorio_rust.html`](guias/rust/laboratorio_rust.html) | Caderno interativo com 35 capítulos: os 23 temas do Guia Completo de Rust na Parte A, integração com Python (PyO3/maturin), trading algorítmico, sistemas IoT embarcados e simuladores em tempo real. | [Guia de uso](guias/rust/README.md) |
| **Rust Embarcado** | [`guias/rust_embarcados/rust-embarcado.html`](guias/rust_embarcados/rust-embarcado.html) | 23 capítulos de eletrônica e ESP32-C3, sete projetos, 15 calculadoras e simuladores, 70 perguntas e práticas de produção e GitHub. | [Guia de uso](guias/rust_embarcados/README.md) |
| **Flutter com IA no bolso** | [`guias/flutter/laboratorio_flutter.html`](guias/flutter/laboratorio_flutter.html) | Guia de Flutter, Dart e IA: módulo 0 disponível com softmax em Dart/Python/Go, demo de concorrência, quizzes e exportações; módulos 1 a 13 planejados. | [Guia de uso](guias/flutter/README.md) |
| **Laboratório LeetCode** | [`guias/leet_code/laboratorio_leet_code.html`](guias/leet_code/laboratorio_leet_code.html) | 18 capítulos de algoritmos com 33 problemas, 99 soluções em Python/Rust/Go, seis simuladores, quizzes e treino de entrevista. | [Guia de uso](guias/leet_code/README.md) |
| **Laboratório Kotlin & Android** | [`guias/android_kotlin/laboratorio_kotlin_android.html`](guias/android_kotlin/laboratorio_kotlin_android.html) | 16 práticas com 32 novos programas Kotlin, diagramas, quizzes, simuladores e fragmentos de integração Android. | [Guia de uso](guias/android_kotlin/README.md) |

---

## 🚀 Como Visualizar os Guias

Cada guia é uma aplicação web interativa independente, contida em um arquivo HTML. Não é necessário instalar Rust, Flutter ou as linguagens dos exemplos para visualizar o conteúdo.

### Opção 1: Servidor Local HTTP (Recomendado)

Iniciar um servidor HTTP local garante que fontes, scripts e recursos externos (CDN) funcionem sem restrições de segurança do protocolo `file://`.

#### Com Python 3
No terminal, execute na raiz do projeto:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Abra o navegador no endereço:

- **Rust:** [http://localhost:8000/guias/rust/laboratorio_rust.html](http://localhost:8000/guias/rust/laboratorio_rust.html)
- **Rust Embarcado:** [http://localhost:8000/guias/rust_embarcados/rust-embarcado.html](http://localhost:8000/guias/rust_embarcados/rust-embarcado.html)
- **Flutter:** [http://localhost:8000/guias/flutter/laboratorio_flutter.html](http://localhost:8000/guias/flutter/laboratorio_flutter.html)
- **LeetCode:** [http://localhost:8000/guias/leet_code/laboratorio_leet_code.html](http://localhost:8000/guias/leet_code/laboratorio_leet_code.html)
- **Kotlin/Android:** [http://localhost:8000/guias/android_kotlin/laboratorio_kotlin_android.html](http://localhost:8000/guias/android_kotlin/laboratorio_kotlin_android.html)

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
xdg-open guias/rust/laboratorio_rust.html
```

#### No macOS
```bash
open guias/rust/laboratorio_rust.html
```

#### No Windows (PowerShell)
```powershell
Start-Process "guias\rust\laboratorio_rust.html"
```

Ou simplesmente dê duplo-clique no arquivo através do gerenciador de arquivos.

Os comandos acima abrem o laboratório Rust. Para Flutter, use `guias/flutter/laboratorio_flutter.html`; para LeetCode, use `guias/leet_code/laboratorio_leet_code.html`. Para Kotlin/Android, use `guias/android_kotlin/laboratorio_kotlin_android.html`. Consulte as instruções específicas de [Flutter](guias/flutter/README.md), [LeetCode](guias/leet_code/README.md) e [Kotlin/Android](guias/android_kotlin/README.md).

Para Rust Embarcado, abra `guias/rust_embarcados/rust-embarcado.html` e consulte seu [guia de uso](guias/rust_embarcados/README.md).

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

## 🔌 Eletrônica e Rust Embarcado

O caderno de Rust Embarcado reúne 23 capítulos sobre eletrônica, ESP32-C3, firmware, web e produção. São sete projetos, 15 calculadoras e simuladores em JavaScript, 70 perguntas e uma biblioteca `calc` que pode ser testada no PC e compilada sem `std`. Os exemplos distinguem os projetos com esp-hal dos que usam esp-idf-svc.

Consulte o [guia de uso de Rust Embarcado](guias/rust_embarcados/README.md) e o [relatório de revisão](guias/rust_embarcados/REVISAO.md). Com Python, Beautiful Soup, Playwright, Rust e Chrome/Chromium instalados, execute na raiz:

```bash
python3 guias/rust_embarcados/validar_laboratorio.py
```

O validador verifica estrutura, exemplos locais e interações no navegador. A compilação dos projetos embarcados e os testes em Wokwi/placa precisam ser feitos no ambiente descrito no guia.

## 📱 Exemplos do Guia de Flutter

O módulo 0 contém exemplos executáveis de softmax e temperatura em Dart puro, Python com NumPy e Go. O HTML também oferece comparação entre thread principal e Web Worker, busca, glossário, tema, progresso, três quizzes, download de notebook e exportação Markdown/PDF.

Veja os pré-requisitos e os comandos na [documentação de Flutter](guias/flutter/README.md).

Os módulos 1 a 13 ainda são planejamentos, incluindo a criação do aplicativo Flutter e do proxy de IA em Go.

---

## 🧩 Algoritmos e Entrevistas com LeetCode

O laboratório reúne 18 capítulos e 33 problemas com soluções comparadas em Python, Rust e Go. Inclui seis simuladores (Big-O, janela deslizante, busca binária, ilhas por BFS, LCS e seleção de padrões), 54 perguntas, treino final e glossário com 49 termos. Os simuladores executam JavaScript; os exemplos das três linguagens são executados separadamente.

Consulte a [documentação de LeetCode](guias/leet_code/README.md) para estudar e reproduzir os exemplos. O script e o relatório da revisão anterior não estão presentes nesta cópia do repositório.

## 🤖 Kotlin, Android e IA

O laboratório segue o formato do caderno Rust: resumo, conceitos, termos, exemplos, diagramas, pegadinhas e quizzes, com pausas de humor. São 16 práticas, 32 novos programas Kotlin/JVM, 16 diagramas SVG, 19 perguntas e quatro demonstrações em JavaScript. A seção 1.6 mantém os exemplos completos de feature engineering em Kotlin/Python e as comparações em C#/Go.

Inclui fragmentos de Compose, ViewModel, Room, coroutines e Flow. Os roteiros de integração estão sinalizados; não há aplicativo Android completo. Tema, conclusões manuais, respostas dos quizzes e ponto de leitura são salvos no navegador. Veja a [documentação de Kotlin/Android](guias/android_kotlin/README.md).

Com Python, beautifulsoup4, Playwright, Kotlin/JVM, Java e Chrome/Chromium instalados:

```bash
python3 guias/android_kotlin/validar_laboratorio.py
```

## 🎯 Recursos do Laboratório Rust

- **Abas Comparativas**: Alterne entre implementações em **Python** e **Rust** com um clique.
- **Simuladores Embutidos**:
  - Simulador de decisão de integração (FFI vs PyO3 vs IPC).
  - Simulador de backtest quantitativo (estratégia RSI + Williams %R).
  - Simulador de pipeline IoT com protocolo MQTT e filtro EMA.
- **Quizzes Interativos**: Questões com diagnóstico instantâneo e explicação de erros de compilação.
- **Progresso e Tema**: Armazenamento automático do seu progresso de estudo e preferência de tema (claro/escuro) no navegador (`localStorage`).

## Onde o progresso fica salvo

Os cinco HTMLs salvam automaticamente no **`localStorage` do perfil do navegador**, no próprio computador. Os dados continuam disponíveis após fechar e reiniciar o navegador; nenhum arquivo de progresso é criado no projeto e nenhum dado de progresso é enviado a um servidor.

| Guia | Chaves no `localStorage` |
| :--- | :--- |
| Rust | `labrs:done` (capítulos concluídos), `labrs:reading` (ponto da leitura), `labrs:quiz:<id>` (respostas), `labrs:treino:placar` e `labrs:treino:ativos` (treino), `labrs:tema` e `labrs:lang` (preferências) |
| Rust Embarcado | `rsemb:done` (conclusões manuais), `rsemb:quiz:<id>` (respostas), `rsemb:treino:placar` e `rsemb:treino:ativos` (treino), `rsemb:tema` e `rsemb:trilha` (preferências) |
| Flutter | `fia:done` (módulos concluídos), `fia:reading` (ponto da leitura), `fia:quiz:<id>` (respostas), `fia:theme` e `fia:lang` (preferências) |
| LeetCode | `lablc:done` (capítulos concluídos), `lablc:reading` (ponto da leitura), `lablc:quiz:<id>` (respostas), `lablc:treino:placar` e `lablc:treino:ativos` (treino), `lablc:tema` e `lablc:lang` (preferências) |
| Kotlin/Android | `kia:done` (conclusões manuais), `kia:reading` (leitura), `kia:quiz:<id>` (respostas), `kia:theme` e `kia:lang` (preferências) |

Os guias de Rust, Flutter, LeetCode e Kotlin/Android retomam a leitura ao reabrir sem uma âncora (`#capitulo`) no endereço, usando o ponto salvo. Links para um capítulo específico têm prioridade. As marcações e respostas são salvas imediatamente; a posição é salva durante a rolagem e ao sair da página.

Nos guias de Rust, Flutter e LeetCode, ao rolar para baixo até o final de um capítulo ou módulo entrar na tela, ele é marcado automaticamente como concluído e salvo. As caixas de conclusão continuam disponíveis para marcar ou desmarcar manualmente. Abrir o guia no meio ou saltar para outro capítulo não conclui os anteriores automaticamente.

No laboratório Kotlin/Android, as conclusões continuam manuais; ponto de leitura e respostas dos quizzes também são persistidos.

No Rust Embarcado, as conclusões são manuais e as respostas dos quizzes ficam salvas. Este guia não salva o ponto de rolagem; a preferência de trilha e o placar do treino final também são persistidos.

Para consultar os dados, abra as ferramentas do navegador (**F12 → Application/Aplicativo → Local Storage/Armazenamento local**) e selecione o endereço do guia.

Use o mesmo navegador, perfil e endereço, incluindo protocolo, domínio e porta. `localhost` e `127.0.0.1`, ou portas diferentes, têm armazenamentos separados. Para arquivos abertos diretamente (`file://`), a persistência depende do navegador e do caminho do arquivo; prefira o servidor local com endereço fixo. Limpar os dados do site apaga o progresso, e janelas anônimas descartam os dados ao encerrar. Quando o armazenamento estiver bloqueado ou cheio, o guia exibirá um aviso junto ao progresso.
