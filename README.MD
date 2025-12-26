
# 🚀 MCP SQL Server: Integre o Claude Desktop ao seu Banco de Dados
-- Claude SQL Bridge: Guia Definitivo para macOS & Windows

## Última atualização: Dezembro 2025
## Versão do SQL Server testada: Microsoft SQL Server 2025 (RTM) Enterprise Developer Edition


Este guia fornece o passo a passo unificado para configurar o **Model Context Protocol (MCP)**, permitindo que o Claude Desktop execute consultas SQL diretamente em instâncias do **Microsoft SQL Server**.

## 📋 1. Pré-requisitos (Ambos os Sistemas)

* **Claude Desktop** instalado.
* **Node.js (v18+)** instalado: Necessário para rodar o servidor MCP.
* **SQL Server** acessível: Pode ser local, via Docker ou servidor remoto.
* **Conectividade:** Certifique-se de que a porta **1433** está aberta no firewall.

---

## 🛠️ 2. Instalação do Servidor MCP

Abra o terminal (Terminal no macOS ou PowerShell/CMD no Windows) e instale o conector globalmente:

```bash
npm install -g @modelcontextprotocol/server-mssql

```

> **Dica:** Verifique o local de instalação do Node.js com o comando `which node` (macOS) ou `where node` (Windows) para usar no arquivo de configuração.

---

## ⚙️ 3. Configuração do Claude Desktop

O Claude Desktop utiliza um arquivo JSON para gerenciar os servidores. O conteúdo é o mesmo, mas o local do arquivo muda conforme o sistema:

### Caminho do Arquivo:

* **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
* **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

### Conteúdo do JSON:

Copie e cole o código abaixo, ajustando os caminhos de acordo com seu usuário:

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

*No **macOS**, o caminho em `args` costuma ser `/usr/local/lib/node_modules/...` ou similar.*

---

## 🔍 4. Verificação de Logs e Troubleshooting

Se o ícone de ferramenta (martelo) não aparecer no Claude, verifique os logs para diagnosticar erros de conexão ou sintaxe.

### Como abrir a pasta de logs:

* **No macOS (Terminal):**
```bash
open ~/Library/Logs/Claude

```


* **No Windows (Executar/Win+R):**
```text
%USERPROFILE%\AppData\Local\Logs\Claude

```



**Arquivos principais:**

* `mcp-server-sql-server.log`: Registra falhas de login ou erros de rede do SQL.
* `claude_desktop.log`: Registra se o Claude conseguiu carregar o Node.js e o arquivo de configuração.

---

## 🚀 5. Executando Consultas

1. **Reinicie o Claude Desktop:** Essencial para carregar novas configurações.
2. **Peça uma consulta:** "Claude, mostre os top 5 produtos da tabela dbo.PRODUTOS".
3. **Autorize o Acesso:** Uma janela de segurança surgirá. Clique em **"Permitir uma vez"** ou **"Sempre permitir"**.
4. **Resultado:** O Claude exibirá os dados formatados em tabela diretamente no chat.

---

## 💡 Dicas de Ouro (Checklist)

* **Sintaxe JSON:** No VS Code, verifique se não há erros de "End of file expected". O arquivo deve terminar com um fechamento de chave `}` limpo.
* **Segurança SQL:** Se usar SQL Server local sem certificado SSL oficial, o parâmetro `TrustServerCertificate=true;` na string de conexão é obrigatório para evitar erros de handshake.
* **Teste de Rede:** Use `nc -zv IP 1433` (macOS) ou `Test-NetConnection IP -Port 1433` (Windows) para garantir que o banco está alcançável.

---

# Limitações do MCP SQL Server

Este documento descreve as capacidades e limitações do Model Context Protocol (MCP) ao conectar-se com SQL Server através do Claude AI.

## 📋 Índice

