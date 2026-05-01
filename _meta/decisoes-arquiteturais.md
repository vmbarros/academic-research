---
arquivo: decisoes-arquiteturais
descricao: Registro das decisões de design que estruturam o sistema. Diferente do changelog (que registra mudanças), este arquivo registra escolhas em pontos onde havia alternativas plausíveis, com a razão da escolha. Útil para revisões futuras e para distinguir decisões consolidadas de questões em aberto.
---

# Decisões arquiteturais

> Formato ADR-leve. Cada decisão é uma seção com: **Contexto**,
> **Alternativas consideradas**, **Decisão**, **Consequências**. Decisões
> revisadas no futuro são marcadas como *superseded by [link]*, mas o
> registro original permanece.

---

## ADR-01 — Ficha como substituto operacional do PDF

**Status**: aceita (2026-05-01)

**Contexto**: Pesquisa acadêmica em direito mobiliza papers, livros e teses
densos. A pergunta arquitetural inicial: o LLM consulta o PDF original a
cada pergunta (RAG convencional) ou opera sobre representação intermediária?

**Alternativas consideradas**:

1. **RAG sobre PDFs**: vetorização de PDFs, recuperação semântica em cada
   consulta, leitura de chunks no contexto do LLM
2. **Ficha como substituto operacional**: leitura profunda na indexação
   (uma vez), produção de ficha estruturada, todas as consultas posteriores
   sobre a ficha
3. **Híbrido**: ficha + RAG sobre PDF para fallback

**Decisão**: Ficha como substituto operacional. PDFs lidos uma vez no
fichamento; nunca reabertos para consulta (exceto via `/aprofundar` em
modo de re-leitura dirigida, ver ADR-04).

**Consequências**:
- (+) Custo de contexto previsível e baixo nas consultas (ficha de 5000
  palavras é constante; chunks de RAG variam e somam quando consulta toca
  múltiplas seções)
- (+) Trabalho intelectual de leitura é frontloaded — o pesquisador faria
  isso de qualquer modo
- (+) Ficha estruturada é queriável (frontmatter YAML), RAG não é
- (−) Erro de indexação se propaga sem detecção até refichagem (mitigado
  por ADR-05: feedback loop entre agentes)
- (−) Custo alto na indexação (compensado pela amortização)

---

## ADR-02 — Frontmatter YAML como contrato, não decoração

**Status**: aceita (2026-05-01)

**Contexto**: O frontmatter pode ser tratado como (a) metadado bibliográfico
opcional (Zotero-style), (b) lista flexível conforme conveniência da
indexação, ou (c) **interface estruturada** queriável por humanos
(Dataview), por LLM (grep) e por scripts.

**Alternativas consideradas**:

1. Frontmatter mínimo (autor, ano, título)
2. Frontmatter rico mas livre (cada ficha define seus campos)
3. Frontmatter como contrato rígido com schema versionado

**Decisão**: Contrato. Listas YAML reais (não strings com `;`), schema
explícito em `indexacao-paper/SKILL.md`, campos obrigatórios marcados.
Drift entre fichas é falha bloqueante.

**Consequências**:
- (+) Consultas estruturais resolvíveis sem ler prosa (filtrar, contar,
  mapear relações)
- (+) Dataview funciona; jq/yq funcionam; grep funciona
- (−) Refatoração do schema exige migração (nenhuma migração necessária
  ainda — vault sem fichas reais)
- (−) Indexador deve aplicar disciplina; risco de campos vazios
  (`null`/`[]`)

---

## ADR-03 — Slugs ASCII paralelos a campos com acentuação

**Status**: aceita (2026-05-01)

**Contexto**: Autores e conceitos jurídicos têm acentos, tremas, caracteres
não-ASCII. Buscar `"Habermas"` deve casar `"Habermas, Jürgen"`; buscar
`"folgenberucksichtigung"` (sem trema) deve casar `"Folgenberücksichtigung"`.
Grep tolerante a variação não é robusto sem normalização.

**Alternativas consideradas**:

1. Não normalizar; aceitar falhas silenciosas em grep
2. Normalizar tudo para ASCII e perder display correto
3. Manter campo display + campo slug ASCII paralelo

**Decisão**: Campo slug ASCII paralelo (`autores_slugs`,
`conceitos_*_slugs`), gerado por regra canônica (NFD + remoção de
combining marks + lowercase + kebab-case). Display preservado em campo
original.

