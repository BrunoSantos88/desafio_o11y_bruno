# Atividade Prática Observabilidade

## Objetivo do Laboratório:

Criar um ambiente de observabilidade usando Prometheus e Grafana para monitorar uma aplicação de exemplo.

## Technologies Used:

* Linux (Ubuntu based)
* Python Application
* Prometheus
* Grafana

## Pré-requisitos:

* Você precisará de uma máquina Linux (pode ser uma VM, um servidor ou mesmo uma máquina local com Docker instalado).
* Conhecimento básico de linha de comando do Linux.
* Docker instalado na máquina.
* Uma aplicação de exemplo para monitorar (exemplo simples em Python).
* Clonar o repositório da atividade.

## Passos:

### Passo 1: Instalação do Prometheus e Grafana

1.1. Baixe o Docker Compose e instale-o em sua máquina se você ainda não o tiver:

  
   ```console
    $ sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    $ sudo chmod +x /usr/local/bin/docker-compose
   ```
  
    
1.2. **Clone o repositório** e acesse o diretório do laboratório:

    $ cd desafio_o11y/observability-lab

1.3. Crie um arquivo **docker-compose.yml** para definir os serviços Prometheus e Grafana:


```yaml
version: '3'
services:
  prometheus:
    image: prom/prometheus
    volumes:
    - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    - ./rules.yml:/etc/prometheus/rules.yml
    command:
    - '--config.file=/etc/prometheus/prometheus.yml'
    ports:
      - '9090:9090'
    network_mode: "host"

  grafana:
    image: grafana/grafana
    ports:
    - '3000:3000'
    volumes:
      - grafana-data:/var/lib/grafana
    network_mode: "host"
  
  alertmanager:
    image: prom/alertmanager
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    ports:
      - 9093:9093
    network_mode: "host"

volumes:
  grafana-data:

```

1.4. Crie um diretório chamado **prometheus** e, dentro dele, crie um arquivo **prometheus.yml** para configurar o Prometheus:

  ```yaml
global:
  scrape_interval: 15s
rule_files:
  - rules.yml
alerting:
  alertmanagers:
  - static_configs:
    - targets:
       - localhost:9093
scrape_configs:
- job_name: 'prometheus'
  static_configs:
  - targets: ['localhost:9090']
- job_name: 'python_server'
  static_configs:
  - targets: ['localhost:3001']
  ```

* **Dica:** Você precisará ajustar a identação do arquivo

### Passo 2: Configurando a Aplicação de Exemplo

2.1 Certifique-se de que você tenha o **Flask** e o **Prometheus Client** Python instalados. Você pode instalá-los usando o pip:

    pip install Flask prometheus_client

2.2 Acesso o diretório da aplicação:

    $ cd python-app

* Abra o arquivo **app.py** e analise o código fonte, perceba que existem algumas rotas criadas.
* A porta que está sendo exposta sua aplicação é a 3001, você pode alterar caso seja necessário.


2.3 Inicie sua aplicação e exponha-a em uma porta específica.

```console
python app.py
```

2.5 Testando a aplicação python.

Sua aplicação estará disponível em http://localhost:3001. Você pode acessar a página inicial e também verificar as métricas expostas em http://localhost:3001/metrics.

### Passo 3: Gerando métricas na aplicação

Acesse sua aplicação em clique em: Gerar Erro e depois em Calcular Duração

### Passo 4: Configurar o Prometheus no Laboratório 

No arquivo **prometheus.yml** no diretório prometheus (conforme configurado anteriormente), adicione ou ajuste a seguinte seção sob **scrape_configs** para coletar métricas da sua aplicação python:


```yaml
- job_name: 'python_server'
  static_configs:
  - targets: ['localhost:3001']
```

* Certifique-se de substituir **'your-app-container:your-app-port'** pelo host e porta onde sua aplicação python está sendo executada.


### Passo 5: Iniciando o Ambiente de Observabilidade

5.1 Volte para o diretório raiz do seu projeto e execute o seguinte comando para iniciar os serviços Prometheus e Grafana:

```console
$ cd ..
$ docker-compose up -d
```

5.2 Certifique-se que os containers do Prometheus e do Grafana subiram e estão funcionando:

```console
$ docker-compose ps
```

### Passo 6: Acessando o Promethues e verificando as métricas da aplicação

6.1 Acesse o painel Promethues em seu navegador em http://localhost:9090.

6.2 Verifique so o Prometheus está conseguindo acessar os dados da sua aplicação. 

* Clique no menu **Status** e depois em **Targets**.

  * O status deve estar UP para ambos os targets (Prometheus e your-app)
  * Caso o status da aplicação não esteja UP, certifique-se que a aplicação esteja rodando (Item 2.3).
  * Caso ainda não esteja UP ou com outro status, reveja a atividade, pois algum ponto pode ter faltado.

6.3 Agora vamos olhar as métricas configuradas em nossa aplicação:

* Métrica de Contagem de Erros (app_errors_total): Esta métrica conta o número total de erros que ocorreram em sua aplicação.
* Métrica de Duração da Função (app_function_duration_seconds): Esta métrica mede o tempo gasto na execução de funções específicas em sua aplicação. Você configurou rótulos para identificar a função específica sendo monitorada.

### Passo 7: Configurando o Grafana

7.1. Acesse o painel Grafana em seu navegador em http://localhost:3000.

7.2. Faça login com as credenciais padrão (username: admin, password: admin).

7.3. Configure o Prometheus como uma fonte de dados:

