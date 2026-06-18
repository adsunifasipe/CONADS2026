# Integração de Agentes de IA com n8n

## IV CONADS – Congresso Acadêmico de Análise e Desenvolvimento de Sistemas

### Objetivo

Neste laboratório vamos aprender a integrar Agentes de Inteligência Artificial utilizando o n8n, Telegram, Google Gemini, Google Sheets e APIs externas.

Ao final deste exercício o aluno será capaz de:

* Criar um chatbot utilizando IA.
* Integrar um bot do Telegram.
* Utilizar memória conversacional.
* Processar perguntas automaticamente.
* Consultar dados em APIs externas.
* Automatizar notificações para grupos.
* Utilizar ferramentas (Tools) dentro de Agentes de IA.

---

# Visão Geral do Projeto

O workflow possui três aplicações práticas:

## Aplicação 01 – Monitoramento de Formulários Google

Quando uma nova resposta é enviada para um Google Forms:

1. O Google Sheets Trigger detecta a nova linha.
2. Os dados são capturados.
3. Uma mensagem é enviada automaticamente para um grupo do Telegram.

---

## Aplicação 02 – Central de Atendimento do IV CONADS

Quando um usuário envia mensagem para o Bot do Telegram:

1. O Telegram Trigger recebe a mensagem.
2. O Agente de IA analisa o conteúdo.
3. O agente classifica a mensagem como:

   * pergunta
   * padrao
4. O usuário recebe uma resposta automática.
5. Se for uma pergunta, a equipe organizadora também é notificada em um grupo.

---

## Aplicação 03 – Consulta de Empresas por CNPJ

O usuário informa um CNPJ.

Exemplo:

```text
Consulte o CNPJ 12.345.678/0001-90
```

Fluxo:

1. O Telegram recebe a mensagem.
2. O Agente identifica o CNPJ.
3. O CNPJ é enviado para uma API externa.
4. A API retorna os dados da empresa.
5. O Agente monta uma resposta amigável.
6. A resposta é enviada ao usuário.

---

# Arquitetura da Solução

```text
Telegram
    ↓
Telegram Trigger
    ↓
AI Agent
    ↓
Google Gemini
    ↓
HTTP Request Tool
    ↓
BrasilAPI
    ↓
Telegram
```

---

# Importando o Workflow

## Passo 1

Abra o n8n.

## Passo 2

Clique em:

```text
Workflows
```

## Passo 3

Selecione:

```text
Import from File
```

## Passo 4

Escolha o arquivo:

```text
ExemploCONADS.json
```

## Passo 5

Confirme a importação.

Após importar, o fluxo aparecerá exatamente como foi desenvolvido durante o minicurso.

---

# Configurando as Credenciais

## Telegram

### Criando o Bot

Abra:

https://t.me/BotFather

Comandos:

```text
/newbot
```

Informe:

* Nome do bot
* Username do bot

O BotFather fornecerá um Token.

Exemplo:

```text
123456789:AAExemploToken
```

---

### Configurando no n8n

Criar credencial:

```text
Telegram API
```

Preencher:

```text
Access Token
```

Cole o token fornecido pelo BotFather.

---

# Google Gemini

Acesse:

https://aistudio.google.com

Criar uma API Key.

No n8n:

```text
Credentials
→ Google Gemini(PaLM) API
```

Informe:

```text
API Key
```

Salvar.

---

# Google Sheets

Criar uma credencial:

```text
Google Sheets OAuth2
```

Realizar login na conta Google.

Permitir acesso às planilhas.

Selecionar a planilha desejada.

---

# Explicação dos Nós

## Telegram Trigger

Responsável por escutar novas mensagens enviadas para o bot.

Saída:

```json
{
  "message": {
    "text": "Olá"
  }
}
```

---

## AI Agent

Responsável por:

* Interpretar mensagens.
* Tomar decisões.
* Chamar ferramentas.
* Produzir respostas.

Neste projeto existem dois agentes:

### Agente de Atendimento

Classifica mensagens.

Retorna:

```json
{
  "tipo": "pergunta",
  "mensagem": "Opa, sugestão ou pergunta enviada!!"
}
```

ou

```json
{
  "tipo": "padrao",
  "mensagem": "Olá! Seja bem-vindo..."
}
```

---

### Agente de Consulta de CNPJ

Responsável por:

* Encontrar o CNPJ na frase.
* Remover formatação.
* Consultar API.
* Responder ao usuário.

---

## Google Gemini Chat Model

Modelo utilizado:

```text
gemini-2.5-flash-lite
```

Função:

* Processamento de linguagem natural.
* Interpretação das mensagens.

---

## Simple Memory

Permite que o agente lembre do histórico da conversa.

Exemplo:

Usuário:

```text
Meu nome é João
```

Depois:

```text
Qual é meu nome?
```

O agente consegue responder utilizando o histórico.

---

## HTTP Request Tool

Ferramenta utilizada pelo Agente.

URL:

```text
https://brasilapi.com.br/api/cnpj/v1/{{$fromAI('cnpj')}}
```

Observe:

```javascript
{{$fromAI('cnpj')}}
```

Este valor é extraído automaticamente pelo Agente.

Exemplo:

Pergunta:

```text
Consulte o CNPJ 12345678000190
```

URL gerada:

```text
https://brasilapi.com.br/api/cnpj/v1/12345678000190
```

---

# Como o Agent Chama uma Tool

Descrição da Tool:

```text
Use esta ferramenta sempre que o usuário informar um CNPJ ou pedir informações de uma empresa.

Parâmetros:
- cnpj: CNPJ somente números.
```

O próprio modelo decide quando utilizar a ferramenta.

Não é necessário programar condições.

---

# Exemplo Prático

Pergunta:

```text
Qual o CNPJ 19131243000197?
```

O agente:

1. Extrai o número.
2. Chama a ferramenta.
3. Consulta a BrasilAPI.
4. Monta a resposta.

Resposta:

```text
Razão Social: Empresa Exemplo LTDA

Situação:
ATIVA

Município:
Sinop - MT
```

---

# Estrutura de Memória Conversacional

Identificador utilizado:

```javascript
{{ $json.message.from.id }}
```

Cada usuário possui sua própria memória.

Isso permite conversas independentes.

---

# Fluxo de Atendimento do CONADS

```text
Usuário
    ↓
Telegram
    ↓
AI Agent
    ↓
Classificação
    ↓

Pergunta?
    ↓ Sim
Grupo Organizador

    ↓ Não
Resposta padrão
```

---

# Boas Práticas

## Limitar respostas

Utilize prompts específicos.

Evite:

```text
Responda livremente.
```

Prefira:

```text
Retorne apenas JSON.
```

---

## Utilizar memória apenas quando necessário

Memória excessiva aumenta:

* custo
* consumo de tokens
* tempo de resposta

---

## Sempre validar dados externos

Antes de utilizar informações de APIs:

* verificar erros
* verificar campos nulos
* tratar exceções

---

# Possíveis Melhorias

* Integração com WhatsApp.
* Banco vetorial.
* RAG com PDFs.
* Consulta em banco de dados.
* Cadastro automático de participantes.
* Geração de certificados.
* Integração com Google Calendar.
* Agente para dúvidas sobre programação.

---

# Conclusão

O n8n permite criar agentes inteligentes sem programação complexa.

Combinando:

* Telegram
* Gemini
* APIs
* Google Sheets
* Ferramentas (Tools)

é possível construir soluções reais de automação e atendimento utilizando Inteligência Artificial em poucos minutos.

---

## IV CONADS

### Programando o seu futuro.
