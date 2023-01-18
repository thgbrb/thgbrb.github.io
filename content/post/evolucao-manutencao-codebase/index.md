+++
author = "Thiago Borba"
title = "Manutenção e evolução de codebase"
slug = "evolucao-manutencao-codebase"
date = "2023-01-16"
description = "Aplicações complexas tendem a ter um codebase grande onde muitas pessoas, com diferentes experiências e habilidades, atuam. O resultado disso tende a ser um spagetti-codebase ao longo do tempo. Como minimizar esse impacto e melhor a manutenção e evolução de um codebase?"
tags = [
"Boas práticas",
]
categories = [
"Coding",
]
image = "linh-ha-KN8W0Q8H3gI-unsplash.jpg"
+++

Quando atuamos em sistemas complexos facilmente percebemos a quantidade de pessoas que atuaram apenas observando os estilos de codificação (inclusive os mais peculiares). Cada pessoa possuis suas habilidades e experiências, e isso influencia diretamente em como o código é escrito. Uma das estratégias para minimizar esse impacto é reduzir ao máximo a complexidade cognitiva.


## Bloco de exemplo
Esse bloco de código tem uma pontuação complexidade cognitiva alta, o que é esperado dado a quantidade de desvios dentro de um looping. O primeiro destaque vai para as branchs **else** que são redundantes, uma vez que a condição é retornada caso não seja verdadeira. 

```csharp
        foreach (var item in request.Apis)
        {
          if (!await _certificationRepository.FindApiAsync(item.IdApi))
            return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.API_NOT_FOUND, GlobalizationConstants.ApiNaoEncontrada.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));
          else if (!await _certificationRepository.ActiveApiAsync(item.IdApi))
            return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.API_NOT_ACTIVE, GlobalizationConstants.ApiNaoEstaAtiva.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));
          else if (!await _certificationRepository.CertificationApiAsync(item.IdApi))
            return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.API_NOT_CERTIFICATION, GlobalizationConstants.ApiNaoExigeCertificacao.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));
          else if (!await _certificationRepository.ScenariosApiAsync(item.IdApi))
            return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.API_NOT_SCENARIOS, GlobalizationConstants.ApiNaoPossuiCenariosParaCertificacao.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));
          else if (!await _developerRepository.DeveloperCredentials(developer.Id))
            return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.DEV_NOT_CREDENTIALS, GlobalizationConstants.DesenvolvedorNaoPossuiCredenciais.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));
        }
```

No C# high-level gerado pelo compilador a partir de um IL Release, observamos que as branchs foram removidas.

```csharp
        foreach (StartCertificationDeveloperRequest api in request.Apis)
        {
          item = api;
          if (!await developerHandler._certificationRepository.FindApiAsync(item.IdApi))
            return developerHandler.Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao((string) null, "API_NOT_FOUND", "ApiNaoEncontrada".Resource(), "MensagemPadraoErroLoremIpsum".Resource()));
          if (!await developerHandler._certificationRepository.ActiveApiAsync(item.IdApi))
            return developerHandler.Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao((string) null, "API_NOT_ACTIVE", "ApiNaoEstaAtiva".Resource(), "MensagemPadraoErroLoremIpsum".Resource()));
          if (!await developerHandler._certificationRepository.CertificationApiAsync(item.IdApi))
            return developerHandler.Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao((string) null, "API_NOT_CERTIFICATION", "ApiNaoExigeCertificacao".Resource(), "MensagemPadraoErroLoremIpsum".Resource()));
          if (!await developerHandler._certificationRepository.ScenariosApiAsync(item.IdApi))
            return developerHandler.Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao((string) null, "API_NOT_SCENARIOS", "ApiNaoPossuiCenariosParaCertificacao".Resource(), "MensagemPadraoErroLoremIpsum".Resource()));
          if (!await developerHandler._developerRepository.DeveloperCredentials(developer.Id))
            return developerHandler.Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao((string) null, "DEV_NOT_CREDENTIALS", "DesenvolvedorNaoPossuiCredenciais".Resource(), "MensagemPadraoErroLoremIpsum".Resource()));
          item = (StartCertificationDeveloperRequest) null;
        }
```

Os compiladores modernos são hábeis para otimizar boa parte do código que é escrito. Além disso, as linguagens modernas entregam **syntax-sugars** para facilitar a leitura e entendimento do código. No exemplo acima podemos observar por exemplo, que além de remover as branchs else redundante, o compilador também substituiu as constantes por seus valores. 

## Escreva código para humanos preferencialmente
Para que um codebase evolua de forma saudável, precisamos dar preferência para a escrita de código para humanos e não para compiladores. Escrever um código complexo, que privilegia o desempenho, deve ser algo que faça sentido e principalmente, tenha métricas e números que justifiquem isso. 

