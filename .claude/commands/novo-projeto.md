---
description: Cria um novo projeto de pesquisa em projects/ com estrutura padrão.
  Solicita ao usuário informações mínimas (slug, título, tipo, pergunta de
  pesquisa, conceitos-chave, autores-âncora) e gera a árvore completa de pastas,
  PROJECT.md preenchido, e _master.md inicial do índice. Use quando iniciar nova
  pesquisa: tese, dissertação, artigo, capítulo, parecer, seminário ou curso.
argument-hint: [slug-do-projeto-opcional]
---

# Criação de novo projeto de pesquisa

Você vai criar um novo projeto sob `projects/`. Siga o protocolo abaixo
sem pular etapas. Não invente respostas para campos não fornecidos —
peça ao usuário.

## 1. Coleta de informações

Se o usuário forneceu um slug em `$ARGUMENTS`, use-o como ponto de
partida (mas valide). Caso contrário, peça-o junto com as demais
informações.

**Validação do slug**:
- kebab-case (palavras separadas por hífen, minúsculas)
- curto e descritivo (3-6 palavras)
- estável: NÃO conter ano nem versão (essas vão no PROJECT.md)
- exemplos válidos: `tese-regulacao-data-centers`, `artigo-barganha-aneel`,
  `seminario-direito-digital`, `parecer-licenciamento-ambiental`

**Verificação prévia**: antes de criar qualquer arquivo, verificar se
`projects/[slug]/` já existe. Se existir, ABORTAR e avisar — não
sobrescrever.

**Solicitação ao usuário** (em uma única mensagem com perguntas
numeradas, em português):

```
Para criar o projeto, preciso das seguintes informações. Pode responder
em formato livre — vou preencher o PROJECT.md a partir das suas respostas.

1. **Slug** (se ainda não definido): identificador curto em kebab-case
2. **Título completo** do trabalho
3. **Tipo de trabalho**: tese / dissertação / artigo / capítulo / parecer
   / seminário / curso / outro
4. **Pergunta de pesquisa** (pode ser provisória, em uma frase)
5. **Hipótese inicial** (provisória; aceitar "sem hipótese ainda")
6. **Tema central** (palavra ou expressão curta — será tag-mãe)
7. **Conceitos-chave iniciais** (3-7 conceitos que já sabe que serão
   centrais)
8. **Autores-âncora** (3-5 autores que já sabe que serão centrais)
9. **Prazo / horizonte temporal** (ex.: "qualificação 2027-03",
   "submissão até 2026-08", "sem prazo")
10. **Orientador(a) / banca / interlocutores** (opcional)
```

NÃO prossiga sem essas respostas. Se o usuário disser "tanto faz" ou
similar para um campo, registrar `[a definir]` no PROJECT.md — nunca
inventar.

## 2. Criação da estrutura

Após coletar, criar exatamente esta árvore (em ordem):

```
projects/[slug]/
├── PROJECT.md
├── corpus/
│   ├── papers/.gitkeep
│   └── indice/
│       ├── _index.md             (Camada 0 — vazio inicialmente)
│       ├── _master.md            (entrada navegável Dataview)
│       ├── papers/.gitkeep
│       ├── conceitos/
│       │   ├── [slug-1].md       (esqueleto, nome = slug ASCII)
│       │   ├── [slug-2].md
│       │   └── ...
│       ├── autores/
│       │   ├── [slug-1].md       (esqueleto, nome = slug ASCII)
│       │   ├── [slug-2].md
│       │   └── ...
│       └── mapas/.gitkeep
├── sessoes/.gitkeep
└── escrita/
    ├── rascunhos/.gitkeep
    └── final/.gitkeep
```

Use `mkdir -p` e `touch` via Bash. Os arquivos `.gitkeep` mantêm pastas
vazias versionáveis.

## 3. Conteúdo dos arquivos gerados

### `PROJECT.md`

```markdown
---
slug: [slug]
titulo: [título completo]
tipo: [tipo]
tema_central: [tema central]
status: ativo
criado_em: [data ISO de hoje]
prazo: [prazo informado, ou "sem prazo"]
---

# [Título completo]

## Pergunta de pesquisa

[pergunta]

## Hipótese atual

[hipótese ou "sem hipótese ainda"]

> Esta hipótese é provisória e deve ser revisada conforme o corpus
> avança. Mantenha histórico das versões anteriores na seção
> "Histórico de hipóteses" abaixo.

## Conceitos-chave iniciais

- [[conceitos/conceito-1]]
- [[conceitos/conceito-2]]
- ...

## Autores-âncora

- [[autores/autor-1]]
- [[autores/autor-2]]
- ...

## Estado atual

- [ ] Corpus inicial reunido
- [ ] Indexação completa do corpus inicial
- [ ] Mapas conceituais derivados
- [ ] Estrutura argumentativa definida
- [ ] Primeiro rascunho integral
- [ ] Revisão crítica
- [ ] Versão para qualificação/submissão

## Decisões metodológicas

> Registre aqui escolhas que afetam o trabalho como um todo: recortes,
> exclusões, tradições teóricas adotadas. Cada decisão com data e
> justificativa breve.

## Questões em aberto

> Pontos que você sabe que precisará resolver mais tarde. Não tente
> resolver agora — só registre.

## Histórico de hipóteses

> Quando a hipótese principal mudar, mover a versão antiga para cá com
> data. Pesquisa madura tem trajetória registrada.

## Interlocutores

[orientador, banca, co-autores, ou "[a definir]"]

---
*Documento vivo. Atualizar quando o foco do projeto mudar.*
```

