# ClimaSeguro

Sistema inteligente de monitoramento de risco climático para eventos educacionais.

Projeto acadêmico desenvolvido para a disciplina de **Integração de APIs** do curso de **IA e Automação Digital da UniFECAF**.

**Aluno:** Paulo Ricardo Takara Stefens  
**RA:** 232231  
**Curso:** IA e Automação Digital  
**Universidade:** UniFECAF  
**Disciplina:** Integração de APIs  

---

## Links do projeto

### Aplicação

https://clima-seguro-hub.lovable.app/

### Vídeo de apresentação

https://youtu.be/GZkEPbu2zKM

### Repositório GitHub

https://github.com/therealteacherpaul/central-inteligente-monitoramento/tree/main

### Airtable

https://airtable.com/appXI5pGDcJMTRiiS/shrIUsZzNh76EASDi

### Cenário público do Make

https://eu1.make.com/public/shared-scenario/q7AlqJqBRr6/clima-seguro

### Relatório teórico

O arquivo:

    CLIMASEGURO-Parte-Teorica.pdf

está versionado neste repositório e contém a documentação teórica, arquitetura, integrações, segurança, LGPD, ética, governança e principais evidências do projeto.

---

# 1. Visão geral

O **ClimaSeguro** é uma aplicação web criada para auxiliar instituições educacionais na tomada de decisão sobre eventos que podem ser impactados por condições climáticas adversas.

A aplicação permite cadastrar eventos contendo:

- nome do evento;
- tipo;
- cidade;
- data;
- horário;
- e-mail do responsável.

A partir dessas informações, o sistema consulta serviços externos de geolocalização e previsão meteorológica, processa os dados automaticamente e classifica o evento em três níveis:

- **Normal**
- **Atenção**
- **Crítico**

Além da classificação, o sistema:

- gera uma recomendação;
- registra os resultados;
- atualiza o Dashboard;
- mantém histórico;
- envia alertas por e-mail quando necessário.

---

# 2. Problema identificado

Instituições educacionais realizam frequentemente:

- atividades esportivas;
- festivais;
- feiras;
- excursões;
- cerimônias;
- eventos culturais;
- apresentações;
- eventos comunitários;
- atividades externas.

Essas atividades podem ser afetadas por fatores como:

- calor excessivo;
- sensação térmica elevada;
- chuva;
- precipitação;
- vento.

Normalmente, o acompanhamento dessas condições depende de consultas manuais e da interpretação do responsável.

O ClimaSeguro foi desenvolvido para automatizar esse processo e centralizar as informações necessárias para apoiar a decisão.

---

# 3. Objetivo

O projeto tem como objetivo criar uma solução capaz de:

1. autenticar usuários;
2. cadastrar eventos;
3. validar os dados informados;
4. localizar geograficamente a cidade;
5. consultar previsão meteorológica;
6. identificar as condições específicas para a data e horário do evento;
7. tratar os dados recebidos;
8. classificar o risco;
9. gerar uma recomendação;
10. persistir os resultados;
11. atualizar o Dashboard;
12. enviar alertas quando necessário.

---

# 4. Arquitetura

O fluxo principal pode ser representado da seguinte forma:

    Usuário
       ↓
    Lovable
       ↓
    Supabase Auth
       ↓
    Supabase Database
       ↓
    API validate-event
       ↓
    Make.com Webhook
       ↓
    Nominatim / OpenStreetMap
       ↓
    Open-Meteo
       ↓
    Tratamento dos dados
       ↓
    Classificação de risco
       ↓
    Geração de recomendação
       ↓
    Airtable
       ↓
    Supabase
       ↓
    Gmail
       ↓
    API events-summary
       ↓
    Dashboard ClimaSeguro

---

# 5. Tecnologias utilizadas

## Lovable

Utilizado para desenvolvimento da interface web.

Principais telas:

- autenticação;
- Dashboard;
- Novo Evento;
- Meus Eventos;
- detalhes;
- arquivamento e restauração.

