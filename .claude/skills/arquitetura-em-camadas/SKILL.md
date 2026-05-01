---
name: arquitetura-em-camadas
description: Define as 5 camadas de representação do conhecimento sobre o corpus
  (_index.md, frontmatter, ficha, subfichas por capítulo, anexo de citações
  expandidas) e o protocolo de transição entre elas. Use ao planejar como
  responder consulta complexa, ao indexar obras grandes, ou ao decidir onde
  registrar uma informação. É o mapa operacional que articula indexacao-paper,
  consulta-cruzada e mapeamento-conceitual.
---

# Arquitetura em camadas

O corpus indexado tem **5 camadas** de representação. Cada uma serve um tipo
distinto de consulta. O sistema é eficiente quando o LLM transita entre
camadas conforme a profundidade real exigida — não quando carrega tudo a cada
pergunta nem quando se contenta com o nível superficial.

## Princípio fundamental

**Compreensão rápida e completa não vivem em uma representação única; vivem
em uma escada que o LLM sobe conforme necessário.** Cada camada superior é
mais barata em contexto e mais informativa por unidade de leitura, ao custo
de menor fidelidade. Cada camada inferior é mais densa e mais cara, ao
benefício de maior fidelidade.

## As cinco camadas

### Camada 0 — `_index.md` (índice denso do projeto)

**Onde**: `corpus/indice/_index.md`

**O que é**: tabela de uma linha por obra indexada, com chave + ano + gênero
+ tese-resumo + tags. Mantida automaticamente pelo `@leitor-profundo` a cada
indexação. Ordenação por chave (alfabética), para que diffs git sejam estáveis.

**Função**: ponto de entrada barato. Permite ao LLM filtrar candidatos
relevantes para uma consulta sem abrir nenhuma ficha integral. Custo: ~1 KB
por 30 obras.

**Formato**:

```markdown
# Índice denso — [Nome do projeto]

> Gerado e mantido pelo `@leitor-profundo`. NÃO editar à mão — alterações são
> sobrescritas na próxima indexação. Para refinar resumo de uma obra, editar
> `tese_central_resumo` no frontmatter da ficha; será propagado.

| chave | ano | gênero | tese (≤80 palavras) | tags | checksum |
|---|---|---|---|---|---|
| ayres-1992-responsive | 1992 | empirico | Regulação eficaz alterna entre persuasão e sanção, escalonando coerção conforme resposta do regulado; punição direta sem persuasão prévia falha. | regulacao-responsiva, enforcement | a3f7 |
| habermas-1992-faktizitat | 1992 | teorico | [obra de tese irredutível — ver corpo e subfichas] | legitimidade, discurso, direito-moral | b8c2 |
| stigler-1971-economic | 1971 | teorico | Regulação é demandada pela própria indústria regulada e desenhada para servi-la, contra interpretação corrente de proteção do interesse público. | captura-regulatoria, escolha-publica | f1d9 |
```

**Coluna `checksum`**: hash curto (4 caracteres hex) do conteúdo da ficha
mestra. Permite detectar drift entre `_index.md` e ficha. Recalculado pelo
agente em cada atualização. Regra: `sha256(conteudo_ficha)[:4]`.

**Quando atualizar**:
- A cada indexação nova → adicionar linha
- A cada refichagem → atualizar linha (e checksum)
- Periodicamente, comando de auditoria pode detectar drift

### Camada 1 — Frontmatter da ficha (YAML estruturado)

**Onde**: bloco `---` no topo de cada ficha (`corpus/indice/papers/[chave].md`
ou `corpus/indice/papers/[chave]/_ficha.md`).

**O que é**: representação queriável do paper. Inclui dados bibliográficos,
classificações temáticas, conceitos com slugs, autores referenciados,
relações com outros papers do corpus, pressupostos identificados, registro
epistêmico, gaps de cobertura.

**Função**: responder consultas estruturais sem ler prosa. "Quais papers
empíricos do corpus citam Habermas como base teórica?" → resolvido lendo
só frontmatter dos candidatos pré-filtrados via Camada 0.

**Definição completa**: ver `.claude/skills/indexacao-paper/SKILL.md`,
seção "Template de frontmatter".

