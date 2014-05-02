---
layout: blog
title: Uso básico de horizon.
menu:
  - Tema 2
---
# Lanzar una instancia desde OpenStack Horizon

## Escenario

Aunque el siguiente procedimiento es bastante general, puede haber pequeñas
diferencias en función de la versión de OpenStack utilizada, por lo que
detallamos las características del montaje utilizado:

* OpenStack Havana (2013.2) sobre CentOS 6.5 utilizando RDO
* OpenStack neutron con plugin OpenvSwitch en una configuración "Per tenant
  routers with private networks"
* Red privada 10.0.0.0/24
* Direcciones IP flotantes 192.168.0.15-192.168.0.25
* Nombre de usuario: demo
* Proyecto: demo

### Crear un par de claves ssh

Por razones de seguridad, las imágenes que se incluyen en OpenStack no suelen
tener definidas las contraseñas de los usuarios, sólo es posible acceder
utilizando pares de claves ssh. Cuando se genera una instancia, la clave pública
ssh se inyecta en la misma y sólo el propietario de la correspondiente clave
privada podrá acceder a ella.

Tras iniciar una sesión en Horizon, el primer paso es crear un par de claves
ssh, por lo que hacemos picamos en la pestaña "Pares de claves" (Acceso y
Seguridad).

Hay dos opciones: Crear un par de claves o importarlo. Optamos por la primera
opción, por lo que ponemos un nombre al par de claves (prueba) y se nos
descargará automáticamente la clave privada correspondiente, la enviamos al
directorio ~/.ssh y la protegemos adecuadamente:

       mv prueba.pem ~/.ssh
       chmod 400 ~/.ssh/prueba.pem

### Asignar IP flotante al proyecto

Las direcciones IP flotantes (IPs elásticas en terminología de Amazon) permiten
a las instancias comunicarse con equipos de fuera de su red.

* Hacemos "click" en la pestaña "IPs flotantes" (Acceso y Seguridad)
* Pulsamos en "Asignar IP al proyecto"
* Pulsamos en "Asignar IP" de la ventana emergente

### Lanzar una instancia

Vamos a "Imágenes e instantáneas" y seleccionamos una imagen 
Click on “Images & Snapshots” and launch a required instance from the list of
images available (in this case we’ll use Debian wheezy image).

images

Provide an instance name and select a flavor in “Details” tab:

instance1

Click on “Access & security” tab and select the right keypair:

instance2

Click on “Networking” tab and select desired network from available
networks. Volume options and post-Creation can be ignored in this simple test,
so click on launch bottom:

instance3

After a few seconds the instance is active and a fixed IP 10.0.0.2 has been
assigned to it. Clicking on “More” button shows several options, click on
“Associate Floating IP”:
instace-running1

Floting IP 172.22.196.59 is now associated to port with IP 10.0.0.2:

floatingip4

It is possible to see instance virtual console, but it is not possible to log in
because user password is not set.

spice
Security Group rules

Instance is launched and floating IP associated so it should be possible to
access via ssh, but this is not yet possible due to default firewall
behavior. Incoming connections must be explicitly allowed as rules in a security
group.

Click on Security groups tab in “Access & security” and click on “Edit Rules”
button:

secgroup1

Add a rule to allow incoming ssh connections (22/tcp):

secgroup2

Add a rule to allow all incoming icmp connections:

secgroup3

Two rules are now defined:

secgroup4
Access to the launched instance

Now we can ping to the instance:

$ ping 172.22.196.59
PING 172.22.196.59 (172.22.196.59) 56(84) bytes of data.
64 bytes from 172.22.196.59: icmp_req=1 ttl=62 time=621 ms
64 bytes from 172.22.196.59: icmp_req=2 ttl=62 time=136 ms
64 bytes from 172.22.196.59: icmp_req=3 ttl=62 time=143 ms
^C
--- 172.22.196.59 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2001ms
rtt min/avg/max/mdev = 136.858/300.532/621.666/227.090 ms

And use the ssh command to make a secure connection to the instance (specifying
the private key to use):

$ ssh -i ~/.ssh/openstack-bisharron.pem debian@172.22.196.59
The authenticity of host '172.22.196.59 (172.22.196.59)' can't be established.
ECDSA key fingerprint is 2f:23:72:6f:13:b8:f0:00:9a:fb:90:64:da:3f:58:9d.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.22.196.59' (ECDSA) to the list of known hosts.
Linux debian.example.com 3.2.0-4-amd64 #1 SMP Debian 3.2.46-1 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
debian@debian-test:~$

Clicking on “Network topology” we can see a nice representation of the project
elements (routers, networks and servers):

###Objetivos

* Conocer los conceptos fundamentales para poder utilizar OpenStack.
* Utilizar OpenStack desde el panel web (Horizon)

### Conceptos Previos

* [Conceptos previos](conceptos_previos.html)

### Lanzar una instancia desde OpenStack Horizon

* [Lanzar una instancia](lanzar_instancia)

###Ejercicios

* [Ejercicio: Trabajar con instancias GNU/Linux](practica_linux)

### Enlaces interesantes

* [How to launch an instance on OpenStack(I): Horizon](http://albertomolina.wordpress.com/2013/11/20/how-to-launch-an-instance-on-openstack-i-horizon/)