### `corpus/indice/_index.md`

Camada 0 da arquitetura (ver `.claude/skills/arquitetura-em-camadas/SKILL.md`).
Mantido automaticamente pelo `@leitor-profundo`. Cabeçalho fixo:

```markdown
# Índice denso — [Título do projeto]

> Gerado e mantido pelo `@leitor-profundo`. NÃO editar à mão — alterações são
> sobrescritas na próxima indexação. Para refinar resumo de uma obra, editar
> `tese_central_resumo` no frontmatter da ficha; será propagado.

| chave | ano | gênero | tese (≤80 palavras) | tags | checksum |
|---|---|---|---|---|---|

*Sem entradas. Será preenchido conforme papers forem indexados.*
```

### `corpus/indice/_master.md`

```markdown
# Índice mestre — [Título]

Entrada raiz do corpus. Toda navegação inicia aqui.

## Papers indexados

\`\`\`dataview
TABLE 
  ano AS "Ano",
  tema_central AS "Tema central",
  file.link AS "Ficha"
FROM "corpus/indice/papers"
SORT ano ASC
\`\`\`

## Conceitos no corpus

\`\`\`dataview
LIST FROM "corpus/indice/conceitos"
SORT file.name ASC
\`\`\`

## Autores referenciados

\`\`\`dataview
LIST FROM "corpus/indice/autores"
SORT file.name ASC
\`\`\`

## Mapas derivados

\`\`\`dataview
LIST FROM "corpus/indice/mapas"
\`\`\`

## Estatísticas do corpus

\`\`\`dataview
TABLE length(rows) AS "Quantidade"
FROM "corpus/indice/papers"
GROUP BY tipo
\`\`\`

---
*Atualizado automaticamente via Dataview conforme novos papers são indexados.*
```

### Esqueletos em `corpus/indice/conceitos/[slug].md`

Nome do arquivo: slug ASCII do conceito (regra de slug em
`.claude/skills/indexacao-paper/SKILL.md`, seção "Geração de slugs ASCII").

Para cada conceito-chave informado:

```markdown
---
conceito: [nome do conceito como o usuário escreveu]
slug: [slug ASCII]
projeto: [slug do projeto]
status: a-indexar
papers_que_mobilizam: []
papers_que_introduzem: []
divergencias_semanticas: []
---

# [Conceito]

> [!note] A indexar
> Este arquivo foi criado como esqueleto. Será preenchido conforme papers
> do corpus que mobilizam o conceito forem indexados pelo @leitor-profundo.

## Definição operacional adotada no projeto

[a definir após indexação dos primeiros papers]

## Origem no corpus

[a indexar]

## Definições no corpus

[a indexar]

## Genealogia conceitual

[a indexar]

## Divergências semânticas

[a indexar]

## Conceitos relacionados

[a indexar]
```

### Esqueletos em `corpus/indice/autores/[slug].md`

Nome do arquivo: slug ASCII do autor — usar `sobrenome-nome` quando há nome
próprio, ou `sobrenome` quando o autor é tradicionalmente citado só pelo
sobrenome no campo (regra de slug em `.claude/skills/indexacao-paper/SKILL.md`).

Para cada autor-âncora informado:

```markdown
---
autor: [Nome completo]
slug: [slug ASCII, ex.: habermas-jurgen]
projeto: [slug do projeto]
obras_no_corpus: []
papers_que_citam: []
funcao_recorrente: a-definir
---

# [Nome do autor]

> [!note] A indexar
> Este arquivo foi criado como esqueleto. Será preenchido conforme papers do
> corpus que citam o autor forem indexados pelo @leitor-profundo.

## Identificação

[áreas, formação, obras-chave — a preencher]

## Conceitos centrais atribuídos

[a indexar]

## Função no corpus

[a definir após indexação dos primeiros papers]

## Papers que mobilizam o autor

[a indexar — será gerado por Dataview ou manualmente]

## Divergências internas

[a indexar — papers do corpus que usam este autor em sentidos incompatíveis]

## Genealogia

[a indexar]
```

## 4. Confirmação ao usuário

Após criar tudo, devolver ao usuário, em português, mensagem com:

1. Caminho absoluto do projeto criado
2. Lista das pastas e arquivos criados (resumida — não listar todos os
   `.gitkeep`)
3. Próximo passo sugerido:
   > "Adicione PDFs em `corpus/papers/` e use `/indexa [arquivo.pdf]`
   > para começar a indexação. Para abrir o projeto no Obsidian, aponte
   > o vault para `[caminho-do-projeto]/`."
4. Lembrete:
   > "PROJECT.md é documento vivo. Atualize quando a pergunta de pesquisa,
   > hipótese ou status mudarem."

NÃO escreva mais nada após a confirmação. Não ofereça ajuda adicional
não solicitada.

## Restrições

- NUNCA criar projeto em diretório que já existe
- NUNCA inventar dados que o usuário não forneceu (usar `[a definir]`)
- NUNCA pular a coleta de informações com pretexto de "agilizar"
- Se Zotero MCP disponível, NÃO sincronizar automaticamente — apenas
  mencionar ao usuário que pode fazer manualmente quando quiser
