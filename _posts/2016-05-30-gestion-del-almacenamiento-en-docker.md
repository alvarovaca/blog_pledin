---
id: 1717
title: Gestión del almacenamiento en docker
date: 2016-05-30T09:56:55+00:00


guid: http://www.josedomingo.org/pledin/?p=1717
permalink: /2016/05/gestion-del-almacenamiento-en-docker/


tags:
  - docker
  - Virtualización
---
<p style="text-align: justify;">
  Cuando un contenedor es borrado, toda la información contenida en él, desaparece. Para tener almacenamiento persistente en nuestros contenedores, que no se elimine al borrar el contenedor, es necesario utilizar volúmenes de datos (<em>data volume</em>). Un volumen es un directorio o un fichero en el <em>docker engine </em>que se monta directamente en el contenedor. Podemos montar varios volúmenes en un contenedor y en varios contenedores podemos montar un mismo volumen.
</p>

<p style="text-align: justify;">
  Tenemos dos alternativas para gestionar el almacenamiento en docker:
</p>

<li style="text-align: justify;">
  Usando volúmenes de datos
</li>
<li style="text-align: justify;">
  Usando contenedores de volúmenes de datos
</li>

## Volúmenes de datos

Los volúmenes de datos tienen las siguientes características:

<li style="text-align: justify;">
  Son utilizados para guardar e intercambiar información de forma independientemente a la vida de un contenedor.
</li>
<li style="text-align: justify;">
  Nos permiten guardar e intercambiar información entre contenedores.
</li>
<li style="text-align: justify;">
  Cuando borramos el contenedor, no se elimina el volumen asociado.
</li>
<li style="text-align: justify;">
  Los volúmenes de datos son directorios del host montados en un directorio del contenedor, aunque también se pueden montar ficheros.
</li>
<li style="text-align: justify;">
  En el caso de montar en un directorio ya existente de un contenedor un volumen de datos , su contenido no será eliminado.
</li>

### Añadiendo volúmenes de datos

Vamos a empezar creando una contenedor al que le vamos a asociar un volumen:

<pre>$ docker run -it --name contenedor1 -v /volumen ubuntu:14.04 bash</pre>

<p style="text-align: justify;">
  Como podemos comprobar con la opción <code>-v</code> hemos creado un nuevo volumen que se ha montado en el directorio<em> /volumen</em> del contenedor. Vamos a crear un fichero en ese directorio:
</p>

<pre>root@d50f89458659:/# cd /volumen/
root@d50f89458659:/volumen# touch fichero.txt
root@d50f89458659:/volumen# exit</pre>

Podemos comprobar los puntos de montajes que tiene nuestro contnedor con la siguiente instrucción:

