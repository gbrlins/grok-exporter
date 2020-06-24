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
O arquivo encontra-se por padrão dentro da pasta "example". Entrar no diretório example e substituir o arquivo config-file.yml pelo conteúdo do arquivo <a href="https://github.com/gbrlins/grok-exporter/blob/master/config-file.yml">config.yml</a>

*obs: A identação é importante para o funcionamento. Verifique!*

### Passo 3: Criar uma entrada na crontab
Com o comando crontab -e, adicionar a seguinte entrada na cron (lembrar de alterar para o caminho correto):

```
* * * * * usb-devices | grep 'Product=' > /home/{user}/grok_exporter-1.0.0.RC3.linux-amd64/example/example.log
```

### Passo 4: Configurando o prometheus

```
vim /etc/prometheus/prometheus.yml
```
Adicionar em seu arquivo de configuração, dentro de scrape_configs o seguinte bloco de entrada:

```
  - job_name: 'grok'
    static_configs:
      - targets:
        - {IP_OU_HOSTNAME_DA_MAQUINA_A_SER_MONITORADA}:9144 #grok_exporter
        labels:
          role: grok-exporter
```
Observe no arquivo exemplo <a href="https://github.com/gbrlins/grok-exporter/blob/master/prometheus.yml">prometheus.yml</a> a presença desse segmento e certifique de preservar a identação. 

Após editado, salvar o arquivo e restartar o Prometheus:
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
Dentro do Grafana, já é possível criar os gráficos de monitoramento. Fique livre para criar, ou importe uma dashboard padrão (disponível aqui nesse link: <a href="https://github.com/gbrlins/grok-exporter/blob/master/dashboard.json">dashboard.json</a>).

No símbolo de + (Create) > Import > Or paste JSON . Colar o conteúdo do <a href="https://github.com/gbrlins/grok-exporter/blob/master/dashboard.json">dashboard.json</a>seguinte JSON: 

### Passo 8: Configurar variáveis no Grafana

Variáveis permitem filtrar dados em uma dashboard. Com o exemplo acima, é necessário criar uma variável.
Para criar uma variável:

Clicar em "Dashboard settings" (canto superior direito, no símbolo de engrenagem) e selecionar a aba "Variables"
Clicando em "New", abrirá a página de configuração. Copiar como mostra a imagem abaixo:

![Screenshot_20200623_114217](https://user-images.githubusercontent.com/22751679/85417988-a5124e80-b546-11ea-94c2-84c13762303f.png)


Em "Regex" = /.*instance="([^"]*),*/

Após criar variável, "Save" e volte para a Dashboard USB. Verificar se a coleta dos dados estão ocorrendo.