* Clique em "Configuration" no menu à esquerda.
* Clique em "Data Sources" e, em seguida, em "Add data source".
* Escolha "Prometheus" como o tipo de fonte de dados.
* Na seção "HTTP", configure o URL para http://prometheus:9090.
* Clique em "Save & Test".

### Passo 8: Criando um Painel no Grafana

8.1. Crie um novo painel clicando em "Create" e escolha "Dashboard".

8.2. Clique em "Add new panel" e escolha "Graph".

8.3. Configure sua consulta Prometheus para visualizar métricas da sua aplicação.

### Passo 9: Configurando e gerando alerta com o Alertmanager.

Agora é com você, configure os passos necessários para ser possível a geração de alertas utilizando o alertmanager e o webhook.

# alertmanager
```yaml

route:
  group_by: ['job','instance']

  group_wait: 5s
  group_interval: 10s
  repeat_interval: 15s

  receiver: discord

receivers:
- name: discord
  discord_configs:
  - webhook_url: 'https://discord.com/api/webhooks/#########'
```
# rules.yml
```yaml
groups:
 - name: Count greater than 5
   rules:
   - alert: CountGreaterThan5
     expr: ping_request_count > 5
     for: 5s

 - name: Uptime app
   rules:
   - alert: ServiceDown
     expr: up{instance="localhost:3001", job="python_server"} == 0
     for: 5s
     labels:
      severity: critical
     annotations:
       summary: "Service is down"
       description: "The Python server on localhost:3001 is not responding"
  
 - name: Error_total
   rules:
   - alert: AppHighErrorRate
     expr: app_errors_total{job="python_server"} > 5
     for: 5s
     labels:
       severity: critical
     annotations:
      summary: "High error rate in the application"
      description: "The total count of errors is higher than 100 for 5s."

 - name: Durarion_App
   rules:
   - alert: HighFunctionExecutionCount
     expr: app_function_duration_seconds_count{job="python_server"} > 5
     for: 5s
     labels:
       severity: warning
     annotations:
      summary: "High function execution count"
      description: "The Python server function is being executed frequently."
```

Envie seu arquivos atualizados (docker-compose.yml, rules.yml, alertmanager.yml) para o e-mail: ####### com prints do seu ambiente funcionando: Webhook disparado.

**Dica:** Utilize a documentação do promethues para fazer consultas e finalizar essa etapa.

# Dashbord Prometheus DataSource, Logs - Prometheus, Cadvisor , NodeExporter e Aplicação.                             
<img src="https://private-user-images.githubusercontent.com/91704169/289210883-5b24d3fb-3cfb-4f1e-8add-eb3aa4b32eb8.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDIwNzkwMzgsIm5iZiI6MTcwMjA3ODczOCwicGF0aCI6Ii85MTcwNDE2OS8yODkyMTA4ODMtNWIyNGQzZmItM2NmYi00ZjFlLThhZGQtZWIzYWE0YjMyZWI4LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFJV05KWUFYNENTVkVINTNBJTJGMjAyMzEyMDglMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjMxMjA4VDIzMzg1OFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTMyYWRjYzRiNzZiNjliMjlkYjBlMjhiMWU0MmEzYTMwNDc1NzYxNWVmNjA2NGU3NzY2ZDIzMzU0NTM1YWUwM2UmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.-tlES5lhpjUoV1SbUoZbfe3Ip6FC_ThYkCNCZ_TPd7k" min-width="200px" max-width="430px" width="420px" align="right" alt="Dashbord"> 

<img src="https://private-user-images.githubusercontent.com/91704169/289211622-da53525d-4c65-471d-9704-a466119c0967.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDIwNzk1MTIsIm5iZiI6MTcwMjA3OTIxMiwicGF0aCI6Ii85MTcwNDE2OS8yODkyMTE2MjItZGE1MzUyNWQtNGM2NS00NzFkLTk3MDQtYTQ2NjExOWMwOTY3LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFJV05KWUFYNENTVkVINTNBJTJGMjAyMzEyMDglMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjMxMjA4VDIzNDY1MlomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTcyM2IzNDk3Y2MzM2ViNjhhZTBiYzJiMGIwMzFmNmQ5NzU2MGM0YWFhNTA0NDg5ZWU2ZWZkMTNhYmYxNzY3Y2EmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.Z5SExT4hTvsUQeRhciAYwVXAodPD_A1SGA3Wa5ixM9Y" min-width="200px" max-width="420px" width="420px" align="left" alt="Dashbord"/>

<img src="https://private-user-images.githubusercontent.com/91704169/289211518-886de773-c62a-4936-9399-1aa1df0b87c1.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDIwNzk0MjksIm5iZiI6MTcwMjA3OTEyOSwicGF0aCI6Ii85MTcwNDE2OS8yODkyMTE1MTgtODg2ZGU3NzMtYzYyYS00OTM2LTkzOTktMWFhMWRmMGI4N2MxLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFJV05KWUFYNENTVkVINTNBJTJGMjAyMzEyMDglMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjMxMjA4VDIzNDUyOVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTkwZWE0NzVmNDVkOGI2MTc0NGE2ZmRlYTBlN2MyMDc4MzQyZWQ3YzliOGRmOGYwZTExYTc2YjM0MDc4OGNkZWImWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.-4qADoX9_3nsdlBWgyN-rCTa8-shsqQp8qPLLuTz0bw" min-width="200px" max-width="420px" width="420px" align="left" alt="Discord">
