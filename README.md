# Checkout E-commerce (Camunda 8 + Spring Boot)

## 🎯 Objetivo

Prova de conceito (POC) de um orquestrador de pedidos utilizando **Camunda 8 (Zeebe)** e **Spring Boot 3**. 
Demonstra o padrão de **Processamento Assíncrono** com Wait States (espera de callback externo).

## 🏗️ Arquitetura (V1 - Local)

| Componente | Tecnologia |
|------------|------------|
| Linguagem | Java 21 |
| Framework | Spring Boot 3.3+ |
| Engine | Camunda 8 Run (Self-Managed Local) |

### Comunicação
- **App → Zeebe:** gRPC (Porta 26500)
- **Cliente → App:** REST API (Porta 8080)
- **Operate (Dashboard):** HTTP (Porta 8081 ou 8080/operate)

## 🔄 Fluxo do Processo (BPMN)

**ID do Processo:** `processo-pagamento-v1`

```
[Start] → [solicitar-pagamento] → [⏸️ msg-pagamento-confirmado] → [End]
```

1. **Start Event:** Pedido recebido via API
2. **Service Task (`solicitar-pagamento`):** Worker Java processa validação inicial e simula envio para adquirente
3. **Intermediate Catch Event (`msg-pagamento-confirmado`):** O processo PARA e aguarda mensagem externa (Correlation Key: `orderId`)
4. **End Event:** Pedido Concluído

## 🔌 API Endpoints

### 1. Criar Pedido (Inicia o Processo)

```http
POST /pedidos
Content-Type: application/json

{
  "produto": "PC Gamer",
  "valor": 5000.00
}
```

**Resposta (200 OK):**
```json
{
  "orderId": "123e4567-e89b-12d3-a456-426614174000",
  "status": "PROCESSANDO"
}
```

### 2. Webhook de Pagamento (Destrava o Processo)

```http
POST /pedidos/{orderId}/pagamento
Content-Type: application/json

{
  "status": "APROVADO"
}
```

**Ação:** Envia comando `publishMessage` para o Zeebe correlacionando pelo `orderId`.

## 🚀 Como Executar

### Pré-requisitos
- Java 21+
- Maven 3.8+
- Camunda 8 Run (local) rodando na porta 26500

### Executar a aplicação

```bash
./mvnw spring-boot:run
```

## 📁 Estrutura do Projeto

```
src/main/java/com/vitorindio/ecommerce_camunda/
├── EcommerceCamundaApplication.java
├── controller/
│   └── PedidoController.java
├── dto/
│   ├── PedidoRequest.java
│   └── PedidoResponse.java
├── service/
│   └── PedidoService.java
└── worker/
    └── SolicitarPagamentoWorker.java
    
src/main/resources/
├── application.properties
└── processo-pagamento-v1.bpmn
```

## 📝 Licença

Este projeto é uma POC para fins educacionais.

