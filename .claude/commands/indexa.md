---
description: Indexa um PDF do corpus invocando o subagente @leitor-profundo,
  que produz fichamento estruturado em corpus/indice/papers/ e atualiza os
  mapas derivados de conceitos e autores. Use quando precisar adicionar paper
  novo ao índice ou refichar paper existente com template atualizado.
argument-hint: [caminho-do-pdf-relativo-ao-projeto] [--projeto slug]
---

# Indexação de paper

Você vai indexar um PDF no projeto ativo. Protocolo:

## 1. Identificação do projeto e do arquivo

1. Identificar o projeto ativo conforme hierarquia em CLAUDE.md (seção
   "Identificação do projeto ativo"): `--projeto [slug]` se fornecido em
   `$ARGUMENTS`, senão inferir por cwd, senão perguntar.
2. Localizar o arquivo em `$ARGUMENTS`. Aceitar:
   - Caminho relativo ao projeto: `corpus/papers/ayres-1992.pdf`
   - Apenas nome do arquivo: `ayres-1992.pdf` (assume `corpus/papers/`)
   - Caminho absoluto: usar como fornecido
3. Verificar que o arquivo existe e é PDF. Se não:
   - Listar PDFs disponíveis em `corpus/papers/` do projeto ativo
   - Pedir ao usuário para esclarecer

## 2. Verificação prévia

Antes de invocar `@leitor-profundo`:

1. Calcular a chave provável do paper a partir do nome do arquivo
   (autor-ano-titulo)
2. Verificar se `corpus/indice/papers/[chave].md` já existe
3. Se existir:
   - Avisar o usuário
   - Perguntar: refichar (sobrescrever)? cancelar? usar chave alternativa?
4. Se NÃO existir, prosseguir

## 3. Invocação do subagente

Invocar `@leitor-profundo` com instrução clara:

> Indexe o PDF em `[caminho-completo]` para o projeto `[slug]`.
> Siga integralmente o protocolo da skill `indexacao-paper`.
> Salve a ficha em `[caminho-do-projeto]/corpus/indice/papers/`.
> Atualize esqueletos de conceitos e autores conforme necessário.
> Devolva: caminho da ficha, tese central, conceitos-chave, lista de
> arquivos novos criados em conceitos/ e autores/, sinalizações de
> problemas.

## 4. Apresentação do resultado ao usuário

Quando o `@leitor-profundo` retornar, apresentar ao usuário:

```markdown
## Indexado: [chave]

**Tese central**:
[parágrafo da tese]

**Conceitos-chave identificados**:
- [conceito 1] → [[conceitos/conceito-1]]
- ...

**Arquivos criados/atualizados**:
- Ficha: `corpus/indice/papers/[chave].md`
- Conceitos novos: [lista]
- Autores novos: [lista]
- Conceitos atualizados: [lista]
- Autores atualizados: [lista]

**Sinalizações** (se houver):
- [problemas reportados pelo subagente]

**Próximo passo sugerido**: 
- Verificar a ficha no Obsidian e refinar manualmente onde necessário
- Continuar indexação com `/indexa [próximo-pdf]`
- Ou consultar o corpus com `/busca`, `/trecho` ou `/mapa`
```

## Restrições

- NÃO indexar PDF que não está no diretório do projeto ativo (a menos
  que o usuário forneça caminho absoluto explicitamente)
- NÃO sobrescrever ficha existente sem confirmação
- NÃO improvisar fichamento — sempre delegar a `@leitor-profundo`
- Se o PDF tem mais de ~200 páginas, avisar o usuário que a indexação
  pode levar tempo significativo e perguntar se quer prosseguir