---

## Supabase

Utilizado como backend principal.

Responsável por:

- autenticação;
- usuários;
- banco de dados;
- perfis;
- eventos;
- Row Level Security;
- Edge Functions;
- APIs REST próprias.

---

## Make.com

Responsável pela principal automação do sistema.

O cenário executa:

1. recebimento dos dados via webhook;
2. geocodificação;
3. consulta meteorológica;
4. extração da previsão no horário do evento;
5. classificação do risco;
6. geração da recomendação;
7. persistência no Airtable;
8. atualização do Supabase;
9. envio condicional de e-mail.

O cenário está configurado para execução:

    Immediately as data arrives

Assim, o webhook pode iniciar automaticamente o processamento quando um novo evento é cadastrado.

---

## OpenStreetMap / Nominatim

Utilizado para geocodificação.

A cidade informada pelo usuário é convertida em:

- latitude;
- longitude;
- país.

Essas informações são utilizadas na consulta meteorológica.

---

## Open-Meteo

Utilizada para obtenção de previsão meteorológica horária.

São consultados dados como:

- temperatura;
- sensação térmica;
- umidade;
- probabilidade de chuva;
- precipitação;
- velocidade do vento.

---

## Airtable

Utilizado como camada complementar de registro operacional e auditoria.

O Airtable permite acompanhar os resultados processados pela automação em uma estrutura no-code de fácil inspeção.

---

## Gmail

Utilizado para envio automático de alertas.

Alertas são enviados quando o evento é classificado como:

- Atenção;
- Crítico.

Eventos Normais não geram alerta.

---

# 6. APIs utilizadas

O ClimaSeguro consome APIs externas e também possui APIs REST próprias.

## APIs externas

### Nominatim / OpenStreetMap

Responsável por transformar o nome da cidade em coordenadas geográficas.

### Open-Meteo

Responsável por fornecer os dados meteorológicos horários.

---

# 7. APIs REST próprias

Foram desenvolvidas duas APIs REST através de Supabase Edge Functions.

## POST validate-event

Endpoint responsável por validar os dados do formulário.

Valida:

- nome;
- tipo;
- cidade;
- data;
- horário;
- e-mail do responsável.

Também exige usuário autenticado.

Exemplo de resposta:

    {
      "valid": true,
      "message": "Evento válido."
    }

A API é utilizada diretamente pelo frontend antes da criação do evento.

---

## GET events-summary

API responsável por gerar os indicadores apresentados no Dashboard.

Retorna:

    total_events
    future_events
    risk_summary.normal
    risk_summary.attention
    risk_summary.critical
    risk_summary.pending
    next_event

A função:

- utiliza autenticação;
- respeita RLS;
- considera o timezone do usuário;
- ignora registros arquivados.

---

# 8. Autenticação

A autenticação é realizada através do Supabase Auth.

O fluxo permite:

1. criação de conta;
2. confirmação de e-mail;
3. login;
4. acesso às rotas protegidas;
5. logout.

Cada usuário visualiza apenas seus próprios registros.

---

# 9. Segurança

Foram implementadas políticas de **Row Level Security (RLS)**.

Essas políticas garantem que usuários autenticados não possam acessar registros pertencentes a outros usuários.

Também existe separação entre:

- credenciais do frontend;
- processos server-side;
- autenticação;
- banco de dados;
- automações.

Credenciais administrativas não são armazenadas no repositório público.

---

# 10. Estrutura principal de dados

A tabela principal é:

    events

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
    archived
    archived_at

---

# 11. Processamento climático

Após o cadastro, o Make consulta a localização da cidade.

Em seguida, utiliza as coordenadas para consultar a Open-Meteo.

A resposta contém previsões organizadas por horário.

O sistema identifica o índice correspondente ao horário cadastrado no evento e extrai especificamente os dados daquela hora.

Exemplo:

    Evento marcado para 14:00
       ↓
    Busca do horário correspondente
       ↓
    Extração da previsão de 14:00
       ↓
    Tratamento
       ↓
    Classificação

