# APIs REST Próprias — ClimaSeguro

## 1. Visão Geral

O ClimaSeguro utiliza APIs externas para geocodificação e obtenção de dados meteorológicos, mas também possui APIs REST próprias responsáveis por regras de negócio e fornecimento de dados para a interface da aplicação.

As APIs próprias foram implementadas como Supabase Edge Functions e utilizam autenticação baseada na sessão do usuário.

APIs implementadas:

- `POST /functions/v1/validate-event`
- `GET /functions/v1/events-summary`

Essas APIs utilizam o Supabase como fonte transacional principal da aplicação.

O Airtable possui papel complementar, sendo utilizado como camada operacional no-code para registro, auditoria e acompanhamento dos eventos processados pela automação.

---

## 2. Arquitetura

Fluxo principal das APIs próprias:

```text
Usuário
   ↓
Lovable
   ↓
Supabase Auth
   ↓
JWT
   ↓
APIs REST próprias
   ↓
Supabase Database
```

O processamento climático ocorre em uma camada complementar:

```text
Evento
   ↓
Make
   ↓
Nominatim
   ↓
Open-Meteo
   ↓
Classificação de risco
   ↓
Airtable
   ↓
Supabase
   ↓
Alerta por e-mail
```

### Responsabilidade do Supabase

O Supabase atua como fonte transacional principal da aplicação.

Responsabilidades:

- autenticação de usuários;
- armazenamento de perfis institucionais;
- armazenamento dos eventos;
- controle de acesso com Row Level Security;
- fornecimento de dados para o frontend;
- fonte de dados das APIs próprias;
- armazenamento dos resultados processados pela automação.

### Responsabilidade do Airtable

O Airtable é utilizado como camada complementar de integração e auditoria operacional.

Responsabilidades:

- espelhamento dos resultados processados;
- acompanhamento visual das automações;
- conferência operacional;
- integração no-code com o fluxo do Make.

Dessa forma, Supabase e Airtable possuem responsabilidades diferentes e não representam uma duplicação acidental da arquitetura.

---

## 3. Autenticação

As APIs exigem que o usuário esteja autenticado pelo Supabase.

As requisições utilizam os seguintes headers:

```http
Authorization: Bearer <SUPABASE_ACCESS_TOKEN>
apikey: <SUPABASE_PUBLISHABLE_KEY>
```

O JWT representa a sessão do usuário autenticado.

As Edge Functions utilizam contexto de usuário autenticado e trabalham em conjunto com as políticas de Row Level Security do Supabase.

Isso garante que cada usuário tenha acesso somente aos dados permitidos pelas políticas da aplicação.

---

## 4. API Validate Event

### Endpoint

```http
POST /functions/v1/validate-event
```

### Objetivo

Centralizar a validação dos dados de um evento antes de seu cadastro e processamento.

A validação não fica exclusivamente sob responsabilidade do frontend, permitindo que a mesma regra de negócio seja utilizada por diferentes clientes.

### Request

```json
{
  "name": "Festival de Primavera",
  "type": "Externo",
  "city": "Osaka",
  "event_date": "2026-08-25",
  "event_time": "14:00",
  "responsible_email": "responsavel@exemplo.com"
}
```

### Campos

| Campo | Tipo | Obrigatório | Regra |
|---|---|---:|---|
| `name` | string | Sim | Não pode estar vazio |
| `type` | string | Sim | Deve ser `Interno` ou `Externo` |
| `city` | string | Sim | Não pode estar vazio |
| `event_date` | string | Sim | Formato `YYYY-MM-DD` |
| `event_time` | string | Sim | Formato `HH:MM` |
| `responsible_email` | string | Sim | Deve possuir formato válido de e-mail |

### Response de sucesso

HTTP `200`

```json
{
  "valid": true,
  "message": "Evento válido.",
  "user": {
    "id": "uuid-do-usuario",
    "email": "usuario@exemplo.com"
  },
  "data": {
    "name": "Festival de Primavera",
    "type": "Externo",
    "city": "Osaka",
    "event_date": "2026-08-25",
    "event_time": "14:00",
    "responsible_email": "responsavel@exemplo.com"
  }
}
```