**Consequências**:
- (+) Grep tem recall 100% sobre slug
- (+) Display permanece correto (Müller, regulação, esfera-pública)
- (+) Path de arquivos passa a usar slug (estável, sem caracteres que
  problemas em filesystem)
- (−) Duplicação de informação no YAML (mitigado por geração automática
  pelo `@leitor-profundo`; risco de drift baixo)

---

## ADR-04 — Comando /aprofundar para refinamento dirigido

**Status**: aceita (2026-05-01)

**Contexto**: Quando o `@consultor-corpus` responde "informação não está
na ficha", as opções eram: (a) aceitar a lacuna, (b) abrir o PDF
manualmente, (c) refichar inteiro. Faltava caminho intermediário
controlado por evidência de gap real.

**Alternativas consideradas**:

1. Aceitar refichagem integral como única forma de refinar (caro)
2. Permitir consulta direta ao PDF como fallback do consultor (quebra
   ADR-01)
3. Re-leitura dirigida: lê só seções/capítulos do tópico, agrega ao índice
   sem reescrever

**Decisão**: Re-leitura dirigida via `/aprofundar`. Modo APROFUNDAMENTO do
`@leitor-profundo` lê só passagens relevantes ao tópico, ADICIONA conteúdo
(subficha tópica nova, ampliação de subficha, anexo, ou atualização da
mestra), marca tudo com `> [!aprofundamento]` (audit trail), atualiza
`cobertura_incompleta`.

**Consequências**:
- (+) Refinamento incremental dirigido por gap real (não preventivo)
- (+) Audit trail explícito (callout) — usuário e LLM distinguem conteúdo
  de fichamento integral vs aprofundamento
- (+) Cumulativo: cada `/aprofundar` enriquece o índice; perda de
  informação só por reescrita explícita
- (−) Reabre PDF (custo de contexto), mas dirigido
- (−) Risco de fragmentação: muitos `/aprofundar` em mesmo paper podem
  produzir índice mosaico — mitigado por convenção de quando criar subficha
  nova vs ampliar existente

---

## ADR-05 — Feedback loop entre @leitor-profundo e @consultor-corpus

**Status**: aceita (2026-05-01)

**Contexto**: Sistema unidirecional (indexador → consultor) significa que
erros de indexação se propagam sem detecção. O consultor tem visão
integrada; o indexador tem visão profunda do paper individual. Faltava
canal entre os dois.

**Alternativas consideradas**:

1. Manter unidirecional; revisão manual periódica do usuário
2. Revisão cruzada automática (consultor revalida fichas a cada consulta)
   — caro
3. Logging passivo: consultor sinaliza inconsistências em log; usuário ou
   `/aprofundar` resolvem quando convém

**Decisão**: Logging passivo via `_log-auditorias.md`. Critério para
registrar: ≥2 fichas envolvidas, divergência substantiva, divergência
não é objeto explícito do paper. Sinalização também replicada em
`flags_para_auditoria` do frontmatter da ficha envolvida.

**Consequências**:
- (+) Sistema ganha memória dos próprios erros prováveis
- (+) Auditoria não bloqueia consulta (resposta vai imediatamente; log é
  efeito colateral)
- (+) Usuário pode disparar `/aprofundar` direcionado por log
- (−) Risco de log inflacionar se o consultor for sensível demais
  (mitigado pelos critérios)
- (−) Resolução exige intervenção humana ou refichagem (sem auto-cura)

---

## ADR-06 — Arquitetura em 5 camadas para acesso progressivo

**Status**: aceita (2026-05-01)

**Contexto**: Compreensão "rápida" exige material leve; compreensão
"completa" exige material denso. Representação única tem que escolher.
Para obras grandes (livros 300+pp), ficha única achata diferenças entre
capítulos e força carregamento de 8000 palavras para extrair 600.

**Alternativas consideradas**:

1. Ficha única para qualquer obra, com densidade variável
2. Múltiplas fichas curtas (uma por seção) — perde visão integral
3. Camadas explícitas com transição governada por protocolo

**Decisão**: 5 camadas. C0 (`_index.md` denso), C1 (frontmatter), C2
(corpo da ficha), C3 (subfichas por capítulo, obrigatórias para 300+pp),
C4 (anexo `citacoes.md`). Protocolo de transição (regra dos 5+2 passos)
no `@consultor-corpus`: nunca pula camadas, sobe só conforme profundidade
exige.

