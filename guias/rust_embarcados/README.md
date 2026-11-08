# Rust Embarcado: Guia Interativo

Este diretório contém o [rust-embarcado.html](rust-embarcado.html), um caderno de eletrônica e Rust para o ESP32-C3. São **23 capítulos**, **134 blocos de código e configuração**, **15 calculadoras e simuladores**, **70 perguntas** e **81 termos de glossário**.

O material apresenta sete projetos, do pisca-LED a um painel web em Rust. Os simuladores desta página executam JavaScript; para executar os firmwares Rust, crie os projetos indicados no caderno e use o Wokwi ou uma placa real. Os blocos incluem programas completos, trechos, configurações e exemplos que mostram erros de compilação de propósito.

## Como visualizar e interagir

Na raiz do repositório:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Acesse [http://localhost:8000/guias/rust_embarcados/rust-embarcado.html](http://localhost:8000/guias/rust_embarcados/rust-embarcado.html).

Se iniciar o servidor dentro de `guias/rust_embarcados`, acesse [http://localhost:8000/rust-embarcado.html](http://localhost:8000/rust-embarcado.html). Também é possível abrir o arquivo pelo gerenciador de arquivos ou, no Linux, executar na raiz:

```bash
xdg-open guias/rust_embarcados/rust-embarcado.html
```

Não é necessário instalar Rust nem ter uma placa para ler o guia. Use um navegador atual com JavaScript para os simuladores, quizzes e buscas. Sem JavaScript, o conteúdo estático e todas as abas de código permanecem visíveis. Fontes Google e Prism são recursos externos; sem conexão, há fontes locais e os exemplos continuam disponíveis sem realce de sintaxe.

## Estrutura dos módulos

| Parte | Capítulos | Conteúdo |
| :--- | :--- | :--- |
| A · Comece aqui | 01–02 | Organização do estudo, ferramentas, Rust, esp-generate, espflash e Wokwi |
| B · Eletrônica sem medo | 03–09 | Lei de Ohm, resistores, divisores, LEDs, botões, capacitores, transistores, ADC, PWM e bateria |
| C · Projetos na placa | 10–14 | Pisca-LED, contador com debounce, ohmímetro, dimmer e termômetro com alarme |
| D · Web com Rust | 15–16 | LED pelo celular com esp-idf-svc e painel de temperaturas com Axum |
| E · Do protótipo ao produto | 17–21 | no_std, memória, testes, CI, PCB, produção, OTA, identificação e venda no Brasil |
| F · GitHub | 22–23 | Workspace, versionamento, releases, biblioteca calc e licenças |
| G · Fechamento | Sem numeração | Treino final, fontes e glossário |

Cada projeto apresenta três trilhas: demonstração nesta página, execução no Wokwi e montagem em placa real. Os capítulos combinam resumo, conceitos, exemplos, diagramas e quizzes.

## Recursos interativos

- **Calculadoras e simuladores:** Lei de Ohm, código de cores, resistor de LED, divisor, carga RC, PWM, bateria, pisca-LED, debounce, ohmímetro, dimmer, NTC com histerese, requisições HTTP, preço e escolha de licença.
- **Código em abas:** seleção por clique ou pelas teclas ←, →, Home e End; botão para copiar o código. Se a área de transferência estiver indisponível, o texto é selecionado para cópia manual.
- **Busca:** filtro de títulos no índice e pesquisa de termos no glossário, sem diferenciar acentos ou maiúsculas. Termos no texto também têm dicas ao passar o mouse ou receber foco.
- **Quizzes:** resposta imediata, explicação e opção de refazer. O treino final permite escolher capítulos, sortear perguntas e zerar o placar.
- **Tema e progresso:** tema, trilha preferida, conclusões manuais, respostas e placar ficam no navegador. Este guia não restaura o ponto de rolagem nem conclui capítulos automaticamente.

As chaves do `localStorage` usam o prefixo `rsemb:`: `tema`, `trilha`, `done`, `quiz:<id>`, `treino:ativos` e `treino:placar`. Quando o acesso ao armazenamento falha, aparece um aviso no índice; as interações continuam durante a sessão. Use o mesmo navegador, perfil e endereço para reencontrar os dados. Veja [onde o progresso fica salvo](../../README.md#onde-o-progresso-fica-salvo).

## Como executar os exemplos

### Cálculos no PC

O capítulo 22 contém `calc/Cargo.toml` e o arquivo completo `calc/src/lib.rs`, sem dependências externas. Ele reúne as funções apresentadas gradualmente nos capítulos anteriores; esses trechos são referências didáticas e não devem ser concatenados ao arquivo completo.

Para experimentar a biblioteca isoladamente, em um diretório de trabalho separado:

```bash
cargo new --lib calc
cd calc
# Substitua src/lib.rs pelo arquivo completo do capítulo 22.
cargo test
cargo build --lib
```

Ao usar a biblioteca isolada, mantenha o `Cargo.toml` criado por `cargo new`. O manifesto mostrado no capítulo herda edição e licença do workspace também mostrado ali. Para reproduzir esse workspace, crie antes as pastas `calc` e `painel` com seus manifestos. A edição 2024 requer Rust 1.85 ou posterior; dependências e ferramentas podem exigir uma versão mais nova.

Os exemplos Python usam apenas a biblioteca padrão. Copie um exemplo para um arquivo `.py` e execute com `python3 arquivo.py`. Os programas Rust de orçamento, BOM e serial podem ser copiados para `src/main.rs` de projetos criados com `cargo new` e executados com `cargo run`.

### ESP32-C3 com esp-hal

Siga o capítulo 02 para instalar Rust e as ferramentas. No diretório em que pretende criar o firmware:

```bash
rustup target add riscv32imc-unknown-none-elf
cargo install esp-generate --locked
cargo install espflash --locked
esp-generate --chip esp32c3 pisca-led
cd pisca-led
```

No gerador, escolha as opções indicadas no guia, incluindo Wokwi e suporte às APIs instáveis para ADC/PWM quando necessário. Respeite a versão de Rust exigida pelo projeto gerado e preserve seus arquivos de link e dependências compatíveis. Copie o firmware do projeto desejado para o caminho do binário configurado, acrescente a dependência local `calc` e ajuste os caminhos relativos.

```bash
cargo build --release
# Com a placa conectada, compila, grava e abre o monitor:
cargo run --release
```

Para simular, abra a pasta do firmware no VS Code, configure `wokwi.toml` e `diagram.json` e execute **Wokwi: Start Simulator**. O Wokwi executa o ELF já compilado. Alguns diagramas do guia mostram somente os objetos a acrescentar às listas `parts` e `connections`, em vez de um JSON completo.

Os exemplos adotam a API do esp-hal 1.0; confira a documentação correspondente à versão fixada pelo gerador, especialmente para ADC e LEDC. Consulte a [API do ADC](https://docs.espressif.com/projects/rust/esp-hal/1.0.0/esp32c3/esp_hal/analog/adc/struct.AdcConfig.html) e a [configuração do Wokwi](https://docs.wokwi.com/vscode/project-config).

### Servidor web e painel

O projeto 6 usa **esp-idf-svc com std**, seguindo o gerador `esp-idf-template` indicado no capítulo 15; ele tem ferramentas e alvo diferentes dos projetos esp-hal sem sistema operacional. O HTML da página fica em `src/pagina.html`, incluído no firmware com `include_str!`. SSID e senha são definidos na compilação, conforme o exemplo.

O projeto 7 roda no PC: crie o projeto `painel`, adicione Axum, Tokio e Serde conforme o bloco do capítulo 16 e copie `main.rs` e `painel.html` para `src`. Rode `cargo run` e abra `http://localhost:3000`. O programa `dispositivo-simulado` envia leituras usando Reqwest e serde_json. Esses exemplos são demonstrações locais; não implementam autenticação nem armazenamento durável.

O trecho de `build.rs` do capítulo 20 deve ser acrescentado ao arquivo gerado, preservando a configuração de link do ESP-IDF. As integrações de OTA, CI e release são roteiros a adaptar ao projeto criado; não há firmware, backend ou pipeline implantado neste diretório.

## Validação e manutenção

A revisão está registrada em [REVISAO.md](REVISAO.md). Para repetir os testes, tenha Python 3.11+, Rust, Beautiful Soup, Playwright e Chrome/Chromium instalados. As dependências Python podem ser instaladas em um ambiente virtual:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install beautifulsoup4 playwright
python guias/rust_embarcados/validar_laboratorio.py
```

O comando acima parte da raiz do repositório. Para usar outro Chrome/Chromium, passe `--chrome /caminho/do/navegador`. Para executar somente estrutura e exemplos locais:

```bash
python3 guias/rust_embarcados/validar_laboratorio.py --sem-navegador
```

A validação cria arquivos temporários, bloqueia CDNs no navegador e usa um servidor HTTP ligado a `127.0.0.1` em porta temporária. Verifica estrutura, âncoras, dados dos quizzes, duplicações integrais, exemplos locais, calculadoras e interações. Não grava hardware, não executa o Wokwi e não compila os projetos que dependem do ESP-IDF ou de crates externos.

Ao editar, mantenha os IDs únicos e as implementações dos trechos didáticos coerentes com a biblioteca completa. Atualize as contagens e o relatório quando alterar o conteúdo. A aprovação dos testes cobre os cenários descritos no relatório, sem garantir ausência absoluta de bugs.

Voltar à [documentação principal](../../README.md).
