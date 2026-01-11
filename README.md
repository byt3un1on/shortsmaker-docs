# Shortsmaker - Documentação do Sistema

O sistema é responsável por automatizar a criação de vídeos curtos (shorts) a partir de um tema e descrição fornecidos pelo usuário.

## 1. Arquitetura e Fluxo do Sistema

Este diagrama descreve as aplicações, serviços e o fluxo assíncrono de processamento.

```mermaid
flowchart TD
    User([Usuário])
    
    subgraph Apps [Aplicações]
        Frontend(Frontend)
        API(API)
        Worker(Worker)
    end

    subgraph DB [Postgres]
        TicketsTable[(Tabela de Tickets)]
    end

    subgraph ObjectStorage [Bucket]
        Files[(Músicas, Vídeos, VTTs)]
    end

    %% Fluxo de Entrada
    User --> Frontend
    Frontend --> API
    API --> TicketsTable

    %% Fluxo de Processamento (Worker)
    Worker -.-> TicketsTable
    Worker --> S1
    
    subgraph Steps [Passos de Produção]
        S1["1. Gerar Roteiro"]
        S2["2. Coletar Música"]
        S3["3. Gerar Vídeo"]
        S4["4. Extrair VTT"]
    end

    S1 --> S2 --> S3 --> S4
    
    S2 --- Files
    S3 --- Files
    S4 --- Files
```

## 2. Diagrama Entidade-Relacionamento (ER) - Normalizado (3NF)

O modelo foi normalizado para separar o ciclo de vida e auditoria do `Ticket` de seus dados dinâmicos e metadados.

```mermaid
erDiagram
    TICKET ||--o{ TICKET_ATTRIBUTES : "possui"
    
    TICKET {
        uuid id PK
        string status "PENDING, PROCESSING, COMPLETED, FAILED"
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
        boolean is_deleted
    }

    TICKET_ATTRIBUTES {
        uuid id PK
        uuid ticket_id FK
        string key "theme, description, script, video_url, etc."
        text value
        timestamp created_at
        timestamp deleted_at
        boolean is_deleted
    }
```

### Detalhes do Funcionamento:
1. **Frontend:** Coleta os dados do usuário.
2. **API:** Recebe a requisição, gera um UUID para o **Ticket**, registra o status inicial e salva os inputs na tabela **TICKET_ATTRIBUTES**.
3. **Worker:** Atua de forma assíncrona. Ele busca no banco por tickets com status de processamento pendente, consulta os atributos necessários em **TICKET_ATTRIBUTES**, executa as ferramentas de IA e edição de vídeo, e persiste os arquivos no **Bucket**. A cada etapa, ele atualiza o status do **Ticket** e insere/atualiza os caminhos dos arquivos em **TICKET_ATTRIBUTES**.

