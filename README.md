# Como rodar o K8s

## 🐳 Imagens Docker necessárias

Essas imagens **não** estão disponíveis no Docker Hub.

- `worker-insert:latest`
- `worker-update:latest`
- `worker-delete:latest`
- `techchallengeapi-dotnet-api:latest`

Para obtê-las, basta executar o `docker-compose` nos repositórios:

- [FIAP_Fase1_e_Fase2_TechChallenge](https://github.com/ThiagoBertoldi1/FIAP_Fase1_e_Fase2_TechChallenge)
- [FIAP_Fase3_TechChallenge](https://github.com/ThiagoBertoldi1/FIAP_Fase3_TechChallenge)

---

## ⚙️ Comandos para criar o ambiente

> Foi utilizado o **kind** como ambiente local.

### 🔨 Criar o cluster

> ⚠️ Importante: você precisa estar no diretório do arquivo `_kind-config.yaml`, baixado do repositório `Configs-K8s`.


```bash
kind create cluster --config _kind-config.yaml
```

### 🧪 Criar o namespace

```bash
kubectl apply -f ns.yaml
```

### 🚀 Criar deploys, services e configurações

Execute os seguintes comandos na raiz do projeto:

```bash
kubectl apply -f ./api
kubectl apply -f ./banco
kubectl apply -f ./monitoramento
kubectl apply -f ./rabbitmq
kubectl apply -f ./permissions
```

### 🐳 Inserir as imagens Docker no cluster

```bash
kind load docker-image worker-insert:latest
kind load docker-image worker-update:latest
kind load docker-image worker-delete:latest
kind load docker-image techchallengeapi-dotnet-api:latest
```

Verifique se todos os pods estão rodando corretamente:

```bash
kubectl get pods -n techchallenge
```

---

## 🛠️ Configuração inicial do SQL Server

A configuração inicial do banco deve ser feita manualmente, seguindo os passos abaixo:

### 1. Abrir conexão com o banco no cluster

```bash
kubectl port-forward sqlserver-0 <porta-local>:1433 -n techchallenge
```

### 2. Acessar o banco com `sqlcmd`

Em outro terminal:

```bash
sqlcmd -S localhost,<porta-local> -U sa -P "MinhaSenhaForte123!"
```

### 3. Executar comandos no banco

```sql
CREATE DATABASE TechChallenge
GO

USE [TechChallenge]
GO

SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

CREATE TABLE [dbo].[Contact] (
    [Id] UNIQUEIDENTIFIER NOT NULL,
    [Name] NVARCHAR(MAX) NOT NULL,
    [Email] NVARCHAR(MAX) NOT NULL,
    [PhoneNumber] BIGINT NOT NULL,
    [Region] NVARCHAR(MAX) NOT NULL,
    [District] NVARCHAR(MAX) NOT NULL,
    CONSTRAINT [PK_Contacts] PRIMARY KEY CLUSTERED ([Id] ASC)
        WITH (
            PAD_INDEX = OFF,
            STATISTICS_NORECOMPUTE = OFF,
            IGNORE_DUP_KEY = OFF,
            ALLOW_ROW_LOCKS = ON,
            ALLOW_PAGE_LOCKS = ON,
            OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF
        )
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
```

### 4. (Opcional) Inserir dados fictícios

```sql
INSERT INTO dbo.Contact (Id, Name, Email, PhoneNumber, Region, District)
VALUES
    (NEWID(), 'John Doe', 'john.doe@email.com', 47999999999, 'SC', '47'),
    (NEWID(), 'Jane Smith', 'jane.smith@email.com', 47988888888, 'SC', '47'),
    (NEWID(), 'Carlos Silva', 'carlos.silva@email.com', 47977777777, 'PR', '41')
GO
```

Você pode conferir se os registros foram inseridos com sucesso

```sql
SELECT COUNT(*) FROM dbo.Contact
GO
```

### ✅ Pronto!

Para usar a API, banco ou outro serviço do cluster, basta abrir uma conexão com o comando:

```bash
kubectl port-forward <nome-pod> <porta-local>:<porta-servico> -n techchallenge
```

Exemplo:

```bash
kubectl port-forward dotnet-api-jkbnakjs7 8080:3000 -n techchallenge
```

A API estará acessível em `http://localhost:8080`

> ⚠️ Lembre-se: `dotnet-api-jkbnakjs7` é um exemplo. Substitua pelo nome real do pod no seu ambiente.
---

## 🌐 Portas de Serviços Padrão

| Serviço     | Porta  |
|-------------|--------|
| API         | 3000   |
| RabbitMQ    | 15672  |
| Grafana     | 3000   |
| Prometheus  | 9090   |
| SQL Server  | 1433   |

---

## 🔐 Credenciais Padrão

| Serviço     | Usuário | Senha                 |
|-------------|---------|------------------------|
| Grafana     | admin   | admin                  |
| RabbitMQ    | guest   | guest                  |
| SQL Server  | sa      | MinhaSenhaForte123!    |

---

## ℹ️ Informações Adicionais

- Os **painéis do Grafana** precisam ser configurados ao criar o cluster, utilizando o `Prometheus` como fonte dos dados.
  - Após isso, não é necessário configurá-los novamente.
- As **filas do RabbitMQ** são criadas automaticamente pelos workers.
  - Nenhuma ação manual é necessária.
