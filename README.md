# Bot de Clima no Telegram com N8N

Bot do Telegram que informa a temperatura atual de cidades brasileiras usando a API do OpenWeather.

## 🌟 Funcionalidades

- ✅ Recebe o nome da cidade via mensagem no Telegram
- ✅ Normaliza automaticamente caracteres acentuados (São Paulo → Sao Paulo)
- ✅ Consulta a API do OpenWeather em tempo real
- ✅ Retorna a temperatura atual em graus Celsius
- ✅ Tratamento de erros para cidades não encontradas
- ✅ Suporte para formato "Cidade,UF" (ex.: São Paulo,SP)

## 📋 Pré-requisitos

- N8N instalado e rodando
- Conta no Telegram (para criar o bot)
- Conta no OpenWeather (para obter API key gratuita)

## 🚀 Como Importar o Workflow

1. Abra o N8N
2. Clique em **"Workflows"** no menu lateral
3. Clique em **"Import from File"** ou use o atalho Ctrl+I
4. Selecione o arquivo `workflow-chatbot-telegram.json`
5. O workflow será importado com todos os nós configurados

## 🔑 Configuração de Credenciais

### 1. Telegram Bot Token

#### Como obter o token:

1. Abra o Telegram e procure por `@BotFather`
2. Envie o comando `/newbot`
3. Siga as instruções:
   - Escolha um nome para o bot (ex.: "Meu Bot do Clima")
   - Escolha um username (deve terminar em "bot", ex.: "MeuClimaBot")
4. Copie o **token de acesso** fornecido pelo BotFather
5. ⚠️ **IMPORTANTE:** Guarde este token em local seguro e nunca o compartilhe publicamente

#### Como configurar no N8N:

1. No N8N, vá em **"Credentials"** (menu lateral)
2. Clique em **"Add Credential"**
3. Procure por **"Telegram API"**
4. Cole o token no campo **"Access Token"**
5. Dê um nome para a credencial (ex.: "Bot do Clima")
6. Clique em **"Save"**
7. Nos nós **"Telegram Trigger"** e **"Send a text message"** do workflow, selecione essa credencial

### 2. OpenWeather API Key

#### Como obter a API Key:

1. Acesse https://openweathermap.org/
2. Crie uma conta gratuita
3. Confirme seu email
4. Acesse https://home.openweathermap.org/api_keys
5. Copie sua API Key (ou crie uma nova se necessário)
6. ⚠️ **Aguarde até 2 horas** para a chave ser ativada (geralmente é instantâneo, mas pode demorar)

#### Como configurar no N8N:

O workflow está configurado para usar a variável de ambiente `OPENWEATHER_API_KEY`.

**Opção 1: Configurar Variável de Ambiente (Recomendado)**

Se você tem acesso ao servidor onde o N8N está rodando:

```bash
# Linux/Mac
export OPENWEATHER_API_KEY=sua_chave_aqui

# Ou adicione ao arquivo de configuração do N8N
```

Depois reinicie o N8N para carregar a variável.

**Opção 2: Editar Diretamente no Workflow**

Se preferir não usar variáveis de ambiente:

1. Abra o workflow importado no N8N
2. Clique no nó **"HTTP Request"**
3. Vá até a seção **"Query Parameters"**
4. Encontre o parâmetro com nome `appid`
5. No campo **"Value"**, substitua `={{ $env.OPENWEATHER_API_KEY }}` pela sua chave real
6. Clique em **"Save"**

Exemplo:
```
Name: appid
Value: sua_api_key_aqui_sem_aspas
```

## ▶️ Como Executar

1. **Ative o workflow** clicando no toggle no canto superior direito (deve ficar verde/azul)
2. Abra o Telegram
3. Procure pelo seu bot usando o username que você criou
4. Envie uma mensagem com o nome da cidade

### Exemplos de Uso:

**Formato recomendado:** `Cidade, UF`

```
São Paulo, SP
Rio de Janeiro, RJ
Belo Horizonte, MG
Curitiba, PR
```

**Também funciona sem o estado:**
```
São Paulo
Rio de Janeiro
```

### Respostas Esperadas:

✅ **Cidade válida:**
```
🌤️ A temperatura em São Paulo é de 25°C.
```