Dessa maneira, a análise não depende apenas de uma média diária.

---

# 12. Variáveis analisadas

O sistema considera:

    Temperatura
    Sensação térmica
    Umidade
    Probabilidade de chuva
    Precipitação
    Velocidade do vento

Essas informações são transformadas em variáveis intermediárias dentro da automação.

---

# 13. Classificação do risco

O sistema utiliza três níveis:

    Normal
    Atenção
    Crítico

## Eventos externos

Um evento pode atingir o nível Crítico quando ocorre, por exemplo:

    Sensação térmica >= 40°C
    Temperatura >= 35°C
    Probabilidade de chuva >= 70%
    Velocidade do vento >= 40 km/h

A classificação Atenção utiliza limites intermediários.

Quando nenhum limite relevante é atingido:

    Normal

Eventos internos utilizam critérios diferentes, considerando principalmente temperatura e sensação térmica.

---

# 14. Recomendações automáticas

## Normal

    Condições favoráveis no momento.
    Recomenda-se manter o acompanhamento até a realização do evento.

## Atenção

    Recomenda-se reforçar medidas preventivas,
    acompanhar a previsão e preparar alternativas para a atividade.

## Crítico

    Recomenda-se reavaliar a realização do evento,
    considerar mudança de horário, local coberto ou adiamento.

A recomendação é utilizada como apoio à decisão e não substitui a avaliação humana.

---

# 15. Alertas por e-mail

O Gmail é acionado através de um filtro no Make.

Condição:

    event_risk = Atenção
    OU
    event_risk = Crítico

O alerta pode apresentar:

- nome do evento;
- tipo;
- cidade;
- data;
- horário;
- classificação;
- temperatura;
- sensação térmica;
- probabilidade de chuva;
- vento;
- recomendação.

Eventos classificados como Normal não geram alerta.

---

# 16. Persistência

## Supabase

É a fonte transacional principal da aplicação.

Armazena:

- usuários;
- perfis;
- eventos;
- dados meteorológicos;
- status;
- recomendações;
- arquivamento.

## Airtable

Atua como camada complementar de integração e auditoria operacional.

Supabase e Airtable possuem responsabilidades diferentes e não representam duplicação acidental.

---

# 17. Dashboard

O Dashboard apresenta:

    Total de eventos
    Eventos futuros
    Normais
    Atenção
    Críticos
    Pendentes

Também apresenta:

- eventos recentes;
- próximo evento.

Os dados resumidos são fornecidos pela API própria `events-summary`.

---

# 18. Timezone

O sistema utiliza o timezone do navegador:

    Intl.DateTimeFormat().resolvedOptions().timeZone

Esse valor é enviado para a Edge Function `events-summary`.

Isso permite calcular corretamente:

- data atual;
- horário atual;
- eventos futuros;
- próximo evento.

A solução foi testada em contextos de timezone diferentes, incluindo Japão e Brasil.

---

# 19. Arquivamento lógico

Eventos antigos podem ser arquivados sem exclusão definitiva.

Campos utilizados:

    archived
    archived_at

Ao arquivar:

    archived = true

O evento:

- deixa de aparecer entre os ativos;
- aparece na área de arquivados;
- deixa de afetar os indicadores do Dashboard.

Ao restaurar:

    archived = false
    archived_at = null

O evento volta a ser considerado pelo sistema.

---

# 20. Testes dos níveis de risco

Foram utilizados testes controlados com previsões reais para exercitar os três caminhos principais da lógica.

## Normal

Cidade utilizada:

    Kushiro

Resultado:

    Normal

Também foi validado que não ocorre envio de alerta.

## Atenção

Cidade utilizada:

    Kobe

Valores observados durante o teste incluíram aproximadamente:

    Temperatura: 29,5°C
    Sensação térmica: 35,6°C
    Probabilidade de chuva: 48%
    Vento: 10,6 km/h

Resultado:

    Atenção

