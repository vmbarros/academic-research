---
arquivo: changelog-metodologico
descricao: Histórico de mudanças significativas no sistema metodológico (.claude/, CLAUDE.md). Mudanças menores de fraseologia ou correções tipográficas não entram aqui — só decisões que alteram a forma como o sistema opera.
---

# Changelog metodológico

> Cada entrada registra: data, commit (se houver), o que mudou, **por que**
> mudou, e o que isso afeta nos projetos existentes ou futuros.

## Convenção

- Ordem reversa-cronológica (mais recente no topo)
- Cada entrada tem cabeçalho `## YYYY-MM-DD — [título curto]`
- Subseções fixas: **Mudança**, **Razão**, **Impacto**

---

## 2026-05-01 — Robustez de invocação (commit `1b22e78`)

**Mudança**: Formalizada hierarquia de identificação do projeto ativo
(`--projeto` > cwd > pergunta) e convenção explícita de wiki links
(relativos a `corpus/indice/` dentro do projeto; absolutos a partir do
vault para cruzamento entre projetos). Os 6 comandos passam a aceitar
`--projeto [slug]` opcional.

**Razão**: Antes, identificação do projeto dependia exclusivamente do
cwd. Falhava em três cenários: consulta cruzada da raiz do vault,
automação sem cwd no projeto, terminais paralelos com projetos distintos.
Wiki links operavam por convenção implícita — risco de resolução
ambígua quando o LLM perdia contexto do projeto ativo em sessão longa.

**Impacto**: Comandos podem ser invocados de qualquer lugar com
`--projeto`. Resolução de wiki link tem regra explícita + ponteiro
canônico (`caminho_canonico` no frontmatter). Nenhuma quebra retroativa
— comportamento default (cwd) preservado.

---

## 2026-05-01 — Auditoria e refinamento dirigido (commit `c39062f`)

**Mudança**: Novo comando `/aprofundar` para re-leitura focada de paper
em tópico específico (modo APROFUNDAMENTO do `@leitor-profundo` — agrega
ao índice, não reescreve). `@consultor-corpus` ganha protocolo de
registro de inconsistências em `_log-auditorias.md` (≥2 fichas,
divergência substantiva) e auto-detecção de incompletude (busca lateral
via relações, conceito, autor antes de devolver "não consta"). Hipóteses
de conexão passam a usar callout `> [!hypothesis]` em vez de marcador
inline.

**Razão**: Antes, fluxo era unidirecional (indexador produz → consultor
consome) e erros de indexação se propagavam em silêncio. Quando o
consultor respondia "não consta", usuário tinha 3 opções caras (aceitar
lacuna, abrir PDF manualmente, refichar inteiro). Faltava (a)
sinalização ativa de inconsistências detectadas, (b) caminho intermediário
de refinamento dirigido por gap real.

**Impacto**: Sistema ganha **memória dos próprios erros prováveis**
(`_log-auditorias.md`) e mecanismo de refinamento incremental sem
reindexação completa. Conteúdo aprofundado tem audit trail
(`> [!aprofundamento]`).

---

## 2026-05-01 — Arquitetura em 5 camadas (commit `87abeb7`)

**Mudança**: Introduzida arquitetura explícita em 5 camadas de
representação: 0 (`_index.md` denso), 1 (frontmatter), 2 (corpo da ficha),
3 (subfichas por capítulo para obras 300+pp), 4 (anexo `citacoes.md`).
Nova skill `arquitetura-em-camadas` define o padrão e o protocolo de
transição. `@leitor-profundo` ganha responsabilidade de manter `_index.md`
com checksum sha256[:4] e criar subfichas obrigatórias para livros/teses
grandes. `@consultor-corpus` ganha regra dos 5+2 passos para transição
entre camadas.

**Razão**: Sem camadas, ficha única achatava diferenças entre capítulos
em obras grandes. Para responder consulta sobre cap. 8 de obra de 700pp,
o LLM precisava carregar a ficha inteira (8000 palavras) para extrair
600. Compreensão "rápida e completa" não cabe em representação única;
exige escada que o LLM sobe conforme profundidade real.

**Impacto**: Obras 300+pp passam a ser obrigatoriamente fichadas com
subfichas. `_index.md` torna-se ponto de entrada barato para qualquer
consulta. Fichas existentes não-grandes não mudam. Anexo `citacoes.md`
permite registrar passagens-chave >30 palavras sem inflar a ficha mestra.

---

## 2026-05-01 — Frontmatter como representação estruturada (commit `6ffef21`)

**Mudança**: Reescrita do template de frontmatter em
`indexacao-paper/SKILL.md`. Listas YAML reais (eliminada forma
`[a; b; c]` que parseava como string). Slugs ASCII paralelos para
autores e conceitos (`autores_slugs`, `conceitos_*_slugs`). Tese central
em frontmatter escalonada por gênero (artigo ≤25, capítulo ≤40, tese ≤60,
livro ≤80) + `teses_centrais` (lista) para obras politéticas. Bloco
`relacoes` com `cita`, `e_citado_por` (mantido bidirecionalmente),
`desenvolve`, `critica`, `compartilha_pressuposto_com`. Bloco
`pressupostos` com `enunciados`, `operacionais_implicitos` (≥1
discriminante por ficha), `rejeitados_explicitamente`. Campos
`projeto`, `caminho_canonico` para resolução determinística. Checklists
de cobertura por gênero.

**Razão**: Frontmatter anterior era metadado bibliográfico, não
representação queriável da análise. Consultas como "quem desenvolve a
tese de X?" exigiam carregar todas as fichas. Slugs sem ASCII causavam
falhas silenciosas em grep (`Müller` ≠ `Mueller`). Listas como string
quebravam parsing.

**Impacto**: O LLM pode responder consultas estruturais (filtragem,
contagem, mapeamento de relações) lendo só YAML. Fichas pré-existentes
precisam de migração (não há, ainda — o vault está vazio de fichas
reais). Bidirecionalidade de `e_citado_por` mantida pelo
`@leitor-profundo` automaticamente.

---

## 2026-05-01 — Estrutura inicial (commit `758802a`)

**Mudança**: Setup inicial do sistema metodológico — CLAUDE.md com
postura de interlocutor acadêmico, agentes (`@leitor-profundo`,
`@consultor-corpus`), skills (`indexacao-paper`, `consulta-cruzada`,
`mapeamento-conceitual`), comandos (`/novo-projeto`, `/indexa`, `/busca`,
`/mapa`, `/trecho`).

**Razão**: Versão 0 do sistema. Captura postura intelectual (densidade,
atribuição cirúrgica, recusa do parecerismo, distinção
dogmática/zetética) e workflow básico de indexação + consulta.

**Impacto**: Base sobre a qual as 4 mudanças posteriores foram aplicadas.
