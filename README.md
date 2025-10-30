# Desafio DIO: Automação Serverless com AWS Lambda e S3

Este repositório documenta a implementação de um pipeline de automação de tarefas *serverless*, utilizando a combinação resiliente de **Amazon S3** e **AWS Lambda**. O foco é consolidar o conhecimento na criação de arquiteturas orientadas a eventos (event-driven).

Criado como parte dos estudos e desafios do **Bootcamp Santander Code Girls | DIO**.

---

## Objetivo do Laboratório

O objetivo principal foi construir uma solução que reage a eventos em tempo real no *storage* de objetos, eliminando a necessidade de gerenciamento de infraestrutura. Demonstramos como os serviços da AWS se integram para automatizar tarefas de processamento de dados ou mídias.

## As Peças da Automação

O projeto se baseia na integração direta de dois serviços essenciais:

| Serviço | Papel na Arquitetura | Detalhamento |
| :--- | :--- | :--- |
| **Amazon S3** | **O Gatilho (Trigger)** | Atua como o depósito seguro de objetos e, crucialmente, como o **emissor de eventos**. É o ponto de partida que avisa a AWS sobre novas ações. |
| **AWS Lambda** | **A Ação (Action)** | É a função de computação *serverless*. Fica "escutando" o S3 e é acionada **sob demanda** para executar o código programado (lógica de negócios). |

---

## Pipeline Orientado a Eventos

A solução implementada cria um **pipeline event-driven clássico**, onde a chegada de um arquivo dispara o processamento imediatamente:

1.  **Evento:** Um novo objeto (ex: `log.txt` ou `imagem.jpg`) é carregado (*PUT*) em um bucket S3 específico (`Source Bucket`).
2.  **Gatilho:** O S3 detecta o evento (`s3:ObjectCreated:*`) e envia uma notificação para o Lambda.
3.  **Invocação:** A AWS acorda e executa a Função Lambda, escalando instantaneamente.
4.  **Processamento:** O código executa a lógica de negócios, como ler o conteúdo do arquivo, processar dados ou, em um cenário real, redimensionar uma imagem.

### Análise do Código (Node.js)

O sucesso da automação depende da correta leitura do JSON de evento enviado pelo S3. A função Lambda se concentra em "desempacotar" esse objeto para identificar o arquivo recém-criado:

```javascript
// Exemplo de como a função lê o evento S3
const AWS = require('aws-sdk');
const s3 = new AWS.S3();

exports.handler = async (event) => {
    
    const bucketName = event.Records[0].s3.bucket.name;
    const objectKey = event.Records[0].s3.object.key; // 'key' é o nome/caminho do arquivo

    console.log(`Novo arquivo detectado: ${objectKey} no bucket ${bucketName}`);

    // Regra de Negócio

    return {
        statusCode: 200,
        body: JSON.stringify('Processamento concluído com sucesso!')
    };
};
```

## Principais Aprendizados e Insights
- A execução deste desafio solidificou a compreensão de conceitos fundamentais da arquitetura de nuvem moderna:
- Fundamentos Serverless: O entendimento prático de que o Lambda elimina a sobrecarga de gerenciamento de servidores, permitindo que o foco total seja no código da aplicação.
- Design Event-Driven: Proficiência na configuração de gatilhos e permissões (IAM Roles) para criar fluxos onde a ação de um serviço (S3) invoca automaticamente outro (Lambda).
- Escalabilidade Automática: Observação de como o Lambda escala instantaneamente para lidar com picos de uploads no S3, garantindo que o processamento em lote seja eficiente.
- Leitura de Eventos: Capacidade de analisar o payload JSON que o S3 envia, o que é essencial para processar o objeto correto dentro da função.
