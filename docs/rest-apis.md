# APIs REST Próprias — ClimaSeguro

## 1. Visão Geral

O ClimaSeguro utiliza APIs externas para geocodificação e obtenção de dados meteorológicos, mas também possui APIs REST próprias responsáveis por regras de negócio e fornecimento de dados para a interface da aplicação.

As APIs próprias foram implementadas como Supabase Edge Functions e utilizam autenticação baseada na sessão do usuário.

APIs implementadas:

- `POST /functions/v1/validate-event`
- `GET /functions/v1/events-summary`

Essas APIs utilizam o Supabase como fonte transacional da aplicação.

O Airtable possui papel complementar, sendo utilizado como camada operacional no-code para registro, auditoria e acompanhamento dos eventos processados pela automação.

---

# 2. Arquitetura

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

O processamento climático ocorre em uma camada complementar:

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
Supabase
   +
Airtable
   ↓
Alerta por e-mail
Responsabilidade do Supabase

O Supabase atua como fonte transacional principal da aplicação.

Responsabilidades:

autenticação de usuários;
armazenamento de perfis institucionais;
armazenamento dos eventos;
controle de acesso com Row Level Security;
fornecimento de dados para o frontend;
fonte de dados das APIs próprias.
Responsabilidade do Airtable

O Airtable é utilizado como camada complementar de integração e auditoria operacional.

Responsabilidades:

espelhamento dos resultados processados;
acompanhamento visual das automações;
conferência operacional;
integração no-code com o fluxo do Make.

Dessa forma, Supabase e Airtable possuem responsabilidades diferentes e não representam uma duplicação acidental da arquitetura.

3. Autenticação

As APIs exigem que o usuário esteja autenticado pelo Supabase.

As requisições utilizam os seguintes headers:

Authorization: Bearer <SUPABASE_ACCESS_TOKEN>
apikey: <SUPABASE_PUBLISHABLE_KEY>

O JWT representa a sessão do usuário autenticado.

As Edge Functions utilizam contexto de usuário autenticado e trabalham em conjunto com as políticas de Row Level Security do Supabase.

Isso garante que cada usuário tenha acesso somente aos dados permitidos pelas políticas da aplicação.

4. API Validate Event
Endpoint
POST /functions/v1/validate-event
Objetivo

Centralizar a validação dos dados de um evento antes de seu cadastro e processamento.

A validação não fica exclusivamente sob responsabilidade do frontend, permitindo que a mesma regra de negócio seja utilizada por diferentes clientes.

Request
{
  "name": "Festival de Primavera",
  "type": "Externo",
  "city": "Osaka",
  "event_date": "2026-08-25",
  "event_time": "14:00",
  "responsible_email": "responsavel@exemplo.com"
}
Campos
Campo	Tipo	Obrigatório	Regra
name	string	Sim	Não pode estar vazio
type	string	Sim	Deve ser Interno ou Externo
city	string	Sim	Não pode estar vazio
event_date	string	Sim	Formato YYYY-MM-DD
event_time	string	Sim	Formato HH:MM
responsible_email	string	Sim	Deve possuir formato válido de e-mail
Response de sucesso

HTTP 200

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
Response de erro de validação

HTTP 400

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
Códigos HTTP
Código	Significado
200	Evento válido
400	Dados inválidos ou corpo da requisição não processável
401	Usuário não autenticado ou credencial inválida
405	Método HTTP não permitido
500	Erro interno
Regras de negócio

A API valida:

nome do evento;
tipo do evento;
cidade;
formato da data;
formato do horário;
formato do e-mail do responsável.

O tipo do evento aceita apenas:

Interno
Externo
Uso previsto na aplicação
Novo Evento
    ↓
validate-event
    ↓
Dados válidos?
 ├── Não → exibe erros
 └── Sim → continua cadastro e processamento
5. API Events Summary
Endpoint
GET /functions/v1/events-summary
Objetivo

Fornecer dados consolidados dos eventos pertencentes ao usuário autenticado.

A API foi criada para alimentar indicadores do Dashboard sem exigir que o frontend faça os cálculos localmente.

Request

A requisição não possui body.

São necessários somente os headers de autenticação.

Response

HTTP 200

