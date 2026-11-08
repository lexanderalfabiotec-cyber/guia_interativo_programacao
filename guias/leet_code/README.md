# Laboratório LeetCode: Guia Interativo

Este diretório contém o [laboratorio_leet_code.html](laboratorio_leet_code.html), um caderno de algoritmos e preparação para entrevistas com soluções comparadas em **Python, Rust e Go**.

O conteúdo inclui **18 capítulos, 33 problemas, 99 exemplos de código, 54 perguntas de quiz, 49 termos de glossário e seis simuladores**. Os enunciados são resumos didáticos; cada problema tem um link para a página original no LeetCode. O laboratório não envia soluções ao juiz e não executa Python, Rust ou Go dentro do navegador.

## Como visualizar e interagir

### Servidor local recomendado

Na raiz do repositório:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Acesse [http://localhost:8000/guias/leet_code/laboratorio_leet_code.html](http://localhost:8000/guias/leet_code/laboratorio_leet_code.html).

Se iniciar o servidor dentro de `guias/leet_code`, acesse [http://localhost:8000/laboratorio_leet_code.html](http://localhost:8000/laboratorio_leet_code.html).

### Abertura direta

Abra o HTML pelo gerenciador de arquivos ou, no Linux, execute na raiz do repositório:

```bash
xdg-open guias/leet_code/laboratorio_leet_code.html
```

Não é necessário instalar as três linguagens para ler o caderno. Use um navegador atual com JavaScript para quizzes, glossário, simuladores e persistência. Sem JavaScript, os textos, diagramas e as três versões dos códigos continuam visíveis.

Google Fonts e Prism são carregados pela internet. Sem conexão, o laboratório usa fontes locais e mostra o código sem realce; as interações continuam funcionando.

## Estrutura dos capítulos

| Parte | Capítulo | Conteúdo e problemas |
| :--- | :--- | :--- |
| A · Fundamentos | 01 · Método e Big-O | Restrições, custo e Contains Duplicate (#217) |
| A · Fundamentos | 02 · O kit das três linguagens | Coleções, assinaturas e Valid Anagram (#242) |
| B · Arrays e strings | 03 · Arrays e hashing | Two Sum (#1), Group Anagrams (#49) |
| B · Arrays e strings | 04 · Dois ponteiros | Container With Most Water (#11), 3Sum (#15) |
| B · Arrays e strings | 05 · Janela deslizante | Best Time to Buy and Sell Stock (#121), Longest Substring Without Repeating Characters (#3) |
| B · Arrays e strings | 06 · Soma de prefixos | Product of Array Except Self (#238), Subarray Sum Equals K (#560) |
| B · Arrays e strings | 07 · Pilha e pilha monotônica | Valid Parentheses (#20), Daily Temperatures (#739) |
| B · Arrays e strings | 08 · Busca binária | Binary Search (#704), Koko Eating Bananas (#875) |
| C · Estruturas ligadas | 09 · Listas ligadas | Reverse Linked List (#206), Merge Two Sorted Lists (#21) |
| C · Estruturas ligadas | 10 · Árvores: DFS e BFS | Maximum Depth of Binary Tree (#104), Binary Tree Level Order Traversal (#102) |
| C · Estruturas ligadas | 11 · Heap e top K | Kth Largest Element (#215), Top K Frequent Elements (#347) |
| D · Grafos e recursão | 12 · Grafos: ilhas e dependências | Number of Islands (#200), Course Schedule (#207) |
| D · Grafos e recursão | 13 · Backtracking | Subsets (#78), Combination Sum (#39) |
| E · Programação dinâmica | 14 · DP 1D | Climbing Stairs (#70), Coin Change (#322) |
| E · Programação dinâmica | 15 · DP 2D | Unique Paths (#62), Longest Common Subsequence (#1143) |
| F · Truques e entrevista | 16 · Guloso e intervalos | Jump Game (#55), Merge Intervals (#56) |
| F · Truques e entrevista | 17 · Manipulação de bits | Single Number (#136), Counting Bits (#338) |
| F · Truques e entrevista | 18 · Simulado de entrevista | Longest Consecutive Sequence (#128), roteiro de 45 minutos e plano de oito semanas |

O fechamento reúne treino final com perguntas sorteadas, fontes e glossário. Essas três seções não entram na contagem de capítulos concluídos.

## Recursos interativos

- **Código em abas:** escolha Python, Rust ou Go; a preferência se aplica aos 33 problemas. As setas esquerda/direita e Home/End navegam entre as abas. Use **Copiar** para copiar o código; se a área de transferência estiver bloqueada, o texto fica selecionado para cópia manual.
- **Índice:** filtre capítulos sem diferenciar acentos e maiúsculas. No celular, use o botão **Capítulos**; `Esc` fecha a gaveta.
- **Quizzes:** três perguntas por capítulo, com explicação imediata. **Refazer o quiz** limpa as respostas daquele capítulo.
- **Treino final:** selecione os capítulos, responda às questões sorteadas e acompanhe o placar. Pelo menos um capítulo permanece selecionado; **Zerar placar** reinicia a contagem.
- **Glossário:** pesquise termos ou filtre por categoria; os termos nos capítulos exibem uma definição ao receber foco ou passar o mouse. Clicar no termo abre sua entrada e limpa filtros que poderiam ocultá-la.
- **Tema e leitura:** tema claro/escuro, linguagem, respostas, ponto de leitura e conclusão são salvos no navegador. Ao reabrir sem âncora, a leitura é retomada; um endereço com `#capitulo` tem prioridade.
- **Conclusão automática:** ao rolar para baixo até o final de um capítulo entrar na tela, ele é marcado como estudado. Saltar pelo índice não conclui os capítulos anteriores. A caixa de conclusão também permite marcar ou desmarcar manualmente.
- **Impressão/PDF:** use a impressão do navegador; as três linguagens aparecem na saída. Não há botão de exportação Markdown ou notebook neste laboratório.

### Os seis simuladores

| Simulador | Como usar |
| :--- | :--- |
| Limite de tempo e Big-O | Altere `n` e a linguagem para comparar oito complexidades. As velocidades são estimativas didáticas, sem executar um benchmark. |
| Janela deslizante | Digite um texto e acompanhe os ponteiros passo a passo ou com **Rodar tudo**. São processados até 24 pontos de código Unicode. |
| Busca binária | Escolha o alvo e avance as comparações; gere outro array para comparar buscas com sucesso e sem resultado. |
| Ilhas por BFS | Edite a grade de 8 × 10, gere uma grade aleatória ou limpe tudo; conte as componentes conectadas por quatro direções. |
| Tabela da LCS | Digite duas strings de até dez pontos de código Unicode e preencha uma célula ou a tabela inteira; a trilha mostra uma subsequência possível. |
| Seletor de padrões | Escolha uma das 20 pistas de enunciado para consultar o padrão, seu custo e o capítulo correspondente. |

Os simuladores de texto contam pontos de código; uma combinação de vários pontos, como alguns emojis compostos, pode ocupar mais de uma posição.

## Onde o progresso fica salvo

Os dados ficam no `localStorage` do perfil do navegador, com prefixo `lablc:`:

| Chave | Conteúdo |
| :--- | :--- |
| `lablc:done` | Capítulos marcados como estudados |
| `lablc:reading` | Seção, fração percorrida e posição da rolagem |
| `lablc:quiz:<id>` | Respostas do quiz de cada capítulo |
| `lablc:treino:placar` | Acertos e total de respostas do treino |
| `lablc:treino:ativos` | Capítulos selecionados para o treino |
| `lablc:tema`, `lablc:lang` | Tema e linguagem preferidos |

Nenhum arquivo de progresso é criado no repositório e o laboratório não envia essas respostas a um servidor. Use o mesmo navegador, perfil e endereço para recuperar os dados. Com armazenamento bloqueado ou cheio, um aviso aparece no índice; o progresso funciona durante a sessão. Veja [as orientações de persistência](../../README.md#onde-o-progresso-fica-salvo).

## Como executar os exemplos

As soluções seguem as assinaturas do LeetCode. Os simuladores são JavaScript independente e não executam esses exemplos. Para testar todos os códigos com as adaptações necessárias, use o script de validação abaixo.

### Python

As classes `Solution` usam tipos como `List` e `Optional`, além de `ListNode` e `TreeNode` nos capítulos de listas e árvores. O juiz fornece esse ambiente. Ao copiar para um arquivo local, acrescente os imports e as definições usados pelo problema e chame o método explicitamente:

```python
from typing import List

# Cole aqui a classe Solution do problema #217.
print(Solution().containsDuplicate([1, 2, 3, 1]))  # True
```

Salve como `solucao.py` e execute `python3 solucao.py`. Os exemplos não exigem bibliotecas externas.

### Rust

Em um projeto separado, criado com `cargo new teste_leetcode`, acrescente `struct Solution;`, copie os imports e o `impl Solution` do problema e faça a chamada em `main`:

```rust
struct Solution;
// Cole aqui os imports e o impl Solution do problema #217.
fn main() {
    println!("{}", Solution::contains_duplicate(vec![1, 2, 3, 1])); // true
}
```

Execute `cargo run`. Os exemplos usam a biblioteca padrão; a validação compila com edição 2021. Para listas e árvores, acrescente também as estruturas que o LeetCode fornece; o script de validação contém essas definições.

### Go

Use Go 1.21+ para `min`, `max` e `slices`. Acrescente `package main`, os imports indicados na legenda e uma função `main` que chame a solução. Exemplo para #217:

```go
package main
import "fmt"

// Cole aqui a função containsDuplicate do problema #217.
func main() {
    fmt.Println(containsDuplicate([]int{1, 2, 3, 1})) // true
}
```

Execute `go run solucao.go`. Os problemas de listas e árvores também precisam de `ListNode` ou `TreeNode`. As soluções seguem as restrições de cada enunciado; versões que indexam bytes ou usam contagens de 26 letras não são implementações genéricas para texto Unicode.

## Validação e manutenção

O [relatório de validação](VALIDACAO.md) registra as correções, os testes executados e os limites da revisão. O script [validar_laboratorio.py](validar_laboratorio.py) extrai os exemplos diretamente do HTML, verifica estrutura e duplicações, executa as 99 soluções com resultados esperados e testa as interações no Chrome.

Com Python 3.12+, `uv`, Rust, Go e Chrome/Chromium instalados, execute na raiz:

```bash
# Validação completa, com fontes e scripts externos bloqueados no navegador.
uv run guias/leet_code/validar_laboratorio.py

# Inclui o carregamento e o realce do Prism pela internet.
uv run guias/leet_code/validar_laboratorio.py --online

# Apenas HTML e soluções nas três linguagens.
uv run guias/leet_code/validar_laboratorio.py --skip-browser

# Apenas estrutura e interações no navegador.
uv run guias/leet_code/validar_laboratorio.py --browser-only
```

O `uv` instala Beautiful Soup e Playwright em seu ambiente. Se necessário, indique o navegador com `BROWSER_EXECUTABLE`. Compilações, executáveis e cache Go são criados em um diretório temporário, removido ao final. Sem Rust ou Go, a validação completa falha explicitamente.

Ao incluir problemas, mantenha IDs e exemplos únicos, três linguagens por problema, uma resposta correta por pergunta e links internos válidos. Atualize as contagens, os casos de teste e este README.

Voltar à [documentação principal](../../README.md).
