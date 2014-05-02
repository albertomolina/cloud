---
layout: blog
title: Uso básico de horizon.
menu:
  - Tema 2
---
# Lanzar una instancia

## Crear un par de claves ssh
      
Por razones de seguridad, las imágenes que se incluyen en OpenStack no
suelen tener definidas las contraseñas de los usuarios, sólo es posible
acceder utilizando pares de claves ssh. Cuando se genera una instancia, la
clave pública ssh se inyecta en la misma y sólo el propietario de la
correspondiente clave privada podrá acceder a ella.

Tras iniciar una sesión en Horizon, el primer paso es crear un par de claves
ssh, por lo que hacemos picamos en la pestaña "Pares de claves" (Acceso y
Seguridad).

Hay dos opciones: Crear un par de claves o importarlo. Optamos por la primera
opción, por lo que ponemos un nombre al par de claves (prueba) y se nos
descargará automáticamente la clave privada correspondiente, la enviamos al
directorio ~/.ssh y la protegemos adecuadamente:

       mv prueba.pem ~/.ssh
       chmod 400 ~/.ssh/prueba.pem

## Asignar IP flotante al proyecto

Las direcciones IP flotantes (IPs elásticas en terminología de Amazon) permiten
a las instancias comunicarse con equipos de fuera de su red.

* Hacemos "click" en la pestaña "IPs flotantes" (Acceso y Seguridad)
* Pulsamos en "Asignar IP al proyecto"
* Pulsamos en "Asignar IP" de la ventana emergente

## Ajustar las reglas del grupo de seguridad

Los grupos de seguridad son conjuntos de reglas de cortafuegos a aplicar a cada
instania. Inicialmente está definido el grupo "default". Accedemos a la pestaña
"Grupos de seguridad" (Acceso y seguridad):

* Borramos todas las reglas inicialmente definidas
* Creamos reglas de salida ("egreso") permisivas, permitiendo todas las
  comunicaciones para los protocolos TCP, UDP e ICMP
* Creamos una regla de entrada ("ingreso") aceptando todo el ICMP desde nuestra
  red
* Creamos una regla de entraga ("ingreso") aceptando conexiones SSH desde
* nuestra red

## Lanzar la instancia

Crea una instancia de una imagen GNU/Linux (**Instancias** -> **Lanzar Instancia**).

Los datos necesarios para crear la instancia son los siguientes:
    
* **Origen de la instancia**: Puede ser Imagen o Instantánea, elegimos la
  primera opción. 
* **Imagen**: Podemos elegir de la lista de imágenes disponibles. Vamos a hacer
  una instancia a partir de la imagen **CentOS**. 
* **Nombre** de la instancia
* El **sabor** de la máquina que vamos a crear.
* Número de instancias, por defecto 1.
* Grupo de seguridad, por defecto "default".
* El par de **claves SSH** con el que vamos a acceder a la instancia.
* La **red** a la que está conectada nuestra instancia.

Una vez creada la instancia, es posible acceder a ella utilizando la consola,
pulsando sobre el nombre de la instancia (desde el apartado **Instancias**) y
luego la pestaña **Consola**. 

Cuando se crea, la instancia toma por DHCP una dirección IP fija de la red a la
que está conectada. Durante la vida de la instancia está IP será siempre la
misma, de ahí el nombre de IP fija, a pesar de que se asigna por DHCP.

## Asociar IP flotante

Para poder acceder a nuestra instancia desde el exterior tenemos que asignarle a
la instancia una IP flotante, para ello elegimos la IP flotante creada
inicialmente. Podemos desasociar esta IP flotante de una instancia y asignarla a
otra, de ahí el nombre de IP flotante, que puede cambiar de una instancia a
otra.

## Acceder a la instancia

Por último, utilizando un cliente ssh accede a la instancia utilizando la IP
flotante que le hemos asignado a la instancia, por ejemplo:

        ssh -i prueba.pem centos@192.168.0.15