### Response de erro de validação

HTTP `400`

```json
{
  "valid": false,
  "message": "Dados do evento inválidos.",
  "errors": [
    "Tipo deve ser Interno ou Externo.",
    "Cidade é obrigatória.",
    "Data do evento deve estar no formato YYYY-MM-DD.",
    "Horário do evento deve estar no formato HH:MM.",
    "E-mail do responsável é inválido."
  ]
}
```

### Códigos HTTP

| Código | Significado |
|---:|---|
| `200` | Evento válido |
| `400` | Dados inválidos ou corpo da requisição não processável |
| `401` | Usuário não autenticado ou credencial inválida |
| `405` | Método HTTP não permitido |
| `500` | Erro interno |

### Regras de negócio

A API valida:

- nome do evento;
- tipo do evento;
- cidade;
- formato da data;
- formato do horário;
- formato do e-mail do responsável.

O tipo do evento aceita apenas:

```text
Interno
Externo
```

### Uso na aplicação

```text
Novo Evento
    ↓
validate-event
    ↓
Dados válidos?
 ├── Não → exibe erros
 └── Sim → continua cadastro e processamento
```

---

## 5. API Events Summary

### Endpoint

```http
GET /functions/v1/events-summary
```

### Objetivo

Fornecer dados consolidados dos eventos pertencentes ao usuário autenticado.

A API foi criada para alimentar os indicadores do Dashboard sem exigir que o frontend faça os cálculos localmente.

### Request

A requisição não possui body.

São necessários somente os headers de autenticação.

### Response

Exemplo de resposta após a integração com eventos reais:

```json
{
  "user": {
    "id": "uuid-do-usuario",
    "email": "usuario@exemplo.com"
  },
  "total_events": 2,
  "future_events": 1,
  "risk_summary": {
    "normal": 0,
    "attention": 2,
    "critical": 0,
    "pending": 0
  },
  "next_event": {
    "id": "uuid-do-evento",
    "name": "Teste Frontend ClimaSeguro",
    "city": "Anjo-Shi",
    "event_date": "2026-08-26",
    "event_time": "08:00:00",
    "weather_status": "Atenção",
    "processing_status": "processed"
  }
}
```

### Estrutura do resumo

#### `total_events`

Quantidade total de eventos visíveis ao usuário autenticado.

#### `future_events`

Quantidade de eventos que ainda não ocorreram.

A lógica considera:

- data do evento;
- horário do evento;
- fuso horário adotado no projeto.

Durante os testes, a função foi ajustada para utilizar o fuso:

```text
Asia/Tokyo
```

Isso evita que um evento já ocorrido no mesmo dia continue sendo contabilizado como futuro.

#### `risk_summary.normal`

Quantidade de eventos classificados como `Normal`.

#### `risk_summary.attention`

Quantidade de eventos classificados como `Atenção`.

#### `risk_summary.critical`

Quantidade de eventos classificados como `Crítico`.

#### `risk_summary.pending`

Quantidade de eventos ainda aguardando processamento climático.

#### `next_event`

Primeiro evento futuro encontrado na ordenação por data e horário.

### Códigos HTTP

| Código | Significado |
|---:|---|
| `200` | Resumo retornado com sucesso |
| `401` | Usuário não autenticado ou credencial inválida |
| `405` | Método HTTP não permitido |
| `500` | Erro ao consultar ou gerar o resumo |

### Uso na aplicação

```text
Dashboard
   ↓
events-summary
   ↓
Supabase events
   ↓
Total de eventos
Eventos futuros
Normal
Atenção
Crítico
Pendentes
Próximo evento
```

---

## 6. Segurança

As duas APIs utilizam autenticação de usuário.

A tabela `events` possui Row Level Security.

As políticas implementadas garantem que um usuário autenticado possa:

