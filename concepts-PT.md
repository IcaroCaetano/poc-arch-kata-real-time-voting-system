# 🧠 O que é CQRS (Command Query Responsibility Segregation)?

CQRS significa Separação de Responsabilidades entre Comandos e Consultas.

Em termos simples:

Escrever dados e ler dados são problemas diferentes — então usamos modelos diferentes para cada um.

## 🧩 Sem CQRS (modelo tradicional)

Normalmente temos:

- Um único serviço

- Um único modelo de dados

- O mesmo banco serve para:

  - Escritas (INSERT / UPDATE)

  - Leituras (SELECT)

*Problemas disso em sistemas grandes:*

- Escrita pesada afeta leitura

- Leitura pesada afeta escrita

- Modelo fica complexo

- Escalar fica caro e difícil

## ✅ Com CQRS

Separarmos o sistema em dois lados independentes:

✍️ *Command Side (Write Side)*

Responsável por:

- Receber comandos (ex: enviar voto)

- Validar regras de negócio

- Persistir eventos/dados

Características:

- Otimizado para alta taxa de escrita

- Modelo simples

- Consistência é prioridade

## 👀 Query Side (Read Side)

Responsável por:

- Responder consultas (ex: resultado da votação)

- Fornecer dados já prontos para leitura

Características:

- Otimizado para leituras rápidas

- Pode usar cache

- Pode ser eventualmente consistente

## 🧬 CQRS aplicado ao nosso sistema de votação
Write Side

- Vote Ingestion Service

- Kafka

- Kafka Streams

- DynamoDB

Objetivo:

- Nunca perder votos

- Garantir exatamente um voto por usuário

- Garantir integridade

Read Side

- Redis

- WebSocket / SSE

Objetivo:

- Mostrar resultados em tempo real

- Responder rápido

- Suportar milhões de usuários lendo ao mesmo tempo

## 📊 Visual simplificado

````text
          WRITE SIDE (Commands)
Usuário ───▶ API ───▶ Kafka ───▶ DynamoDB
                        │
                        ▼
          READ SIDE (Queries)
                   Redis ───▶ WebSocket ───▶ Usuário
````

## 🔥 Por que CQRS é essencial aqui?
### 1️⃣ Escala independentemente

- Escrita: 250k votos/s

- Leitura: milhões de usuários acompanhando

Sem CQRS, um derruba o outro.

### 2️⃣ Modelos de dados diferentes

Write:

Estrutura simples

Append-only

Read:

- Dados agregados

- Contadores prontos

### 3️⃣ Performance

- Escrita não depende de cache

- Leitura não depende de joins ou cálculos

### 4️⃣ Resiliência

Se o Redis cair:

- Escrita continua funcionando

Se o Write estiver lento:

- Últimos dados ainda estão no cache

## ⚠️ Trade-off importante (sempre explique)

CQRS não garante leitura 100% consistente o tempo todo.

No nosso caso:

- Durante a votação → eventual consistency

- Ao final → consistência forte (job de recontagem)

👉 Isso é uma decisão consciente, não um erro.

## 🗣️ Explicação curta para o time (30 segundos)

“Usamos CQRS porque escrever votos e ler resultados são problemas completamente diferentes. O write side é otimizado para não perder dados e garantir integridade. O read side é otimizado para responder rápido e escalar. Isso nos permite suportar picos enormes sem comprometer consistência nem tempo real.”

--
# 🧠 O que significa Exactly-Once Semantics?

*Exactly-once significa:*

*Cada evento* é processado uma única vez, nem zero, nem duas — exatamente uma.

Ou seja:

- O evento não pode ser perdido

- O evento não pode ser processado duas vezes

- Mesmo com:

  - falhas

  - retries

  - crashes

  - rede instável

## 🔁 O problema real que o exactly-once resolve

Em sistemas distribuídos, falhas são normais.

Exemplo sem exactly-once:

1. Um voto chega

2. O sistema processa

3. O sistema cai antes de confirmar

4. O evento é reenviado

5. O voto é contado duas vezes

Ou o inverso:

- O evento some → voto perdido

### 📌 As três semânticas clássicas

### 1️⃣ At-most-once

- Processa no máximo uma vez

- Pode perder eventos

### ❌ Inaceitável para votação

### 2️⃣ At-least-once

- Processa uma ou mais vezes

- Não perde eventos

### ❌ Pode contar voto duplicado

### 3️⃣ Exactly-once ✅

- Processa exatamente uma vez

- Não perde

- Não duplica

- ✔️ Obrigatório para votação

## 🧬 Exactly-once no seu sistema de votação

No seu desenho, exactly-once é garantido em camadas, não por mágica.

### 1️⃣ Kafka como base

Kafka:

  - Persiste eventos

  - Mantém ordenação por partição

  - Permite replay

Mas Kafka sozinho não garante exactly-once.

## 2️⃣ Kafka Streams (EOS v2)

Kafka Streams oferece Exactly-Once Semantics (EOS):

Ele garante que:

  - Ler evento

  - Atualizar State Store

  - Produzir resultado

  - Commit de offsets

👉 Tudo acontece de forma atômica.

Se o processo cair:

  - Ou tudo foi aplicado

  - Ou nada foi aplicado

### 3️⃣ State Stores para deduplicação

As State Stores:

  - Guardam o estado local do processamento

  - Registram quais userId já votaram

Quando um voto chega:

  - Se o usuário já existe na store → rejeita

  - Se não existe → processa e grava

Isso evita:

  - Retries

  - Eventos duplicados

  - Replays acidentais

### 4️⃣ Particionamento por userId

Ao particionar por userId:

  - Todos os votos do mesmo usuário vão para a mesma partição

  - Mantemos ordem garantida

  - Eliminamos race conditions

### 5️⃣ Escrita consistente no DynamoDB

O processamento só confirma o offset:

  - Depois de atualizar a store

  - Depois de persistir o voto

  - Depois de atualizar a contagem

Isso fecha o ciclo do exactly-once.

## 📊 Visão resumida do fluxo

````text
Evento chega
   ↓
Kafka Streams lê
   ↓
Verifica State Store
   ↓
Atualiza contagem
   ↓
Persiste no DynamoDB
   ↓
Commit atômico
````

Se algo falhar → rollback automático.

## ⚠️ Ponto importante (sempre explique)

Exactly-once NÃO significa:

- Zero latência

- Zero complexidade

- Zero custo

Significa:

- Mais controle

- Mais estado

Mais disciplina arquitetural

## 🗣️ Explicação curta para o time (20–30 segundos)

“Exactly-once significa que cada voto será processado uma única vez, mesmo se houver falhas, retries ou quedas de serviço. Usamos Kafka Streams com state stores e particionamento por usuário para garantir que nenhum voto seja perdido e nenhum seja contado duas vezes.”

## 🧠 Frase de ouro para banca / revisão técnica

“Exactly-once não é uma feature isolada, é uma propriedade emergente da combinação entre processamento transacional, estado local e controle de offsets.”
