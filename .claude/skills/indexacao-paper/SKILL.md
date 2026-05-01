---
name: indexacao-paper
description: Protocolo padronizado de indexação de paper/livro/capítulo/tese
  no corpus. Use quando precisar fichar um documento novo, refichar um existente
  com template atualizado, ou validar fichas existentes contra o padrão. Contém
  o template completo, regras de preenchimento, checklists de cobertura por
  gênero e protocolo de bidirecionalidade de relações.
---

# Protocolo de indexação de paper

Esta skill define o padrão de fichamento operado pelo `@leitor-profundo` e
validado pelo `@consultor-corpus`. É o **contrato de forma** das fichas em
`corpus/indice/papers/`.

## Princípios

1. **Ficha = substituto operacional do paper**. PDFs não serão reabertos em
   consultas futuras. O que não está na ficha está perdido.

2. **Frontmatter é interface, não decoração**. O YAML é consultado por Dataview
   no Obsidian, por grep no Claude Code, e pode ser parseado por scripts. Sua
   estrutura não é flexível — é contrato. **Sempre listas YAML reais**, nunca
   strings com separador `;`.

3. **Densidade controlada e escalonada por gênero**. Ficha curta demais é falha;
   ficha longa demais queima contexto futuro. Calibragem na seção "Densidade".

4. **Citação textual é recurso escasso na ficha mestra**. Para citações longas
   ou volumosas, usar anexo `citacoes.md` (ver skill complementar de
   arquitetura em camadas).

5. **Relações entre papers são dado, não prosa**. Quem cita quem, quem desenvolve
   quem, quem compartilha pressuposto com quem — tudo no frontmatter, em
   estrutura queriável e bidirecional.

## Template de frontmatter

Toda ficha começa com este YAML. Listas são listas YAML reais (`- item` por
linha), nunca strings com `;`. Campos não preenchidos: `null` ou `[]`, não
omitir.

```yaml
---
# === Identificação ===
chave: ayres-1992-responsive            # kebab-case, único no projeto
projeto: tese-regulacao-data-centers    # slug do projeto que contém esta ficha
caminho_canonico: projects/tese-regulacao-data-centers/corpus/indice/papers/ayres-1992-responsive.md

# === Bibliográficos ===
autores:
  - Ayres, Ian
  - Braithwaite, John
autores_slugs:                          # ASCII kebab-case, sem acento
  - ayres-ian
  - braithwaite-john
ano: 1992
titulo: Responsive Regulation
subtitulo: Transcending the Deregulation Debate
tipo: livro                             # artigo|capitulo|livro|tese|dissertacao|parecer|revisao
revista_ou_editora: Oxford University Press
volume: null
numero: null
paginas: 1-205
doi: null
idioma: en                              # pt|en|es|de|fr|it|la|outro

# === Classificação temática ===
tema_central: regulacao-responsiva
campos:
  - direito-administrativo
  - direito-regulatorio
  - sociologia-juridica

# === Conceitos ===
conceitos_introduzidos:                 # cunhados pelo autor neste texto
  - regulacao-responsiva
  - piramide-de-enforcement
conceitos_introduzidos_slugs:
  - regulacao-responsiva
  - piramide-de-enforcement
conceitos_mobilizados:                  # usados de forma central, origem em outros
  - captura-regulatoria
  - dilema-do-prisioneiro
conceitos_mobilizados_slugs:
  - captura-regulatoria
  - dilema-do-prisioneiro

# === Referências ===
autores_referenciados:                  # autores citados com peso significativo
  - Stigler, George
  - Bardach, Eugene
  - Kagan, Robert
autores_referenciados_slugs:
  - stigler-george
  - bardach-eugene
  - kagan-robert
divergencias_explicitas_com:            # autores criticados nominalmente
  - Stigler, George

# === Tese ===
# Para obra monotética, usar tese_central_resumo (string).
# Para obra politética, usar teses_centrais (lista) e deixar tese_central_resumo: null.
# Limites por gênero: artigo ≤25, capítulo/parecer ≤40, tese/dissertação ≤60, livro ≤80 palavras.
tese_central_resumo: "Regulação eficaz alterna entre persuasão e sanção, escalonando coerção conforme resposta do regulado; punição direta sem persuasão prévia falha."
teses_centrais: null

# === Relações com outros papers do corpus ===
relacoes:
  cita:                                 # papers do corpus citados por este
    - chave: stigler-1971-economic
      paginas: [9, 10, 11, 12]
      funcao: contraste                 # base_teorica|dado_empirico|operacional|contraste|apoio_periferico
  e_citado_por: []                      # atualizado automaticamente quando outros papers entram
  desenvolve: []                        # papers cuja tese este expande
  critica:
    - stigler-1971-economic
  reapropria_de_modo_divergente: []
  compartilha_pressuposto_com: []
  pressuposto_compartilhado: null       # se compartilha_pressuposto_com tem itens, descrever o pressuposto

# === Pressupostos do próprio paper ===
pressupostos:
  enunciados:                           # autor declara explicitamente
    - "Atores regulados respondem racionalmente a incentivos calibrados"
  operacionais_implicitos:              # autor opera sem enunciar (deve ter ≥1)
    - "Estado tem capacidade institucional para escalonar enforcement de modo confiável"
  rejeitados_explicitamente:            # autor critica nominalmente
    - "Captura regulatória é inevitável (Stigler)"

# === Caracterização epistêmica ===
genero_argumentativo: empirico          # empirico|teorico|dogmatico-sistematico|zetetico-critico
registro_epistemico: zetetico-critico   # dogmatico-sistematico|dogmatico-aplicativo|zetetico-critico|hibrido

# === Auditoria ===
indexado_em: 2026-05-01
reindexado_em: null
cobertura_incompleta: []                # lista de itens do checklist de cobertura não preenchidos
flags_para_auditoria: []                # sinalizações abertas no _log-auditorias.md
---
```