- visualizar somente os próprios eventos;
- cadastrar somente eventos associados à própria conta;
- atualizar somente os próprios eventos;
- excluir somente os próprios eventos.

A identificação do usuário não é recebida por parâmetro da URL.

Ela é obtida através do token de autenticação.

Isso evita estruturas inseguras como:

```text
GET /events?user_id=123
```

nas quais um cliente poderia tentar alterar manualmente o identificador do usuário.

O frontend não utiliza `service_role` ou outras credenciais administrativas.

Credenciais server-side são mantidas exclusivamente em componentes de backend, como o fluxo do Make.

---

## 7. APIs Próprias e APIs Externas

O projeto trabalha com dois tipos de integração.

### APIs externas consumidas

#### OpenStreetMap Nominatim

Utilizada para transformar uma cidade em coordenadas geográficas.

Fluxo:

```text
Cidade
↓
Nominatim
↓
Latitude + Longitude
```

#### Open-Meteo

Utilizada para obtenção da previsão meteorológica.

Entre os dados consumidos estão:

- temperatura;
- sensação térmica;
- umidade;
- probabilidade de chuva;
- precipitação;
- velocidade do vento.

### APIs próprias desenvolvidas

#### Validate Event

Responsável pela validação das regras de negócio do cadastro de eventos.

#### Events Summary

Responsável pela consulta e consolidação dos dados dos eventos para consumo pelo frontend.

A arquitetura demonstra, portanto:

```text
Consumo de APIs externas
+
Desenvolvimento e disponibilização de APIs próprias
```

---

## 8. Testes Realizados

As funções foram testadas através de chamadas HTTP utilizando `curl`.

### Teste da Validate Event

Foram testados cenários válidos e inválidos.

#### Cenário válido

Resultado:

```json
{
  "valid": true,
  "message": "Evento válido."
}
```

#### Cenário inválido

Foram enviados propositalmente:

- tipo inválido;
- cidade vazia;
- data fora do formato esperado;
- horário inválido;
- e-mail inválido.

A API retornou corretamente os erros de validação.

Exemplo:

```json
{
  "valid": false,
  "message": "Dados do evento inválidos.",
  "errors": [
    "Tipo deve ser Interno ou Externo.",
    "Cidade é obrigatória.",
    "Data do evento deve estar no formato YYYY-MM-DD.",
    "Horário do evento deve estar no formato HH:MM.",
    "E-mail do responsável é inválido."
  ]
}
```

### Teste inicial da Events Summary

No primeiro teste, a tabela `events` ainda estava vazia.

Resultado:

```json
{
  "total_events": 0,
  "future_events": 0,
  "risk_summary": {
    "normal": 0,
    "attention": 0,
    "critical": 0,
    "pending": 0
  },
  "next_event": null
}
```

Esse resultado era esperado naquele estágio do desenvolvimento.

### Teste com dados reais

Após a integração do frontend e a criação de eventos reais, a API passou a retornar dados consolidados do usuário autenticado.

Exemplo validado:

```json
{
  "total_events": 2,
  "future_events": 1,
  "risk_summary": {
    "normal": 0,
    "attention": 2,
    "critical": 0,
    "pending": 0
  },
  "next_event": {
    "name": "Teste Frontend ClimaSeguro",
    "city": "Anjo-Shi",
    "event_date": "2026-08-26",
    "event_time": "08:00:00",
    "weather_status": "Atenção",
    "processing_status": "processed"
  }
}
```

---

## 9. Evidências

As evidências técnicas da implementação são mantidas na pasta:

```text
screenshots/
```

Entre as evidências estão registros da:

- criação das Edge Functions;
- execução da API de validação;
- rejeição de dados inválidos;
- execução autenticada da API de resumo;
- resposta JSON das APIs;
- estrutura da tabela `events`;
- políticas de Row Level Security;
- inclusão do `event_id` no webhook;
- integração Make → Supabase;
- criação de evento real pelo Lovable;
- atualização automática do evento;
- exibição dos eventos reais;
- Dashboard utilizando dados reais;
- correção de binding do card Atenção;
- detalhes meteorológicos do evento;
- envio de alerta por e-mail.