- [Operações Suportadas](#operações-suportadas)
- [Limitações](#limitações)
- [Permissões e Segurança](#permissões-e-segurança)
- [Exemplo de Análise de Permissões](#exemplo-de-análise-de-permissões)
- [Alternativas para Operações Restritas](#alternativas-para-operações-restritas)

## ✅ Operações Suportadas

### Consultas e Leitura de Dados

- Execução de comandos `SELECT` em tabelas, views e funções
- JOINs complexos e subconsultas
- Consultas ao `INFORMATION_SCHEMA` e `sys.objects`
- Execução de stored procedures de leitura
- Uso de funções escalares e de tabela

### Modificações de Dados (DML)

- `INSERT`, `UPDATE`, `DELETE`
- `MERGE` e operações complexas de manipulação de dados
- Gerenciamento de transações:
  - `BEGIN TRANSACTION`
  - `COMMIT`
  - `ROLLBACK`

### Definição de Dados (DDL)

- `CREATE`, `ALTER`, `DROP` de:
  - Tabelas
  - Views
  - Índices
  - Stored procedures
  - Functions
- Gerenciamento de constraints e triggers

### Análise e Metadata

- Consulta de estruturas de banco de dados
- Verificação de estatísticas e performance
- Análise de planos de execução (`SET STATISTICS`)
- Inspeção de schemas e objetos do banco

## ❌ Limitações

### 1. Operações de Arquivo

**Não suportado:**
- `BACKUP DATABASE` / `RESTORE DATABASE` ⚠️ (mesmo com permissões adequadas)
- Acesso ao sistema de arquivos do servidor
- Comandos como `xp_cmdshell`
- `OPENROWSET` com arquivos
- Criação ou leitura de arquivos `.bak`, `.csv`, `.txt` no servidor
- `BULK INSERT` de arquivos externos
- Importação/exportação de dados para arquivos físicos

> **Nota Importante**: Mesmo que o usuário possua as permissões `BACKUP DATABASE` e `BACKUP LOG`, o MCP não consegue executar comandos de backup porque eles requerem acesso ao sistema de arquivos do servidor, o que é uma **limitação da interface MCP**, não das permissões do usuário.

### 2. Operações de Sistema

**Limitado ou não suportado:**
- Gerenciamento completo de logins e usuários
- Configurações de servidor (`sp_configure`)
- Linked servers
- Replicação e espelhamento de banco de dados
- SQL Server Agent Jobs e agendamentos

### 3. Limitações de Rede e Performance

- **Timeout**: 15 segundos por consulta
- Sem transferência de arquivos binários pela conexão
- Sem acesso direto ao sistema de arquivos do servidor
- Conexão TCP/IP na porta padrão 1433

### 4. Comandos Administrativos Avançados

**Restritos:**
- Alguns comandos `DBCC` (manutenção avançada)
- Operações que requerem permissões `sysadmin` específicas
- Alterações em databases de sistema (`master`, `msdb`, `model`, `tempdb`)

## 🔐 Permissões e Segurança

As limitações dependem das **permissões do usuário** configurado na conexão MCP.

### Verificar Permissões Atuais

```sql
-- Ver todas as permissões do usuário atual
SELECT * FROM fn_my_permissions(NULL, 'DATABASE');

-- Ver roles do usuário
SELECT 
    r.name AS role_name,
    m.name AS member_name
FROM sys.database_role_members AS drm
JOIN sys.database_principals AS r ON drm.role_principal_id = r.principal_id
JOIN sys.database_principals AS m ON drm.member_principal_id = m.principal_id
WHERE m.name = USER_NAME();

-- Ver o usuário atual
SELECT USER_NAME() AS CurrentUser, SUSER_NAME() AS CurrentLogin;
```

### Roles Comuns do SQL Server

| Role | Descrição | Permissões Principais |
|------|-----------|----------------------|
| **db_owner** | Proprietário do banco | Controle total sobre o banco de dados |
| **db_ddladmin** | Administrador DDL | CREATE, ALTER, DROP de objetos |
| **db_datawriter** | Escritor de dados | INSERT, UPDATE, DELETE |
| **db_datareader** | Leitor de dados | SELECT em todas as tabelas |
| **db_backupoperator** | Operador de backup | BACKUP DATABASE, BACKUP LOG |
| **db_securityadmin** | Administrador de segurança | Gerenciar roles e permissões |

### Aumentar Permissões (Se Necessário)

#### Para Permitir Backups via T-SQL

```sql
-- Conceder permissão de backup ao usuário
USE master;
GO
GRANT BACKUP DATABASE TO [seu_usuario];
GRANT BACKUP LOG TO [seu_usuario];
GO
```

#### Para Operações de Arquivo (⚠️ Risco de Segurança)

```sql
-- Habilitar xp_cmdshell (use com EXTREMA cautela!)
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

> ⚠️ **Atenção**: Habilitar `xp_cmdshell` representa riscos significativos de segurança. Use apenas em ambientes controlados e de desenvolvimento.

#### Para Operações DDL Completas

```sql
-- Adicionar usuário ao role db_ddladmin
USE [seu_banco];
GO
ALTER ROLE db_ddladmin ADD MEMBER [seu_usuario];
GO
```

#### Para Adicionar Usuário como DB Owner

```sql
-- Adicionar usuário ao role db_owner (controle total)
USE [seu_banco];
GO
ALTER ROLE db_owner ADD MEMBER [seu_usuario];
GO
```

## 📊 Exemplo de Análise de Permissões

Abaixo está um exemplo real de análise de permissões de um usuário com role `db_owner`:

### Informações do Usuário

```
Role: db_owner
Usuário: dbo (Database Owner)
```

### Permissões Disponíveis (Resumo)

**Operações de Dados:**
- ✅ SELECT, INSERT, UPDATE, DELETE
- ✅ EXECUTE (procedures e functions)
- ✅ REFERENCES

**Definição de Estrutura:**
- ✅ CREATE TABLE, VIEW, PROCEDURE, FUNCTION
- ✅ CREATE SCHEMA, ROLE, USER
- ✅ ALTER ANY SCHEMA, USER, ROLE
- ✅ DROP (todos os objetos)

**Backup e Recuperação:**
- ✅ BACKUP DATABASE
- ✅ BACKUP LOG
- ⚠️ Nota: Mesmo com essas permissões, o MCP não consegue executar comandos de backup

**Administração:**
- ✅ ALTER (modificar banco e objetos)
- ✅ CONTROL (controle total do banco)
- ✅ TAKE OWNERSHIP
- ✅ VIEW DATABASE STATE
- ✅ KILL DATABASE CONNECTION
- ✅ CHECKPOINT

**Segurança e Criptografia:**
- ✅ ALTER ANY MASK / UNMASK
- ✅ ALTER ANY SENSITIVITY CLASSIFICATION
- ✅ VIEW ANY SENSITIVITY CLASSIFICATION
- ✅ CREATE/ALTER CERTIFICATE, SYMMETRIC KEY, ASYMMETRIC KEY
- ✅ VIEW ANY COLUMN ENCRYPTION KEY DEFINITION

**Recursos Avançados:**
- ✅ EXECUTE ANY EXTERNAL SCRIPT
- ✅ ADMINISTER DATABASE BULK OPERATIONS
- ✅ ALTER LEDGER (tabelas Ledger)
- ✅ VIEW DATABASE PERFORMANCE STATE
- ✅ CREATE/ALTER DATABASE AUDIT
- ✅ SHOWPLAN (ver planos de execução)

### Interpretação

Com a role **db_owner**, o usuário possui:

1. **Controle Total**: Pode executar qualquer operação no banco de dados
2. **Permissões de Backup**: Possui `BACKUP DATABASE` e `BACKUP LOG`, mas o MCP não consegue executá-los devido à limitação de acesso ao sistema de arquivos
3. **Administração Completa**: Pode criar, modificar e deletar qualquer objeto
4. **Segurança**: Pode gerenciar usuários, roles e permissões
5. **Performance**: Pode visualizar e analisar estado e performance do banco

**Diferença entre Permissão e Capacidade do MCP:**
- ✅ **Permissão**: O usuário tem autorização para executar `BACKUP DATABASE`
- ❌ **Capacidade do MCP**: A interface MCP não consegue acessar o sistema de arquivos para salvar o arquivo `.bak`

## 🔄 Alternativas para Operações Restritas

### Backup de Dados

Como não é possível criar arquivos `.bak` diretamente através do MCP, utilize estas alternativas:

#### 1. Scripts SQL de INSERT

Melhor para migração entre servidores diferentes:

```sql
-- Gerar script completo com estrutura e dados
-- Estrutura da tabela
CREATE TABLE [dbo].[PEDIDOS](
    [ID_PEDIDO] [int] NOT NULL PRIMARY KEY,
    [ID_CLIENTE] [int] NULL,
    [ID_PRODUTO] [int] NULL,
    [DATA_PEDIDO] [datetime] NULL,
    [DATA_ENTREGA] [date] NULL,
    [QUANTIDADE] [int] NULL,
    [VALOR_TOTAL] [numeric](18, 0) NULL
);

-- Dados
INSERT INTO [dbo].[PEDIDOS] VALUES (1, 1, 1, '2023-01-05 14:05:00', '2023-05-22', 10, 200);
-- ... mais linhas
```

**Vantagens:**
- Portável entre diferentes versões do SQL Server
- Versionável no Git
- Fácil de revisar e modificar
- Funciona perfeitamente com o MCP

#### 2. Exportação para JSON

```sql
-- Exportar dados em formato JSON
SELECT * FROM tabela
FOR JSON AUTO, INCLUDE_NULL_VALUES;
```

**Vantagens:**
- Formato universal
- Fácil de processar em outras linguagens
- Estruturado e legível

#### 3. Geração de Scripts Completos do Banco

```sql
-- Listar todas as tabelas
SELECT TABLE_SCHEMA, TABLE_NAME 
FROM INFORMATION_SCHEMA.TABLES 
WHERE TABLE_TYPE = 'BASE TABLE';

-- Para cada tabela, gerar CREATE TABLE e INSERT statements
```

#### 4. Backup Manual no Servidor

Para backups binários completos (`.bak`), execute diretamente no servidor via SSMS, Azure Data Studio ou T-SQL:

```sql
-- Execute este comando diretamente no SQL Server (não via MCP)
BACKUP DATABASE [NomeDoBanco] 
TO DISK = 'C:\Backups\NomeDoBanco.bak'
WITH FORMAT, 
     MEDIANAME = 'SQLServerBackups',
     NAME = 'Full Backup of NomeDoBanco',
     COMPRESSION;
GO

-- Verificar o backup
RESTORE VERIFYONLY 
FROM DISK = 'C:\Backups\NomeDoBanco.bak';
```

### Operações de Sistema

Para configurações de sistema e jobs agendados:
- Use SQL Server Management Studio (SSMS)
- Utilize Azure Data Studio
- Acesse diretamente o servidor via RDP ou console
- Use PowerShell com módulo SqlServer

### Transferência de Arquivos

Para importar/exportar grandes volumes de dados:
- Use BCP (Bulk Copy Program)
- Use SQL Server Import/Export Wizard
- Use ferramentas ETL como SSIS
- Use Azure Data Factory (para Azure SQL)

## 📊 Casos de Uso Ideais

O MCP SQL Server é ideal para:

- ✅ Análise exploratória de dados
- ✅ Consultas ad-hoc e relatórios
- ✅ Modificações de dados (INSERT, UPDATE, DELETE)
- ✅ Criação e manutenção de estruturas (DDL)
- ✅ Extração de dados para migração
- ✅ Análise de performance e otimização
- ✅ Geração de scripts de banco de dados
- ✅ Verificação de permissões e metadata
- ✅ Prototipagem e desenvolvimento

## 🚫 Não Recomendado Para

- ❌ Backups binários completos do banco (use SSMS ou scripts diretos)
- ❌ Operações que requerem acesso ao sistema de arquivos
- ❌ Configurações avançadas de servidor
- ❌ Gerenciamento de infraestrutura
- ❌ Operações que exigem mais de 15 segundos
- ❌ Importação/exportação de arquivos grandes
- ❌ Agendamento de jobs

## 💡 Dicas e Boas Práticas

### Para Migração de Dados

1. **Use scripts SQL**: Mais portável e versionável
2. **Teste em ambiente de desenvolvimento primeiro**
3. **Faça backup antes de modificações**: Use SSMS para criar `.bak`
4. **Divida operações grandes**: Evite timeout de 15 segundos
5. **Use transações**: Garanta integridade dos dados

### Para Análise de Permissões

1. **Sempre verifique permissões antes de operações críticas**
2. **Documente as permissões necessárias para cada operação**
3. **Use o princípio do menor privilégio**
4. **Revise permissões periodicamente**

### Para Performance

1. **Use índices apropriados**
2. **Analise planos de execução**: Use `SET STATISTICS IO, TIME ON`
3. **Evite SELECT * em tabelas grandes**
4. **Use paginação para grandes resultados**

## 📝 Notas Adicionais

- O MCP conecta-se via TCP/IP na porta padrão 1433
- Requer credenciais válidas de SQL Server Authentication ou Windows Authentication
- Todas as operações respeitam as permissões configuradas para o usuário
- Transações são suportadas e devem ser usadas para garantir integridade dos dados
- O timeout padrão é de 15 segundos por consulta
- Não há limite de queries sequenciais, mas cada uma deve completar em 15 segundos

## 🔍 Troubleshooting

### Erro de Timeout

```
ERRO DE CONEXÃO: Failed to connect in 15000ms
```

**Soluções:**
- Verifique se o servidor está online
- Confirme que a porta 1433 está acessível
- Verifique configurações de firewall
- Teste conectividade de rede

### Erro de Permissão

```
The user does not have permission to perform this action
```

**Soluções:**
- Verifique permissões com `fn_my_permissions`
- Verifique roles do usuário
- Solicite permissões adequadas ao DBA
- Use conta com privilégios apropriados

### Query Muito Lenta

**Soluções:**
- Adicione filtros WHERE para reduzir dados
- Use índices apropriados
- Divida query em partes menores
- Analise plano de execução


## 🤝 Contribuições

Sinta-se à vontade para contribuir com este documento reportando limitações adicionais ou sugerindo melhorias.

---


📞 Contato
Alexandre
alemiti@gmail.com
😊

---