### Regras de preenchimento

- **chave**: `[primeiro-autor-sobrenome]-[ano]-[palavra-chave-titulo]`. Único no
  projeto. Estável (mudar chave quebra wiki links em outras fichas).
- **caminho_canonico**: path absoluto a partir da raiz do vault. Permite
  resolução determinística sem inferência por cwd.
- **autores_slugs / *_slugs**: gerar automaticamente. Regra: minúsculas,
  decomposição NFD (separar caracteres + diacríticos), remover combining marks,
  espaços e pontuação viram hífen, comprimir hífens consecutivos. Ex.:
  `Müller, Friedrich` → `muller-friedrich`; `Folgenberücksichtigung` →
  `folgenberucksichtigung`; `co-originariedade` → `co-originariedade`.
- **conceitos_introduzidos** vs **conceitos_mobilizados**: distinção crucial.
  *Introduzidos* = cunhados pelo autor neste texto OU em texto anterior do mesmo
  autor. *Mobilizados* = usados de forma central, mas com origem em outro autor.
- **tese_central_resumo**: gancho de filtragem. Limites:
  - artigo: ≤ 25 palavras
  - capítulo / parecer: ≤ 40 palavras
  - tese / dissertação: ≤ 60 palavras
  - livro / monografia: ≤ 80 palavras

  Para obra politética (várias teses imbricadas, irredutíveis a uma), usar
  `teses_centrais` (lista) e deixar `tese_central_resumo: null`. Para obra
  irredutível mesmo no limite escalonado:
  `tese_central_resumo: "[obra de tese irredutível — ver corpo e subfichas]"`.
- **relacoes.cita**: incluir apenas papers do corpus (não bibliografia geral).
  A bibliografia geral entra em `autores_referenciados`.
- **relacoes.e_citado_por**: NÃO preencher manualmente. O `@leitor-profundo`
  atualiza este campo nas fichas pré-existentes quando indexa um paper novo
  que as cita. Bidirecionalidade é mantida pelo agente.