<pre>$ docker inspect contenedor1
...
"Mounts": [
            {
                "Name": "c7665edfb4505d6ac85fb0f3db118f6c7bb63958157ec722d6d3ee15ca8f3427",
                "Source": "/mnt/sda1/var/lib/docker/volumes/c7665edfb4505d6ac85fb0f3db118f6c7bb63958157ec722d6d3ee15ca8f3427/_data",
                "Destination": "/volumen",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
...</pre>

<p style="text-align: justify;">
  <!--more-->A continuación podemos comprobar en el docker engine el directorio correspondiente al volumen y que efectivamente contiene el fichero que hemos creado.
</p>

<pre># cd /mnt/sda1/var/lib/docker/volumes/c7665edfb4505d6ac85fb0f3db118f6c7bb63958157ec722d6d
3ee15ca8f3427/_data
fichero1.txt</pre>

Con el cliente docker también podemos comprobar los volúmenes que hemos creados ejecutando:

<pre>$ docker volume ls
DRIVER              VOLUME NAME
local               c7665edfb4505d6ac85fb0f3db118f6c7bb63958157ec722d6d3ee15ca8f3427</pre>

Y a continuación podemos obtener información del volumen:

<pre>$ docker volume inspect c7665edfb4505d6ac85fb0f3db118f6c7bb63958157ec722d6d3ee15ca8f3427
[
    {
        "Name": "c7665edfb4505d6ac85fb0f3db118f6c7bb63958157ec722d6d3ee15ca8f3427",
        "Driver": "local",
        "Mountpoint": "/mnt/sda1/var/lib/docker/volumes/c7665edfb4505d6ac85fb0f3db118f6c7bb63958157ec722d6d3ee15ca8f3427/_data",
        "Labels": null
    }
]</pre>

Por último podemos comprobar que aunque borremos el contenedor, el volumen no se borra:

<pre>$ docker rm contenedor1

$ docker volume ls
DRIVER              VOLUME NAME
local               c7665edfb4505d6ac85fb0f3db118f6c7bb63958157ec722d6d3ee15ca8f3427</pre>

### Creando volúmenes de datos

<p style="text-align: justify;">
  En el apartado anterior al añadir un nuevo volumen al contenedor, se ha creado un nuevo volumen pero con un nombre muy poco significativo. A la hora de gestionar una gran cantidad de volúmenes es muy importante poner nombres significativos a los volúmenes, para ello vamos a crear primero el volumen (indicando el nombre) y a continuación lo asociamos al contenedor:
</p>

<pre>$ docker volume create --name vol1
vol1
$ docker volume ls
DRIVER              VOLUME NAME
local               c7665edfb4505d6ac85fb0f3db118f6c7bb63958157ec722d6d3ee15ca8f3427
local               vol1
$ docker run -it --name contenedor2 -v vol1:/data ubuntu:14.04 bash
</pre>

<p style="text-align: justify;">
  Hemos creado el contenedor2 que tendrá asociado el volumen <em>vol1</em> y lo tendrá montado en el directorio <em>/data</em>.
</p>

<p style="text-align: justify;">
  Realmente no es necesario realizar la operación de crear el volumen con anterioridad. Si creamos un contenedor y asociamos un volumen con un nombre que no existe, el volumen se creará. Vemos un ejemplo:
</p>

<pre>$ docker run -it --name contenedor3 -v vol2:/data ubuntu:14.04 bash
root@8f36b1c407b9:/# exit

$ docker volume ls
DRIVER              VOLUME NAME
local               c7665edfb4505d6ac85fb0f3db118f6c7bb63958157ec722d6d3ee15ca8f3427
local               vol1
local               vol2</pre>

<p style="text-align: justify;">
  De forma similar, pero utilizando el nombre largo, podemos crear un nuevo contenedor al que asociamos nuestro primer volumen y podamos comprobar que la información guardada en el volumen es persistente:
</p>

<pre>$ docker run -it --name contenedor4 -v c7665edfb4505d6ac85fb0f3db118f6c7bb63958157ec722d6d3ee15ca8f3427:/data ubuntu:14.04 bash
root@1fbbe52788b5:/# cd data/
root@1fbbe52788b5:/data# ls
fichero.txt</pre>

<p style="text-align: justify;">
  Por último para borrar un volumen tenemos que asegurarnos que no está asociado a ningún contenedor:
</p>

<pre>$ docker volume rm vol1
Error response from daemon: Unable to remove volume, volume still in use: remove vol1: volume is in use - [969b3bd0ec30193b9ce2cef837b6231e3c39310caca4660ea0fcab695d99d065]

$ docker rm contenedor2
contenedor2
$ docker volume rm vol1
vol1</pre>

### Montando directorios del host en un contenedor

<p style="text-align: justify;">
  Una aplicación particular del trabajo con volúmenes de datos, es la posibilidad de montar en el contenedor un directorio del docker engine. En este caso hay que tener en cuenta que si el directorio de montaje del contenedor ya existe, no se borra su contenido, simplemente se monta encima. Veamos un ejemplo:
</p>

<pre>$ docker run -it --name contenedor5 -v /home/docker:/informacion ubuntu:14.04 bash
root@c1cf905f170a:/# cd informacion/
root@c1cf905f170a:/informacion# ls
log.log</pre>

<p style="text-align: justify;">
  En este ejemplo hemos montado en el directorio <em>/informacion</em> del contenedor, el directorio <em>/home/docke</em>r del Docker Engine. Y comprobamos que podemos acceder a los ficheros del hosts.
</p>

<h3 style="text-align: justify;">
  Montando ficheros del host en un contenedor
</h3>

<p style="text-align: justify;">
  Además de poder montar un directorio, podemos montar un fichero del Docker Engine. En el siguiente ejemplo vamos a montar el fichero <em>/etc/hosts</em> en el contenedor.
</p>

<pre>$ docker run -it --name contenedor6 -v /etc/hosts:/etc/hosts.bak ubuntu:14.04 bash
root@ff58bc448b57:/# cat /etc/hosts.bak</pre>

## Contenedores de volúmenes de datos

<p style="text-align: justify;">
  Otra posibilidad que tenemos para conseguir el almacenamiento persistente es la creación de un contenedor donde creamos un volumen de datos y que podemos asociar a uno o varios contenedores para guardar la información en él.
</p>

<p style="text-align: justify;">
  Vamos a crear un contenedor con un volumen de datos asociado:
</p>

<pre>$ docker create -it --name data_vol_container -v /shared_folder ubuntu:14.04 bash
ec45dbf110e14f6f4097d2b2dfaec7092669c18c7424913106a46342af54ce23</pre>

<p style="text-align: justify;">
  A continuación con el parámetro <code>--volumes-from</code>, creamos un contenedor asociado al contenedor anterior y que montará el volumen de datos:
</p>

<pre>$ docker run -it --name container1 --volumes-from data_vol_container ubuntu:14.04 bash
root@93aed9e94393:/# cd /shared_folder/
root@93aed9e94393:/shared_folder# echo "hola"&gt;fichero.txt
root@93aed9e94393:/shared_folder# exit</pre>

<p style="text-align: justify;">
  Finalmente creamos un nuevo contenedor asociado al contenedor con el volumen de datos, para comprobar que, efectivamente, se comparte el volumen:
</p>

<pre>$ docker run -it --name container2 --volumes-from data_vol_container ubuntu:14.04 bash
root@a2733d38c49a:/# cat /shared_folder/fichero.txt
hola</pre>

## Conclusiones

<p style="text-align: justify;">
  En esta entrada se ha descrito de forma introductoria las distintas posibilidades que tenemos para conseguir que la información gestionada por nuestros contenedores sea persistentes, es decir, no sea eliminada cuando destruimos el contenedor. Para más información sobre el almacenamiento en Docker puedes consultar la <a href="https://docs.docker.com/engine/userguide/containers/dockervolumes/">documentación oficial</a>.
</p>

<!-- AddThis Advanced Settings generic via filter on the_content -->

<!-- AddThis Share Buttons generic via filter on the_content -->