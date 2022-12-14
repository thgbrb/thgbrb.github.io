+++
author = "Thiago Borba"
title = "Monad Either"
slug = "monad-either"
date = "2022-12-14"
unsafe = true 
description = "A estrutura Either, comum na programação funcional, permite representar dois possíveis tipos de valores. Um bom exemplo de uso é o retorno de um método, que pode ser de sucesso e outro de falha."
tags = [
"Monads",
"Coding",
]
categories = [
"coding",
]
image = "saad-ahmad-BQLw0OrA6F4-unsplash.jpg"

+++

## Conceito do Either
O objetivo do Either é representar um valor dado duas possibilidades. Um Either com valores A e B representará A ou B, nunca os dois.

Nesse exemplo, **s** poderá representar uma string ou um int.

```haskell
let s = Left "foo" :: Either String Int
Left "foo"

let n = Right 3 :: Either String Int
Right 3
```


## Aplicação Prática: Retornos de validações
As estratégias mais comuns de retornos de validações tratam dois tipos nos seus retornos, o **sucesso** e **falha**.

No exemplo abaixo, o método **SalvarDadosAdicionais** retorna o resultado de uma operação de negócio.
Na sequencia é verificado o tipo do retorno do objeto. Quando ocorre um ou mais erros, é retornado o objeto do tipo **ValidationResult**, que é um container para os erros. 
Quando ocorre um sucesso, é retornado o objeto do tipo **SalvarDadosAdicionaisCommandResult**, que contém o **Id** do objetivo persistido.
```csharp
var retorno = await _fluxoDadosAdicionaisService.SalvarDadosAdicionais(request);

if (retorno is ValidationResult validationResult)
    return Response(validationResult);
else
    return Response((SalvarDadosAdicionaisCommandResult)retorno);
```

É interessante observar que a validação do objeto está encapsulada no método **SalvarDadosAdicionais**, que herda da classe **CommandResponse**. Essa estratégia não deixa claro se o objeto é válido ou não.
Para identificar é necessário fazer o **if** de verificação de tipo. O **CommandResponse** também faz **if** para verificar se existem propriedades nulas.
```csharp
public CommandResponse(
    TResult result,
    ValidationResult validationResult,
    HttpStatusCode? httpStatusCodeOnFailure)
{
    this.Result = result;
    this.Errors = validationResult == null ?
        Array.Empty<ValidationFailure>() :
        validationResult.Errors.ToArray();
    HttpStatusCodeOnFailure = httpStatusCodeOnFailure;
}
```

## Solução com Either
A solução Either traz clareza e remove verificações, uma vez que são definidos os dois possíveis tipos de retorno.
No exemplo abaixo, ao invés de retornarmos um tipo **object**, é retornado um Either com os seus dois possíveis valores.

```csharp
public Either<string, ArgumentNullException> ReturnsSuccessOrFail(string value)
{
    return !string.IsNullOrEmpty(value) ? value : new ArgumentNullException(nameof(value));
}
```        

A assinatura do método deixa claro que será retornado uma **string** ou uma exceção do tipo **ArgumentNullException**.

No caso de uso, o método **ReturnsSuccessOrFail** pode receber dois tipos de valores, uma **string** ou **ArgumentNullException**.
Depois de definido o valor, o método **Match** é invocado para decidir qual a função será executada.

```csharp
// Arrange
var usageCases = new UsageCases();
var value = "success case";

// Act
var result = usageCases
        .ReturnsSuccessOrFail(value)
        .Match(
            success => $"{success}",
            error => $"{error}");

// Assert
Assert.Equal(value, result);
```

## Implementação Either
A implementação do Either tem uma baixa complexidade. Utilizando generics, é possível definir um valor do tipo TL ou TR.
Com uma sobrecarga de construtor para definir TL ou TR é possível marcar qual o lado que foi setado.

```csharp
public class Either<TL, TR>
{
    private readonly TL _left;
    private readonly TR _right;
    private readonly bool _isLeft;
    
    public Either(TL left)
    {
        _left = left;
        _isLeft = true;
    }

    public Either(TR right)
    {
        _right = right;
        _isLeft = false;
    }
```

O método Match verifica se o objeto da esquerda está marcado. Se estiver marcado invoca a função da esqueda, senão a função da direita.
```csharp
public T Match<T>(Func<TL, T> leftFunction, Func<TR, T> rightFunction)
    => _isLeft ? leftFunction(_left) : rightFunction(_right);
```

A utilização de conversão implícita permite o uso do Either de forma idiomática, o que é visto na assinatura do método de exemplo.
```csharp
public static implicit operator Either<TL, TR>(TL left)
    => new Either<TL, TR>(left);

public static implicit operator Either<TL, TR>(TR right)
    => new Either<TL, TR>(right);
```

## Manutenção e Testes
Uma vez que o Either trata os dois tipos, a necessidade de verificação de tipos ou verificação de nulos em cada ponto de retorno de um método é eliminada.
Isso reduz a chance de erros, uma vez que os possíveis retornos fazem parte da assinatura do método. Essa estratégia também apoia os testes de unidade, justamente por não ter as verificações de tipo e tratamento de nulls.

> Fontes: https://github.com/thgbrb/monads