- **pressupostos.operacionais_implicitos**: deve ter ≥1 item para qualquer paper
  acadêmico não-trivial. Se você não consegue identificar um, ou o paper é
  meta-discussão metodológica (legítimo, então registrar como tal), ou a leitura
  foi superficial. Pressuposto trivial ("o autor pressupõe que o direito existe")
  é pior que ausência — deve ser **operacionalmente discriminante**, distinguindo
  este autor de tradição teórica diferente.
- **genero_argumentativo**: classificação operacional, escolher o predominante.
  Em mistura genuinamente equilibrada, registrar duas tags como lista.
- **registro_epistemico**: distinto de `genero_argumentativo`. *Dogmático* opera
  dentro do sistema jurídico positivo; *zetético* problematiza pressupostos
  mobilizando ferramentas externas. *Híbrido* só quando a mistura é estrutural,
  não fuga da decisão.
- **cobertura_incompleta**: preenchido pelo `@leitor-profundo` ao final, com itens
  do checklist de cobertura (abaixo) que não puderam ser preenchidos. Vazio = ficha
  completa. Não vazio = sinalização de gap conhecido.

## Densidade escalonada por gênero

Alvos de extensão para o corpo da ficha (excluindo frontmatter, em palavras):

| Gênero | Faixa de páginas | Ficha | Subfichas |
|---|---|---|---|
| Artigo | até 30 pp | 1500–3500 | n/a |
| Artigo extenso | 30–80 pp | 3500–5000 | opcional por seção numerada |
| Capítulo de livro | qualquer | 2000–4000 | n/a |
| Tese / dissertação 100–300 pp | 100–300 | 5000–8000 ficha mestra | opcionais |
| Livro / tese 300+ pp | 300+ | 6000–10000 ficha mestra | **obrigatórias** por capítulo (1500–3000 cada) |

Subfichas: ver skill `arquitetura-em-camadas` (Camada 3).

## Limites de citação literal

Citação literal direta na ficha:

| Gênero | Citações na ficha mestra (≤30 palavras cada) | Citações na subficha (≤30 palavras cada) | Anexo `citacoes.md` (sem teto) |
|---|---|---|---|
| Artigo | até 5 | n/a | 0–3 |
| Capítulo | até 5 | n/a | 0–3 |
| Livro / tese | até 10 | até 5 por subficha | até 10 |

Citações > 30 palavras vão **obrigatoriamente** para o anexo `citacoes.md`.
Citação que excede o teto da ficha mas é essencial: mover para anexo, deixar
referência cruzada na ficha (`> Ver C7 em [[citacoes]]`).

## Estrutura obrigatória do corpo

Em ordem fixa:

1. **Tese central** (1 parágrafo de prosa rigorosa; para obra politética, 1 parágrafo
   por tese central, marcadas como T1, T2, T3...)
2. **Estrutura argumentativa** (mapa seção→função, ou capítulo→função para livros;
   se subfichas: `[[chave/cap-XX-titulo]]` por capítulo)
3. **Conceitos operativos** (com definição literal e link wiki para `[[conceitos/...]]`)
4. **Método** (se empírico) ou **Arquitetura argumentativa** (se teórico)
5. **Referências mobilizadas** (tabela classificada por função argumentativa)
6. **Citações textuais selecionadas** (na ficha; ≤30 palavras cada; expandidas
   no anexo `citacoes.md` se necessário)
7. **Pressupostos** (eco em prosa do que está no frontmatter, com justificativa
   por pressuposto operacional implícito identificado)
8. **Lacunas e silêncios**
9. **Crítica metodológica**
10. **Registro epistêmico** (eco do frontmatter, com justificativa breve)
11. **Conexões com o corpus** (eco de `relacoes`, em prosa qualitativa)
12. **Notas para uso futuro**

## Tipologia da função argumentativa de referências

