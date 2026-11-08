# Laboratório Kotlin & Android · IA no bolso

Abra [laboratorio_kotlin_android.html](laboratorio_kotlin_android.html). O caderno segue o formato do laboratório Rust: resumo, conceito, aplicação, termos explicados, exemplos, diagrama, pegadinha, desafio e quiz, com humor durante a leitura.

São **16 práticas, 32 novos programas Kotlin/JVM, 16 diagramas SVG e 19 perguntas**. A seção 1.6 mantém o exemplo completo de feature engineering em Kotlin/Python e as comparações em C#/Go. Há também fragmentos de Compose, ViewModel, Room, coroutines e Flow.

As listas identificadas como **roteiro de aprofundamento** descrevem integrações a desenvolver. Este diretório não contém um aplicativo Android, projeto Gradle ou backend completo.

## Como abrir

Na raiz do repositório:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Acesse [o laboratório local](http://localhost:8000/guias/android_kotlin/laboratorio_kotlin_android.html). Também é possível abrir o HTML diretamente; prefira HTTP para ter um endereço estável para o progresso.

## Trilhas

| Práticas | Conteúdo |
| --- | --- |
| 1–4 | Tipos, null safety, funções, estado, Compose, arquitetura e dados |
| 5–9 | Contratos de API, orçamento de contexto, regressão, métricas, árvores, OCR e memória de modelos |
| 10–16 | Roteamento híbrido, RAG, agentes, avaliação, desempenho, segurança e projeto de domínio |

Cada prática tem dois programas independentes. Copie **um por vez**, preveja a saída e compare com o resultado publicado. Para execução local, com Kotlin/JVM e um JDK compatível instalados:

```bash
kotlinc Main.kt -include-runtime -d pratica.jar
java -jar pratica.jar
```

O botão **Abrir no Playground** abre o Playground oficial e tenta copiar o código para colar. Os fragmentos de integração explicam o ambiente e as dependências necessárias; os trechos Android precisam de um projeto próprio. Os exemplos de coroutines precisam de `kotlinx-coroutines-core`.

## Interações

- **Quatro demonstrações em JavaScript:** thread principal versus Worker, null safety, z-score e escolha entre execução local/nuvem. Elas não executam Kotlin nem modelos de IA no navegador.
- **Quiz com explicação:** 16 perguntas das práticas e três em coleções. Respostas são salvas ao clicar e restauradas ao reabrir.
- **Busca e navegação:** pesquisa ignora acentos; `/` abre a busca e `Esc` fecha os controles. O sumário oferece acesso direto às práticas e aos roteiros.
- **Tema e progresso:** preferências e conclusões manuais persistem no navegador. A leitura é retomada ao abrir sem uma âncora; um link para uma seção tem prioridade.
- **Código e exportação:** copiar, abas comparativas, notebook Python, Markdown e impressão/PDF. Todas as abas e as fontes Mermaid entram no Markdown.

Os 16 diagramas das práticas usam SVG nativo e funcionam offline. Os dois diagramas anteriores em Mermaid dependem da CDN para renderização; sem internet, a fonte textual permanece disponível. Sem conexão, o realce de sintaxe pode faltar e a exportação usa o conversor interno. Fontes do sistema dispensam downloads.

## Onde os dados ficam salvos

| Chave no localStorage | Conteúdo |
| --- | --- |
| `kia:done` | IDs dos 16 capítulos marcados manualmente |
| `kia:reading` | Capítulo e posição relativa da leitura |
| `kia:quiz:<id>` | Alternativa escolhida em cada pergunta |
| `kia:theme`, `kia:lang` | Tema e linguagem preferida |

Use o mesmo navegador, perfil e endereço. Limpar os dados do site remove o progresso; aba anônima descarta os dados ao encerrar. Quando o armazenamento está bloqueado, o caderno continua em memória e exibe aviso.

## Validação

O script [validar_laboratorio.py](validar_laboratorio.py) verifica estrutura, âncoras, contagens, saídas dos 32 programas Kotlin e interações no Chrome. Se o compilador fornecer o JAR de coroutines, também compila e executa os dois programas correspondentes.

Requer Python 3, `beautifulsoup4`, Kotlin/JVM e Java; para navegador, `playwright` e Chrome/Chromium instalados. Defina `KOTLINC` e `BROWSER_EXECUTABLE` se necessário.

```bash
python3 guias/android_kotlin/validar_laboratorio.py
python3 guias/android_kotlin/validar_laboratorio.py --skip-browser
python3 guias/android_kotlin/validar_laboratorio.py --skip-kotlin
```

O teste do navegador bloqueia recursos externos para conferir o funcionamento offline, exercita as oito combinações do roteador, persistência, busca, exportação, retomada e layout móvel. Compilações e downloads ficam em diretório temporário. Esses testes não substituem um build Android ou medições em aparelho.

[Voltar ao repositório](../../README.md).