**Consequências**:
- (+) Compreensão rápida: C0 dá entrada barata (1 KB/30 obras)
- (+) Compreensão completa: C2-C4 disponíveis quando exigidas
- (+) Custo de contexto proporcional à pergunta, não ao corpus
- (−) Indexação de livro grande mais cara (subfichas obrigatórias)
- (−) Risco de drift entre `_index.md` e fichas (mitigado por checksum
  sha256[:4])

---

## ADR-07 — Bidirecionalidade de `relacoes.e_citado_por`

**Status**: aceita (2026-05-01)

**Contexto**: Relação "X cita Y" é declarada por X. A relação inversa
"Y é citado por X" é fato derivável. Pode ser computada por (a) busca
exaustiva a cada consulta, (b) campo derivado mantido bidirecionalmente.

**Alternativas consideradas**:

1. Busca exaustiva a cada consulta (lazy, mas O(n) por query)
2. Campo derivado computado por script periódico
3. Bidirecionalidade automática pelo `@leitor-profundo` na indexação

**Decisão**: Bidirecionalidade pelo `@leitor-profundo`. Ao indexar X que
cita [Y, Z], abre fichas de Y e Z e adiciona X em
`relacoes.e_citado_por`. Para citações órfãs (Y ainda não indexado),
registra em `cobertura_incompleta: [referencia-orfa: Y]`; quando Y é
finalmente indexado, varredura reversa fecha pendências.

**Consequências**:
- (+) Consultas "quem cita Y?" são O(1) (lê `Y.relacoes.e_citado_por`)
- (+) Grafo de citações disponível como dado, não como prosa
- (−) Indexação modifica fichas pré-existentes (risco de drift se
  bidirecionalidade falhar parcialmente — mitigado por checksum)
- (−) `desenvolve`, `critica`, `reapropria_de_modo_divergente` ficam
  unidirecionais (declarações da fonte sobre o alvo, não fatos sobre o
  alvo) — assimetria intencional

---

## ADR-08 — Sem RAG semântico até o corpus pedir

**Status**: aceita (2026-05-01)

**Contexto**: Para corpus pequeno (10-50 obras), grep + LLM resolve a
maioria das consultas. Para corpus grande (200+ obras), busca semântica
torna-se necessária. Decisão sobre quando investir em embeddings.

**Alternativas consideradas**:

1. Implementar embeddings agora preventivamente
2. Manter grep apenas, indefinidamente
3. Manter grep até o corpus pedir (sinal: queries semânticas falhando
   por vocabulário)

**Decisão**: Diferir. Manter grep + LLM enquanto corpus < ~100 obras.
Reavaliar quando o pesquisador relatar consultas que falham por
vocabulário (ex.: "tratamento de legitimidade democrática" não casa
papers que tratam do tema sem usar o termo literal).

**Consequências**:
- (+) Sem dependência adicional (Python + PyYAML bastam)
- (+) Sem custo de manutenção de índice vetorial
- (−) Recall semântico limitado ao que o frontmatter captura
- (−) Eventual migração exige indexar corpus inteiro

---

## Questões em aberto

Decisões plausíveis ainda não tomadas, listadas para revisão futura:

- **Corpus compartilhado entre projetos**: hoje cada projeto tem corpus
  isolado. Para pesquisador maduro, a mesma ficha de Habermas serve à
  tese e a um parecer. Opções: (a) `_meta/biblioteca-comum/` com fichas
  compartilhadas, (b) `_meta/projects-registry.md` mantém mapa
  paper×projetos. Adiada até surgir caso real.
- **Direito positivo (leis, decretos, jurisprudência) como tipo
  distinto**: hoje todo o template assume corpus secundário (papers/
  livros/teses). Lei tem estrutura própria (artigos/incisos), vinculação
  diferente, citação específica. Adiar até primeiro projeto que precise.
- **Protocolo de escrita**: a árvore prevê `escrita/rascunhos/` e
  `escrita/final/`, mas nenhum comando opera ali. O mapa termina em
  "Implicações para a pesquisa" — perfeito para preparar escrita —, mas
  a passagem ao parágrafo ainda recai em decisão manual. Comando
  `/rascunho [secao]` que produza texto com atribuições integradas é
  candidato natural.
- **Citação canônica formatada (ABNT/Chicago/Bluebook)**: o frontmatter
  coleta dados suficientes para gerar citação, mas não há comando para
  produzir bibliografia formatada. Adiar até primeira escrita real.