**Quando carregar**: consulta que pede contagem, listagem ou mapeamento de
relações sobre subset filtrado de obras.

### Camada 2 — Corpo da ficha (estrutura analítica em prosa)

**Onde**: corpo do mesmo arquivo da Camada 1.

**O que é**: tese central em parágrafo, estrutura argumentativa por seção
(ou capítulo, para livros), conceitos operativos com definição literal,
método ou arquitetura argumentativa, tabela de referências mobilizadas,
citações textuais selecionadas, pressupostos em prosa, lacunas, crítica
metodológica, registro epistêmico justificado, conexões com o corpus.

**Função**: responder consultas analíticas sobre o paper específico. "Como
Ayres & Braithwaite sustentam a pirâmide de enforcement?" exige leitura da
estrutura argumentativa + conceitos operativos + citações.

**Definição completa**: ver `.claude/skills/indexacao-paper/SKILL.md`,
seção "Estrutura obrigatória do corpo".

**Quando carregar**: consulta que pede análise sobre obra única, ou sobre
até ~5 obras integradas (acima de 5, considere se a Camada 1 não basta).

### Camada 3 — Subfichas por capítulo (apenas para obras grandes)

**Onde**: diretório `corpus/indice/papers/[chave]/`, com:
- `_ficha.md` — ficha mestra (Camadas 1-2 acima)
- `cap-01-titulo.md` — subficha do capítulo 1
- `cap-02-titulo.md` — subficha do capítulo 2
- ... uma subficha por capítulo

**Critério de quando criar subfichas**:

| Gênero | Páginas | Subfichas |
|---|---|---|
| Artigo | até 30 | n/a |
| Artigo extenso | 30–80 | opcional, apenas se densidade conceitual exigir |
| Capítulo de livro | qualquer | n/a (a ficha já é "subficha" do livro pai) |
| Tese / dissertação | 100–300 | opcional, a critério da densidade |
| Livro / tese | 300+ | **obrigatórias** |
| Livro / tese | 100–300 com alta densidade conceitual | **recomendadas** |

**Convenção de nomeação**:
- `cap-XX-slug-titulo.md` onde `XX` é número zero-padded (`01`, `02`, ..., `12`),
  `slug-titulo` é slug ASCII curto do título do capítulo
- Se a obra usa "Parte I, Cap. 1", numerar continuamente: `cap-01`, `cap-02`...
- Nunca usar romanos no nome de arquivo

**Template de subficha**:

```yaml
---
chave_subficha: ayres-1992-responsive/cap-02-piramide
chave_paper_pai: ayres-1992-responsive
projeto: tese-regulacao-data-centers
caminho_canonico: projects/tese-regulacao-data-centers/corpus/indice/papers/ayres-1992-responsive/cap-02-piramide.md

capitulo: 2
titulo_capitulo: "Enforcement Pyramids"
paginas: 19-53

# === Conteúdo específico do capítulo ===
tese_capitulo_resumo: "A pirâmide de enforcement formaliza escalonamento dinâmico: persuasão na base, revogação no topo; credibilidade da ameaça superior potencializa a inferior."

conceitos_introduzidos_no_capitulo:
  - piramide-de-enforcement
conceitos_introduzidos_slugs:
  - piramide-de-enforcement
conceitos_mobilizados_no_capitulo:
  - dilema-do-prisioneiro
conceitos_mobilizados_slugs:
  - dilema-do-prisioneiro

autores_referenciados_no_capitulo:
  - Bardach, Eugene
  - Kagan, Robert A.
autores_referenciados_slugs:
  - bardach-eugene
  - kagan-robert-a

# === Conexões internas (com outros capítulos do mesmo livro) ===
conexoes_internas:
  prepara: [cap-03-aplicacoes, cap-04-tit-for-tat]
  desenvolve_de: [cap-01-impasse]

# === Conexões externas (com outros papers do corpus) ===
conexoes_externas:
  cita:
    - chave: bardach-1982-enforcement
      paginas: [24, 67-72]
      funcao: base_teorica
  e_citado_por: []
---

# Cap. 2 — Enforcement Pyramids (pp. 19-53)

## Tese do capítulo

[parágrafo de prosa rigorosa, similar à seção "Tese central" da ficha mestra,
mas restrito ao argumento do capítulo]

## Estrutura interna do capítulo

- **§ 2.1 (pp. 19-25)** — função: introdução do problema da escolha entre
  punir e persuadir
- **§ 2.2 (pp. 26-35)** — função: formulação da pirâmide
- ...

## Conceitos operativos introduzidos no capítulo

### Pirâmide de enforcement

> "The escalating responses available to regulators form a pyramid: persuasion at the base, license revocation at the apex." (p. 35)

- **Função no argumento do capítulo**: instrumento institucional que opera a tese
- **Conexão com o corpus**: [[conceitos/piramide-de-enforcement]]

## Citações textuais do capítulo

[≤ 5 citações ≤ 30 palavras cada, mesmas regras da ficha mestra]

## Função no argumento total do livro

[1 parágrafo: como esse capítulo se encaixa na arquitetura do livro inteiro.
Esta seção é a *cola* entre subfichas e ficha mestra.]

## Notas para uso futuro

[anotações específicas do capítulo]
```