{
  "user": {
    "id": "uuid-do-usuario",
    "email": "usuario@exemplo.com"
  },
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

No teste inicial, os valores retornados foram 0 porque a tabela events ainda não possuía registros.

Esse resultado confirma que a função:

autenticou o usuário;
acessou a tabela events;
respeitou as políticas de segurança;
calculou o resumo;
retornou uma resposta JSON válida.
Estrutura do resumo
total_events

Quantidade total de eventos visíveis ao usuário autenticado.

future_events

Quantidade de eventos cuja data é igual ou posterior à data atual.

risk_summary.normal

Quantidade de eventos classificados como Normal.

risk_summary.attention

Quantidade de eventos classificados como Atenção.

risk_summary.critical

Quantidade de eventos classificados como Crítico.

risk_summary.pending

Quantidade de eventos ainda aguardando processamento climático.

next_event

Primeiro evento futuro encontrado na ordenação por data e horário.

Códigos HTTP
Código	Significado
200	Resumo retornado com sucesso
401	Usuário não autenticado ou credencial inválida
405	Método HTTP não permitido
500	Erro ao consultar ou gerar o resumo
Uso previsto na aplicação
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
6. Segurança

As duas APIs utilizam autenticação de usuário.

A tabela events possui Row Level Security.

As políticas implementadas garantem que um usuário autenticado possa:

visualizar somente os próprios eventos;
cadastrar somente eventos associados à própria conta;
atualizar somente os próprios eventos;
excluir somente os próprios eventos.

A identificação do usuário não é recebida por parâmetro da URL.

Ela é obtida através do token de autenticação.

Isso evita estruturas inseguras como:

GET /events?user_id=123

nas quais um cliente poderia tentar alterar manualmente o identificador do usuário.

7. APIs Próprias e APIs Externas

O projeto trabalha com dois tipos de integração.

APIs externas consumidas
OpenStreetMap Nominatim

Utilizada para transformar uma cidade em coordenadas geográficas.

Open-Meteo

Utilizada para obtenção da previsão meteorológica.

APIs próprias desenvolvidas
Validate Event

Responsável pela validação das regras de negócio do cadastro de eventos.

Events Summary

Responsável pela consulta e consolidação dos dados dos eventos para consumo pelo frontend.

A arquitetura demonstra, portanto:

Consumo de APIs externas
+
Desenvolvimento e disponibilização de APIs próprias
8. Testes Realizados

As funções foram testadas através de chamadas HTTP utilizando curl.

Teste da Validate Event

Foram testados cenários válidos e inválidos.

Cenário válido

Resultado:

{
  "valid": true,
  "message": "Evento válido."
}
Cenário inválido

Foram enviados propositalmente:

tipo inválido;
cidade vazia;
data fora do formato esperado;
horário inválido;
e-mail inválido.

A API retornou corretamente os erros de validação.

Exemplo:

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
Teste da Events Summary

A API foi executada utilizando um JWT válido de usuário autenticado.

Resultado inicial:

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

A resposta com valores zerados era esperada porque a tabela events ainda estava vazia no momento do teste.

9. Evidências

As evidências técnicas da implementação são mantidas na pasta:

screenshots/

Entre as evidências estão registros da:

criação das Edge Functions;
execução da API de validação;
rejeição de dados inválidos;
execução autenticada da API de resumo;
resposta JSON da API;
estrutura da tabela events;
políticas de Row Level Security.
10. Integração Futura com o Frontend

As APIs serão integradas diretamente à interface do ClimaSeguro.

Fluxo previsto:

Novo Evento
   ↓
validate-event
   ↓
Supabase events
   ↓
Make
   ↓
Nominatim
   ↓
Open-Meteo
   ↓
Classificação de risco
   ↓
Supabase
   ↓
events-summary
   ↓
Dashboard

A API validate-event será utilizada no formulário de criação de eventos.

A API events-summary será utilizada para substituir os valores mockados atualmente exibidos no Dashboard.

11. Papel do Supabase e do Airtable

O Supabase é a fonte de verdade transacional do ClimaSeguro.

Ele é responsável por:

autenticação;
usuários;
perfis;
eventos;
segurança;
APIs próprias;
dados utilizados pelo frontend.

O Airtable é mantido intencionalmente como uma camada complementar de integração e auditoria operacional.

Ele permite:

visualizar resultados processados;
acompanhar automações;
conferir dados sem acessar diretamente o banco principal;
preservar uma camada no-code de operação.

Essa separação foi adotada como decisão arquitetural do projeto.

12. Status Atual

As duas APIs REST próprias já foram:

implementadas;
publicadas;
autenticadas;
testadas;
validadas por chamadas HTTP.

APIs disponíveis:

POST /functions/v1/validate-event
GET  /functions/v1/events-summary