---

## 10. Atualização do Supabase pelo Make

O fluxo de automação recebe o `event_id` gerado pelo Supabase.

Esse identificador permite atualizar exatamente o mesmo registro após o processamento climático.

Fluxo:

```text
Evento criado no Supabase
↓
event_id
↓
Webhook do Make
↓
Nominatim
↓
Open-Meteo
↓
Classificação de risco
↓
Recomendação
↓
PATCH no Supabase
```

Exemplo conceitual da atualização:

```http
PATCH /rest/v1/events?id=eq.<event_id>
```

Entre os campos atualizados estão:

- `country`
- `latitude`
- `longitude`
- `temperature`
- `apparent_temperature`
- `humidity`
- `rain_probability`
- `precipitation`
- `wind_speed`
- `weather_status`
- `recommendation`
- `forecast_status`
- `processing_status`
- `weather_updated_at`

Após um processamento bem-sucedido:

```text
processing_status = processed
```

---

## 11. Papel do Supabase e do Airtable

O Supabase é a fonte de verdade transacional do ClimaSeguro.

Ele é responsável por:

- autenticação;
- usuários;
- perfis;
- eventos;
- segurança;
- Row Level Security;
- APIs próprias;
- dados utilizados pelo frontend;
- resultados processados.

O Airtable é mantido intencionalmente como uma camada complementar de integração e auditoria operacional.

Ele permite:

- visualizar resultados processados;
- acompanhar automações;
- conferir dados sem acessar diretamente o banco principal;
- preservar uma camada no-code de operação.

Essa separação foi adotada como decisão arquitetural do projeto.

Supabase e Airtable não representam duplicação acidental.

---

## 12. Fluxo Ponta a Ponta

O fluxo funcional atualmente implementado é:

```text
Usuário
↓
Lovable
↓
validate-event
↓
Supabase events
↓
UUID do evento
↓
event_id
↓
Make
↓
Nominatim
↓
Open-Meteo
↓
Tratamento dos dados
↓
Classificação de risco
↓
Recomendação
↓
Airtable
↓
PATCH no Supabase
↓
Atualização do evento
↓
Filtro de alerta
↓
Gmail
↓
Lovable exibe o resultado
```

Esse fluxo foi validado com um evento criado diretamente pela interface.

O evento foi:

- validado pela API própria;
- inserido no Supabase;
- processado pelo Make;
- enriquecido com dados meteorológicos;
- classificado como risco climático;
- atualizado no mesmo registro;
- exibido novamente no frontend;
- utilizado no Dashboard;
- enviado por e-mail quando o risco exigiu alerta.

---

## 13. Integração das APIs com o Frontend

As APIs REST próprias passaram a ser utilizadas diretamente pela interface do ClimaSeguro.

### Validate Event no frontend

A tela `Novo Evento` utiliza:

```http
POST /functions/v1/validate-event
```

Fluxo implementado:

```text
Usuário preenche formulário
↓
validate-event
↓
Dados válidos?
├── Não → exibe erros e interrompe
└── Sim → continua
        ↓
INSERT em public.events
        ↓
UUID do evento é retornado
        ↓
event_id é enviado ao webhook do Make
```

Quando a API retorna erros de validação, o evento não é inserido no banco.

Quando os dados são válidos:

1. o frontend insere o evento em `public.events`;
2. o registro recebe `processing_status = pending`;
3. o UUID real é obtido;
4. o UUID é enviado ao Make como `event_id`;
5. o usuário é redirecionado para Meus Eventos;
6. o frontend informa que a previsão está sendo processada.

### Meus Eventos

A tela `Meus Eventos` deixou de utilizar mocks.

Os registros são carregados diretamente de:

```text
public.events
```

A consulta respeita as policies RLS do Supabase.

Estados apresentados:

```text
pending
→ Processando previsão

processed
→ Exibe weather_status

error
→ Erro no processamento
```

