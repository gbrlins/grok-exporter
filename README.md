# Monitorando dispositivos com o grok-exporter

*Documentação disponível em https://github.com/fstab/grok_exporter*

O exemplo foi feito em um host conectado ao SUSE Manager com o Grafana e Prometheus instalado através de formulas do SUMA.
Caso não seja a sua situação, será necessário baixar e configurar o Grafana e Prometheus antes de iniciar o passo-a-passo.


### Passo 1: Baixar grok_exporter
```
wget https://github.com/fstab/grok_exporter/releases/download/v1.0.0.RC3/grok_exporter-1.0.0.RC3.linux-amd64.zip
```
Após download, explodir o .zip e entrar no diretório que foi explodido:
```
unzip grok_exporter-1.0.0.RC3.linux-amd64.zip
cd grok_exporter-1.0.0.RC3.linux-amd64/
``` 
### Passo 2: Configurar arquivo config.yml
O arquivo encontra-se por padrão dentro da pasta "example". Entrar no diretório example e substituir o arquivo config-file.yml pelo conteúdo abaixo, ou copie o conteúdo do arquivo <a href=“https://github.com/gbrlins/grok-exporter/blob/master/config-file.yml“>config.yml</a>
*obs: A identação é importante para o funcionamento. Verifique!*
```
global:
  config_version: 3
input:
  type: file
  path: ./example/example.log
  readall: true
grok_patterns:
- 'DEVICE [^=]*$'
metrics:
- type: counter
  name: usb_activity
  help: Exibe se um dispositivo USB está conectado ou não
  match: '%{DEVICE:device}'
  cumulative: false
  labels:
      device: '{{.device}}'
server:
  protocol: http
  port: 9144
```

### Passo 3: Criar uma entrada na crontab
Com o comando crontab -e, adicionar a seguinte entrada na cron (lembrar de alterar para o caminho correto):

```
* * * * * usb-devices | grep 'Product=' > /home/{user}/grok_exporter-1.0.0.RC3.linux-amd64/example/example.log
```

### Passo 4: Configurando o prometheus

```
vim /etc/prometheus/prometheus.yml
```
Adicionar em seu arquivo de configuração, dentro de scrape_configs a seguinte entrada:

```
  - job_name: 'grok'
    static_configs:
      - targets:
        - {IP_OU_HOSTNAME_DA_MAQUINA_A_SER_MONITORADA}:9144 #grok_exporter
        labels:
          role: grok-exporter
```
O resultado final deve ficar parecido com o exemplo <a href=“https://github.com/gbrlins/grok-exporter/blob/master/prometheus.yml“>prometheus.yml</a> 
Salvar o arquivo e restartar o Prometheus:
```
systemctl restart prometheus.service
```

### Passo 5: Start no serviço do grok
Dentro do diretório do grok_exporter, rodar o seguinte comando para iniciar o serviço do grok_eporter:
```
./grok_exporter -config ./example/config-file.yml
```
Você perceberá que o terminal ficou travado. Tudo bem. Não termine o processo, se não o grok será finalizado.
Nesse momento, já será possível verificar que o Prometheus está recebendo as métricas. Para verificar, acessar localhost:9090 e digitar usb_activity no campo "Expression"

### Passo 6: Métricas sendo exportadas
É possível verificar que a url localhost:9144/metrics está disponível. Verifique! Acessando a URL, você encontrará algumas métricas com o nome "usb_activity". Essas métricas estão prontas para serem analisadas pelo Grafana.

