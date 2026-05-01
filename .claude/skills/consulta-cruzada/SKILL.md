---
name: consulta-cruzada
description: Protocolo para consultas que cruzam múltiplas fichas do corpus —
  comparações entre autores, mapeamento de divergências, identificação de
  cadeias de citação, rastreamento de conceitos através de papers. Use quando
  a resposta exige integração analítica de informações dispersas no índice.
---

# Protocolo de consulta cruzada

Esta skill governa consultas que exigem integração de informações de
múltiplas fichas. Diferentemente de uma busca direta (resolvida por
grep + leitura de uma ficha), a consulta cruzada **produz análise**
sobre o corpus.

## Princípio fundamental

Análise cruzada não é compilação. **Não** liste "Autor A diz X. Autor B
diz Y." Identifique a estrutura analítica subjacente: pressupostos
compartilhados, divergências reais, deslocamentos conceituais.

## Tipos de consulta cruzada

### Tipo 1 — Comparação entre autores sobre uma questão

**Forma da pergunta**: "Como [autor A] e [autor B] divergem sobre [questão]?"

Protocolo:

1. **Especificação da questão**
   Formular a questão em UMA frase. Se não couber em uma frase, a questão
   está mal posta — refinar antes de prosseguir.

2. **Posição de cada autor**
   Para cada autor envolvido:
   - Tese sobre a questão (1 parágrafo, paráfrase rigorosa)
   - Citação textual de suporte (com página) — máximo 1 por autor, sob
     30 palavras
   - Argumento que sustenta a tese (paráfrase, com referência à seção)
   - Pressupostos não enunciados (o que precisa ser verdade para a tese
     funcionar?)
   - Localização canônica: ficha consultada (link wiki)

3. **Tipologia da divergência**
   Classificar OBRIGATORIAMENTE:
   - **Empírica**: discordam sobre fatos do mundo
   - **Conceitual**: usam conceitos diferentes para o mesmo fenômeno, ou
     o mesmo conceito com sentidos distintos
   - **Normativa**: concordam nos fatos, divergem na avaliação
   - **De ênfase**: mesma posição substantiva, prioridades diferentes
   - **Aparente**: divergência some quando precisamos os termos

   A categoria orienta a estratégia argumentativa subsequente.

4. **Mapa do campo (se 3+ autores envolvidos)**
   Posicionar cada autor em duas dimensões relevantes para a controvérsia.
   Identificar quadrantes vazios — eles indicam posições logicamente
   possíveis mas não ocupadas (potencial contribuição original).

5. **Avaliação**
   Qual posição tem maior suporte no corpus? NUNCA tomar partido sem
   evidência textual. Quando o corpus não permite decisão, declarar
   empate e indicar que tipo de evidência adicional resolveria.

6. **Implicação para o projeto**
   Como essa divergência afeta o argumento da pesquisa? Se não afeta,
   ela não deveria estar no texto.

7. **Salvamento (se solicitado)**
   `corpus/indice/mapas/[questao].md` ou `analise/comparacoes/[questao].md`
   conforme estrutura do projeto.

### Tipo 2 — Genealogia de citação

**Forma da pergunta**: "Como [conceito] foi sendo deslocado/desenvolvido
ao longo do corpus?", "Qual a cadeia de citação de [autor X]?"

Protocolo:

1. Identificar o "ponto de origem" no corpus (paper mais antigo ou
   formulação mais influente)
2. Listar papers subsequentes que citam o origem (grep no frontmatter
   `autores_referenciados`)
3. Para cada paper subsequente:
   - Função da citação (base teórica? contraste? operacional?)
   - Deslocamento semântico (o uso é fiel? ampliado? distorcido?
     reapropriado?)
   - Página exata
4. Construir narrativa da genealogia: como o conceito se moveu pelo
   corpus, quem o reformulou, em que direção
5. Identificar pontos de divergência: onde a "linha principal" se
   bifurcou em interpretações distintas

### Tipo 3 — Mapeamento de pressupostos compartilhados

**Forma da pergunta**: "Que autores compartilham pressuposto P, mesmo
sem dialogar diretamente?"

Esta é a consulta mais analítica e a mais valiosa para descobrir
estrutura latente do campo.

Protocolo:

1. Formular o pressuposto em proposição simples
2. Para cada ficha do corpus, examinar:
   - Se o autor enuncia o pressuposto explicitamente
   - Se o autor o usa sem enunciar (operacional implícito)
   - Se o autor o rejeita explicitamente
   - Se o autor não toma posição (silêncio)
3. Construir mapa: aceitam / rejeitam / silenciam
4. Identificar agrupamentos não-óbvios: autores de tradições teóricas
   diferentes que compartilham pressuposto, ou autores da mesma tradição
   que divergem nele

### Tipo 4 — Detecção de uso divergente do mesmo conceito

**Forma da pergunta**: "O conceito X é usado com o mesmo sentido em todo
o corpus?" ou variações.

Protocolo:

1. Listar todos os papers que mobilizam o conceito (grep frontmatter
   `conceitos_mobilizados` e `conceitos_introduzidos`)
2. Para cada um, extrair da ficha:
   - Definição usada (citação literal se a ficha registra)
   - Função no argumento
   - Distinções estabelecidas em relação a outros usos
3. Comparar definições. Identificar:
   - Usos equivalentes (sinônimos legítimos)
   - Usos deslocados (mesmo termo, sentido modificado por boa razão)
   - Usos divergentes (mesmo termo, sentidos incompatíveis — fonte de
     confusão argumentativa)
4. Sinalizar para o pesquisador as divergências problemáticas: usar o
   termo sem qualificação será impreciso.

## Output padrão

Toda consulta cruzada produz output com:

1. **Cabeçalho**: tipo de consulta + questão formulada com precisão
2. **Mapa**: tabela ou estrutura comparativa
3. **Análise**: prosa analítica que não é mero comentário sobre a tabela
4. **Avaliação**: peso relativo das posições, com base no corpus
5. **Lacunas**: o que o corpus NÃO permite decidir
6. **Implicação**: para a pesquisa em curso

## Anti-padrões

NÃO faça:
- Sintetizar uma "posição média" entre divergentes (descaracteriza a
  divergência)
- Listar posições sem classificar tipo de divergência (parecerismo
  disfarçado)
- Tomar partido sem evidência textual
- Apresentar como divergência o que é apenas diferença de ênfase, ou
  o inverso
- Inventar conexões entre autores que não estão nas fichas
  (sugerir como hipótese é OK; afirmar como fato, não)
