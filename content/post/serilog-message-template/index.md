+++
authors= ["Thiago Borba", "ChatGPT"]
title = "Utilizando MessageTemplate em log estruturado."
slug = "serilog-message-template"
date = "2023-04-19"
description = "Log estruturado é um método de armazenamento de dados de registro em um formato pré-definido e organizado, facilitando a análise e busca de informações específicas. Neste post, discutiremos como usar o MessageTemplate do Serilog para log estruturado em aplicações."
tags = [
"DevOps",
"Logging",
]
categories = [
"infrastructure",
"csharp",
]
image = "jatin-jangid-1f0DB1u7p8Q-unsplash.jpg"
+++

## Definição de log estruturado
Log estruturado é uma técnica de registro de eventos em sistemas de computador, que envolve o armazenamento de informações em um formato padronizado e organizado, facilitando a análise e busca de informações específicas. Em contraste com o log não estruturado, onde as informações são gravadas em texto simples e sem formatação, o log estruturado segue uma estrutura definida, geralmente em formato de chave-valor. Essa estrutura permite que os dados sejam organizados de maneira coerente e significativa, tornando mais fácil a análise de problemas e a identificação de padrões. Além disso, o log estruturado pode ser armazenado em vários tipos de banco de dados, permitindo um acesso mais rápido e eficiente às informações de registro. Em resumo, o log estruturado é uma técnica importante para melhorar a visibilidade e a confiabilidade de sistemas de computador, tornando mais fácil a identificação e resolução de problemas.

### Exemplo de logs
_Exemplo de log não estruturado_
```log
2023-04-18 14:32:45,943 [INFO] Aplicativo iniciado
2023-04-18 14:32:46,156 [DEBUG] Conexão com banco de dados estabelecida com sucesso
2023-04-18 14:33:02,234 [ERROR] Erro ao processar solicitação de login do usuário "steve_jobs"
2023-04-18 14:34:17,345 [WARN] Quantidade de memória disponível abaixo do limite mínimo
2023-04-18 14:35:09,876 [INFO] Aplicativo encerrado
```

Nesse exemplo, as informações são gravadas em texto simples e sem formatação definida. As linhas do log contêm um carimbo de data/hora, um nível de log (INFO, DEBUG, ERROR, WARN), e uma mensagem descritiva sobre o evento que ocorreu. No entanto, não há estrutura definida para as informações, tornando difícil a análise e busca de informações específicas, especialmente em grandes conjuntos de dados de log.

_Exemplo de log estruturado_
```json
{"timestamp": "2023-04-18T14:32:45.943Z", "level": "INFO", "message": "Aplicativo iniciado"}
{"timestamp": "2023-04-18T14:32:46.156Z", "level": "DEBUG", "message": "Conexão com banco de dados estabelecida com sucesso"}
{"timestamp": "2023-04-18T14:33:02.234Z", "level": "ERROR", "message": "Erro ao processar solicitação de login", "username": "steve_jobs"}
{"timestamp": "2023-04-18T14:34:17.345Z", "level": "WARN", "message": "Quantidade de memória disponível abaixo do limite mínimo"}
{"timestamp": "2023-04-18T14:35:09.876Z", "level": "INFO", "message": "Aplicativo encerrado"}
```

Observe que agora há um campo adicional chamado "username" que foi adicionado ao evento de log que gerou o erro. O valor "joao" que antes estava na mensagem agora foi transformado em uma propriedade, tornando a informação mais facilmente acessível e buscável. Essa é uma das principais vantagens do log estruturado: a capacidade de separar as informações em campos específicos para que possam ser facilmente pesquisadas e analisadas.

## O que é o MessageTemplate no Serilog
O MessageTemplate no Serilog é um recurso que permite que as mensagens de log sejam formatadas de maneira mais sofisticada e flexível. Ele é um modelo de mensagem que define uma string de formato contendo marcadores de posição que podem ser substituídos por valores dinâmicos durante a gravação do log. Em vez de simplesmente gravar uma mensagem de log em uma string fixa, o MessageTemplate permite que as mensagens de log sejam formatadas de maneira personalizada, com base em informações específicas do evento de log.
O uso de MessageTemplates também pode melhorar o desempenho, reduzindo a sobrecarga de alocação de memória e a conversão de tipos, uma vez que os valores de substituição são processados de forma mais eficiente do que se fossem concatenados em uma única string fixa. Em resumo, o MessageTemplate é uma ferramenta útil para tornar a gravação de logs mais flexível, legível e eficiente.

Para utilizar o MessageTemplate do Serilog, é necessário definir uma string de formato que contenha marcadores de posição para os valores que serão substituídos durante a gravação do log. Os marcadores de posição são definidos entre chaves e podem conter um nome opcional para identificar o valor que será substituído.

```csharp
_log.Debug("{Count} eventos para serem emitidos", _filaEventos.Count);

// {"Count":10, "Message":"10 eventos para serem emitidos", "Level":"Debug", "Timestamp":"2023-04-19T15:30:00.0000000Z"}
```

Neste exemplo, a propriedade "Count" é adicionada ao log estruturado para representar o número de eventos pendentes na fila. O valor dessa propriedade é 10, que é o valor de _filaEventos.Count. Além disso, a mensagem de log também é incluída na propriedade "Message", e o nível de log "Debug" é registrado na propriedade "Level". A propriedade "Timestamp" contém a data e hora em que o log foi registrado.

Com essa saída, é possível realizar análises mais precisas e filtrar logs de maneira mais eficiente, utilizando as propriedades definidas no log estruturado. O uso de logs estruturados é uma prática recomendada para melhorar a qualidade e a eficiência da gestão de logs em aplicações e sistemas.

## Erro comum ao gerar log estruturado
Um erro comum ao gerar uma mensagem de log é ignorar o messageTemplate. No exemplo abaixo é feito a concatenação da propriedade Count com a string definida. 
Essa prática não permite que o log seja indexado ou que seja utilizado para geração de métricas quantitativas. Também é observado uma ineficiencia na alocação de memória, onde ocorre uma operação de box/unbox do int e a alocação de uma nova string.

```csharp
_log.Debug($"{_filaEventos.Count} eventos para serem emitidos");

// [Debug] 10 eventos para serem emitidos
```


## Conclusão
Em conclusão, o MessageTemplate do Serilog é uma ferramenta poderosa para personalizar e formatar mensagens de log de maneira estruturada. Ao definir uma string de formato com marcadores de posição, é possível adicionar propriedades significativas aos logs que permitem análises precisas e filtragem eficiente. 


> 🤯 **Informação**
>
> Conheça mais sobre o messagem template em https://messagetemplates.org