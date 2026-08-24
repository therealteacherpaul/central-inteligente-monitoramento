# ClimaSeguro

Projeto desenvolvido para a disciplina de Integração de APIs da UniFECAF.

## Objetivo

Desenvolver uma plataforma de apoio à gestão de eventos institucionais, capaz de integrar serviços externos por meio de APIs, consultar dados climáticos, classificar riscos, armazenar informações e emitir alertas para responsáveis pelos eventos.

O sistema permite o cadastro de eventos, autenticação de usuários, análise climática e acompanhamento de status de risco.

## Arquitetura Atual

Usuário → Lovable → Supabase → Make → Nominatim → Open-Meteo → Supabase / Airtable → E-mail

## Tecnologias

- Lovable
- Supabase
- Make
- Airtable
- OpenStreetMap Nominatim API
- Open-Meteo API
- Gmail
- GitHub

## Funcionalidades Implementadas

- Autenticação por e-mail e senha com Supabase
- Confirmação de cadastro por e-mail
- Rotas protegidas
- Cadastro de perfil institucional
- RLS para isolamento de dados por usuário
- Estrutura da tabela de eventos no Supabase
- Recebimento de dados por webhook
- Geocodificação de cidade
- Consulta de previsão climática
- Tratamento dos dados climáticos
- Classificação de risco climático
- Geração de recomendação
- Persistência complementar no Airtable
- Envio de alertas por e-mail

## Estrutura de Dados

### profiles

Armazena informações básicas da instituição associada ao usuário autenticado.

Principais campos:

- id
- institution_name
- created_at

### events

Armazena os eventos cadastrados e os dados climáticos processados.

Principais campos:

- id
- user_id
- name
- type
- city
- event_date
- event_time
- responsible_email
- country
- latitude
- longitude
- temperature
- apparent_temperature
- humidity
- rain_probability
- precipitation
- wind_speed
- weather_status
- recommendation
- forecast_status
- processing_status
- weather_updated_at
- created_at
- updated_at

## Segurança

O Supabase Row Level Security (RLS) é utilizado para garantir que cada usuário visualize e altere apenas os próprios dados.

As rotas internas da aplicação também exigem autenticação.

## Fluxo de Processamento Climático

Evento cadastrado
→ processamento iniciado
→ geocodificação com Nominatim
→ consulta climática com Open-Meteo
→ classificação de risco
→ geração de recomendação
→ atualização dos dados
→ persistência
→ envio de alerta quando necessário

## Status de Processamento

Os eventos podem assumir os seguintes estados internos:

- pending
- processed
- error

## Status Climático

A classificação atual utiliza:

- Normal
- Atenção
- Crítico

## Evidências

Os screenshots da implementação estão disponíveis na pasta `screenshots/`.

As evidências incluem:

- funcionamento do webhook
- geocodificação
- consulta climática
- classificação de risco
- recomendação
- persistência no Airtable
- envio de alerta por e-mail
- autenticação no Supabase
- criação automática de profile
- RLS
- dashboard autenticado
- criação da tabela events

## Estratégia de Produto

O projeto evoluiu de uma central simples de consulta climática para uma solução voltada ao apoio à gestão de eventos institucionais.

O objetivo passou a ser reduzir riscos operacionais relacionados a condições climáticas, oferecendo cadastro de eventos, autenticação, monitoramento e alertas.

Essa mudança de escopo foi adotada para aumentar a utilidade prática e a coerência do projeto.

## Próximos Passos

- Conectar o formulário de Novo Evento à tabela events
- Substituir dados mockados por dados reais
- Atualizar dashboard e tela Meus Eventos
- Integrar criação de eventos ao Make
- Atualizar o Supabase com os resultados processados
- Simplificar a tela de configurações
- Implementar duas APIs REST próprias
- Documentar as APIs
- Realizar testes finais
- Produzir documentação e vídeo de apresentação

## Status

🚧 Em desenvolvimento