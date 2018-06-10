---
id: 517
title: Instalación de Alfresco en Debian Squeeze
date: 2011-11-10T23:12:33+00:00
author: admin

guid: http://www.josedomingo.org/pledin/?p=517
permalink: /2011/11/instalacion-alfresco-en-debian-squeeze/

  
tags:
  - Alfresco
  - Apache
  - CMS
  - Manuales
  - Tomcat
---
[<img class="aligncenter size-medium wp-image-518" title="alfresco1" src="{{ site.url }}{{ site.baseurl }}/assets/wp-content/uploads/2011/11/alfresco1-300x86.jpg" alt="" width="300" height="86" srcset="https://www.josedomingo.org/pledin/wp-content/uploads/2011/11/alfresco1-300x86.jpg 300w, https://www.josedomingo.org/pledin/wp-content/uploads/2011/11/alfresco1-1024x294.jpg 1024w, https://www.josedomingo.org/pledin/wp-content/uploads/2011/11/alfresco1.jpg 1600w" sizes="(max-width: 300px) 100vw, 300px" />]({{ site.url }}{{ site.baseurl }}/assets/wp-content/uploads/2011/11/alfresco1.jpg)

Este artículo está basado en este [otro](http://widget.linkwithin.com/redirect?url=http%3A//yoadminsis.blogspot.com/2010/04/instalacion-alfresco-33-community-war_15.html&vars=%5B%22http%3A//yoadminsis.blogspot.com/search/label/alfresco%22%2C%20290792%2C%201%2C%20%22http%3A//yoadminsis.blogspot.com/2010/04/03-instalacion-alfresco-33-community.html%22%2C%2044707827%2C%200%2C%2044707835%5D&ts=1320959033698), y sobre todo en mi propia experiencia instalando Alfresco Comunity 4.0 en  Debian Squeeze. La instalación se ha realizado sobre una máquina virtual con 3 gigabytes de memoria RAM. Empecemos&#8230;

Alfresco es un gestor de Documentación profesional y Open Source. Tiene dos versiones. Una versión Enterprise (de pago y con servicio técnico, etc.) y Community (gratuita y sin garantía). Para la instalación de Alfresco 4.0 Community usaremos los ficheros WAR que sirven para desplegar aplicaciones en un servidor Tomcat ya existente. Necesitaremos hacer los siguientes pasos:

**Instalación de Java**
  
En mi caso voy a usar el Java de Oracle, recuerda que debes tener los repositorios non-free configurados:

<pre class="brush: bash; gutter: false; first-line: 1">aptitude install sun-java6-bin sun-java6-jre sun-java6-jdk sun-java6-plugin sun-java6-fonts libcommons-el-java</pre>

Cuando nos lo pida en la consola aceptamos la licencia. Si tuviéramos ya instalado otro java podríamos hacer que se use por defecto el de Sun con:

<pre class="brush: bash; gutter: false; first-line: 1">update-alternatives --set java /usr/lib/jvm/java-6-sun/jre/bin/java</pre>

**Instalación de MySQL**
  
Instalamos el servidor mysql:

<pre class="brush: bash; gutter: false; first-line: 1">aptitude install mysql-server-5.1 mysql-client-5.1 libmysql-java</pre>

libmysql-java es el controlador JDBC de MySQL version 5.1.10. Lo configuramos en la instalación de tomcat6.

**Instalamos el servidor de aplicaciones Tomcat 6.**

<pre class="brush: bash; gutter: false; first-line: 1">aptitude install tomcat6 tomcat6-admin tomcat6-docs tomcat6-examples tomcat6-user</pre>

Para manejar el gestor de aplicaciones y el gestor de hosts de tomcat desde el explorador web debemos añadir nuestro usuario a los roles manager y admin de tomcat. Para ello configuramos el fichero /etc/tomcat6/tomcat-user.xml y añadimos el usuario a la sección tomcat-users. Sustituimos usuario y contraseña por los nuestros.

<pre class="brush: xml; gutter: false; first-line: 1">&lt;tomcat-users&gt;
&lt;user username="usuario" password="contraseña" roles="admin,manager"/&gt;
&lt;/tomcat-users&gt;</pre>

Para configurar el conector AJP y que así el servidor web Apache envíe las peticiones a Tomcat configuramos el siguiente fichero y descomentamos la linea del conector.

<pre class="brush: bash; gutter: false; first-line: 1">nano /etc/tomcat6/server.xml

&lt;!-- Define an AJP 1.3 Connector on port 8009 --&gt;
&lt;Connector port="8009" protocol="AJP/1.3" redirectPort="8443" /&gt;</pre>

Creamos el directorio endorsed en /var/lib/tomcat6/common

<pre class="brush: bash; gutter: false; first-line: 1">mkdir /var/lib/tomcat6/common/endorsed
chown -R tomcat6:tomcat6 /var/lib/tomcat6/common/endorsed</pre>

Para que podamos usar alfresco correctamente debemos cambiar unos parámetros en el fichero /etc/default/tomcat6

Pra resolver el problema de &#8220;Out of Memory&#8221; que nos puede surgir con alfresco lo cambiamos dependiendo de la memoria que tengamos, se recomienda tener al menos 3G.

<pre class="brush: bash; gutter: false; first-line: 1">JAVA_OPTS="-Djava.awt.headless=true  -Dfile.encoding=UTF-8 -server -Xms1536m -Xmx1536m -XX:NewSize=256m -XX:MaxNewSize=256m -XX:PermSize=256m -XX:MaxPermSize=256m -XX:+DisableExplicitGC  -Xmx3G  -Djava.endorsed.dirs=/usr/share/tomcat6/endorsed:/var/lib/tomcat6/common/endorsed"</pre>

Habilitamos esta opción para que tomcat o sus aplicaciones puedan usar puertos por debajo de 1024 con usuarios no privilegiados (como tomcat6).

<pre class="brush: bash; gutter: false; first-line: 1">AUTHBIND=yes</pre>

Hemos terminado con ese fichero, por otro lado indicar que libmysql-java es el controlador JDBC de MySQL y libcommons-el-java son componentes reusables opensource de java. Esta instalación instala los jar JDBC de mysql y commons en java pero para que funcione correctamente con tomcat debemos incluirlo en el classpath de tomcat. Esto lo hacemos añadiendo un link simbólico al jar de mysql y commons en java en el directorio /usr/share/tomcat6/lib.

<pre class="brush: bash; gutter: false; first-line: 1">cd /usr/share/tomcat6/lib
ln -s ../../java/mysql.jar mysql.jar
ln -s ../../java/commons-el.jar commons-el.jar</pre>

En /usr/share/java tenemos un link simbólico llamado mysql.jar que apunta al jdbc de mysql (mysql-connector-java-5.1.10.jar) en el mismo directorio. Por eso apuntamos mysql.jar en el directorio de tomcat a mysql.jar en el directorio de java. Un link a otro link.

Reiniciamos tomcat

<pre class="brush: bash; gutter: false; first-line: 1">/etc/init.d/tomcat6 restart</pre>

Y vemos que funciona en http://localhost:8080 y que podemos acceder al manager y al hostmanager con el usuario y contraseña que pusimos.

**Instalación de Apache 2.2**

Instalamos el servidor web de apache. No es necesario en principio pero lo haremos para que sea apache el que se encargue de las peticiones web.

<pre class="brush: applescript; gutter: false; first-line: 1">aptitude install apache2 apache2-utils</pre>

No voy a entrar en la configuración de apache, solo configuraremos el conector de apache a tomcat.

Para conectar el servidor web con el servidor de aplicaciones (contenedor de servlets y JSP) se usa el protocolo AJP. Para configurar esto se pueden usar, o el módulo de apache mod\_jk o el módulo mod\_proxy. La recomendación es usar el módulo mod_proxy ya que es más moderno y es el que configuraremos. También se puede usar para balanceo de carga, clusters&#8230; Por defecto, ya viene instalado con apache2.

En la instalación de Tomcat ya configuramos el conector AJP para que funcionara correctamente. Ahora nos ocupamos de la parte de configuración de apache. Para configurar el conector editamos el fichero de configuración de mod proxy y lo dejamos así (Cambia la dirección ip del servidor tomcat):

<pre class="brush: bash; gutter: false; first-line: 1">nano /etc/apache2/mods-available/proxy.conf</pre>

<pre class="brush: bash; gutter: true; first-line: 1">&lt;IfModule mod_proxy.c&gt;
        #turning ProxyRequests on and allowing proxying from all may allow
        #spammers to use your proxy to send email.
        # Con esta directiva en Off hacemos que se deshabilite la redirección del
        # proxy excepto para las entradas que nosotros pongamos con ProxyPass
        ProxyRequests Off

        # Hace que las peticiones originales de información de host se mantengan a
        # través de la conexión del conector AJP. Es útil para aplicaciones que
        # necesitan mantener esta información.
        ProxyPreserveHost On

        # Indica que todos los hosts (*) pueden acceder a traves del proxy
        &lt;Proxy *&gt;
                AddDefaultCharset off
                Order deny,allow
                Allow from all
                #Allow from .example.com
        &lt;/Proxy&gt;

        # Enable/disable the handling of HTTP/1.1 "Via:" headers.
        # ("Full" adds the server version; "Block" removes all outgoing Via: headers)
        # Set to one of: Off | On | Full | Block
        ProxyVia On

        # Indica que las peticiones a apache para /alfresco se pasen por el protocolo AJP
        # a la dirección IP y puerto donde Tomcat está escuchando mediante el protocolo
        # AJP con un conector.
        ProxyPass /alfresco ajp://192.168.1.35:8009/alfresco

        # Indica que cualquier peticion de cabeceras (request headers) del reverse proxy
        # debería ser reescrita de forma adecuada para asegurar que las redirecciones que
        # haga el servidor Tomcat sean manejadas de forma correcta.
        ProxyPassReverse /alfresco ajp://192.168.1.35:8009/alfresco

        # Permitimos el acceso a la aplicación alfresco
        &lt;Location /alfresco &gt;
                Order allow,deny
                Allow from all
        &lt;/Location&gt;

        # Bloque para share.war. Con cada aplicación desplegada que queramos incluir en
        # el conector deberemos poner el bloque siguiente.
        ProxyPass /share ajp://192.168.1.35:8009/share
        ProxyPassReverse /share ajp://192.168.1.35:8009/share
        &lt;Location /share&gt;
                Order allow,deny
                Allow from all
        &lt;/Location&gt;

&lt;/IfModule&gt;</pre>

Activamos los módulos proxy (se configuran en el mismo archivo anterior)

<pre class="brush: applescript; gutter: false; first-line: 1">a2enmod proxy_balancer proxy_ajp proxy</pre>

Y reiniciamos apache2.

En la siguiente imagen vemos como el conector, apache y tomcat están funcionando. No usamos directamente el puerto 8080 de tomcat si no el 80 de apache. El error de Alfresco es simplemente debido a que todavía no lo hemos instalado en su sitio.

[<img class="aligncenter size-medium wp-image-532" title="Pantallazo-Apache Tomcat-6.0.28 - Informe de Error - Google Chrome" src="{{ site.url }}{{ site.baseurl }}/assets/wp-content/uploads/2011/11/Pantallazo-Apache-Tomcat-6.0.28-Informe-de-Error-Google-Chrome-300x157.png" alt="" width="300" height="157" srcset="https://www.josedomingo.org/pledin/wp-content/uploads/2011/11/Pantallazo-Apache-Tomcat-6.0.28-Informe-de-Error-Google-Chrome-300x157.png 300w, https://www.josedomingo.org/pledin/wp-content/uploads/2011/11/Pantallazo-Apache-Tomcat-6.0.28-Informe-de-Error-Google-Chrome-1024x538.png 1024w, https://www.josedomingo.org/pledin/wp-content/uploads/2011/11/Pantallazo-Apache-Tomcat-6.0.28-Informe-de-Error-Google-Chrome.png 1366w" sizes="(max-width: 300px) 100vw, 300px" />]({{ site.url }}{{ site.baseurl }}/assets/wp-content/uploads/2011/11/Pantallazo-Apache-Tomcat-6.0.28-Informe-de-Error-Google-Chrome.png)**Instalación de herramientas adicionales**

Para un correcto funcionamiento necesitamos instalar las siguientes herramientas.

  * Flash 10.x.
  * SWF Tools: para la conversion de pdf y swf usar la vista previa de pdfs.
  * OpenOffice.org
  * Imagemagick: Para manipulación de imágenes. Ya viene por defecto, pero por si acaso lo incluimos.

Las herramientas que están en los repositorios de Debian las instalamos:

<pre class="brush: bash; gutter: false; first-line: 1">aptitude install  flashplugin-nonfree openoffice.org imagemagick</pre>

Sin embargo no existe en Debian un paquete para instalar SWF Tools, por lo tanto es necesario compilarlas, para ello:

Descargate la última versión de la herramienta con:

<pre class="brush: bash; gutter: false; first-line: 1">wget http://www.swftools.org/swftools-2011-10-10-1647.tar.gz</pre>

Antes de compilarla instala algunas herramientas necesarias:

<pre class="brush: bash; gutter: false; first-line: 1">apt-get install libjpeg62-dev libfreetype6-dev libpng3-dev libt1-dev libungif4-dev make build-essential</pre>

Descomprimimos el fichero que hemos bajado y lo compilamos:

<pre class="brush: bash; gutter: false; first-line: 1">./configure
make
make install</pre>

Puedes probar que se ha instalado de manera adecuada ejecuntando:

<pre class="brush: bash; gutter: false; first-line: 1">pdf2swf -V</pre>

Por último crea el siguinete enlace símbolico para que se pueda localizar el programa:

<pre class="brush: bash; gutter: false; first-line: 1">ln -s /usr/local/bin/pdf2swf /usr/bin/pdf2swf</pre>

**Instalación de Alfresco 4.0**

Alfresco se compone de varios elementos con distintas funcionalidades. Al instalar el archivo WAR es necesario instalar estos componentes por separado. Si usáramos el paquete de instalación, incluiría tomcat, openoffice, etc. Pero nosotros queremos instalar el fichero war solo.

Nos bajaremos los ficheros desde http://wiki.alfresco.com/wiki/Community\_file\_list_4.0.b

En concreto yo me bajo los siguientes. Todos no son necesarios para una instalación básica de alfresco y share.

_alfresco-community-4.0.b.zip
  
alfresco-community-webeditor-4.0.b.zip
  
alfresco-community-deployment-4.0.b.zip_

En principio, para la configuración inicial usaremos el primero de la lista que contienen los WAR. Lo bajamos, descomprimimos y comprobamos que tenemos dos archivos WAR (en el directorio /web-server/webapps) que se auto desplegarán en el servidor de aplicaciones y otros directorios y ficheros.

  * Alfresco.war es la aplicación core de gestión de la documentación.
  * Share.war es la aplicación de gestión de contenidos y documentación.

Una vez descomprimidos vamos a crear la base de datos en MySQL. Para ello debemos usar el script

<pre class="brush: bash; gutter: false; first-line: 1">nano db_setup.sql
create database alfresco default character set utf8 collate utf8_bin;
grant all on alfresco.* to 'alfresco'@'localhost' identified by 'alfresco' with grant option;
grant all on alfresco.* to 'alfresco'@'localhost.localdomain' identified by 'alfresco' with grant option;</pre>

Vemos que lo que hace es crear la base de datos para utf8 por defecto y los usuarios por defecto alfresco y contraseña alfresco. Lo dejamos así por ahora y creamos la base de datos y comprobamos que existe. Si queremos podemos poner nuestra contraseña ahora y luego acordarnos de cambiarla en el fichero de propiedades (más adelante) También comprobamos que existen los usuarios nuevos y recargamos las tablas de permisos.

<pre class="brush: bash; gutter: false; first-line: 1">mysql -u root -p &lt; db_setup.sql
mysql -u root -p -e "select user,host,password from user where user like 'alfresco'" mysql
mysqladmin -u root -p flush-privileges</pre>

Creamos el directorio donde vamos a guardar el repositorio de alfresco que es donde se van a guardar los índices y los ficheros que subamos (documentos, etc.). Después cambiamos los permisos para que tomcat pueda leer los datos.

<pre class="brush: bash; gutter: false; first-line: 1">mkdir /srv/alfresco/alf_data
chown -R tomcat6:tomcat6 /srv/alfresco</pre>

Copiamos todos los ficheros del directorio endorsed al directorio de tomcat, endorsed.

<pre class="brush: bash; gutter: false; first-line: 1">cd web-server
cp endorsed/* /var/lib/tomcat6/common/endorsed/
chown -R tomcat6:tomcat6 /var/lib/tomcat6/common/endorsed</pre>

Copiamos los ficheros .war al directorio de aplicaciones de tomcat donde se autodesplegarán simplemente copiándolos en el directorio.

<pre class="brush: bash; gutter: false; first-line: 1">cd webapps
cp alfresco.war share.war /var/lib/tomcat6/webapps/</pre>

**Configuración básica de Alfresco**

Una vez desplegados los war, Alfresco tiene dos directorios de configuración, uno para el mismo y otro para share que son:

  * <configroot>: /var/lib/tomcat6/webapps/alfresco/WEB-INF
  * <configrootshare>: /var/lib/tomcat6/webapps/share/WEB-INF

Dentro de estos directorios se puede configurar alfresco o share.

El fichero general de configuración de alfresco lo tenemos en /var/lib/tomcat6/webapps/alfresco/WEB-INF/classes/alfresco-global.properties.sample. Lo debemos copiar con el mismo nombre pero sin la extensión sample.

Y debe quedar de la siguiente manera:

<pre class="brush: bash; gutter: true; first-line: 1">###############################
## Common Alfresco Properties #
###############################

#
# Sample custom content and index data location
#-------------
# Repositorio de documentación de Alfresco que
# creamos antes.
dir.root=/srv/alfresco/alf_data

#
# Sample database connection properties
#-------------
# Datos de conexión de la base de datos. Si cambiamos
# la contraseña o el usuario al configurar mysql
# debemos cambiarlos aquí.
db.name=alfresco
db.username=alfresco
db.password=alfresco
db.host=localhost
db.port=3306

#
# External locations
#-------------
# Rutas a los programas que necesita alfresco y que
# instalamos previamente. Podemos usar whereis.
ooo.exe=/usr/bin/soffice
#ooo.enabled=false
#img.root=./ImageMagick
img.exe=/usr/bin/convert
#swf.exe=/usr/bin/pdf2swf

#
# MySQL connection
#-------------
# Driver de mysql. Por defecto usa el dialecto
# hybernate de InnoDB asi que no ponemos nada más.
db.driver=org.gjt.mm.mysql.Driver
db.url=jdbc:mysql://${db.host}:${db.port}/${db.name}

#
# Index Recovery Mode
#-------------
index.recovery.mode=FULL

#
# Outbound Email Configuration
#-------------
mail.host=localhost
mail.port=25
#mail.username=anonymous
#mail.password=
mail.encoding=UTF-8
mail.from.default=admin@dominio.com
#mail.smtp.auth=false</pre>

**Últimos pasos**

Vamos terminando. Tenemos que indicar en que fichero se van a guardar los log de alfresco. Para ello modificamos el fichero /var/lib/tomcat6/webapps/alfresco/WEB-INF/classes/log4j.properties y modificar la siguiente línea:

<pre class="brush: bash; gutter: false; first-line: 1">log4j.appender.File.File=/var/log/tomcat6/alfresco.log</pre>

Del mismo modo lo tendrás que hacer en el fichero: /var/lib/tomcat6/webapps/share/WEB-INF/classes/log4j.properties

Si vamos ahora a http://localhost/alfresco veremos, por fin, la página de alfresco funcionando.

[<img class="aligncenter size-medium wp-image-535" title="Pantallazo-Mi Alfresco - Google Chrome" src="{{ site.url }}{{ site.baseurl }}/assets/wp-content/uploads/2011/11/Pantallazo-Mi-Alfresco-Google-Chrome-300x240.png" alt="" width="300" height="240" srcset="https://www.josedomingo.org/pledin/wp-content/uploads/2011/11/Pantallazo-Mi-Alfresco-Google-Chrome-300x240.png 300w, https://www.josedomingo.org/pledin/wp-content/uploads/2011/11/Pantallazo-Mi-Alfresco-Google-Chrome-1024x819.png 1024w, https://www.josedomingo.org/pledin/wp-content/uploads/2011/11/Pantallazo-Mi-Alfresco-Google-Chrome.png 1280w" sizes="(max-width: 300px) 100vw, 300px" />]({{ site.url }}{{ site.baseurl }}/assets/wp-content/uploads/2011/11/Pantallazo-Mi-Alfresco-Google-Chrome.png)Recomiendo que en los últimos pasos de la instalación se vayan monitorizando los ficheros de logs: catalina.out y alfresco.log. Si surge algún error la mejor manera de solucionar es buscar en internet donde he encontrado mucha información sobre la instalación de alfresco. Espero quele sirva a alguien.

Un saludo.

<!-- AddThis Advanced Settings generic via filter on the_content -->

<!-- AddThis Share Buttons generic via filter on the_content -->