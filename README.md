![alt tag](https://www.elastic.co/assets/blt244a845f141977c3/elastic-logo.svg)

Criado por Leandro Moreira\
E-Mail: leandro@leandromoreira.eti.br\

Os serviços que compõe essa stack tem a finalidade de realizar a instalação e configuração de um ambiente de monitoramento usando o Zabbix, realizando a integração do mesmo com o Elasticsearch possibilitando dessa forma melhorar a performance de acesso aos dados históricos do zabbix.

# ° NOTAS DE VERSÃO

°  Versão 1.0 - 15/07/2018 ==> Criação do Dockerfile e do Docker composse.
°  Update 1.1 - 30/08/2018 ==> Atualizaco da versao do zabbix server, inclusao do traefik como servico de proxy reverso. 
°  Update 1.2 - 30/08/2018 ==> Configuração do modulo SSL no apache. 

# ° PRÉ-REQUISITO

Os pre-requisitos necessários para execução da stack de serviço de monitoramento com Zabbix são, Docker e Swarm instalados e configurados.

IMG

# ° Instalação do Docker
   # curl -fsSL https://get.docker.com | bash

# ° Configuração do Swarm
   # docker swarm init --advertise-addr  <IP_DO_SERVIDOR>

   Onde <IP_DO_SERVIDOR> deve ser substituído pelo ip do seu servidor.


# ° SERVIÇO ZABBIX SERVER
Para configurar o serviço do Zabbix Server, editar o arquivo server.config que localiza-se no diretório configs, alterando as variáveis:

PGSQL_HOST=<IP_DO_SERVIDOR>
PGSQL_ZABBIX_USER=<USUARIO_ZABBIX>
PGSQL_ZABBIX_BD=<ZABBIX_DATABASE>
PGSQL_ZABBIX_PASS=<SENHA_ZABBIX_USER>

# ° SERVIÇO ZABBIX WEB
Para configurar o serviço do Zabbix Web, editar o arquivo web.config que localiza-se no diretório configs, alterando
as variáveis:

PGSQL_HOST=<IP_DO_SERVIDOR>
PGSQL_ZABBIX_USER=<USUARIO_ZABBIX>
PGSQL_ZABBIX_BD=<ZABBIX_DATABASE>
PGSQL_ZABBIX_PASS=<SENHA_ZABBIX_USER>
ZABBIX_HOST=zabbix-server-pgsql
APACHE_VHOST=<FDQN_VIRTUAL_HOST_APACHE>
APACHE_SRV_ADMIN=<EMAIIL_SERVER_ADMIN>
TZ=America/Sao_Paulo
CERT_FILE=<NOME_ARQUIVO_CERTIFICADO>
CERT_KEY=<NOME_ARQUIVO_KEY>

# ° SERVIÇO GRAFANA
Para configurar o serviço do Zabbix Web, editar o arquivo grafana.config que localiza-se no diretório configs, alterando a variavel que armazena a senha do usuario "admin" do grafana.

GF_SECURITY_ADMIN_PASSWORD=<SENHA_USUARIO_ADMIN_GRAFANA>

# ° SERVIÇO ELASTICSEARCH
Após iniciado o serviço do “elasticsearch” temos de criar os índices e os chards para que o Zabbix possa gravar os dados no ”elasticsearch” e a interface web possa exibi-lo.
Para criar os índices, executar no prompt do container do ”elasticsearch” o comando contido no arquivo elasticsearch.config que encontra-se no diretório configs.

Exemplo:
  
 # curl -X PUT \
 http://elasticsearch:9200/text \
 -H 'content-type:application/json' \
 -d '{
 "settings" : {
    "index" : {
       "number_of_replicas" : 1,
       "number_of_shards" : 5
    }
 },
 "mappings" : {
    "values" : {
       "properties" : {
          "itemid" : {
             "type" : "long"
          },
          "clock" : {
             "format" : "epoch_second",
             "type" : "date"
          },
          "value" : {
             "fields" : {
                "analyzed" : {
                   "index" : true,
                   "type" : "text",
                   "analyzer" : "standard"
                }
             },
             "index" : false,
             "type" : "text"
          }
       }
    }
 }
}'
