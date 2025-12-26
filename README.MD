# 🚀 MCP SQL Server: Integre o Claude Desktop ao seu Banco de Dados

> **Claude SQL Bridge**: Guia Definitivo para macOS & Windows

**Última atualização**: Dezembro 2025  
**SQL Server testado**: Microsoft SQL Server 2025 (RTM) Enterprise Developer Edition

Este guia fornece o passo a passo para configurar o **Model Context Protocol (MCP)**, permitindo que o Claude Desktop execute consultas SQL diretamente em instâncias do **Microsoft SQL Server**.

---

## 📋 Índice

- [Pré-requisitos](#-pré-requisitos)
- [Instalação](#-instalação)
- [Configuração](#-configuração)
- [Executando Consultas](#-executando-consultas)
- [Operações Suportadas](#-operações-suportadas)
- [Limitações](#-limitações)
- [Permissões e Segurança](#-permissões-e-segurança)
- [Alternativas para Operações Restritas](#-alternativas-para-operações-restritas)
- [Troubleshooting](#-troubleshooting)
- [Contato](#-contato)

---

## 🎯 Pré-requisitos

- **Claude Desktop** instalado
- **Node.js (v18+)** instalado
- **SQL Server** acessível (local, Docker ou remoto)
- **Porta 1433** aberta no firewall

---

## 🛠️ Instalação

Instale o conector MCP globalmente:

```bash
npm install -g @modelcontextprotocol/server-mssql
```

> **Dica**: Use `which node` (macOS) ou `where node` (Windows) para verificar o caminho de instalação do Node.js.

---

## ⚙️ Configuração

### 1. Localize o arquivo de configuração do Claude:

- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

### 2. Configure o MCP Server:

```json
{
  "mcpServers": {
    "sql-server": {
      "command": "node",
      "args": [
        "C:/Users/SEU_USUARIO/AppData/Roaming/npm/node_modules/@modelcontextprotocol/server-mssql/dist/index.js"
      ],
      "env": {
        "MSSQL_CONNECTION_STRING": "Server=SEU_IP,1433;Database=NOME_BANCO;User Id=sa;Password=SUA_SENHA;Encrypt=true;TrustServerCertificate=true;"
      }
    }
  }
}
```

> **macOS**: O caminho em `args` geralmente é `/usr/local/lib/node_modules/...`

### 3. Verificação de Logs:

Se o ícone de ferramenta (🔨) não aparecer no Claude, verifique os logs:

- **macOS**: `~/Library/Logs/Claude`
- **Windows**: `%USERPROFILE%\AppData\Local\Logs\Claude`

**Arquivos principais**:
- `mcp-server-sql-server.log`: Erros de conexão SQL
- `claude_desktop.log`: Erros de carregamento do MCP

---

## 🚀 Executando Consultas

1. **Reinicie o Claude Desktop**
2. **Envie uma consulta**: "Mostre os top 5 produtos da tabela dbo.PRODUTOS"
3. **Autorize o acesso**: Clique em "Permitir uma vez" ou "Sempre permitir"
4. **Veja o resultado**: Dados formatados diretamente no chat

### 💡 Checklist Rápido

- ✅ **Sintaxe JSON**: Verifique no VS Code se não há erros de sintaxe
- ✅ **SSL**: Use `TrustServerCertificate=true;` para SQL Server local sem certificado oficial
- ✅ **Teste de rede**: 
  - macOS: `nc -zv IP 1433`
  - Windows: `Test-NetConnection IP -Port 1433`

---

## ✅ Operações Suportadas

### Consultas e Leitura
- SELECT, JOINs complexos, subconsultas
- Consultas ao INFORMATION_SCHEMA e sys.objects
- Stored procedures de leitura
- Funções escalares e de tabela

### Modificações de Dados (DML)
- INSERT, UPDATE, DELETE, MERGE
- Transações (BEGIN TRANSACTION, COMMIT, ROLLBACK)

### Definição de Dados (DDL)
- CREATE, ALTER, DROP de tabelas, views, índices
- CREATE/ALTER stored procedures e functions
- Gerenciamento de constraints e triggers

### Análise e Metadata
- Consulta de estruturas de banco
- Verificação de estatísticas e performance
- Análise de planos de execução (SET STATISTICS)

---

## ❌ Limitações

### Operações de Arquivo (Não Suportadas)

- ❌ BACKUP DATABASE / RESTORE DATABASE (mesmo com permissões)
- ❌ Acesso ao sistema de arquivos (xp_cmdshell, OPENROWSET)
- ❌ BULK INSERT de arquivos externos
- ❌ Importação/exportação para arquivos físicos

> **Importante**: O MCP não pode executar comandos de backup porque requer acesso ao sistema de arquivos do servidor - isso é uma limitação da interface MCP, não das permissões.

### Operações de Sistema (Limitadas)

- ❌ Configurações de servidor (sp_configure)
- ❌ Linked servers
- ❌ Replicação e espelhamento
- ❌ SQL Server Agent Jobs

### Limitações Técnicas

- ⏱️ **Timeout**: 15 segundos por consulta
- 🚫 **Sem transferência de arquivos binários**
- 🔌 **Conexão**: TCP/IP na porta 1433

---

## 🔐 Permissões e Segurança

### Verificar Permissões Atuais

```sql
-- Ver todas as permissões do usuário
SELECT * FROM fn_my_permissions(NULL, 'DATABASE');

-- Ver roles do usuário
SELECT 
    r.name AS role_name,
    m.name AS member_name
FROM sys.database_role_members AS drm
JOIN sys.database_principals AS r ON drm.role_principal_id = r.principal_id
JOIN sys.database_principals AS m ON drm.member_principal_id = m.principal_id
WHERE m.name = USER_NAME();
```

### Roles Comuns do SQL Server

| Role | Permissões Principais |
|------|----------------------|
| **db_owner** | Controle total sobre o banco |
| **db_ddladmin** | CREATE, ALTER, DROP de objetos |
| **db_datawriter** | INSERT, UPDATE, DELETE |
| **db_datareader** | SELECT em todas as tabelas |
| **db_backupoperator** | BACKUP DATABASE, BACKUP LOG |

### Exemplo: Permissões com db_owner

Com a role `db_owner`, o usuário possui:

**✅ Operações de Dados**: SELECT, INSERT, UPDATE, DELETE, EXECUTE  
**✅ Definição de Estrutura**: CREATE/ALTER/DROP de todos os objetos  
**✅ Backup**: BACKUP DATABASE e BACKUP LOG (mas MCP não executa)  
**✅ Administração**: CONTROL, TAKE OWNERSHIP, VIEW DATABASE STATE  
**✅ Segurança**: Gerenciar máscaras, classificações e criptografia

**Diferença importante**:
- ✅ **Permissão**: Usuário autorizado para BACKUP DATABASE
- ❌ **Capacidade MCP**: Interface não acessa sistema de arquivos para salvar .bak

---

## 🔄 Alternativas para Operações Restritas

### Backup de Dados

#### 1. Scripts SQL (Recomendado para migração)

```sql
-- Script completo com estrutura e dados
CREATE TABLE [dbo].[PEDIDOS](
    [ID_PEDIDO] [int] NOT NULL PRIMARY KEY,
    [ID_CLIENTE] [int] NULL,
    [VALOR_TOTAL] [numeric](18, 0) NULL
);

INSERT INTO [dbo].[PEDIDOS] VALUES (1, 1, 200);
```

**Vantagens**: Portável, versionável no Git, funciona com MCP

#### 2. Exportação JSON

```sql
SELECT * FROM tabela
FOR JSON AUTO, INCLUDE_NULL_VALUES;
```

#### 3. Backup Manual (.bak)

Execute diretamente no servidor via SSMS ou Azure Data Studio:

```sql
BACKUP DATABASE [NomeDoBanco] 
TO DISK = 'C:\Backups\NomeDoBanco.bak'
WITH COMPRESSION;
```

### Outras Ferramentas

**Para grandes volumes**:
- BCP (Bulk Copy Program)
- SQL Server Import/Export Wizard
- SSIS, Azure Data Factory

---

## 📊 Casos de Uso

### ✅ Ideal para:
- Análise exploratória de dados
- Consultas ad-hoc e relatórios
- Modificações de dados (DML)
- Criação e manutenção de estruturas (DDL)
- Extração de dados para migração
- Análise de performance
- Prototipagem e desenvolvimento

### ❌ Não recomendado para:
- Backups binários completos
- Operações com sistema de arquivos
- Configurações de servidor
- Operações > 15 segundos
- Agendamento de jobs

---

## 🔍 Troubleshooting

### Erro de Timeout
```
ERRO: Failed to connect in 15000ms
```

**Soluções**:
- Verifique se o servidor está online
- Confirme que a porta 1433 está acessível
- Verifique configurações de firewall
- Teste conectividade de rede

### Erro de Permissão
```
The user does not have permission
```

**Soluções**:
- Verifique permissões: `SELECT * FROM fn_my_permissions(NULL, 'DATABASE')`
- Verifique roles do usuário
- Solicite permissões ao DBA
- Use conta com privilégios apropriados

### Query Lenta

**Soluções**:
- Adicione filtros WHERE
- Use índices apropriados
- Divida query em partes menores (< 15s)
- Analise plano de execução: `SET STATISTICS IO, TIME ON`

---

## 💡 Boas Práticas

### Migração de Dados
1. Use scripts SQL (mais portável)
2. Teste em desenvolvimento primeiro
3. Faça backup antes de modificações
4. Divida operações grandes (< 15s)
5. Use transações para integridade

### Performance
1. Use índices apropriados
2. Analise planos de execução
3. Evite SELECT * em tabelas grandes
4. Use paginação para grandes resultados

### Segurança
1. Verifique permissões antes de operações críticas
2. Use o princípio do menor privilégio
3. Documente permissões necessárias
4. Revise permissões periodicamente

---

## 🤝 Contribuições

Contribuições são bem-vindas! Reporte limitações adicionais ou sugira melhorias.

---

## 📞 Contato

**Alexandre**  
📧 alemiti@gmail.com

---