Após o Make atualizar o evento, a página passa a mostrar o resultado real após recarregamento.

### Detalhes do Evento

A tela de detalhes utiliza os dados reais armazenados no Supabase.

São exibidos:

- nome;
- cidade;
- data;
- horário;
- tipo;
- e-mail do responsável;
- status de processamento;
- status climático;
- temperatura;
- sensação térmica;
- umidade;
- probabilidade de chuva;
- precipitação;
- velocidade do vento;
- recomendação.

Os campos meteorológicos utilizados são:

```text
temperature
apparent_temperature
humidity
rain_probability
precipitation
wind_speed
weather_status
recommendation
```

### Events Summary no Dashboard

O Dashboard utiliza:

```http
GET /functions/v1/events-summary
```

A resposta alimenta diretamente:

- total de eventos;
- eventos futuros;
- eventos normais;
- eventos em atenção;
- eventos críticos;
- eventos pendentes;
- próximo evento.

O contrato utilizado pelo frontend é:

```text
summary.total_events
summary.future_events
summary.risk_summary.normal
summary.risk_summary.attention
summary.risk_summary.critical
summary.risk_summary.pending
summary.next_event
```

### Correção de binding no Dashboard

Durante os testes de integração foi identificado um erro no card `Atenção`.

O frontend tentava utilizar valores como:

```text
summary.attention
summary.atencao
```

Porém a API retorna o valor dentro de:

```text
summary.risk_summary.attention
```

O frontend foi corrigido para seguir exatamente o contrato da API.

Após a correção, o Dashboard passou a apresentar corretamente:

```text
Total de eventos: 2
Eventos futuros: 1
Normal: 0
Atenção: 2
Crítico: 0
Pendentes: 0
```

### Correção de eventos futuros

Durante a validação também foi identificado que eventos já ocorridos no mesmo dia ainda eram considerados futuros.

A causa era a comparação apenas pela data.

A Edge Function foi refinada para considerar:

```text
data + horário
```

e o fuso:

```text
Asia/Tokyo
```

Após o ajuste:

- o evento já ocorrido deixou de ser contado como futuro;
- `future_events` passou de `2` para `1`;
- `next_event` passou a apontar corretamente para o próximo evento real.

### Resultado da integração

As APIs deixaram de ser apenas endpoints isolados e passaram a integrar o fluxo funcional real da aplicação.

Fluxo validado:

```text
Lovable
↓
validate-event
↓
Supabase events
↓
event_id
↓
Make
↓
Nominatim
↓
Open-Meteo
↓
Risco + recomendação
↓
Airtable
↓
PATCH no Supabase
↓
Gmail
↓
Lovable exibe resultado atualizado
```

---

## 14. Configurações da Aplicação

A tela de Configurações inicialmente possuía limites climáticos editáveis para chuva e vento.

Esses campos foram removidos porque os limites são atualmente definidos pela lógica de backend.

A interface passou a informar explicitamente:

```text
Os limites climáticos são definidos pelo backend e não são editáveis aqui.
```

São exibidos apenas:

- nome da instituição;
- e-mail para notificações.

Esses campos são apresentados em modo somente leitura.

O botão `Salvar preferências` também foi removido porque não havia persistência associada a ele.

Essa decisão evita apresentar controles que aparentem possuir funcionalidade sem produzir efeito real.

---

## 15. Status Atual das APIs

As duas APIs REST próprias já foram:

- implementadas;
- publicadas;
- autenticadas;
- testadas isoladamente;
- integradas ao frontend;
- utilizadas com dados reais;
- validadas no fluxo ponta a ponta.

APIs disponíveis:

```text
POST /functions/v1/validate-event
GET  /functions/v1/events-summary
```

O fluxo atual demonstra:

```text
Consumo de APIs externas
+
Criação de APIs REST próprias
+
Autenticação
+
Banco de dados
+
Automação
+
Integração frontend/backend
```

As APIs fazem parte efetiva da solução ClimaSeguro e não são apenas demonstrações isoladas.