O exemplo abaixo é a parte de uma solução complexa que utiliza estratégia de bit-field para fazer classificações. Essa solução foi desenvolvida para resolver um problema muito específico que demandava performance e controle do uso de recursos computacionais. 
Além da documentação em código, foi necessária uma página inteira de wiki para garantir o seu entendimento e evolução.

```csharp
// 11100000000000000000000000000000-11100000000000000000000000101111 | 1F 1P 0-3O
// 11100000000000000000000000111000-11100000000000000000000000111111 | 2P 1F 0O-3O
if (value is >= 0xE0000000 and <= 0xE000002F || value is >= 0xE0000038 and <= 0xE000003F)
{
    _classificacaoManager.EnqueueClienteParaClassificacao(data.Key);

    while (!_classificacaoManager.Cpfs.TryRemove(data.Key, out _))
        spinner.SpinOnce(-1);

    _contadores.CpfEnviadoParaClassificacaoEspeculativa();

    _logger.Debug($"CPF enviado para classificação especulativamente Reason: {data.Value.BitsToString()}");
}
```

> Códigos que privilegiam performance tendem a possuir complexidade cognitiva alta!

## Como deixar o exemplo mais legível
Para exercitar a redução da complexidade cognitiva vamos refatorar o bloco de código do início desse artigo.
A primeira ação é a quebra de linha utilizando uma recomendação de 120 colunas. Também adicionamos uma quebra de linha entre os blocos IF.
Essa alteração melhora a leitura do código.

```csharp
        foreach (var item in request.Apis)
        {
          if (!await _certificationRepository.FindApiAsync(item.IdApi))
            return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.API_NOT_FOUND,
              GlobalizationConstants.ApiNaoEncontrada.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));

          if (!await _certificationRepository.ActiveApiAsync(item.IdApi))
            return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.API_NOT_ACTIVE,
              GlobalizationConstants.ApiNaoEstaAtiva.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));

          if (!await _certificationRepository.CertificationApiAsync(item.IdApi))
            return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.API_NOT_CERTIFICATION,
              GlobalizationConstants.ApiNaoExigeCertificacao.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));

          if (!await _certificationRepository.ScenariosApiAsync(item.IdApi))
            return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.API_NOT_SCENARIOS,
              GlobalizationConstants.ApiNaoPossuiCenariosParaCertificacao.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));

          if (!await _developerRepository.DeveloperCredentials(developer.Id))
            return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.DEV_NOT_CREDENTIALS,
              GlobalizationConstants.DesenvolvedorNaoPossuiCredenciais.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));
        }
```

A boa prática recomenda que o IF teste uma condição TRUE preferencialmente, o que reduz complexidade cognitiva. Quando invertemos a condição IF precisamos encadear os IFs, o que aumenta muiti a complexidade. 

> Uma boa prática é uma recomendação e não uma regra.

```csharp
        foreach (var item in request.Apis)
        {
          if (await _certificationRepository.FindApiAsync(item.IdApi))
          {
            if (await _certificationRepository.ActiveApiAsync(item.IdApi))
            {
              if (await _certificationRepository.CertificationApiAsync(item.IdApi))
              {
                if (await _certificationRepository.ScenariosApiAsync(item.IdApi))
                {
                  if (!await _developerRepository.DeveloperCredentials(developer.Id))
                    return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.DEV_NOT_CREDENTIALS,
                      GlobalizationConstants.DesenvolvedorNaoPossuiCredenciais.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));
                }
                else
                {
                  return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.API_NOT_SCENARIOS,
                    GlobalizationConstants.ApiNaoPossuiCenariosParaCertificacao.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));
                }
              }
              else
              {
                return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.API_NOT_CERTIFICATION,
                  GlobalizationConstants.ApiNaoExigeCertificacao.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));
              }
            }
            else
            {
              return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.API_NOT_ACTIVE,
                GlobalizationConstants.ApiNaoEstaAtiva.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));
            }
          }
          else
            return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.API_NOT_FOUND,
              GlobalizationConstants.ApiNaoEncontrada.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));
        }
```

No início do looping, é percebido que a propriedade Apis é uma lista dentro de um objeto que pode ser null. Nesse fluxo seria gerado uma NullException. 
De acordo com a boa prática, deve ser verificado se o objeto é nulo antes do acesso.
```csharp
if (request.Apis != null)
{
    foreach (var item in request.Apis)
    (...)
```


Analisando as chamadas de repositório é observado que cada chamada faz um SELECT no banco de forma independente. 
Além disso, essas chamadas não estão possui dependências entre si. Isso significa que elas podem ser executadas em paralelo.

