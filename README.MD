# MCP SQL Server Connector: Claude Desktop Integration

Documentação técnica para a integração do Claude Desktop com instâncias Microsoft SQL Server (RTM Enterprise Developer Edition e superiores) através do Model Context Protocol (MCP).

## Sumário

- [Funcionalidades](#Funcionalidades)
- [Arquitetura](#Arquitetura)
- [Instalação](#instalação)
- [Configuração](#Configuração)
- [Limitações](#Limitações)
- [Contribuição](#Contribuição)
- [Licença](#Licença)
- [Contato](#Contato)

## Funcionalidades

* **Manipulação de Dados (DML)**: Execução de instruções SELECT, JOINs complexos, INSERT, UPDATE, DELETE e MERGE com suporte a transações.
* **Definição de Estrutura (DDL)**: Capacidade de criar, alterar e remover tabelas, views, índices, stored procedures e triggers.
* **Análise de Metadados**: Consulta ao `INFORMATION_SCHEMA`, verificação de estatísticas e análise de planos de execução via `SET STATISTICS`.
* **Segurança**: Validação de permissões via roles do sistema como `db_owner`, `db_ddladmin`, `db_datareader` e `db_datawriter`.

## Arquitetura

A integração utiliza o protocolo MCP para permitir que o modelo de IA execute comandos SQL em tempo real através de um servidor local Node.js.

* **Protocolo**: Model Context Protocol (MCP).
* **Runtime**: Node.js v18 ou superior.
* **Conectividade**: Protocolo TCP/IP via porta 1433.
* **Sistemas Operacionais**: Compatibilidade verificada em Windows 10+ e macOS 11+.

## Instalação

### Pré-requisitos

* Node.js v18 ou superior instalado.
* Instância SQL Server ativa e acessível (Local, Docker ou Remoto).
* Porta 1433 liberada no firewall do sistema.

### Procedimento

Instale o pacote do servidor globalmente utilizando o gerenciador de pacotes npm:

```bash
npm install -g @modelcontextprotocol/server-mssql

```

## Configuração

O arquivo `claude_desktop_config.json` deve ser editado nos seguintes diretórios, dependendo do sistema:

* **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`
* **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`

### Estrutura do Arquivo

```json
{
  "mcpServers": {
    "mcp-sql-server": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sqlserver"],
      "env": {
        "SQL_CONNECTION_STRING": "SUA_STRING_DE_CONEXAO",
        "TZ": "America/Sao_Paulo"
      }
    }
  }
}

```

## Limitações

### Restrições de Operação

* **Sistema de Arquivos**: Incompatibilidade com operações que exigem acesso físico ao disco do servidor, como `BACKUP DATABASE`, `RESTORE` ou `BULK INSERT`.
* **Timeout**: Limite operacional de 15 segundos por requisição.
* **Administração de Sistema**: Operações de configuração global (`sp_configure`) e gerenciamento de jobs via SQL Server Agent não são suportadas.

## Contribuição

Para propor melhorias ou relatar inconsistências técnicas, utilize o sistema de issues ou envie um Pull Request seguindo os padrões de código do projeto.

## Licença

Este projeto segue os termos de licenciamento padrão para conectores MCP. Para detalhes sobre uso e distribuição, consulte a licença no repositório oficial.

---
## Contato

**Alexandre**  
📧 alemiti@gmail.com
