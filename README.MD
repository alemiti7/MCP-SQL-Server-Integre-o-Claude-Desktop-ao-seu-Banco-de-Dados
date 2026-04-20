# 🔌 MCP SQL Server Connector — Claude Desktop Integration

![Claude Desktop](https://img.shields.io/badge/Claude_Desktop-MCP-D97757?style=flat&logo=anthropic&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-18+-339933?style=flat&logo=nodedotjs&logoColor=white)
![SQL Server](https://img.shields.io/badge/SQL_Server-2025-CC2927?style=flat&logo=microsoftsqlserver&logoColor=white)
![npm](https://img.shields.io/badge/npm-@modelcontextprotocol-CB3837?style=flat&logo=npm&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-10+-0078D4?style=flat&logo=windows&logoColor=white)
![macOS](https://img.shields.io/badge/macOS-11+-000000?style=flat&logo=apple&logoColor=white)
![Licença](https://img.shields.io/badge/Licença-MCP_Standard-blue?style=flat)

> Documentação técnica para integração do **Claude Desktop** com instâncias **Microsoft SQL Server** via **Model Context Protocol (MCP)**.
> 
> 🗓️ *Última atualização: Dezembro 2025* — SQL Server testado: Microsoft SQL Server 2025 (RTM) Enterprise Developer Edition

---

## Sumário

- [Funcionalidades](#funcionalidades)
- [Arquitetura](#arquitetura)
- [Instalação](#instalação)
- [Configuração](#configuração)
- [Limitações](#limitações)
- [Contribuição](#contribuição)
- [Licença](#licença)
- [Contato](#contato)

---

## ✨ Funcionalidades

| Categoria | Descrição |
|---|---|
| **DML** | `SELECT`, JOINs complexos, `INSERT`, `UPDATE`, `DELETE` e `MERGE` com suporte a transações |
| **DDL** | Criação, alteração e remoção de tabelas, views, índices, stored procedures e triggers |
| **Metadados** | Consulta ao `INFORMATION_SCHEMA`, estatísticas e análise de planos via `SET STATISTICS` |
| **Segurança** | Validação de permissões via roles: `db_owner`, `db_ddladmin`, `db_datareader`, `db_datawriter` |

---

## 🏗️ Arquitetura

A integração utiliza o protocolo MCP para permitir que o Claude execute comandos SQL em tempo real através de um servidor local Node.js.

```
Claude Desktop  ──(MCP)──▶  Node.js Server  ──(TCP 1433)──▶  SQL Server
```

| Componente | Detalhe |
|---|---|
| **Protocolo** | Model Context Protocol (MCP) |
| **Runtime** | Node.js v18 ou superior |
| **Conectividade** | TCP/IP via porta 1433 |
| **Sistemas Operacionais** | Windows 10+ e macOS 11+ |

---

## 🛠️ Instalação

### Pré-requisitos

- Node.js v18 ou superior instalado
- Instância SQL Server ativa e acessível (Local, Docker ou Remoto)
- Porta `1433` liberada no firewall do sistema

### Procedimento

Instale o pacote do servidor globalmente via npm:

```bash
npm install -g @modelcontextprotocol/server-mssql
```

---

## ⚙️ Configuração

Edite o arquivo `claude_desktop_config.json` no diretório correspondente ao seu sistema:

| Sistema | Caminho |
|---|---|
| **Windows** | `%APPDATA%\Claude\claude_desktop_config.json` |
| **macOS** | `~/Library/Application Support/Claude/claude_desktop_config.json` |

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

> ⚠️ **Atenção:** Substitua `SUA_STRING_DE_CONEXAO` pela string de conexão real da sua instância SQL Server antes de salvar o arquivo.

---

## ⚠️ Limitações

| Restrição | Detalhe |
|---|---|
| **Sistema de Arquivos** | Sem suporte a `BACKUP DATABASE`, `RESTORE` ou `BULK INSERT` (requerem acesso físico ao disco) |
| **Timeout** | Limite de **15 segundos** por requisição |
| **Administração** | `sp_configure` e gerenciamento de jobs via SQL Server Agent não são suportados |

---

## 🤝 Contribuição

Contribuições são bem-vindas! Para propor melhorias ou relatar inconsistências técnicas:

- 🐛 Abra uma [issue](../../issues) descrevendo o problema
- 💡 Envie um [Pull Request](../../pulls) seguindo os padrões de código do projeto

---

## 📄 Licença

Este projeto segue os termos de licenciamento padrão para conectores MCP. Consulte a licença no repositório oficial para detalhes sobre uso e distribuição.

---

## 📞 Contato

**Alexandre**  
📧 [alemiti@gmail.com](mailto:alemiti@gmail.com)
