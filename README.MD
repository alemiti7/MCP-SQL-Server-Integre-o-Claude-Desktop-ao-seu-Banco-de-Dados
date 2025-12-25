
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
## 🤝 Contribuindo

Formas de contribuir:
- Sugerir melhorias e reportar bugs
- Compartilhar scripts de automação do SEI!

---

📞 Contato
Alexandre
📧 alemiti@gmail.com
⌨️ com ❤️ por [@alemiti7]([https://github.com/alemiti7]) 😊

---
