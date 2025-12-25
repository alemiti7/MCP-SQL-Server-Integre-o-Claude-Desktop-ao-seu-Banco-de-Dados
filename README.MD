
# 🚀 MCP SQL Server: Integre o Claude Desktop ao seu Banco de Dados
-- Claude SQL Bridge: Guia Definitivo para macOS & Windows


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

Limitações do Model Context Protocol (MCP) ao conectar-se com SQL Server através do Claude AI.

## 📋 Índice

- [Operações Suportadas](#operações-suportadas)
- [Limitações](#limitações)
- [Permissões e Segurança](#permissões-e-segurança)
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
- `BACKUP DATABASE` / `RESTORE DATABASE`
- Acesso ao sistema de arquivos do servidor
- Comandos como `xp_cmdshell`
- `OPENROWSET` com arquivos
- Criação ou leitura de arquivos `.bak`, `.csv`, `.txt` no servidor
- `BULK INSERT` de arquivos externos
- Importação/exportação de dados para arquivos físicos

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
```

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

## 🔄 Alternativas para Operações Restritas

### Backup de Dados

Como não é possível criar arquivos `.bak` diretamente através do MCP, utilize estas alternativas:

#### 1. Scripts SQL de INSERT

Melhor para migração entre servidores diferentes:

```sql
-- Gerar script completo com estrutura e dados
-- (ideal para versionamento e migração)
```

**Vantagens:**
- Portável entre diferentes versões do SQL Server
- Versionável no Git
- Fácil de revisar e modificar

#### 2. Exportação para JSON

```sql
-- Exportar dados em formato JSON
SELECT * FROM tabela
FOR JSON AUTO, INCLUDE_NULL_VALUES;
```

#### 3. Exportação para CSV (via código)

Processar resultados e formatar como CSV através de scripts.

#### 4. Backup Manual no Servidor

Para backups binários completos (`.bak`), execute diretamente no servidor:

```sql
BACKUP DATABASE [NomeDoBanco] 
TO DISK = 'C:\Backups\NomeDoBanco.bak'
WITH FORMAT, 
     MEDIANAME = 'SQLServerBackups',
     NAME = 'Full Backup';
```

### Operações de Sistema

Para configurações de sistema e jobs agendados:
- Use SQL Server Management Studio (SSMS)
- Utilize Azure Data Studio
- Acesse diretamente o servidor via RDP ou console

## 📊 Casos de Uso Ideais

O MCP SQL Server é ideal para:

- ✅ Análise exploratória de dados
- ✅ Consultas ad-hoc e relatórios
- ✅ Modificações de dados (INSERT, UPDATE, DELETE)
- ✅ Criação e manutenção de estruturas (DDL)
- ✅ Extração de dados para migração
- ✅ Análise de performance e otimização
- ✅ Geração de scripts de banco de dados

## 🚫 Não Recomendado Para

- ❌ Backups binários completos do banco
- ❌ Operações que requerem acesso ao sistema de arquivos
- ❌ Configurações avançadas de servidor
- ❌ Gerenciamento de infraestrutura
- ❌ Operações que exigem mais de 15 segundos

## 📝 Notas Adicionais

- O MCP conecta-se via TCP/IP na porta padrão 1433
- Requer credenciais válidas de SQL Server Authentication ou Windows Authentication
- Todas as operações respeitam as permissões configuradas para o usuário
- Transações são suportadas e devem ser usadas para garantir integridade dos dados

## 🤝 Contribuições

Sinta-se à vontade para contribuir com este documento reportando limitações adicionais ou sugerindo melhorias.

---


📞 Contato
Alexandre
alemiti@gmail.com
😊

---
