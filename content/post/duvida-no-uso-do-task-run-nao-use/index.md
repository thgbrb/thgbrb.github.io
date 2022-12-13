+++
author = "Thiago Borba"
title = "Está com dúvida no uso do Task.Run? NÃO USE!"
slug = "esta-com-duvida-no-uso-do-task-run-nao-use"
date = "2019-07-03"
description = "O time .NET acertou a mão quando lançou a TPL (task parallel library) no .NET 4.0, porém, isso tem sido um tiro no pé dos projetos .NET em função da falta de conhecimento dos desenvolvedores sobre a TPL."
tags = [
    "C#",
]
categories = [
    "Csharp",
]
image = "featured.jpg"
+++

Quando uma aplicação, principalmente legada, demostra problemas de performance, a primeira solução que vem na cabeça é paralelismo. Com o paralelismo vem a TPL e consequentemente o método Task.Run. Veja esse método:

```csharp
public static Task<List<InsulinDosageComplement>> GetAllAsync()
{
    return Task.Run(() => new InsulinDosageComplementWrapper().GetAll());
}
```

Esse é o erro mais comum na utilização da TPL. O que esse método faz é criar uma nova Task. Não há mágica, é isso! O método que é passado como parâmetro do método Run será executado em uma thread existente ou uma nova thread será criada.

Se considerarmos a chamada desse método em qualquer que seja o contexto, esse método irá consumir uma thread sem necessidade. O mais curioso é que o código que consome o método acima não utiliza await, ele utiliza .Result, ou seja, a thread do request fica bloqueada aguardando uma outra thread processar a chamada.

Certamente Task.Run não é a solução para problemas de performance se não for bem utilizada. Por isso, se você estiver pensando em utilizar Task.Run e está com dúvida, Não use!