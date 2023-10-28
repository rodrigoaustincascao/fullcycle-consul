# Iniciando o Server

```
# Entrar no container
docker exec -it consul01 sh

# Iniciar o dev mode
consul agent -dev

#listar os membros
consul members

#acessar o catálago
curl localhost:8500/v1/catalog/nodes
```
Rodar com pelo menos 3 server. Se for mais, sempre números ímpares.

Consul roda um servidor DNS na porta 8600

## Instalando o dig
Consul roda um servidor DNS na porta 8600
```
apk -U add bind-tools

# consultar DNS
dig @localhost -p 8600

# consultar o dns do node
dig @localhost -p 8600 consul01.node.consul
```
## Configurando a comunicação entre os servers
É necessário saber qual é o IP da rede docker. Então, dentro do container, executar ifconfig e localizar o IP da rede eth0.

Repetir esses comandos em cada server. Não esquecendo de trocar os IPs
```
# Criando os diretórios utilizados na configuração
mkdir /etc/consul.d
mkdir /var/lib/consul

# Iniciando o agent
consul agent -server -bootstrap-expect=3 -node=consulserver01 -bind=172.21.0.4 -data-dir=/var/lib/consul -config-dir=/etc/consul.d

```
Para os servers se reconhecerem
```
consul join 172.21.0.3
# saída do terminal: 2023-10-26T14:18:16.081Z [INFO]  agent: (LAN) joining: lan_addresses=[172.21.0.4]
```
## Utilizando JSON de configuração para automação

O Json de configuração está em /servers/server01
Não esquecer de configurar os IPs
Não esquecer de configurar a chave de criptografia
```
# Gerar chave de criptografia, essa chave precisa ser configurada em todos os servers
consul keygen

mkdir /var/lib/consul
consul agent -config-dir=/etc/consul.d
```


# Configurando o Client

```
docker exec -it consulclient01 sh

# Ativando o client
consul agent


mkdir /etc/consul.d
mkdir /var/lib/consul

consul agent -bind=172.21.0.4 -data-dir=/var/lib/consul -config-dir=/etc/consul.d

consul join 172.21.0.3
```
## Registrando o Serviço

Para registrar o serviço é necessário criar um json de configuração
```
# Atualiza os serviços
consul reload
```

Para buscar o node que contém o serviço
```
consul catalog nodes -service nginx
consul catalog nodes -detailed
```

Registrar o service automaticamente quando o agent subir
```
consul agent -bind=172.21.0.5 -data-dir=/var/lib/consul -config-dir=/etc/consul.d -retry-join=<IPDeAlgumServer>
```

### Nginx
```
apk add nginx

mkdir /run/nginx

nginx

```
#### Configurar o Nginx
```
apk add vim
mkdir /usr/share/nginx/html -p


vim /etc/nginx/conf.d/default.conf
# Remover location { return 404;}
# adicionar root /usr/share/nginx/html;


vim /usr/share/nginx/html/index.html
# adicionar <h1>Hello World!</h1>
```

# Gerar chave de criptografia
No server
```
consul keygen
```
## TCPDump
Utilizado para verificar se as informações trafegadas entre os serves estão criptografadas
```
apk -U add bind-tools
apk add tcpdump

tcpdump -i eth0 -an port 8301 -A
```

# Interface
Para acessar a interface no navegador
```
localhost:8500/ui
```
## Configurando na Interface no server
No arquivo de configuração deve ser adicionado o seguinte trecho:
```
    "client_addr": "0.0.0.0",
    "ui_config": {
        "enable": true
    }
```
No docker-composer deve ser disponibilizada a porta 8500