![alt tag](https://assets.zabbix.com/img/logo.svg)

Os serviços que compõe essa stack tem por finalidade realizar a instalação e configuração de um ambiente de monitoramento do Zabbix usando o Docker e realizando a integração do mesmo com o Elasticsearch possibilitando dessa forma melhorar a performance de acesso aos dados históricos do Zabbix.

# VERSÃO DO ZABBIX
  3.4.15

# NOTAS DE VERSÃO
° Versão 1.0 - 15/07/2018 ==> Criação do Dockerfile e do Docker composse.\
° Update 1.1 - 30/08/2018 ==> Atualizaco da versao do zabbix server, inclusao do traefik como servico de proxy reverso.\
° Update 1.2 - 30/08/2018 ==> Configuração do modulo SSL no apache.\
° Update 1.3 - 04/12/2018 ==> Inclusão do Everyz e atualização da versão do elasticsearch para a versão 6.5.0.

# PRÉ-REQUISITO
Os pre-requisitos necessários para execução da stack de serviço:
 1) Docker instalado e configurado.
 2) Docker Swarm configurado.

# INSTALANDO E CONFIGURANDO DOCKER
   curl -fsSL https://get.docker.com | bash

# CONFIGURANDO DOCKER SWARM
   docker swarm init --advertise-addr  <IP_DO_SERVIDOR>

   Onde <IP_DO_SERVIDOR> deve ser substituído pelo ip do seu servidor.

# BAIXANDO AS IMAGENS
   docker pull leandromoreirajfa/zabbix-server:1.2\
   docker pull leandromoreirajfa/zabbix-web:1.2\
   docker pull grafana/grafana\
   docker pull docker.elastic.co/elasticsearch/elasticsearch:6.4.0\
   docker pull traefik

# SERVIÇO ZABBIX SERVER
Para configurar o serviço do Zabbix Server, editar o arquivo server.config que localiza-se no diretório configs, alterando as variáveis:

PGSQL_HOST=<IP_DO_SERVIDOR>\
PGSQL_ZABBIX_USER=<USUARIO_ZABBIX>\
PGSQL_ZABBIX_BD=<ZABBIX_DATABASE>\
PGSQL_ZABBIX_PASS=<SENHA_ZABBIX_USER>

# SERVIÇO ZABBIX WEB
Para configurar o serviço do Zabbix Web, editar o arquivo web.config que localiza-se no diretório configs, alterando
as variáveis:

PGSQL_HOST=<IP_DO_SERVIDOR>\
PGSQL_ZABBIX_USER=<USUARIO_ZABBIX>\
PGSQL_ZABBIX_BD=<ZABBIX_DATABASE>\
PGSQL_ZABBIX_PASS=<SENHA_ZABBIX_USER>\
ZABBIX_HOST=zabbix-server-pgsql\
APACHE_VHOST=<FDQN_VIRTUAL_HOST_APACHE>\
APACHE_SRV_ADMIN=<EMAIIL_SERVER_ADMIN>\
TZ=America/Sao_Paulo\
CERT_FILE=<NOME_ARQUIVO_CERTIFICADO>\
CERT_KEY=<NOME_ARQUIVO_KEY>

# SERVIÇO GRAFANA
Para configurar o serviço do Zabbix Web, editar o arquivo grafana.config que localiza-se no diretório configs, alterando a variavel que armazena a senha do usuario "admin" do grafana.

GF_SECURITY_ADMIN_PASSWORD=<SENHA_USUARIO_ADMIN_GRAFANA>

# SERVIÇO ELASTICSEARCH
Após iniciado o serviço do “elasticsearch” temos de criar os índices e os chards para que o Zabbix possa gravar os dados no ”elasticsearch” e a interface web possa exibi-lo.
Para criar os índices, executar no prompt do container do ”elasticsearch” o comando contido no arquivo elasticsearch.config que encontra-se no diretório configs.

Exemplo:
  
curl -X PUT \
 http://elasticsearch:9200/text \
 -H 'content-type:application/json' \
 -d '{\
 "settings" : {\
    "index" : {\
       "number_of_replicas" : 1,\
       "number_of_shards" : 5\
    }\
 },\
 "mappings" : {\
    "values" : {\
       "properties" : {\
          "itemid" : {\
             "type" : "long"\
          },\
          "clock" : {\
             "format" : "epoch_second",\
             "type" : "date"\
          },\
          "value" : {\
             "fields" : {\
                "analyzed" : {\
                   "index" : true,\
                   "type" : "text",\
                   "analyzer" : "standard"\
                }\
             },\
             "index" : false,\
             "type" : "text"\
          }\
       }\
    }\
 }\
}'
