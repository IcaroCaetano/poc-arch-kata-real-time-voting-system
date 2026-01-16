# 🧬 Sistema de Votação em Tempo Real

Leitura de Arquitetura para Apresentação ao Time

## 1️⃣ Contexto e Problema

Estamos projetando um sistema de votação em tempo real que precisa atender requisitos extremamente rígidos:

- Não podemos perder votos

- Precisamos evitar bots e ataques

- O sistema deve suportar centenas de milhões de usuários

- Precisamos lidar com picos de até 250 mil votos por segundo

- Cada usuário só pode votar uma única vez

- Os resultados precisam ser exibidos em tempo real

- Além disso, existem restrições importantes:

- Nada de serverless

- Nada de soluções monolíticas

- Precisamos de uma arquitetura distribuída, resiliente e auditável

## 2️⃣ Abordagem Arquitetural

Para resolver esse problema, adotamos quatro decisões centrais:

- Arquitetura orientada a eventos

- Microsserviços

- CQRS para separar leitura e escrita

- Processamento em streaming com garantias de exactly-once

- Essas decisões nos permitem escalar, isolar falhas e garantir consistência sem sacrificar tempo real.

## 3️⃣ Visão Geral da Arquitetura

O sistema é dividido em três grandes fluxos:

- Entrada de votos (Write Path)

- Processamento e contagem em tempo real

- Leitura e entrega dos resultados (Read Path)

- O DynamoDB é a fonte da verdade.
  
- O Kafka é o coração do sistema.
  
- O Kafka Streams é responsável por garantir que nenhum voto seja contado duas vezes.

## 4️⃣ Fluxo de Votação (Passo a Passo)

Vamos acompanhar o caminho de um voto:

1. O usuário acessa o sistema através de uma camada de CDN + WAF, que protege contra bots e ataques.

2. A requisição passa pelo API Gateway, que valida autenticação e regras básicas.

3. O Vote Ingestion Service recebe o voto.

  - Ele não conta votos

  - Ele apenas valida o formato e publica um evento

4. O voto é publicado no Kafka, particionado por ID do usuário.

5. O Kafka Streams consome esse evento e:

  - Verifica se o usuário já votou usando State Stores

  - Garante semântica exactly-once

  - Conta o voto

6. O voto é persistido no DynamoDB, que funciona como source of truth.

7. A contagem parcial é atualizada no Redis, que é o Read Model.

8. O usuário recebe a atualização via WebSocket ou SSE, em tempo real.

## 5️⃣ Garantia de “Um Voto por Usuário”

Essa é uma das partes mais críticas da arquitetura.

A garantia acontece em camadas:

- Kafka Streams State Store:

    - Mantém um registro local dos usuários que já votaram

    - Evita dupla contagem em tempo real

- Tópicos particionados por userId:

    - Garante ordenação e consistência por usuário

- DynamoDB:

    - Mantém o histórico completo e imutável

    - Permite auditoria e validações raras do tipo “o usuário já votou?”

Essa combinação evita race conditions e elimina contagens duplicadas.

## 6️⃣ Leitura em Tempo Real vs Consistência Final

Aceitamos conscientemente um trade-off:

- Durante a votação:

    - Redis mostra resultados quase em tempo real

- No encerramento da votação:

    - Um job agendado executa uma recontagem completa no DynamoDB

    - O Redis é atualizado com o resultado final e oficial

Isso nos dá:

- Baixa latência durante a votação

- Consistência forte no resultado final

## 7️⃣ Casos de Uso Atendidos

*Usuário*

- Autenticar

- Votar

- Receber confirmação

- Acompanhar resultados em tempo real

*Sistema*

- Validar votos

- Deduplicar

- Contar votos

- Persistir eventos

- Atualizar projeções

*Administrador*

- Criar eleições

- Abrir e encerrar votações

- Monitorar métricas

*Auditoria*

- Auditar votos

- Reprocessar eventos

- Garantir integridade do resultado

## 8️⃣ Por que essa Arquitetura Funciona

Essa arquitetura funciona porque:

- Eventos são imutáveis

- Falhas são recuperáveis

- Leitura e escrita escalam de forma independente

- Nenhum voto é perdido

- Exactly-once é garantido

- Auditoria é nativa do sistema

Ela foi pensada para escala real, não apenas para funcionar em laboratório.

## 9️⃣ Conclusão

Em resumo:

- Kafka é o backbone

- DynamoDB é a fonte da verdade

- Kafka Streams garante integridade e deduplicação

- Redis entrega velocidade

- O job final garante confiança no resultado

Essa arquitetura nos permite operar com segurança, escala, tempo real e consistência, mesmo sob picos extremos de carga.
