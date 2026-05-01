---
description: Re-leitura dirigida de paper já indexado, focada em tópico específico
  que a ficha atual não cobre adequadamente. Invoca @leitor-profundo em modo de
  refichamento focado, que LÊ o PDF buscando o tópico e ADICIONA ao índice (não
  reescreve a ficha mestra). Resultado é cumulativo. Use quando @consultor-corpus
  responde "esta informação não está na ficha" mas você sabe que está no paper.
argument-hint: [chave-do-paper] [tópico entre aspas] [--projeto slug]
---

# Aprofundamento dirigido de ficha

Você vai disparar re-leitura focada de paper já indexado, sem refichá-lo
inteiro. O resultado se acumula no índice como ampliação, não como
substituição.

## 1. Identificação do projeto, paper e tópico

1. Identificar o projeto ativo conforme hierarquia em CLAUDE.md (seção
   "Identificação do projeto ativo"): `--projeto [slug]` se fornecido em
   `$ARGUMENTS`, senão inferir por cwd, senão perguntar.
2. Receber em `$ARGUMENTS`: `[chave-do-paper] [tópico entre aspas]`
3. Validar:
   - A ficha existe? Localizar `corpus/indice/papers/[chave].md` (paper único)
     ou `corpus/indice/papers/[chave]/_ficha.md` (obra com subfichas)
   - O PDF original existe em `corpus/papers/`? Se NÃO, abortar com mensagem:
     "PDF não encontrado em corpus/papers/. /aprofundar exige re-leitura do
     PDF original; sem ele, não há fonte para ampliar a ficha."
   - O tópico está formulado de modo específico? Se for vago ("aprofunda
     mais", "amplia"), pedir reformulação:
     > "Para aprofundar, preciso de tópico específico. Exemplos:
     >  - 'tratamento da co-originariedade entre autonomia pública e privada'
     >  - 'crítica a Luhmann no cap. 2'
     >  - 'método de seleção de casos do estudo empírico'"

## 2. Decisão sobre destino do conteúdo

Antes de invocar o subagente, decidir onde o conteúdo aprofundado vai morar:

- **Subficha tópica nova**: se o tópico é tratamento extenso (>1500 palavras
  esperadas), criar subficha dedicada em
  `corpus/indice/papers/[chave]/topico-[slug].md` (para obra com diretório)
  ou converter ficha única em obra com diretório se o ganho justificar.
- **Ampliação de subficha existente**: se o tópico cabe dentro de capítulo
  já fichado, ampliar a subficha (`cap-XX-titulo.md`) com nova seção
  "Aprofundamento: [tópico]".
- **Adição ao anexo de citações**: se o aprofundamento é principalmente
  registro de passagens literais que faltavam, ir para `citacoes.md`.
- **Atualização da ficha mestra**: se o tópico afeta a tese central ou
  conceitos operativos, atualizar Camadas 1-2 (com cuidado para não perder
  conteúdo existente).

A decisão é do `@leitor-profundo` durante a re-leitura, com base no que
encontrar. Mas o orquestrador deve passar a decisão preliminar.

## 3. Invocação do subagente

Invocar `@leitor-profundo` em **modo aprofundamento**:

> Modo: APROFUNDAMENTO (não fichamento integral).
>
> Paper: `[chave]`
> Caminho do PDF: `corpus/papers/[arquivo.pdf]`
> Caminho da ficha existente: `corpus/indice/papers/[chave].md` ou
>   `corpus/indice/papers/[chave]/_ficha.md`
> Tópico solicitado: `"[tópico]"`
>
> Procedimento:
> 1. Ler a ficha existente (Camadas 1-2 e subfichas relevantes, se houver)
>    para conhecer o que já está registrado e EVITAR duplicação.
> 2. Reabrir o PDF original e LER apenas as seções/capítulos onde o tópico
>    aparece. Usar índice/sumário do PDF como ponto de entrada; se não tem
>    índice, fazer leitura focada por palavras-chave do tópico.
> 3. Produzir conteúdo novo (não reescrever conteúdo existente) seguindo
>    o template aplicável (subficha, ampliação ou anexo, conforme o caso
>    decidido pelo orquestrador).
> 4. Marcar todo conteúdo novo com bloco de origem:
>    ```
>    > [!aprofundamento] Adicionado por /aprofundar em [data]
>    > Tópico: "[tópico]"
>    > Páginas relidas: [intervalo]
>    ```
> 5. Atualizar `cobertura_incompleta` na ficha mestra: remover qualquer item
>    relacionado ao tópico aprofundado.
> 6. Atualizar `_index.md` se o resumo da obra precisar refletir o novo
>    conteúdo.
> 7. Registrar a operação em `corpus/indice/_log-aprofundamentos.md`.
>
> Devolver: arquivos modificados, novo conteúdo (resumo), itens de
> cobertura_incompleta fechados, sinalizações de problema.

## 4. Apresentação do resultado

Apresentar ao usuário:

```markdown
## Aprofundamento concluído: [chave] — "[tópico]"

**Origem**: `corpus/papers/[arquivo.pdf]`, pp. [intervalo relido]

**Conteúdo adicionado**:
- [Local]: [arquivo modificado/criado, com resumo do que foi adicionado]

**Sinalizações fechadas em `cobertura_incompleta`**:
- [item antes aberto, agora resolvido]

**Sinalizações novas** (se o aprofundamento expôs outros gaps):
- [item novo]

**Próximo passo sugerido**: 
- Verificar conteúdo no Obsidian e refinar manualmente se necessário
- Para outro aprofundamento: `/aprofundar [chave] "[outro tópico]"`
- Se o aprofundamento expôs inconsistências entre fichas, considerar
  registro em `_log-auditorias.md`
```

## Restrições

- NÃO reescrever conteúdo existente da ficha sem confirmação explícita
  (aprofundamento agrega, não substitui — exceção apenas para erros
  factuais identificados durante a re-leitura)
- NÃO disparar `/aprofundar` para tópico vago (exigir formulação específica)
- NÃO ampliar ficha de paper sem PDF disponível (não há fonte para a
  ampliação)
- O conteúdo novo SEMPRE marcado com `> [!aprofundamento]` para que o
  consultor-corpus saiba que veio de re-leitura dirigida (audit trail)
