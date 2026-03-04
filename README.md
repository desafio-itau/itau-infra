# 🐳 Docker Compose - Infraestrutura do Sistema

Este documento explica como configurar e executar a infraestrutura completa do Sistema de Compra Programada de Ações usando Docker Compose.

---

## 🎯 Visão Geral

Este `docker-compose.yml` provisiona toda a infraestrutura necessária para rodar os microsserviços:

- **PostgreSQL**: Banco de dados relacional
- **Kafka**: Message broker para eventos assíncronos
- **Zookeeper**: Coordenação do cluster Kafka
- **Kafka UI**: Interface web para visualizar tópicos e mensagens

Todos os serviços rodam em containers Docker isolados e se comunicam através de uma rede bridge dedicada (`itau-network`).

---

## 🛠️ Serviços Incluídos

### 1. **PostgreSQL** (`itau-postgres`)
- **Imagem**: `postgres:16-alpine`
- **Porta**: `5433:5432` (host:container)
- **Função**: Banco de dados principal do sistema
- **Features**:
  - Inicialização automática com `schema.sql`
  - Volume persistente para dados
  - Health check integrado

### 2. **Zookeeper** (`itau-zookeeper`)
- **Imagem**: `confluentinc/cp-zookeeper:7.6.0`
- **Porta**: `2181:2181`
- **Função**: Coordenação e gerenciamento do cluster Kafka
- **Necessário**: Para o funcionamento do Kafka

### 3. **Kafka** (`itau-kafka`)
- **Imagem**: `confluentinc/cp-kafka:7.6.0`
- **Portas**: 
  - `9092` (conexões externas - localhost)
  - `29092` (conexões internas - containers)
- **Função**: Message broker para eventos assíncronos
- **Tópicos utilizados**:
  - `ir-dedo-duro`: Eventos de IR 0,005% por operação
  - `ir-vendas`: Eventos de IR 20% sobre lucro em vendas
  - `rebalanceamento-cesta`: Eventos de rebalanceamento

### 4. **Kafka UI** (`itau-kafka-ui`)
- **Imagem**: `provectuslabs/kafka-ui:latest`
- **Porta**: `8090:8080`
- **Função**: Interface web para monitoramento do Kafka
- **Acesso**: http://localhost:8090

---

## ✅ Pré-requisitos

Antes de começar, certifique-se de ter instalado:

- [Docker](https://docs.docker.com/get-docker/) (versão 20.10+)
- [Docker Compose](https://docs.docker.com/compose/install/) (versão 2.0+)

---

## 🔌 Portas Utilizadas

| Serviço | Porta Host | Porta Container | Descrição |
|---------|------------|-----------------|-----------|
| PostgreSQL | 5433 | 5432 | Conexão ao banco de dados |
| Zookeeper | 2181 | 2181 | Coordenação Kafka |
| Kafka (externo) | 9092 | 9092 | Conexão de aplicações externas |
| Kafka (interno) | 29092 | 29092 | Conexão entre containers |
| Kafka UI | 8090 | 8080 | Interface web |

**Por que porta 5433 para PostgreSQL?**
- A porta padrão `5432` frequentemente já está em uso no macOS/Linux
- Usar `5433` evita conflitos com instalações locais do PostgreSQL

---

## 🔐 Variáveis de Ambiente

### PostgreSQL

| Variável | Descrição | Valor Padrão |
|----------|-----------|--------------|
| `DESAFIO_ITAU_DB_USER` | Usuário do banco | `itau_user` |
| `DESAFIO_ITAU_DB_PASSWORD` | Senha do banco | `itau_password_secure_2024` |
| `DESAFIO_ITAU_DB_NAME` | Nome do database | `compra_programada` |
| `DESAFIO_ITAU_DB_HOST` | Host do banco | `localhost` |
| `DESAFIO_ITAU_DB_PORT` | Porta do banco | `5433` |

### Kafka

| Variável | Descrição | Valor Padrão |
|----------|-----------|--------------|
| `KAFKA_BOOTSTRAP_SERVERS` | Endereço do broker | `localhost:9092` |