**Densidade**: 1500–3000 palavras por subficha. Mais que isso, considerar
sub-subdividir em outra rodada de fichamento.

**Manutenção da ficha mestra com subfichas**:
- A ficha mestra (`_ficha.md`) tem `tese_central_resumo` que sintetiza o livro
  inteiro
- Sua "Estrutura argumentativa" lista capítulos com link wiki para subfichas:
  ```markdown
  - **Cap. 1** [[ayres-1992-responsive/cap-01-impasse]] — Função: ...
  - **Cap. 2** [[ayres-1992-responsive/cap-02-piramide]] — Função: ...
  ```
- Os campos `relacoes`, `pressupostos`, `conceitos_*` no frontmatter da ficha
  mestra são **agregados** das subfichas, não duplicação. Quando uma subficha
  introduz conceito novo, o agente atualiza o frontmatter da mestra.

**Quando carregar subficha**: consulta que pede análise localizada na obra.
"Qual a função do conceito de esfera pública no cap. 8 de *Faktizität*?" → ler
`habermas-1992-faktizitat/cap-08-esfera-publica.md`, ~2000 palavras, em vez
de carregar a obra inteira.

### Camada 4 — Anexo de citações expandidas (opcional por paper)

**Onde**: `corpus/indice/papers/[chave]/citacoes.md` (para obra com diretório
próprio) ou `corpus/indice/papers/[chave]-citacoes.md` (para obra com ficha
única — convenção do hífen evita confundir com diretório).

**O que é**: arquivo de citações literais sem limite duro de comprimento.
Cada citação tem identificador (`C1`, `C2`...), texto integral, página,
edição, função no argumento do paper, uso futuro previsto, tags semânticas.

**Quando criar**:
- Quando o paper tem passagem-chave (definição constitutiva, tese-âncora)
  que excede 30 palavras
- Quando o paper tem mais citações dignas de registro literal que o teto da
  ficha permite (5 para artigo/capítulo, 10 para livro)

**Template**:

```markdown
---
chave_paper: habermas-1992-faktizitat
projeto: tese-regulacao-data-centers
caminho_canonico: projects/tese-regulacao-data-centers/corpus/indice/papers/habermas-1992-faktizitat/citacoes.md
---

# Citações expandidas — Habermas (1992)

> Citações que excedem 30 palavras ou que ultrapassam o teto da ficha mestra.
> Identificadores `C1`, `C2`... estáveis: a ficha mestra e subfichas referenciam
> por id (`> Ver C3`).

## C1 — Definição operacional de "discurso prático"

> "Por discurso prático entendo aquela forma de argumentação na qual pretensões
> de validade normativa são tematizadas hipoteticamente e examinadas com base em
> razões. Diferentemente do discurso teórico, que se ocupa de pretensões de
> verdade, o prático tem por objeto a correção (Richtigkeit) de normas de ação
> e dos juízos morais que delas derivam."
> — *Faktizität und Geltung*, p. 138, ed. 1992

- **Função no argumento**: definição constitutiva da arquitetura de legitimidade
- **Uso futuro previsto**: referência canônica em capítulo sobre fundamentação procedimental
- **Tags**: discurso, validade, legitimidade, correção
- **Capítulo de origem**: cap. 3 (procedimento)
```

