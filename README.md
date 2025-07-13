# Sistema de Venda de Ingressos

## Visão Geral

Este projeto apresenta a arquitetura de um sistema de venda de ingressos projetado para lidar com alta concorrência e garantir fairness no processo de compra. O sistema foi desenvolvido para situações como a venda de ingressos para mega shows, onde a demanda é muito superior à oferta disponível.

## Problema Resolvido

O sistema resolve dois problemas críticos:

1. **Fairness (Justiça no Acesso)**: Garantir que clientes com conexões mais lentas tenham as mesmas oportunidades de compra que aqueles com conexões mais rápidas
2. **Consistência de Inventário**: Evitar overselling, garantindo que apenas ingressos realmente disponíveis sejam vendidos

## Arquitetura UML - Casos de Uso

O diagrama UML de casos de uso segue o padrão da imagem de referência, mas adaptado para o sistema de venda de ingressos. O diagrama mostra:

### Atores

- **Cliente**: Usuário final que quer comprar ingressos
- **Administrador**: Gerencia o sistema e eventos
- **Sistema de Pagamento**: Ator externo que processa pagamentos

### Casos de Uso Principais

#### Para o Cliente:
- Entrar na fila de espera
- Verificar posição na fila
- Comprar ingresso

#### Para o Administrador:
- Consultar inventário
- Gerenciar eventos
- Monitorar sistema

#### Para o Sistema de Pagamento:
- Validar transação
- Notificar pagamento

### Relacionamentos

#### «include» (sempre acontece):
- Comprar ingresso → Reservar ingresso
- Reservar ingresso → Processar pagamento
- Processar pagamento → Confirmar compra

#### «extend» (pode acontecer):
- Verificar posição ← Entrar na fila
- Notificar pagamento ← Validar transação

## Principais Soluções Implementadas

### 1. Fairness (Justiça no Acesso)
- **Fila FIFO com Timestamp**: Todos os usuários entram numa fila baseada na ordem de chegada, não na velocidade da conexão
- **Rate Limiting**: Previne que usuários com conexões mais rápidas façam múltiplas requisições
- **Token de Posição**: Informa a posição na fila e tempo estimado

### 2. Consistência de Inventário
- **Pessimistic Locking**: Quando um usuário inicia a compra, o ingresso fica reservado temporariamente
- **2-Phase Commit**: Garante que pagamento e confirmação sejam atômicos
- **Cache Distribuído**: Contador de ingressos em Redis para verificação rápida

### 3. Escalabilidade
- **Load Balancer**: Distribui carga entre múltiplas instâncias
- **Queue Processor**: Controla o fluxo de liberação de usuários para compra
- **API Gateway**: Valida tokens e controla acesso aos serviços

### 4. Monitoramento
- Acompanha tamanho da fila, tempo de processamento e taxa de sucesso

## Componentes Chave

### Sistema de Fila
- **QueueManager**: Gerencia a fila FIFO com timestamps
- **QueueProcessor**: Processa usuários da fila de forma controlada
- **Position Token**: Fornece feedback sobre posição na fila

### Serviços de Negócio
- **TicketService**: Gerencia inventário e reservas
- **PaymentService**: Processa pagamentos com transações distribuídas
- **APIGateway**: Controla acesso e valida tokens

### Camada de Dados
- **Database**: Armazena transações e inventário com ACID
- **RedisCache**: Cache distribuído para performance
- **MonitoringService**: Coleta métricas e alertas

## Padrões Arquiteturais Utilizados

- **Producer-Consumer Pattern**: Para o sistema de filas
- **Gateway Pattern**: Para controle de acesso
- **Two-Phase Commit**: Para transações distribuídas
- **Repository Pattern**: Para abstração de dados
- **Observer Pattern**: Para monitoramento
- **Circuit Breaker**: Para rate limiting
- **Pessimistic Locking**: Para controle de concorrência

## Fluxo Principal

1. **Cliente** entra na fila de espera com timestamp
2. **Sistema** gera token de posição e informa tempo estimado
3. **Queue Processor** libera usuários de forma controlada
4. **Cliente** autorizado pode iniciar compra
5. **TicketService** reserva ingresso (lock pessimista)
6. **PaymentService** processa pagamento
7. **Sistema** confirma transação ou libera reserva
8. **MonitoringService** acompanha todo o processo

## Benefícios

- ✅ **Fairness**: Ordem baseada em timestamp, não velocidade de conexão
- ✅ **Consistência**: Sem overselling através de locks e transações ACID
- ✅ **Escalabilidade**: Arquitetura distribuída com load balancing
- ✅ **Observabilidade**: Monitoramento completo do sistema
- ✅ **Resiliência**: Circuit breakers e tratamento de falhas
- ✅ **Performance**: Cache distribuído e otimizações

## Tecnologias Sugeridas

- **Queue**: Redis Queue
- **Cache**: Redis
- **Database**: PostgreSQL (ACID compliance)
- **Load Balancer**: NGINX ou HAProxy
- **Monitoring**: Prometheus + Grafana
- **API Gateway**: Kong ou AWS API Gateway

---

*Este README descreve a arquitetura conceitual do sistema.*
