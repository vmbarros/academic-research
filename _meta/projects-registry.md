---
arquivo: projects-registry
descricao: Índice global dos projetos ativos e arquivados. Atualizado pelo comando /novo-projeto e por revisões periódicas. Função operacional - permite ao @consultor-corpus localizar projetos sem listar diretórios, e identificar papers compartilhados entre projetos.
---

# Registro de projetos

## Ativos

```dataview
TABLE
  titulo AS "Título",
  tipo AS "Tipo",
  status AS "Status",
  prazo AS "Prazo",
  criado_em AS "Criado em"
FROM "projects"
WHERE file.name = "PROJECT" AND status = "ativo"
SORT criado_em DESC
```

## Arquivados

```dataview
TABLE
  titulo AS "Título",
  tipo AS "Tipo",
  prazo AS "Prazo",
  arquivado_em AS "Arquivado em"
FROM "projects"
WHERE file.name = "PROJECT" AND status = "arquivado"
SORT arquivado_em DESC
```

## Papers compartilhados entre projetos

> Quando um mesmo paper é fichado em múltiplos projetos, registrar aqui para
> evitar drift entre fichas. Atualizado quando o `@leitor-profundo` detecta que
> a chave do paper já existe em outro projeto.

| chave | projetos | sincronizado_em |
|---|---|---|

---

*Atualizar via `/novo-projeto` ao criar projeto, ou manualmente ao arquivar.*