Ao classificar referências citadas pelo autor (no campo `relacoes.cita.funcao` ou
em tabela do corpo), usar UMA das categorias:

- **base_teorica**: autoridade fundadora; sem ela, o argumento desmorona
- **dado_empirico**: fonte de evidência factual
- **operacional**: fornece ferramenta conceitual aplicada
- **contraste**: interlocutor combatido nominalmente
- **apoio_periferico**: citação ornamental ou de erudição

Categorias mistas são aceitáveis (`base_teorica+operacional`), mas raras. Se você
está hesitando entre 3+ categorias, releia a passagem em que o autor é citado:
o uso geralmente é mais específico do que parece.

## Tipologia do registro epistêmico

A questão a responder: o autor *toma a norma como dado* e a sistematiza
(dogmática), ou *problematiza as premissas* da própria normatividade (zetética)?

- **dogmatico-sistematico**: opera dentro do sistema jurídico positivo,
  reconstruindo coerência interna, hierarquias, harmonizações
- **dogmatico-aplicativo**: idem, com ênfase em aplicação a casos
- **zetetico-critico**: questiona pressupostos do sistema, mobiliza ferramentas
  externas (filosofia, sociologia, economia, história) para analisar o direito
- **hibrido**: combina os dois — só quando a mistura é estrutural, não fuga

## Bidirecionalidade de relações — protocolo

Quando o `@leitor-profundo` indexa paper X e o frontmatter contém
`relacoes.cita: [{chave: A, ...}, {chave: B, ...}]`:

1. Para cada item Y em `relacoes.cita`:
   - Verificar se `corpus/indice/papers/Y.md` (ou `corpus/indice/papers/Y/_ficha.md`
     para livros com subfichas) existe.
   - Se existir, abrir a ficha de Y e adicionar `X` em `relacoes.e_citado_por`.
   - Se NÃO existir, registrar em `cobertura_incompleta: [referencia-orfa: Y]`
     da ficha de X. Quando Y for indexado posteriormente, o agente preenche
     `e_citado_por` com base nesse log.

2. Espelhamento de `desenvolve` / `critica` / `reapropria_de_modo_divergente`:
   atualmente unidirecional (a fonte declara). Não espelhar automaticamente —
   se Y "é desenvolvido por X", isso é declaração de X, não fato sobre Y.
   O `@consultor-corpus` consulta a relação fazendo grep em ambas direções.

3. Espelhamento de `compartilha_pressuposto_com`: bidirecional. Se X declara
   compartilhar com Y, Y também passa a ter X na lista, e ambos têm o mesmo
   `pressuposto_compartilhado`.

## Checklist de cobertura por gênero

Aplicar **antes de salvar**. Itens não atendidos vão para `cobertura_incompleta`.
A presença do campo é convite à `/aprofundar`, não falha bloqueante.

### Artigo empírico

- [ ] Pergunta de pesquisa identificada
- [ ] Desenho do estudo descrito (amostra, método, instrumentos)
- [ ] Achados principais sumarizados (com magnitudes/direções, não só "encontrou efeito")
- [ ] Limitações declaradas pelo autor registradas
- [ ] Implicações teóricas e práticas distintas
- [ ] Replicações/extensões mencionadas (se houver no texto)

### Artigo teórico

- [ ] Tese central isolada
- [ ] Tradição teórica em que se inscreve identificada
- [ ] Interlocutores constitutivos (autores sem os quais o argumento não se sustenta)
- [ ] Movimento argumentativo (analítico / dialético / hermenêutico / crítico-genealógico)
- [ ] Posição em relação a obras anteriores do próprio autor (continuidade/ruptura)
- [ ] Implicações desdobradas no texto vs deixadas em aberto

### Capítulo de livro

- [ ] Função do capítulo no argumento total do livro
- [ ] Tese específica do capítulo (distinta da tese do livro)
- [ ] Posicionamento relativo (introdutório / pivô / aplicativo / desfecho)

### Tese / dissertação

