+++
author = "Thiago Borba"
title = "Boas práticas em commits"
slug = "boas-praticas-em-commits"
date = "2021-07-08"
description = "O commit é uma ação tão trivial e automática que às vezes esquecemos ou até mesmo não paramos para pensar na sua importância."
tags = [
"Boas práticas",
]
categories = [
"Qualidade",
]
image = "featured.jpg"
+++

Durante um apoio a projeto me deparei com o seguinte cenário quando acessei git log:

```text
391a8dad Elaboração dos contadores de performance para os comando da Api Pedidos
fdde9224 Merge branch 'release/v8.0.3' of https://dev.azure.com/XXXXXXXXXXXXXX/v8.0.3
3c2b525f publicações
a3cb5825 merge apresentação da url do boleto com o desconto do cupom frete enviado ao ax
f9852c90 ajuste para mandar o valor de desconto do cupom tipo frete correto ao ax
64785991 Pedidos: Exibindo o link do Pdf no nó de boleto de cobrança + XXXXXX.Produtos: ajuste no nome de um dos eventos do contador de performance + XXXXXX.YYYYYYYY: exibição do link do Pdf do boleto
c9c79c0d publicaçoes
07570cf7 CatalogoDeProdutos: CatalogoDeProdutosPerformanceMonitor herda a interface IPerformanceMonitor
f113e074 Conflict resolution
424246b0 Update azure-pipelines-unit.yml for Azure Pipelines
d0e870bd Update azure-pipelines-unit.yml for Azure Pipelines
8c5255d2 Update azure-pipelines-unit.yml for Azure Pipelines
7fdf5e97 Update azure-pipelines-unit.yml for Azure Pipelines
```

A situação do histórico era tão caótica que era quase impossível entender qual o propósito de um commit,
tornando inútil a sua descrição. A história daquele fonte estava muito mal contada e é aí que está o problema.
Quando era necessário entender a implementação de uma regra de negócio ou o que motivou as alterações no código não existia contexto,
não existia história.
Uma sequência de commits com o mesmo título "Update azure-pipelines-unit.yml for Azure Pipelines" não deixa claro o motivo da alteração.
Apenas sabemos que ocorreu uma sequência de alterações em um arquivo. 

A falta de relação do commit com uma issue em uma ferramenta ALM contribuía para a falta de contexto. Em algumas situações
era necessário recorrer ao time de negócio para resgatar o histórico que motivou a alteração de uma regra de negócio.

> Os commits em um repositório git contam uma história, a história de vida de um código. Eles descrevem o que motivou a fazer 
> as alterações que foram feitas de uma forma muito clara, mantendo o histórico e apoiando os futuros desenvolvedores. 

Um commit bem escrito é dividido em duas partes, o assunto com no máximo 50 caracteres e que de forma imperativa expressa 
a sua intenção e um body que descreve o porquê do commit. Um bom commit também faz referência ou está vinculado a uma issue
que contextualiza o commit. No exemplo abaixo, é possível verificar algumas boas práticas na escrita de commits aplicadas no 
repositório do Erlang.

```text
21b7ad04ce Clarify the asynchronousness of signal reception
5b762b79a4 erts: Document hibernate resume function must be exported
18e9843e44 Improve receive docs
410b279d41 erts: Fix super alignment in etp-commands
f0ae5f12d6 erts: Short circuit immediates in match spec compiler
220a46789e erts: Micro optimize arityval -> make_arityval
f12067111a erts: Silence harmless compiler warnings
68cfb519ca erts: Optimize erts_qsort_partion_array
e01cfccd2f crypto: Test generated EC private key length
18fa73579d dialyzer: Do not expose line number 0 in message locations
```

O projeto Erlang possui uma qualidade alta na escrita de seus commits. No exemplo abaixo o assunto demonstra a intenção e 
no body o porquê.

> [Exemplo de um commit com boas práticas](https://github.com/erlang/otp/commit/a65d0d571c13d632eed9b0a6b61052c1b4b758bb)
>```text
> Don't keep a stacktrace longer than necessary
>
> After an exception had been caught, the stacktrace (`p->ftrace`) would
> be retained in the process until another exception occurred.
> 
> While at it, also clear the exception term (`p->fvalue`) in the common
> error handling code instead of in the code for `try_case` in each module
> ```

## Guia de Boas Práticas
Mais do que uma simples descrição, as boas práticas na escrita de commit servem para apoiar o processo de revisão, 
contribuir para um release notes rico e ajudar os futuros desenvolvedores.

- O commit é composto por um **assunto** e um **body** <sup>_opcional_</sup>.
- Sempre mantenha uma linha em branco entre o assunto e body quando existir um.
- Não utilize ponto final no final do assunto.
- Mantenha o assunto com no máximo 50 caracteres.
- Utilize o body para **explicar** o que foi feito ao invés de **como** foi feito.
- Dê preferência para uma linguagem imperativa no assunto.
- Relacione as issues no final do body.  
- Conheça o git (squash é um excelente exemplo).  
- Respeite os padrões acordados com o time.

> [Para mais informações acesse o git book.](https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project)