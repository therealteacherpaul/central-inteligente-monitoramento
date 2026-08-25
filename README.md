# ClimaSeguro

Projeto acadêmico desenvolvido para a disciplina de Integração de APIs da UniFECAF, no contexto do curso de IA e Automação Digital.

## Objetivo

O ClimaSeguro é uma aplicação voltada ao apoio à gestão de eventos institucionais, com foco na análise de riscos climáticos.

A plataforma permite cadastrar eventos, consultar condições meteorológicas previstas, classificar o risco climático, gerar recomendações e emitir alertas aos responsáveis.

O projeto combina autenticação, banco de dados, automações, APIs externas, APIs REST próprias e uma interface web desenvolvida com ferramentas no-code e low-code.

---

## Problema

Instituições de ensino realizam eventos internos e externos que podem ser impactados por chuva, calor excessivo, vento forte e outras condições climáticas.

O ClimaSeguro foi criado para reduzir esse risco operacional, concentrando em um único fluxo:

- cadastro do evento;
- consulta da previsão;
- análise de risco;
- recomendação;
- armazenamento dos resultados;
- alerta ao responsável.

---

## Arquitetura Atual

Fluxo da aplicação:

```text
Usuário
   ↓
Lovable
   ↓
Supabase Auth
   ↓
Supabase Database
   ↓
APIs REST próprias

Fluxo de processamento climático:

Evento
   ↓
Make
   ↓
OpenStreetMap Nominatim
   ↓
Open-Meteo
   ↓
Classificação de risco
   ↓
Recomendação
   ↓
Supabase + Airtable
   ↓
Alerta por e-mail

Tecnologias
Lovable
Supabase Database
Supabase Auth
Supabase Edge Functions
Make
Airtable
OpenStreetMap Nominatim API
Open-Meteo API
Gmail
GitHub
Funcionalidades Implementadas
Autenticação por e-mail e senha com Supabase
Confirmação de cadastro por e-mail
Login e logout
Rotas protegidas
Cadastro automático de perfil institucional
Row Level Security no Supabase
Estrutura real da tabela de eventos
Webhook para recebimento dos dados do evento
Geocodificação de cidade
Consulta de previsão climática por data e horário
Tratamento dos dados meteorológicos
Classificação de risco climático
Geração de recomendação
Persistência complementar no Airtable
Envio de alertas por e-mail
Duas APIs REST próprias implementadas com Supabase Edge Functions
API de validação de eventos
API de resumo de eventos para o dashboard
Autenticação das APIs com JWT
Integração das APIs com as políticas RLS do Supabase
Banco de Dados
Tabela profiles

Armazena as informações básicas da instituição vinculada ao usuário autenticado.

Principais campos:

id
institution_name
created_at

O registro de perfil é criado automaticamente após o cadastro do usuário.

Tabela events

A tabela events será a fonte principal dos eventos cadastrados na aplicação.

Principais campos:

id
user_id
name
type
city
event_date
event_time
responsible_email
country
latitude
longitude
temperature
apparent_temperature
humidity
rain_probability
precipitation
wind_speed
weather_status
recommendation
forecast_status
processing_status
weather_updated_at
created_at
updated_at

O campo processing_status permite acompanhar o processamento do evento.

Estados previstos:

pending
processed
error
Segurança

O projeto utiliza Supabase Auth para autenticação dos usuários.

A tabela events possui Row Level Security.

As políticas implementadas garantem que cada usuário autenticado possa:

visualizar somente os próprios eventos;
cadastrar somente eventos vinculados à própria conta;
atualizar somente os próprios eventos;
excluir somente os próprios eventos.

As rotas internas do frontend também exigem autenticação.

As APIs REST próprias utilizam JWT do Supabase para identificar o usuário autenticado.

APIs Externas Utilizadas
OpenStreetMap Nominatim

Utilizada para transformar o nome da cidade em coordenadas geográficas.

Fluxo:

Cidade
↓
Nominatim
↓
Latitude + Longitude
Open-Meteo

Utilizada para consultar a previsão climática correspondente à cidade, data e horário do evento.

Entre os dados utilizados estão:

temperatura;
sensação térmica;
umidade;
probabilidade de chuva;
precipitação;
velocidade do vento.
APIs REST Próprias

Além de consumir APIs externas, o ClimaSeguro também possui APIs REST próprias.

As APIs foram implementadas utilizando Supabase Edge Functions.

Validate Event

Endpoint:

POST /functions/v1/validate-event

Responsável pela validação dos dados de um evento.

Valida:

nome;
tipo;
cidade;
data;
horário;
e-mail do responsável.

O campo type aceita somente:

Interno
Externo

A API também exige usuário autenticado.

Events Summary

Endpoint:

GET /functions/v1/events-summary

Responsável por gerar um resumo dos eventos do usuário autenticado.

A resposta inclui:

total de eventos;
eventos futuros;
eventos normais;
eventos em atenção;
eventos críticos;
eventos pendentes;
próximo evento.

Essa API foi criada para alimentar os indicadores do Dashboard.

Documentação das APIs

A documentação técnica completa das APIs REST próprias está disponível em:

docs/rest-apis.md

O documento apresenta:

endpoints;
métodos HTTP;
autenticação;
requests;
responses;
códigos HTTP;
regras de negócio;
testes;
arquitetura;
justificativas técnicas.
Classificação de Risco Climático

O ClimaSeguro classifica os eventos em três níveis:

Normal
Atenção
Crítico

A análise utiliza informações como:

temperatura;
sensação térmica;
chuva;
vento;
tipo do evento.

Eventos classificados como Externo possuem maior sensibilidade às condições climáticas.

Recomendações

Após a classificação, o sistema gera uma recomendação.

Exemplos:

Normal

Condições favoráveis no momento. Recomenda-se manter o acompanhamento até a realização do evento.

Atenção

Recomenda-se reforçar medidas preventivas, acompanhar a previsão e preparar alternativas para a atividade.

Crítico

Recomenda-se reavaliar a realização do evento, considerar mudança de horário, local coberto ou adiamento.

Alertas

Quando o risco climático exige atenção, o Make envia uma notificação por e-mail ao responsável informado no cadastro do evento.

O alerta apresenta:

nome do evento;
cidade;
data;
horário;
classificação;
informações climáticas;
recomendação.

Os e-mails são enviados utilizando uma conta exclusiva do projeto ClimaSeguro.

Supabase e Airtable

O uso conjunto de Supabase e Airtable é uma decisão arquitetural do projeto.

Supabase

O Supabase atua como fonte de verdade transacional do ClimaSeguro.

Responsabilidades:

autenticação;
usuários;
perfis;
eventos;
segurança;
Row Level Security;
APIs próprias;
dados utilizados pelo frontend.
Airtable

O Airtable é mantido intencionalmente como uma camada complementar de integração e auditoria operacional.

Responsabilidades:

espelhar dados processados;
facilitar a inspeção dos resultados;
acompanhar visualmente as automações;
permitir conferência operacional;
manter uma camada no-code de integração.

Dessa forma, Supabase e Airtable não representam duplicação acidental.

O Supabase atende às necessidades transacionais e de segurança da aplicação, enquanto o Airtable oferece visibilidade operacional e integração no-code.

Interface

O frontend foi desenvolvido no Lovable.

As principais telas são:

Login
Cadastro
Dashboard
Novo Evento
Meus Eventos
Configurações

A aplicação já possui autenticação real integrada ao Supabase.

Os dados dos eventos ainda estão em processo de migração dos mocks do frontend para a tabela real events.

Decisão de Escopo

O projeto inicialmente foi desenvolvido como uma central genérica de consulta climática.

Durante a evolução da solução, o escopo foi refinado para atender a um problema mais específico e relevante: o monitoramento de riscos climáticos em eventos institucionais.

Essa evolução deu origem ao ClimaSeguro.

A mudança permitiu adicionar:

contexto de negócio;
cadastro de eventos;
autenticação;
usuários institucionais;
classificação de risco;
recomendações;
alertas;
banco de dados;
APIs próprias.

A evolução foi realizada de forma incremental, preservando as integrações já desenvolvidas e ampliando gradualmente a arquitetura.

Evidências

As evidências técnicas do projeto estão organizadas na pasta:

screenshots/

Entre elas estão registros de:

webhook;
geocodificação;
consulta à Open-Meteo;
processamento das variáveis climáticas;
classificação de risco;
recomendação;
persistência no Airtable;
envio de alerta por e-mail;
telas do Lovable;
autenticação;
confirmação de e-mail;
Supabase Auth;
tabela profiles;
tabela events;
políticas RLS;
Supabase Edge Functions;
execução das APIs REST próprias.
Próximos Passos
Integrar o formulário de Novo Evento à tabela events
Utilizar a API validate-event no cadastro de eventos
Integrar o cadastro ao fluxo do Make
Atualizar o mesmo evento no Supabase após o processamento climático
Substituir os dados mockados da tela Meus Eventos
Utilizar a API events-summary no Dashboard
Simplificar a tela de Configurações
Manter somente configurações realmente utilizadas pelo backend
Integrar o Supabase ao fluxo de atualização do Make
Realizar testes completos de ponta a ponta
Atualizar a documentação final
Preparar o vídeo de apresentação acadêmica

## Status

🚧 Em desenvolvimento