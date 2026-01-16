# 🧠 O que é CQRS (Command Query Responsibility Segregation)?

CQRS significa Separação de Responsabilidades entre Comandos e Consultas.

Em termos simples:

Escrever dados e ler dados são problemas diferentes — então usamos modelos diferentes para cada um.

## 🧩 Sem CQRS (modelo tradicional)

Normalmente temos:

Um único serviço

Um único modelo de dados

O mesmo banco serve para:

Escritas (INSERT / UPDATE)

Leituras (SELECT)

Problemas disso em sistemas grandes:

Escrita pesada afeta leitura

Leitura pesada afeta escrita

Modelo fica complexo

Escalar fica caro e difícil

## ✅ Com CQRS

Separarmos o sistema em dois lados independentes:

✍️ Command Side (Write Side)

Responsável por:

Receber comandos (ex: enviar voto)

Validar regras de negócio

Persistir eventos/dados

Características:

Otimizado para alta taxa de escrita

Modelo simples

Consistência é prioridade

## 👀 Query Side (Read Side)

Responsável por:

Responder consultas (ex: resultado da votação)

Fornecer dados já prontos para leitura

Características:

Otimizado para leituras rápidas

Pode usar cache

Pode ser eventualmente consistente

##🧬 CQRS aplicado ao nosso sistema de votação
Write Side

Vote Ingestion Service

Kafka

Kafka Streams

DynamoDB

Objetivo:

Nunca perder votos

Garantir exatamente um voto por usuário

Garantir integridade

Read Side

Redis

WebSocket / SSE

Objetivo:

Mostrar resultados em tempo real

Responder rápido

Suportar milhões de usuários lendo ao mesmo tempo

## 📊 Visual simplificado
          WRITE SIDE (Commands)
Usuário ───▶ API ───▶ Kafka ───▶ DynamoDB
                        │
                        ▼
          READ SIDE (Queries)
                   Redis ───▶ WebSocket ───▶ Usuário

🔥 Por que CQRS é essencial aqui?
1️⃣ Escala independentemente

Escrita: 250k votos/s

Leitura: milhões de usuários acompanhando

Sem CQRS, um derruba o outro.

2️⃣ Modelos de dados diferentes

Write:

Estrutura simples

Append-only

Read:

Dados agregados

Contadores prontos

3️⃣ Performance

Escrita não depende de cache

Leitura não depende de joins ou cálculos

4️⃣ Resiliência

Se o Redis cair:

Escrita continua funcionando

Se o Write estiver lento:

Últimos dados ainda estão no cache

⚠️ Trade-off importante (sempre explique)

CQRS não garante leitura 100% consistente o tempo todo.

No nosso caso:

Durante a votação → eventual consistency

Ao final → consistência forte (job de recontagem)

👉 Isso é uma decisão consciente, não um erro.

🗣️ Explicação curta para o time (30 segundos)

“Usamos CQRS porque escrever votos e ler resultados são problemas completamente diferentes. O write side é otimizado para não perder dados e garantir integridade. O read side é otimizado para responder rápido e escalar. Isso nos permite suportar picos enormes sem comprometer consistência nem tempo real.”
