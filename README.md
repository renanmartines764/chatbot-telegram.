# Chatbot de Temperatura no Telegram (n8n)

## Descrição
Chatbot do Telegram que informa a temperatura atual de qualquer cidade do Brasil usando a API OpenWeather. O usuário envia `Cidade,UF,BR` e recebe, por exemplo:
`🌤️ A temperatura em Belo Horizonte é de 25°C.`
Se a cidade não existir, a API falhar ou houver timeout, recebe:
`❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).`

## Pré-requisitos
- n8n (self-hosted ou Cloud)
- Bot do Telegram criado no @BotFather (token)
- Chave de API gratuita da OpenWeather: https://home.openweathermap.org/api_keys (chaves novas podem levar até 2 h para ativar)

## Como importar o workflow
1. No n8n, crie um workflow novo.
2. Menu `...` > **Import from File**.
3. Selecione `workflow-telegram-chatbot.json`.
4. Configure as credenciais e variáveis abaixo e ative (publique) o workflow.

## Configuração
O JSON exportado não contém credenciais nem tokens. Veja `.env.example`.

| Variável | Como configurar |
|----------|-----------------|
| `TELEGRAM_BOT_TOKEN` | Crie a credencial **Telegram API** no n8n (Credentials > New > Telegram API) com o token do @BotFather e associe aos 3 nós do Telegram: Telegram Trigger, Enviar temperatura e Enviar erro. |
| `OPENWEATHER_API_KEY` | Defina como variável de ambiente do n8n (`.env`/docker-compose) e reinicie. O nó HTTP lê `{{ $env.OPENWEATHER_API_KEY }}`. |

Exemplo (docker-compose): `environment: - OPENWEATHER_API_KEY=${OPENWEATHER_API_KEY}`

## Fluxo
1. **Telegram Trigger**: recebe mensagens de texto.
2. **Formatar entrada (queue)** (Set): grava `queue` sem espaços extras, em minúsculas e sem acentos.
3. **Consultar OpenWeather** (HTTP Request, "Send Query Parameters"): `q={{queue}}`, `units=metric`, `lang=pt_br`, `appid={{ $env.OPENWEATHER_API_KEY }}`. Timeout de 10 s; erros de conexão seguem para o IF.
4. **Resposta valida?** (IF): status 200 e `main.temp` presente.
5. **Formatar resposta** (Set): arredonda `main.temp` e monta a mensagem.
6. **Enviar temperatura** / **Enviar erro** (Telegram Send Message).

## Uso
Envie `Belo Horizonte,MG,BR` ou `São Paulo,SP,BR` ao bot.

## Troubleshooting
- **"access to env vars denied"**: o n8n Cloud bloqueia `$env`. Use self-hosted ou troque o `appid` por uma credencial OpenWeatherMap API.
- **401 / Invalid API key**: chave errada ou ainda não ativada.
- **Sempre responde "Cidade não encontrada"**: use `Cidade,UF,BR` e confira a chave.
- **Bot não responde**: o workflow precisa estar ativo, e cada token de bot só funciona em um workflow ou agente por vez.