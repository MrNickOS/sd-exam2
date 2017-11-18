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
```
. elasticsearch-6.0.0/bin/elasticsearch
. kibana-6.0.0-darwin-x86_64/bin/kibana
```

