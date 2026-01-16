# Diagrama de Arquitetura Geral
## Sistema de Votação em Tempo Real

```mermaid
graph TB
    subgraph "Camada de Entrada"
        User[👤 Usuário]
        CDN[CDN + WAF<br/>Proteção contra Bots]
        Gateway[API Gateway<br/>Autenticação & Validação]
    end

    subgraph "Write Path - Ingestão de Votos"
        VoteIngestion[Vote Ingestion Service<br/>Validação de Formato]
    end

    subgraph "Kafka Event Backbone"
        KafkaTopic[Kafka Topic<br/>Particionado por userId]
        KafkaStreams[Kafka Streams<br/>Exactly-Once Processing]
        StateStore[(State Store<br/>Deduplicação)]
    end

    subgraph "Persistência"
        DynamoDB[(DynamoDB<br/>Source of Truth<br/>Histórico Imutável)]
    end

    subgraph "Read Path - Resultados em Tempo Real"
        Redis[(Redis<br/>Read Model<br/>Contagem Parcial)]
        ResultService[Result Service<br/>Agregação de Resultados]
        WebSocket[WebSocket/SSE<br/>Push em Tempo Real]
    end

    subgraph "Administração e Auditoria"
        Admin[Admin Service<br/>Gestão de Eleições]
        Audit[Audit Service<br/>Recontagem & Validação]
        ScheduledJob[Scheduled Job<br/>Recontagem Final]
    end

    subgraph "Observabilidade"
        Metrics[Métricas & Logs<br/>Monitoring]
    end

    %% Fluxo de Votação
    User -->|1. Requisição de Voto| CDN
    CDN -->|2. Filtro de Segurança| Gateway
    Gateway -->|3. Voto Validado| VoteIngestion
    VoteIngestion -->|4. Publica Evento| KafkaTopic

    %% Processamento em Streaming
    KafkaTopic -->|5. Consome Evento| KafkaStreams
    KafkaStreams <-->|6. Verifica Duplicação| StateStore
    KafkaStreams -->|7. Persiste Voto| DynamoDB
    KafkaStreams -->|8. Atualiza Contagem| Redis

    %% Retorno em Tempo Real
    Redis -->|9. Busca Resultado| ResultService
    ResultService -->|10. Push Update| WebSocket
    WebSocket -->|11. Resultado em Tempo Real| User

    %% Administração
    Admin -->|Gerencia| DynamoDB
    Admin -->|Controla| KafkaTopic

    %% Auditoria
    DynamoDB -->|Reprocessamento| Audit
    ScheduledJob -->|Recontagem Final| DynamoDB
    ScheduledJob -->|Atualiza Resultado Oficial| Redis

    %% Observabilidade
    VoteIngestion -.->|Logs| Metrics
    KafkaStreams -.->|Métricas| Metrics
    ResultService -.->|Monitoramento| Metrics

    style DynamoDB fill:#4DB33D,stroke:#2E7D32,color:#fff
    style Redis fill:#DC382D,stroke:#B71C1C,color:#fff
    style KafkaTopic fill:#231F20,stroke:#000,color:#fff
    style KafkaStreams fill:#231F20,stroke:#000,color:#fff
    style User fill:#2196F3,stroke:#1565C0,color:#fff
```

## Decisões Arquiteturais Chave

### 1️⃣ Arquitetura Orientada a Eventos
- **Kafka** como backbone central
- Eventos imutáveis e auditáveis
- Desacoplamento entre produtores e consumidores

### 2️⃣ CQRS (Command Query Responsibility Segregation)
- **Write Path**: DynamoDB como source of truth
- **Read Path**: Redis como read model otimizado
- Escalabilidade independente de leitura e escrita

### 3️⃣ Exactly-Once Processing
- **Kafka Streams** com semântica exactly-once
- **State Stores** para deduplicação em tempo real
- Particionamento por userId garante ordenação

### 4️⃣ Consistência Eventual + Reconciliação
- Redis fornece resultados em tempo real (baixa latência)
- DynamoDB mantém histórico completo e imutável
- Job agendado executa recontagem final para resultado oficial

## Componentes Principais

| Componente | Responsabilidade | Tecnologia |
|------------|------------------|------------|
| **CDN + WAF** | Proteção contra bots e DDoS | CloudFlare / AWS CloudFront |
| **API Gateway** | Autenticação, rate limiting | Kong / AWS API Gateway |
| **Vote Ingestion Service** | Validação e publicação de eventos | Java / Spring Boot |
| **Kafka** | Event streaming e persistência de eventos | Apache Kafka |
| **Kafka Streams** | Processamento e deduplicação | Kafka Streams API |
| **DynamoDB** | Source of truth, histórico imutável | AWS DynamoDB |
| **Redis** | Read model, contagem em tempo real | Redis Cluster |
| **Result Service** | Agregação e entrega de resultados | Java / Spring Boot |
| **WebSocket/SSE** | Push de atualizações em tempo real | Spring WebFlux |
| **Admin Service** | Gestão de eleições e configurações | Java / Spring Boot |
| **Audit Service** | Auditoria e recontagem | Java / Spring Batch |

## Fluxo de Dados Detalhado

### Caminho de Escrita (Write Path)
1. **Usuário** submete voto através do frontend
2. **CDN + WAF** filtra requisições maliciosas
3. **API Gateway** valida autenticação e rate limits
4. **Vote Ingestion Service** valida formato do voto
5. **Kafka** recebe evento de voto (particionado por userId)
6. **Kafka Streams** processa evento:
   - Verifica no State Store se usuário já votou
   - Aplica lógica de deduplicação
   - Garante exactly-once semantics
7. **DynamoDB** persiste voto de forma imutável
8. **Redis** é atualizado com contagem parcial

### Caminho de Leitura (Read Path)
1. **Usuário** solicita resultados
2. **Result Service** busca dados agregados no Redis
3. **WebSocket/SSE** envia atualizações em tempo real
4. **Frontend** exibe resultados com baixa latência

### Caminho de Auditoria
1. **Scheduled Job** executa recontagem completa no DynamoDB
2. **Audit Service** valida integridade dos dados
3. **Redis** é atualizado com resultado oficial
4. **Logs e métricas** são armazenados para compliance

## Garantias do Sistema

✅ **Nenhum voto é perdido** - DynamoDB + Kafka com persistência

✅ **Um voto por usuário** - State Store + particionamento por userId

✅ **Exactly-once processing** - Kafka Streams com transações

✅ **Tempo real** - Redis + WebSocket

✅ **Auditável** - Eventos imutáveis no DynamoDB

✅ **Resiliente** - Arquitetura distribuída com failover

✅ **Escalável** - Suporta 250k votos/segundo em picos

## Escalabilidade

- **Kafka**: Particionamento horizontal (100+ partições)
- **DynamoDB**: Auto-scaling com provisioned capacity
- **Redis**: Cluster mode com sharding
- **Microservices**: Instâncias replicadas com load balancer
- **Kafka Streams**: Paralelização automática por partição