**Limite**: o anexo não tem teto rígido, mas tem disciplina — cada citação
deve ter função registrada e uso futuro previsto. Citação sem propósito é
ruído.

**Quando carregar**: consulta que pede passagem literal completa, ou quando
ficha registra "ver C3" e o LLM precisa do trecho integral para responder.

## Protocolo de transição entre camadas

Implementado pelo `@consultor-corpus`. Ordem fixa:

```
1. TODA consulta começa em Camada 0 (_index.md). Identificar candidatos.

2. Se a consulta é ESTRUTURAL (contagem, listagem, mapeamento de relações):
   → Ler Camada 1 (frontmatter) dos candidatos.
   → Resolver. Stop.

3. Se a consulta é ANALÍTICA sobre paper específico:
   → Ler Camada 2 (corpo da ficha) do paper.
   → Se a ficha aponta para subficha relevante (Camada 3), abrir só ela.
   → Se a ficha aponta para citação expandida (Camada 4) e a consulta exige
     a passagem literal completa, abrir o anexo.

4. Se a consulta envolve ATÉ 5 PAPERS integrados (consulta cruzada típica):
   → Ler Camada 1 dos 5 (filtragem fina)
   → Ler Camada 2 dos 2-3 mais relevantes (apenas)
   → Subir para Camada 3/4 só se o usuário pedir o trecho específico

5. Se a consulta envolve 6+ PAPERS:
   → Camada 1 apenas, em todos
   → Decisão analítica do consultor: vale subir Camada 2 em algum?
     Geralmente NÃO — consultas sobre 6+ papers são panorâmicas;
     resposta integrada vive em mapas (skill mapeamento-conceitual)
   → Considerar gerar mapa derivado em vez de responder ad hoc

6. Em qualquer momento, NUNCA pular camadas:
   - Não ler Camada 4 (citação expandida) sem ter visto Camadas 0-2 antes
     — anexo descontextualizado induz erro de atribuição
   - Não ler Camada 3 (subficha) sem ter passado pela ficha mestra
     — perde a função do capítulo no argumento total

7. Se a resposta requer citação literal e a ficha registra apenas paráfrase
   do indexador, sinalizar ao usuário:
   "esta passagem está como paráfrase na ficha; para citar literalmente,
    [opção A: /aprofundar para refichar focado] [opção B: o usuário verifica
    no PDF original e me devolve o trecho]"
   NUNCA inventar citação literal a partir de paráfrase.
```

## Como cada camada nasce

| Camada | Cria/atualiza | Quando |
|---|---|---|
| 0 (`_index.md`) | `@leitor-profundo` | Toda indexação ou refichagem |
| 1 (frontmatter) | `@leitor-profundo` | Toda indexação |
| 2 (corpo da ficha) | `@leitor-profundo` | Toda indexação |
| 3 (subficha) | `@leitor-profundo` | Indexação de livro 300+pp (obrigatório) ou tese 100-300pp denso (opcional/recomendado) |
| 4 (citação expandida) | `@leitor-profundo` ou usuário | Quando passagem-chave excede 30 palavras ou teto da ficha |

## Anti-padrões

NÃO faça:

- Indexar livro de 500pp em ficha única de 8000 palavras (achata diferenças
  entre capítulos; o LLM precisa carregar tudo para extrair informação local)
- Criar subfichas para artigo de 25pp (overhead sem ganho; ficha única basta)
- Editar `_index.md` à mão (será sobrescrito; editar `tese_central_resumo` na
  ficha em vez disso)
- Citar literalmente passagem >30 palavras na ficha mestra (vai para anexo,
  com referência cruzada `> Ver C3`)
- Pular camadas no consumo (anexo sem ficha, subficha sem mestra)
- Inventar checksum (se não puder calcular, deixar `null` e reprocessar
  depois — mas o agente sempre deve ter capacidade de calcular)
- Numerar subficha com algarismo romano no nome de arquivo (ordenação git
  quebra)