```csharp
        foreach (var item in request.Apis)
        {
          var findApiAsync = _certificationRepository.FindApiAsync(item.IdApi);
          var activeApiAsync = _certificationRepository.ActiveApiAsync(item.IdApi);
          var certificationApiAsync= _certificationRepository.CertificationApiAsync(item.IdApi);
          var scenariosApiAsync = _certificationRepository.ScenariosApiAsync(item.IdApi);
          var developerCredentialsAsync = _developerRepository.DeveloperCredentials(developer.Id);

          await Task.WhenAll(new Task[] { findApiAsync, activeApiAsync, certificationApiAsync, scenariosApiAsync, developerCredentialsAsync });
          
          if (! await findApiAsync)
            return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.API_NOT_FOUND, GlobalizationConstants.ApiNaoEncontrada.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));
          
          if (!await activeApiAsync)
            return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.API_NOT_ACTIVE, GlobalizationConstants.ApiNaoEstaAtiva.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));
          
          if (!await certificationApiAsync)
            return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.API_NOT_CERTIFICATION, GlobalizationConstants.ApiNaoExigeCertificacao.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));
          
          if (!await scenariosApiAsync)
            return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.API_NOT_SCENARIOS, GlobalizationConstants.ApiNaoPossuiCenariosParaCertificacao.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));
          
          if (!await developerCredentialsAsync)
            return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, StatusOperacaoConstants.DEV_NOT_CREDENTIALS, GlobalizationConstants.DesenvolvedorNaoPossuiCredenciais.Resource(), GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource()));
        }
```


Analisando o código observamos que os retornos das verificações são objetos de erro. Esse cenário permite extrair a construção do objeto de erro para um método.
Uma estratégia melhor seria implementar essas construções no próprio objeto de erro utilizando o factory pattern.
```csharp
        CommandResponse<Result<StartCertificationDeveloperResult>> CreateResponseInternal(string statusOperacao, string mensagemAmigavel,
                                                                                          string mensagemTecnica)
        {
          return Response(Result<StartCertificationDeveloperResult>.RetornarErrosDeValidacao(null, statusOperacao, mensagemAmigavel, mensagemTecnica));
        }

        _unitOfWork.Begin();

        foreach (var item in request.Apis)
        {
          var findApiAsync = _certificationRepository.FindApiAsync(item.IdApi);
          var activeApiAsync = _certificationRepository.ActiveApiAsync(item.IdApi);
          var certificationApiAsync = _certificationRepository.CertificationApiAsync(item.IdApi);
          var scenariosApiAsync = _certificationRepository.ScenariosApiAsync(item.IdApi);
          var developerCredentialsAsync = _developerRepository.DeveloperCredentials(developer.Id);

          await Task.WhenAll(new Task[] { findApiAsync, activeApiAsync, certificationApiAsync, scenariosApiAsync, developerCredentialsAsync });

          if (!await findApiAsync)
            return CreateResponseInternal(
              StatusOperacaoConstants.API_NOT_FOUND,
              GlobalizationConstants.ApiNaoEncontrada.Resource(),
              GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource());

          if (!await activeApiAsync)
            return CreateResponseInternal(
              StatusOperacaoConstants.API_NOT_ACTIVE,
              GlobalizationConstants.ApiNaoEstaAtiva.Resource(),
              GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource());

          if (!await certificationApiAsync)
            return CreateResponseInternal(
              StatusOperacaoConstants.API_NOT_CERTIFICATION,
              GlobalizationConstants.ApiNaoExigeCertificacao.Resource(),
              GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource());

          if (!await scenariosApiAsync)
            return CreateResponseInternal(
              StatusOperacaoConstants.API_NOT_SCENARIOS,
              GlobalizationConstants.ApiNaoPossuiCenariosParaCertificacao.Resource(),
              GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource());

          if (!await developerCredentialsAsync)
            return CreateResponseInternal(
              StatusOperacaoConstants.DEV_NOT_CREDENTIALS,
              GlobalizationConstants.DesenvolvedorNaoPossuiCredenciais.Resource(),
              GlobalizationConstants.MensagemPadraoErroLoremIpsum.Resource());
        }

```

Após a análise e refatoração do método conseguimos obter:
- Redução da complexidade cognitiva;
- Paralelismo nas chamadas para banco de dados;
- Tratamento de null exception;
- Encapsulamento de construção de objetos;


## Conclusão
A complexidade cognitiva é um ofensor na manutenção e evolução de um codebase, seja por gerar ruídos no código, por não utilizar os recursos da linguagem ou até mesmo por privilegiar performance. Aplicações empresariais são complexas por natureza e a eliminação da complexidade é quase uma utopia, porém com o investimento de algum tempo na análise, aplicando os recursos da linguagem e aplicando técnicas de programação é possível reduzir consideravelmente o impacto desse ofensor. 

> 👉 Dica ☠️
> 
> Por padrão, escreva código para humanos e não para compiladores.

> 👉 Dica2 ☠️
>
> Invista 18 minutos do seu dia para conhecer e se aprofundar na sua linguagem principal. Parace pouco, mas em um ano isso representa ~90h :)