# ☁️ Arquitetura Orientada a Eventos (EDA) na AWS

Este repositório documenta o laboratório intensivo da Semana do Desenvolvedor AWS (Escola da Nuvem), focado na construção de uma Arquitetura Orientada a Eventos para Processamento de Pedidos e Arquivos.

## 🎯 Visão Geral do Sistema e Arquitetura

O desafio de negócio consiste em gerenciar pedidos de forma eficiente e escalável a partir de múltiplas fontes (em tempo real via API e em lotes via S3). O sistema foi projetado utilizando computação Serverless e mensageria assíncrona para garantir desacoplamento, resiliência e alta disponibilidade.

![Arquitetura Completa](imagens/arquitetura_completa.png)

### Pilares da Solução:
* **Ingestão Múltipla:** API REST para pedidos em tempo real e S3 para processamento em lote.
* **Processamento Assíncrono:** SQS para desacoplamento e EventBridge como barramento de eventos central.
* **Persistência e Notificações:** DynamoDB para dados e SNS para alertas de erro em tempo real.
* **Resiliência Integrada:** DLQs para tratamento de falhas e Lambda Layers para reutilização de código.

---

## 🗓️ Diário de Construção e Evidências

O provisionamento da infraestrutura foi dividido em 4 fases lógicas, documentadas abaixo com o desenho da arquitetura e as evidências de configuração e execução.

### 📘 Dia 1: Ingestão de Pedidos via API e EventBridge
**Objetivo:** Criar o fluxo inicial para a ingestão de pedidos via endpoint de API REST. Os dados passam por uma Lambda de pré-validação, são enfileirados em uma SQS FIFO para garantir a ordem, e processados por uma segunda Lambda que publica um evento em um barramento customizado no EventBridge.

![Arquitetura Dia 1](imagens/dia1/arquitetura_dia1.png)

* **Infraestrutura de Mensageria e Orquestração:**
  * ![SQS FIFO](imagens/dia1/01_sqs_filas_fifo.png)
  * ![EventBridge Custom Bus](imagens/dia1/05_eventbridge_custom_bus.png)
* **Segurança (Princípio do Menor Privilégio):**
  * ![IAM Role Pré-validação](imagens/dia1/02_iam_role_pre_validacao.png)
  * ![IAM Role Validação](imagens/dia1/06_iam_role_validacao_pedidos.png)
* **Computação e API:**
  * ![Lambda Pre-validação Env](imagens/dia1/03_lambda_pre_validacao_env.png)
  * ![API Gateway](imagens/dia1/04_api_gateway_endpoint.png)
  * ![Lambda Trigger](imagens/dia1/07_lambda_validacao_trigger_2.png)
* **Prova de Execução:**
  * ![CloudWatch Logs](imagens/dia1/08_cloudwatch_logs_dia1.png)

---

### 📗 Dia 2: Ingestão de Arquivos via S3 e Rastreamento
**Objetivo:** Implementar um canal alternativo para processamento em lote. Arquivos JSON no S3 disparam notificações para uma SQS Standard. Uma Lambda valida o conteúdo, envia pedidos válidos para o pipeline principal (FIFO) e registra o histórico no DynamoDB. Erros disparam notificações via SNS.

![Arquitetura Dia 2](imagens/dia2/arquitetura_dia2.png)

* **Mensageria Standard, Auditoria e Alertas:**
  * ![SQS Standard](imagens/dia2/09_sqs_filas_standard.png)
  * ![DynamoDB Histórico](imagens/dia2/10_dynamodb_tabela_historico.png)
  * ![SNS Topico](imagens/dia2/11_sns_topico_alertas.png)
* **Segurança e Orquestração S3:**
  * ![IAM Role Validação S3](imagens/dia2/12_iam_role_validacao_s3.png)
  * ![S3 Event Notification](imagens/dia2/14_s3_event_notification.png)
  * ![Lambda S3 Trigger](imagens/dia2/13_lambda_validacao_s3_trigger.png)
* **Prova de Execução e Auditoria:**
  * ![DynamoDB Itens Histórico](imagens/dia2/15_dynamodb_itens_historico.png)

---

### 📙 Dia 3: Processamento Central de Pedidos e Persistência
**Objetivo:** Construir a lógica que consome os eventos unificados. Uma regra no EventBridge captura eventos de `NovoPedidoValidado` e os direciona a uma nova fila SQS. Uma Lambda simula o processamento do pedido e persiste o estado na tabela principal do DynamoDB.

![Arquitetura Dia 3](imagens/dia3/arquitetura_dia3.png)

* **Buffer Central e Base de Dados Principal:**
  * ![SQS Pendentes](imagens/dia3/16_sqs_filas_pendentes.png)
  * ![DynamoDB Pedidos](imagens/dia3/17_dynamodb_tabela_pedidos.png)
* **Segurança, Roteamento e Computação:**
  * ![IAM Role Processamento](imagens/dia3/18_iam_role_processa_pedidos.png)
  * ![EventBridge Rule Novo Pedido](imagens/dia3/20_eventbridge_regra_novo_pedido.png)
  * ![Lambda Processa Pedidos](imagens/dia3/19_lambda_processa_pedidos.png)
* **Prova de Execução (Omnicanalidade):**
  * ![DynamoDB Processados API e S3](imagens/dia3/21_dynamodb_pedidos_processados.png)

---

### 📕 Dia 4: Ciclo de Vida e Resiliência (Tolerância a Falhas)
**Objetivo:** Expandir a funcionalidade para lidar com cancelamentos e alterações de pedidos utilizando o EventBridge e SQS para roteamento. Teste prático de stress para demonstrar o funcionamento de uma Dead Letter Queue (DLQ) capturando falhas no sistema.

![Arquitetura Dia 4](imagens/dia4/arquitetura_dia4.png)

* **Novas Operações (Altera e Cancela):**
  * ![SQS Operações](imagens/dia4/22_sqs_filas_operacoes.png)
  * ![IAM Role Operações](imagens/dia4/23_iam_role_altera_cancela.png)
  * ![EventBridge Regras Operações](imagens/dia4/25_eventbridge_regras_operacoes.png)
  * ![Lambdas Operações](imagens/dia4/24_lambdas_operacoes.png)
* **Simulações e Prova de Modificação:**
  * ![Simula Cancela](imagens/dia4/26_eventbridge_simula_cancela.png)
  * ![Log Cancela](imagens/dia4/27_cloudwatch_log_cancela.png)
  * ![DynamoDB Cancelado](imagens/dia4/28_dynamodb_pedido_cancelado.png)
  * ![Simula Altera](imagens/dia4/29_eventbridge_simula_altera.png)
  * ![Log Altera](imagens/dia4/30_cloudwatch_log_altera.png)
  * ![DynamoDB Alterado](imagens/dia4/31_dynamodb_pedido_alterado.png)
* **Resumo de Resiliência (DLQ em Ação):**
  * ![Log Erro Forçado DLQ](imagens/dia4/32_cloudwatch_log_erro_dlq.png)
  * ![DLQ Retenção](imagens/dia4/33_sqs_dlq_retencao.png)