❌ **Cidade inválida:**
```
❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP).
```

## 🏗️ Estrutura do Workflow

O workflow é composto por 8 nós:

1. **Telegram Trigger** - Escuta mensagens enviadas ao bot
2. **Edit Fields** - Captura o texto da mensagem do usuário
3. **Code in JavaScript** - Normaliza o texto:
   - Remove acentos (São → Sao)
   - Converte formato "Cidade,SP" para "Cidade,BR"
4. **HTTP Request** - Faz a chamada para a API do OpenWeather
5. **IF** - Verifica se a cidade foi encontrada (código 200)
6. **Edit Fields1** (TRUE) - Formata mensagem de sucesso com a temperatura
7. **Edit Fields2** (FALSE) - Formata mensagem de erro
8. **Send a text message** - Envia a resposta de volta ao usuário no Telegram

### Fluxo de Execução:

```
Usuário envia mensagem
    ↓
Telegram Trigger captura
    ↓
Edit Fields extrai o texto
    ↓
Code normaliza (remove acentos, ajusta formato)
    ↓
HTTP Request consulta OpenWeather API
    ↓
IF verifica se teve sucesso
    ├─ TRUE → Formata mensagem com temperatura → Envia ao usuário
    └─ FALSE → Formata mensagem de erro → Envia ao usuário
```

## 🔧 Resolução de Problemas

### Bot não responde

1. ✅ Verifique se o workflow está **ativado** (toggle verde)
2. ✅ Verifique se as **credenciais do Telegram** estão configuradas
3. ✅ Teste enviar `/start` para o bot primeiro
4. ✅ Verifique a aba **"Executions"** no N8N para ver se há erros

### Erro "City not found" para cidades válidas

1. ✅ Verifique se a **API Key do OpenWeather está configurada**
2. ✅ Aguarde até 2 horas após criar a conta (ativação da chave)
3. ✅ Tente o formato: `Cidade, UF` (com vírgula e espaço)
4. ✅ Teste a API diretamente no navegador:
   ```
   https://api.openweathermap.org/data/2.5/weather?q=Sao Paulo,BR&appid=SUA_CHAVE&units=metric&lang=pt_br
   ```

### Erro "Bad request: chat not found"

- ✅ O nó **"Send a text message"** precisa usar a mesma credencial do Telegram Trigger
- ✅ Verifique se o Chat ID está configurado como: `={{ $('Telegram Trigger').item.json.message.chat.id }}`

## 📚 Variáveis de Ambiente

| Variável | Descrição | Obrigatória |
|----------|-----------|-------------|
| `OPENWEATHER_API_KEY` | Chave da API do OpenWeather | Sim |

**Nota:** O token do Telegram é configurado como credencial no N8N, não como variável de ambiente.

## ⚙️ Configurações Importantes

- **Continue On Fail:** O nó HTTP Request está configurado com "Continue" no erro para permitir tratamento de cidades não encontradas
- **Query Parameters:** O HTTP Request usa parâmetros de query para enviar:
  - `q`: Nome da cidade
  - `units`: metric (para Celsius)
  - `lang`: pt_br (para nomes em português)
  - `appid`: Sua API key

## 🎓 Aprendizados do Projeto

Este projeto demonstra:
- ✅ Integração de APIs externas (OpenWeather)
- ✅ Automação com Telegram
- ✅ Manipulação de strings em JavaScript
- ✅ Tratamento de erros em workflows
- ✅ Uso de variáveis de ambiente
- ✅ Lógica condicional (IF nodes)
- ✅ Transformação de dados

## 📄 Licença

Este projeto é open source e está disponível para uso livre.

## 🤝 Contribuições

Sinta-se à vontade para fazer fork, sugerir melhorias ou reportar problemas!

## ⚠️ Aviso Importante

- ⚠️ Nunca compartilhe suas credenciais (tokens, API keys) publicamente
- ⚠️ O arquivo JSON deste repositório NÃO contém credenciais reais
- ⚠️ Você precisa configurar suas próprias credenciais para usar o bot
- ⚠️ A API gratuita do OpenWeather tem limite de chamadas (60 por minuto, 1.000.000 por mês)

---

**Desenvolvido como parte do Desafio Fase 2 - Rocketseat** 🚀
