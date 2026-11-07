# Flutter com IA no bolso: Guia Interativo

Este diretório contém o [guia-flutter-ia.html](guia-flutter-ia.html), um caderno interativo de Flutter, Dart e inteligência artificial para quem já programa em Python, Go ou C#.

**Estado atual:** o módulo 0 está disponível, com ambiente, referências e uma seção prática de softmax e temperatura em Dart, Python e Go. Os módulos 1 a 13 contêm roteiros de conteúdo planejado. Ainda não há um aplicativo Flutter completo ou um backend de IA implementado neste diretório.

## Como visualizar e interagir

### Servidor local recomendado

Na raiz do repositório:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Acesse [http://localhost:8000/guias/flutter/guia-flutter-ia.html](http://localhost:8000/guias/flutter/guia-flutter-ia.html).

Se iniciar o servidor dentro de `guias/flutter`, o endereço será [http://localhost:8000/guia-flutter-ia.html](http://localhost:8000/guia-flutter-ia.html).

### Abertura direta

Também é possível abrir o HTML pelo gerenciador de arquivos ou, no Linux, executar na raiz do repositório:

```bash
xdg-open guias/flutter/guia-flutter-ia.html
```

Não é necessário instalar Flutter, Dart, Go ou dependências Python para ler o guia. Use um navegador atual, com suporte a CSS `light-dark()`; o JavaScript habilita as ferramentas interativas. O conteúdo e todos os exemplos permanecem visíveis quando o JavaScript está desabilitado.

## Estrutura dos módulos

| Módulo | Conteúdo | Situação |
| :--- | :--- | :--- |
| 0 · Comece aqui | Ambiente, versões, referências, softmax, temperatura, entropia e amostragem | Disponível |
| 1 · Dart | Tipos, null safety, classes, Futures, Streams e Isolates aplicados à IA | Planejado |
| 2 · Flutter | Widgets, layout, navegação, estado, HTTP, formulários e persistência | Planejado |
| 3 · IA via API | Proxy em Go, streaming, tool calling, retries e custo por token | Planejado |
| 4 · Decisão no aparelho | Árvores, treino em Python, métricas e exportação de modelos | Planejado |
| 5 · Visão e áudio | OCR, extração de dados, detecção de objetos e transcrição | Planejado |
| 6 · SLM no celular | Quantização, GGUF, llama.cpp, LiteRT-LM e IA do sistema | Planejado |
| 7 · Arquitetura híbrida | Roteamento entre aparelho e nuvem, fallback e offline-first | Planejado |
| 8 · RAG no celular | Embeddings, similaridade, busca vetorial e recuperação local | Planejado |
| 9 · Agentes no app | Loop do agente, ferramentas, permissões, estado e testes | Planejado |
| 10 · MLOps e AIOps mobile | Artefatos, OTA de modelos, teste A/B, telemetria, drift e rollback | Planejado |
| 11 · Desempenho | Memória, bateria, tamanho do app, GPU/NPU e profiling | Planejado |
| 12 · Segurança | Privacidade, prompt injection, armazenamento e proteção de modelos | Planejado |
| 13 · Projeto final | Assistente financeiro offline-first integrando os módulos anteriores | Planejado |

O guia também inclui mapa de dependências, glossário com analogias e fontes por módulo. A ordem de entrega está descrita no próprio HTML.

## Recursos interativos

- **Demonstração da thread principal e Worker:** compare uma tarefa que bloqueia a interface com outra executada em uma linha de execução separada. É uma simulação em JavaScript do conceito de Isolate, sem carregar um modelo de IA real.
- **Código em abas:** alterne entre Dart, Python e Go usando clique ou as setas do teclado. A preferência se aplica às caixas que oferecem a mesma linguagem.
- **Cópia e notebook:** copie os exemplos ou baixe o exemplo Python como `.ipynb`, com células Markdown e células executáveis.
- **Quizzes:** três perguntas na seção 0.4, com resposta e explicação imediatas.
- **Busca:** pesquisa sem diferenciar maiúsculas ou acentos; `/` abre a busca, as setas percorrem os resultados e `Esc` fecha os controles abertos.
- **Glossário:** definições por mouse, teclado ou toque, com navegação de volta ao trecho de leitura.
- **Tema e progresso:** preferências, módulos marcados, ponto da leitura e respostas dos quizzes ficam nas chaves `fia:theme`, `fia:lang`, `fia:done`, `fia:reading` e `fia:quiz:<id>` do `localStorage`, mesmo após reiniciar o navegador. Sem acesso ao armazenamento, o progresso funciona durante a sessão e um aviso aparece no sumário. A contagem inclui os 14 módulos do roteiro, inclusive os planejados. Veja [onde os dados ficam salvos e como consultá-los](../../README.md#onde-o-progresso-fica-salvo).
- **Conclusão automática:** ao rolar para baixo até o final de um módulo aparecer na tela, ele é marcado como concluído e salvo. A caixa "Terminei este módulo" e as caixas do sumário permitem marcar ou desmarcar manualmente.
- **Exportação:** baixe o conteúdo em Markdown, com as três linguagens, tabelas, fontes Mermaid e âncoras dos links internos. Para PDF, use a impressão do navegador; todas as abas e explicações dos quizzes entram na impressão.

Fontes do Google, highlight.js, Mermaid e Turndown são carregados pela internet. Sem esses recursos, o guia usa fontes locais, mostra os códigos sem realce, preserva os diagramas como texto Mermaid e exporta Markdown com o conversor interno.

## Como executar os exemplos

A referência do guia, conferida em **7 de novembro de 2026**, é Flutter 3.47.x com Dart 3.13.x, Python 3.12+ e Go 1.24+. Os exemplos atuais de Dart usam apenas a biblioteca padrão. Consulte as [notas do Flutter](https://docs.flutter.dev/release/release-notes) e as [novidades do Dart](https://dart.dev/resources/whats-new) ao atualizar o ambiente.

### Dart

Copie o exemplo Dart da seção 0.4 para `softmax.dart` e execute:

```bash
dart --version
dart run softmax.dart
```

Também é possível colar esse exemplo no [DartPad](https://dartpad.dev). Não são necessários plugins Flutter.

### Python e notebook

Copie o exemplo Python para `softmax.py`. Em um diretório de trabalho separado:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install numpy
python softmax.py
```

No Windows, ative o ambiente com `.venv\Scripts\Activate.ps1`. Para usar o notebook, clique em **Baixar .ipynb** na aba Python e abra o arquivo no Jupyter ou na extensão Jupyter do VS Code; selecione o ambiente com NumPy instalado.

### Go

Copie o exemplo Go para `softmax.go` e execute:

```bash
go version
go run softmax.go
```

Esse exemplo demonstra o cálculo e uma política de temperatura. O servidor proxy HTTP faz parte do planejamento do módulo 3.

## Validação e manutenção

O [relatório de validação](VALIDACAO.md) descreve os problemas corrigidos, os testes e suas limitações. O script [validar_guia.py](validar_guia.py) valida o HTML, as referências internas, duplicações de exemplos, saídas Python/Go e as interações no navegador.

Com [uv](https://docs.astral.sh/uv/), Python 3.12+ e Chrome ou Chromium instalado, execute na raiz:

```bash
# Testa as funcionalidades essenciais com os recursos externos bloqueados.
uv run guias/flutter/validar_guia.py

# Inclui os diagramas renderizados, realce de código e exportação por Turndown.
uv run guias/flutter/validar_guia.py --online
```

O `uv` instala as dependências do teste em seu próprio ambiente. Se o navegador estiver em outro caminho, defina `BROWSER_EXECUTABLE`. Go é usado quando disponível; a ausência é informada no resultado. A compilação do exemplo Dart deve ser feita separadamente com o SDK.

Ao editar o HTML, mantenha IDs únicos, links de glossário válidos, uma alternativa correta por quiz e saídas de exemplo coerentes com o código. Atualize a situação dos módulos nesta documentação quando novos capítulos forem implementados.

Voltar à [documentação principal](../../README.md).