- [ ] Pergunta de pesquisa
- [ ] Hipótese
- [ ] Metodologia declarada
- [ ] Estrutura por capítulo com função argumentativa de cada um
- [ ] Contribuição original reivindicada (com qualificação: teórica? metodológica? empírica?)
- [ ] Banca / orientação (paratexto relevante para situar)

### Livro / monografia

- [ ] Tese mestra do livro
- [ ] Cada capítulo com função explicada na estrutura argumentativa
- [ ] Capítulos pivotais (onde o argumento avança) marcados como tal
- [ ] Conceitos cunhados no livro distinguidos de conceitos mobilizados
- [ ] Interlocutores constitutivos
- [ ] Posição em relação à obra anterior do próprio autor
- [ ] Subfichas por capítulo criadas (obrigatório se 300+ pp)

### Parecer / comentário

- [ ] Posição defendida (clara em uma frase)
- [ ] Oponentes identificados
- [ ] Técnica argumentativa predominante (analógica / dogmática / consequencialista / etc.)
- [ ] Caso ou norma específica que motiva o parecer

### Revisão de literatura

- [ ] Critérios de seleção declarados
- [ ] Recorte temporal e disciplinar
- [ ] Tipologia adotada para organizar o campo
- [ ] Lacunas identificadas pela revisão

## Checklist de qualidade — antes de salvar

Independente do gênero, verificar:

- [ ] Frontmatter parseável como YAML válido (todas as listas como listas reais)
- [ ] `chave`, `projeto`, `caminho_canonico` consistentes
- [ ] Slugs ASCII gerados para autores e conceitos
- [ ] `tese_central_resumo` dentro do limite de palavras do gênero, OU
      `teses_centrais` preenchido para obra politética
- [ ] `pressupostos.operacionais_implicitos` com ≥1 item (não trivial)
- [ ] `relacoes.cita` apenas para papers do corpus, com função argumentativa
- [ ] Bidirecionalidade aplicada (`e_citado_por` atualizado nas fichas dos citados)
- [ ] Tabela de referências classifica TODAS as citações de peso
- [ ] Citações literais ≤ 30 palavras na ficha; >30 vão para `citacoes.md`
- [ ] Lacunas identificadas são SUBSTANTIVAS (não nitpicks)
- [ ] Crítica metodológica é analítica, não panegírica nem destrutiva
- [ ] Registro epistêmico identificado e justificado
- [ ] Links wiki para conceitos e autores existentes no corpus
- [ ] `cobertura_incompleta` listado se algum item do checklist de cobertura ficou aberto

## Refichar paper existente

Quando refichar paper já indexado:

1. Verificar se a chave é a mesma (NÃO trocar — quebra links em outras fichas)
2. Preservar `indexado_em` original; preencher `reindexado_em`
3. Comparar nova ficha com antiga; registrar diferenças significativas em
   `corpus/indice/_log-reindexacoes.md`
4. Re-aplicar bidirecionalidade: `e_citado_por` deve ser preservado/recomputado

## Anti-padrões

NÃO faça:

- Listas YAML fake (`[a; b; c]` como string única) — usar listas reais
- Resumir parágrafo a parágrafo (use estrutura argumentativa por função)
- Listar autores citados sem função (use tabela classificada)
- Citar literalmente porque "é bonito" (use por utilidade futura prevista)
- Marcar todo paper como "híbrido" no registro epistêmico (essa classificação
  é informativa quando é o caso real, mas se você sempre marca híbrido está
  fugindo da decisão)
- Inventar links wiki para conceitos/autores que não existem no índice
  (criar primeiro o esqueleto do arquivo)
- Inventar página, citação, ano, título ou afirmação. Página desconhecida → `[s/p]`.
  Dado não localizado → `null` no YAML, `[a verificar]` no corpo.
- Pressuposto operacional trivial ("o autor pressupõe que existe linguagem")
  — se não consegue identificar pressuposto discriminante, releia
- Ficha mestra inflada com citação que devia estar no anexo
