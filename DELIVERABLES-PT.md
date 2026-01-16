# 📦 Entregáveis - Sistema de Votação em Tempo Real

Este documento consolida todos os entregáveis gerados para a **Seção 4.1 (Overall Architecture)** e **Seção 4.3 (Use Cases)** do projeto.

---

## 📋 Índice de Entregáveis

### 1️⃣ Diagrama de Arquitetura Geral (Overall Architecture)

#### 📄 Documentação Completa
- **Arquivo**: [`overall-architecture-diagram-PT.md`](./overall-architecture-diagram-PT.md)
- **Conteúdo**:
  - Diagrama Mermaid interativo
  - Decisões arquiteturais chave
  - Detalhamento de componentes
  - Fluxo de dados (Write Path, Read Path, Auditoria)
  - Garantias do sistema
  - Estratégias de escalabilidade

#### 🖼️ Imagens do Diagrama
- **PNG (Alta resolução)**: [`overall-architecture-diagram-PT.png`](./overall-architecture-diagram-PT.png) - 236 KB
  - Formato raster, ideal para apresentações e documentos
- **SVG (Vetorial)**: [`overall-architecture-diagram-PT.svg`](./overall-architecture-diagram-PT.svg) - 44 KB
  - Formato vetorial, escalável sem perda de qualidade

**Preview do Diagrama:**
![Diagrama de Arquitetura](./overall-architecture-diagram-PT.png)

---

### 2️⃣ Diagrama de Casos de Uso (Use Cases)

#### 📄 Documentação Completa
- **Arquivo**: [`use-case-diagram.md`](./use-case-diagram.md)
- **Conteúdo**:
  - Diagrama Mermaid com 21 casos de uso
  - Detalhamento completo de cada caso de uso
  - 4 atores identificados (Usuário, Sistema, Administrador, Auditor)
  - Fluxos principais e alternativos
  - Pré-condições e pós-condições
  - Relações e dependências
  - Requisitos não funcionais
  - Priorização (MVP, Fase 2, Fase 3)

#### 🖼️ Imagens do Diagrama
- **PNG (Alta resolução)**: [`use-case-diagram.png`](./use-case-diagram.png) - 301 KB
  - Formato raster, ideal para apresentações e documentos
- **SVG (Vetorial)**: [`use-case-diagram.svg`](./use-case-diagram.svg) - 60 KB
  - Formato vetorial, escalável sem perda de qualidade

**Preview do Diagrama:**
![Diagrama de Casos de Uso](./use-case-diagram.png)

---

## 🎯 Resumo dos Casos de Uso

### 👤 Usuário (4 casos de uso)
1. **UC-01**: Autenticar no Sistema
2. **UC-02**: Submeter Voto
3. **UC-03**: Receber Confirmação de Voto
4. **UC-04**: Visualizar Resultados em Tempo Real

### 🤖 Sistema Automatizado (6 casos de uso)
5. **UC-05**: Validar Voto
6. **UC-06**: Deduplicar Votos
7. **UC-07**: Contar Votos
8. **UC-08**: Persistir Eventos
9. **UC-09**: Atualizar Projeções de Leitura
10. **UC-10**: Detectar e Bloquear Bots

### 👨‍💼 Administrador (6 casos de uso)
11. **UC-11**: Criar Eleição
12. **UC-12**: Configurar Opções de Voto
13. **UC-13**: Abrir Votação
14. **UC-14**: Encerrar Votação
15. **UC-15**: Monitorar Métricas do Sistema
16. **UC-16**: Visualizar Dashboard Administrativo

### 🔍 Auditor (5 casos de uso)
17. **UC-17**: Auditar Votos Registrados
18. **UC-18**: Reprocessar Eventos
19. **UC-19**: Executar Recontagem Final
20. **UC-20**: Validar Integridade dos Resultados
21. **UC-21**: Gerar Relatório de Auditoria

---

## 🏗️ Componentes Principais da Arquitetura

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

---

## ✅ Garantias do Sistema

- ✅ **Nenhum voto é perdido** - DynamoDB + Kafka com persistência
- ✅ **Um voto por usuário** - State Store + particionamento por userId
- ✅ **Exactly-once processing** - Kafka Streams com transações
- ✅ **Tempo real** - Redis + WebSocket
- ✅ **Auditável** - Eventos imutáveis no DynamoDB
- ✅ **Resiliente** - Arquitetura distribuída com failover
- ✅ **Escalável** - Suporta 250k votos/segundo em picos

---

## 📊 Decisões Arquiteturais

### 1. Arquitetura Orientada a Eventos
- Kafka como backbone central
- Eventos imutáveis e auditáveis
- Desacoplamento entre produtores e consumidores

### 2. CQRS (Command Query Responsibility Segregation)
- **Write Path**: DynamoDB como source of truth
- **Read Path**: Redis como read model otimizado
- Escalabilidade independente

### 3. Exactly-Once Processing
- Kafka Streams com semântica exactly-once
- State Stores para deduplicação em tempo real
- Particionamento por userId garante ordenação

### 4. Consistência Eventual + Reconciliação
- Redis fornece resultados em tempo real
- DynamoDB mantém histórico completo
- Job agendado executa recontagem final

---

## 📁 Estrutura de Arquivos

```
.
├── DELIVERABLES.md                      # Este arquivo (índice de entregáveis)
├── overall-architecture-diagram.md       # Documentação completa da arquitetura
├── overall-architecture-diagram.png      # Diagrama de arquitetura (PNG)
├── overall-architecture-diagram.svg      # Diagrama de arquitetura (SVG)
├── use-case-diagram.md                   # Documentação completa dos casos de uso
├── use-case-diagram.png                  # Diagrama de casos de uso (PNG)
├── use-case-diagram.svg                  # Diagrama de casos de uso (SVG)
├── overall-diagrams-PT.md                # Documento fonte com análise arquitetural
└── plan.md                               # Plano de pesquisa e entregáveis
```

---

## 🚀 Como Utilizar os Entregáveis

### Para Apresentações
- Use os arquivos **PNG** para incluir em slides (PowerPoint, Google Slides, Keynote)
- Os diagramas têm alta resolução e fundo transparente

### Para Documentação Técnica
- Use os arquivos **SVG** para inclusão em documentos web ou wikis
- Formato vetorial permite zoom sem perda de qualidade

### Para Revisão e Edição
- Os arquivos **.md** contêm o código Mermaid original
- Podem ser editados e re-renderizados conforme necessário

### Para Versionamento
- Todos os arquivos markdown podem ser versionados no Git
- Diagramas Mermaid são tratados como código-fonte

---

## 🔗 Referências

- **Documento Base**: [`overall-diagrams-PT.md`](./overall-diagrams-PT.md)
- **Plano de Pesquisa**: [`plan.md`](./plan.md)
- **Conceitos (PT)**: [`concepts-PT.md`](./concepts-PT.md)
- **Conceitos (EN)**: [`concepts-EN.md`](./concepts-EN.md)

---

## 📝 Notas

- Todos os diagramas foram gerados usando **Mermaid** via **Kroki API**
- Os diagramas podem ser facilmente atualizados editando os blocos Mermaid nos arquivos .md
- Para re-gerar as imagens, use a API Kroki ou ferramentas locais como mermaid-cli

---

**Gerado em**: 16 de Janeiro de 2026
**Responsável**: Person 2 - High-Level Architecture & Use Cases
**Status**: ✅ Completo