O alerta foi enviado corretamente.

## Crítico

Cidade utilizada:

    Fukuoka

Valores observados incluíram aproximadamente:

    Temperatura: 31°C
    Sensação térmica: 37,3°C
    Probabilidade de chuva: 79%
    Vento: 4 km/h

Resultado:

    Crítico

O alerta também foi enviado corretamente.

---

# 21. Matriz de testes

| Cenário | Cidade | Esperado | Obtido |
|---|---|---|---|
| Normal | Kushiro | Normal | Normal |
| Atenção | Kobe | Atenção | Atenção |
| Crítico | Fukuoka | Crítico | Crítico |

---

# 22. Principais desafios técnicos

Durante o desenvolvimento foram tratados problemas relacionados a:

- extração correta dos dados horários;
- manipulação dos arrays da Open-Meteo;
- normalização de textos;
- regras condicionais;
- constraints do banco;
- integração com Airtable;
- timezone;
- cache do frontend;
- consumo correto da resposta JSON;
- arquivamento;
- atualização dos indicadores.

O processo de depuração foi documentado através de screenshots versionados no repositório.

---

# 23. LGPD, ética e governança

O projeto considera princípios relacionados à LGPD.

Os dados pessoais utilizados são limitados principalmente a:

- e-mail do usuário;
- e-mail do responsável;
- identificação da instituição.

Não existe necessidade de coleta de dados pessoais sensíveis.

A aplicação utiliza autenticação e RLS para restringir o acesso às informações.

As recomendações climáticas são apresentadas como **apoio à decisão**.

O sistema não cancela automaticamente eventos.

A decisão final permanece sob responsabilidade humana, considerando também que previsões meteorológicas possuem incerteza e podem sofrer alterações.

---

# 24. Observações importantes para testes

## Janela de previsão meteorológica

O ClimaSeguro utiliza previsões disponibilizadas pela Open-Meteo.

Por esse motivo, para testar o processamento climático, recomenda-se cadastrar um evento **próximo à data atual e dentro da janela de previsão disponibilizada pela API**.

Eventos cadastrados muito distantes no futuro podem ainda não possuir previsão meteorológica disponível.

Essa situação representa uma limitação natural da fonte meteorológica e não uma falha da integração.

---

## E-mails automáticos

E-mails de:

- confirmação de cadastro;
- alertas climáticos;

podem eventualmente ser direcionados para a pasta de **Spam**, **Lixo eletrônico** ou equivalente, dependendo das políticas do provedor de e-mail do destinatário.

Caso a mensagem não apareça na caixa de entrada, recomenda-se verificar essas pastas.

---

# 25. Como testar a aplicação

1. Acesse:

   https://clima-seguro-hub.lovable.app/

2. Crie uma conta.

3. Confirme o endereço de e-mail.

4. Faça login.

5. Cadastre um evento dentro da janela de previsão meteorológica disponível.

6. Informe:

   - nome;
   - tipo;
   - cidade;
   - data;
   - horário;
   - e-mail do responsável.

7. Aguarde o processamento automático.

8. Consulte o resultado em:

   - Meus Eventos;
   - Dashboard.

9. Caso o resultado seja Atenção ou Crítico, verifique também o e-mail informado.

10. Caso algum e-mail não seja encontrado, verifique Spam ou Lixo eletrônico.

---

# 26. Evidências

O repositório contém screenshots produzidos ao longo do desenvolvimento.

As evidências demonstram:

- configuração do webhook;
- Nominatim;
- Open-Meteo;
- dados horários;
- regras de risco;
- recomendações;
- Airtable;
- Supabase;
- autenticação;
- frontend;
- Gmail;
- APIs próprias;
- timezone;
- arquivamento;
- restauração;
- testes;
- diagnóstico e correção de bugs.

Entre as evidências finais estão:

    113-supabase-weather-status-raw-values-diagnostic.png
    114-events-summary-response-diagnostic.png
    115-lovable-dashboard-risk-summary-fix.png
    116-dashboard-risk-summary-correct.png
    117-dashboard-after-event-archive.png
    118-dashboard-after-event-restore.png
    119-validate-event-api-network-success.png