### Passo 7: Criando uma dash no Grafana
Dentro do Grafana, já é possível criar os gráficos de monitoramento. Fique livre para criar, ou importe uma dashboard padrão para exemplo (serão necessárias algumas adequações. No símbolo de + (Create) > Import > Or paste JSON . Colar o seguinte JSON:

```{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": "-- Grafana --",
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      }
    ]
  },
  "editable": true,
  "gnetId": null,
  "graphTooltip": 0,
  "id": 5,
  "iteration": 1592933943815,
  "links": [],
  "panels": [
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "fill": 1,
      "gridPos": {
        "h": 8,
        "w": 16,
        "x": 0,
        "y": 0
      },
      "id": 5,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "options": {},
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "usb_activity{instance=~\"$maquina\"} - usb_activity{instance=~\"$maquina\"} offset 1m",
          "format": "time_series",
          "instant": false,
          "intervalFactor": 1,
          "legendFormat": "{{device}}",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Estado atual",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "transparent": true,
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "cacheTimeout": null,
      "columns": [
        {
          "text": "Current",
          "value": "current"
        }
      ],
      "datasource": "Prometheus",
      "fontSize": "100%",
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 0,
        "y": 8
      },
      "id": 2,
      "links": [],
      "options": {},
      "pageSize": null,
      "pluginVersion": "6.2.5",
      "scroll": true,
      "showHeader": true,
      "sort": {
        "col": 0,
        "desc": true
      },
      "styles": [
        {
          "alias": "Status",
          "colorMode": "cell",
          "colors": [
            "#73BF69",
            "#73BF69",
            "rgba(245, 54, 54, 0.9)"
          ],
          "decimals": 2,
          "link": false,
          "mappingType": 2,
          "pattern": "Current",
          "preserveFormat": false,
          "rangeMaps": [
            {
              "from": "1",
              "text": "Conectado",
              "to": "Infinity"
            }
          ],
          "thresholds": [
            ""
          ],
          "type": "string",
          "unit": "short",
          "valueMaps": [
            {
              "text": "Conectado",
              "value": "1"
            },
            {
              "text": "Desconectado",
              "value": "0"
            }
          ]
        }
      ],
      "targets": [
        {
          "expr": "usb_activity{instance=~\"$maquina\"} > usb_activity{instance=~\"$maquina\"} offset 1m ",
          "format": "time_series",
          "instant": true,
          "intervalFactor": 1,
          "legendFormat": "{{device}}",
          "refId": "A"
        }
      ],
      "timeFrom": null,
      "timeShift": null,
      "title": "Dispositivos USB Conectados",
      "transform": "timeseries_aggregations",
      "type": "table"
    },
    {
      "cacheTimeout": null,
      "columns": [
        {
          "text": "Current",
          "value": "current"
        }
      ],
      "datasource": "Prometheus",
      "fontSize": "100%",
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 8,
        "y": 8
      },
      "id": 3,
      "links": [],
      "options": {},
      "pageSize": null,
      "pluginVersion": "6.2.5",
      "scroll": true,
      "showHeader": true,
      "sort": {
        "col": 0,
        "desc": true
      },
      "styles": [
        {
          "alias": "Status",
          "colorMode": "cell",
          "colors": [
            "#F2495C",
            "#F2495C",
            "rgba(245, 54, 54, 0.9)"
          ],
          "decimals": 2,
          "link": false,
          "mappingType": 2,
          "pattern": "Current",
          "rangeMaps": [
            {
              "from": "1",
              "text": "Desconectado",
              "to": "Infinity"
            }
          ],
          "sanitize": false,
          "thresholds": [
            ""
          ],
          "type": "string",
          "unit": "short",
          "valueMaps": [
            {
              "text": "Conectado",
              "value": "1"
            },
            {
              "text": "Desconectado",
              "value": "0"
            }
          ]
        }
      ],
      "targets": [
        {
          "expr": "usb_activity{instance=~\"$maquina\"} == usb_activity{instance=~\"$maquina\"} offset 1m ",
          "format": "time_series",
          "instant": true,
          "intervalFactor": 1,
          "legendFormat": "{{device}}",
          "refId": "A"
        }
      ],
      "timeFrom": null,
      "timeShift": null,
      "title": "Dispositivos USB Desconectados",
      "transform": "timeseries_aggregations",
      "type": "table"
    }
  ],
  "refresh": "5s",
  "schemaVersion": 18,
  "style": "dark",
  "tags": [],
  "templating": {
    "list": [
      {
        "allValue": null,
        "current": {
          "text": "All",
          "value": "$__all"
        },
        "datasource": "Prometheus",
        "definition": "up",
        "hide": 0,
        "includeAll": true,
        "label": "Server",
        "multi": false,
        "name": "maquina",
        "options": [],
        "query": "up",
        "refresh": 1,
        "regex": "/.*instance=\"([^\"]*),*/",
        "skipUrlSync": false,
        "sort": 2,
        "tagValuesQuery": "",
        "tags": [],
        "tagsQuery": "",
        "type": "query",
        "useTags": false
      }
    ]
  },
  "time": {
    "from": "now-2m",
    "to": "now"
  },
  "timepicker": {
    "refresh_intervals": [
      "5s",
      "10s",
      "30s",
      "1m",
      "5m",
      "15m",
      "30m",
      "1h",
      "2h",
      "1d"
    ],
    "time_options": [
      "5m",
      "15m",
      "1h",
      "6h",
      "12h",
      "24h",
      "2d",
      "7d",
      "30d"
    ]
  },
  "timezone": "",
  "title": "USB",
  "uid": "Rw1IuzWGk",
  "version": 33
}
```

### Passo 8: Configurar variáveis no Grafana

Variáveis permitem filtrar dados em uma dashboard. Com o exemplo acima, é necessário criar uma variável.
Para criar uma variável:

Clicar em "Dashboard settings" (canto superior direito, no símbolo de engrenagem) e selecionar a aba "Variables"
Clicando em "New", abrirá a página de configuração. Copiar como mostra a imagem abaixo:

![Screenshot_20200623_114217](https://user-images.githubusercontent.com/22751679/85417988-a5124e80-b546-11ea-94c2-84c13762303f.png)


Em "Regex" = /.*instance="([^"]*),*/

Após criar variável, "Save" e volte para a Dashboard USB. Verificar se a coleta dos dados estão ocorrendo.

