# Parcial 2 Sistemas Distribuidos
### Autor: Nicolás Machado Sáenz
### Tema: Manejo de contenedores con Docker y Docker-Compose

En este ejercicio se aprovisionará un ambiente virtualizado del stack ELK similar al parcial 1
con la diferencia que se utilizará tecnología de manejo de contenedores. Para ello se requiere
de las siguientes herramientas.
  * Equipo Cliente Ubuntu 16.04
  * Docker + Docker-Compose
  
La infraestructura final estará compuesta de 4 contenedores virtuales, a los que será posible
acceder via IP del host (192.168.130.130) especificando el puerto mapeado a cada contenedor.
  * Servicio web httpd, puerto 8082
  * Servicio Fluentd, puerto 24224 para manejo de logs
  * Elasticsearch, puerto 9200
  * Kibana, puerto 5601 y a este se puede acceder vía web.
  
### Configuración del paquete ELK

Para el despliegue de cada servicio es necesario consignar los siguientes comandos.
```bash
java -version
mkdir -p /home/distribuidos/Documents/talleres/installs/efk
cd /home/distribuidos/Documents/talleres/installs/efk
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.0.0.tar.gz
wget https://artifacts.elastic.co/downloads/kibana/kibana-6.0.0-amd64.deb
tar -xzvf elasticsearch-6.0.0.tar.gz 
sudo dpkg -i kibana-6.0.0-amd64.deb
```

La configuración de Java debería ser similar a esta.
```
openjdk version "1.8.0_131"
OpenJDK Runtime Environment (build 1.8.0_131-8u131-b11-0ubuntu1.16.10.2-b11)
OpenJDK 64-Bit Server VM (build 25.131-b11, mixed mode)
```

Los comandos para ejecutar ambos servicios se especifican a continuación. Las evidencias de
funcionamiento se encuentran al final del documento.
```bash
. elasticsearch-6.0.0/bin/elasticsearch
. kibana-6.0.0-darwin-x86_64/bin/kibana
```

### Configuración de Fluentd
Fluentd es una herramienta que permite la gestión de los logs en ELK (la función de Logstash,
en el parcial 1). Es posible hacer la instalación de manera automatica utilizando un script *sh*
para luego iniciar y  verificar el servicio. Se debe seguir entonces estas instrucciones.
```
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-trusty-td-agent2.sh | sh
/etc/init.d/td-agent restart
/etc/init.d/td-agent status
```

Es posible hacer integración de Fluentd con ELK para el manejo de los registros; para ello se
ingresan estas lineas.
```bash
sudo apt-get install make libcurl4-gnutls-dev --yes
sudo /opt/td-agent/embedded/bin/fluent-gem install fluent-plugin-elasticsearch
sudo /opt/td-agent/embedded/bin/fluent-gem install fluent-plugin-record-reformer
sudo nano /etc/td-agent/td-agent.conf
```

La última linea indica que se editará el archivo de configuración del td-agent insertando
estas especificaciones.
```conf
<source>
    type syslog
    port 5140
    tag  system
</source>
 
<match system.*.*>
    type record_reformer
    tag efkl
    facility ${tag_parts[1]}
    severity ${tag_parts[2]}
</match>
 
<match efkl>
    type copy
    <store>
       type elasticsearch
       logstash_format true
       flush_interval 15s
    </store>
</match>
```

Se reinicia el servicio con ```sudo service td-agent start```. Luego, se le indicará al sistema de
registros que envíe los logs a fluentd, con la instrucción ```sudo vim /etc/rsyslog.conf``` insertando
esta línea

```*.*                             @127.0.0.1:5140```

que hace que el servidor escuche por el puerto 5140 en localhost como predeterminado. Finalmente, se
reinicia el stack ELK con el servicio Fluentd.
```curl -XPUT 'http://localhost:9200/kibana/' -d '{"index.mapper.dynamic": true}'```


## Provisioning Docker + Docker-Compose
La herramienta de contenedores permite ejecutar los comandos previos de manera automática, usando
scripts denominados Dockerfile y docker-compose (de extensión .yml). En ese sentido, se procede a
ejecutar el archivo DC en la carpeta raíz que contiene los subdirectorios de aprovisionamiento.

```distribuidos@Equipo-Lab-306C:~/Documents/talleres/sd-exam2$ docker-compose up --build```

Se utiliza la opción --build para construir desde cero los nuevos contenedores, con estas líneas de
código.

```yml
version: '2'
services:
  web:
    image: httpd
    ports:
      - "8082:8082"
    links:
      - fluentd
    logging:
      driver: "fluentd"
      options:
        fluentd-address: 127.0.0.1:24224
        tag: httpd.access

  fluentd:
    build: ./fluentd
    volumes:
      - ./fluentd/conf:/fluentd/etc
    links:
      - "elasticsearch"
    ports:
      - "24224:24224"
      - "24224:24224/udp"

  elasticsearch:
    image: elasticsearch
    expose:
      - 9200
    ports:
      - "9200:9200"

  kibana:
    image: kibana
    links:
      - "elasticsearch"
    ports:
      - "5601:5601"
```

Este DC ejecuta el archivo Dockerfile para Fluentd, que consta de estas líneas.

```Dockerfile
# fluentd/Dockerfile
FROM fluent/fluentd:v0.12-debian
RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-rdoc", "--no-ri", "--version", "1.9.2"]
```

Y la configuración del servicio Fluentd es la siguiente:

```conf
# fluentd/conf/fluent.conf
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>
<match *.**>
  @type copy
  <store>
    @type elasticsearch
    host elasticsearch
    port 9200
    logstash_format true
    logstash_prefix fluentd
    logstash_dateformat %Y%m%d
    include_tag_key true
    type_name access_log
    tag_key @log_name
    flush_interval 1s
  </store>
  <store>
    @type stdout
  </store>
</match>
```

## Evidencias de funcionamiento

Se puede acceder desde el host anfitrión o, con la IP de éste, desde un equipo en la misma LAN.

![Evidencia](https://github.com/MrNickOS/sd-exam2/blob/A00052208/sd-exam2/00.png)

![Evidencia](https://github.com/MrNickOS/sd-exam2/blob/A00052208/sd-exam2/01.png)

![Evidencia](https://github.com/MrNickOS/sd-exam2/blob/A00052208/sd-exam2/02.png)

![Evidencia](https://github.com/MrNickOS/sd-exam2/blob/A00052208/sd-exam2/03.png)

URL: https://github.com/MrNickOS/sd-exam2/tree/A00052208