---

# 27. Fluxo completo

    1. Usuário cria conta
    2. Confirma e-mail
    3. Realiza login
    4. Preenche Novo Evento
    5. validate-event verifica os dados
    6. Evento é criado no Supabase
    7. Webhook chama o Make
    8. Nominatim localiza a cidade
    9. Open-Meteo retorna a previsão
    10. Make seleciona o horário do evento
    11. Dados meteorológicos são tratados
    12. Risco é calculado
    13. Recomendação é gerada
    14. Airtable recebe o registro
    15. Supabase é atualizado
    16. Gmail envia alerta quando necessário
    17. events-summary consolida os indicadores
    18. Dashboard apresenta o resultado
    19. Eventos antigos podem ser arquivados
    20. Eventos arquivados podem ser restaurados

---

# 28. Estado final do projeto

Estão implementados e validados:

- autenticação;
- confirmação por e-mail;
- login;
- logout;
- RLS;
- cadastro de evento;
- validação por API própria;
- webhook;
- automação Make;
- geocodificação;
- previsão meteorológica;
- análise horária;
- classificação de risco;
- recomendações;
- Supabase;
- Airtable;
- Gmail;
- alertas condicionais;
- Dashboard;
- API de resumo;
- timezone;
- arquivamento;
- restauração;
- testes Normal, Atenção e Crítico;
- vídeo de apresentação;
- relatório teórico;
- documentação e evidências versionadas.

---

# 29. Limitações

Entre as limitações atuais estão:

- dependência da disponibilidade das APIs externas;
- previsão meteorológica limitada a determinada janela temporal;
- previsão sujeita a alterações;
- critérios de risco atualmente fixos;
- ausência de integração com alertas oficiais de defesa civil;
- dependência de conexão com internet;
- limites dos planos gratuitos utilizados;
- possibilidade de e-mails automáticos serem classificados como spam.

---

# 30. Possíveis evoluções

Como trabalhos futuros:

- atualização automática da previsão próxima ao evento;
- alertas recorrentes;
- APIs oficiais de emergência;
- limiares configuráveis;
- gráficos históricos;
- relatórios;
- filtros avançados;
- notificações por outros canais;
- arquivamento automático;
- painel administrativo;
- exportação de dados.

---

# 31. Conclusão

O ClimaSeguro demonstra uma aplicação prática dos conceitos de integração de APIs e automação digital.

A solução integra:

- frontend;
- autenticação;
- banco de dados;
- APIs externas;
- APIs REST próprias;
- automação;
- persistência;
- regras de negócio;
- alertas;
- Dashboard.

O projeto transforma uma consulta meteorológica simples em um fluxo automatizado capaz de receber eventos, consultar dados reais, interpretar previsões, classificar riscos e fornecer informações úteis para apoiar a tomada de decisão.

Os três principais cenários da aplicação foram validados:

    Normal
    Atenção
    Crítico

O resultado é uma solução funcional, modular, de baixo custo e adequada ao contexto acadêmico da disciplina de **Integração de APIs**.

---

# Autor

**Paulo Ricardo Takara Stefens**  
RA: 232231  
IA e Automação Digital  
UniFECAF  
Disciplina: Integração de APIs

---

## Links finais

**Aplicação:**  
https://clima-seguro-hub.lovable.app/

**Vídeo:**  
https://youtu.be/GZkEPbu2zKM

**GitHub:**  
https://github.com/therealteacherpaul/central-inteligente-monitoramento/tree/main

**Airtable:**  
https://airtable.com/appXI5pGDcJMTRiiS/shrIUsZzNh76EASDi

**Cenário Make:**  
https://eu1.make.com/public/shared-scenario/q7AlqJqBRr6/clima-seguro

**Relatório teórico:**  
`CLIMASEGURO-Parte-Teorica.pdf`