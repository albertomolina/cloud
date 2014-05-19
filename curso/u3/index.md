---
layout: blog
tittle: Utilización desde línea de comandos
menu:
  - Tema 4
---

## OpenStack CLI

Es muchas ocasiones es adecuado utilizar OpenStack desde la línea de comandos,
para lo cual hay que tener instalado en la máquina cliente la o las aplicaciones
adecuadas. Estas aplicaciones son programas en Python que utilizan las APIs
RESTful de cada uno de los componentes de OpenStack, así hay un
python-novaclient, python-cinderclient, etc. aunque el primero es el denominado
cliente unificado ya que no solo interactúa con nova, sino que además incluye una
funcionalidad básica para comunicarse con otros componentes de OpenStack
(cinder, glance y neutron)

### Instalación de las aplicaciones cliente

Podríamos utilizar de OpenStack de la distribución del equipo cliente, pero
también es posible instalarlo desde Pypi y quizás más conveniente porque son
paquetes bastante independientes y pequeños.

Lo primero es instalar pip en nuestro sistema: 

    # apt-get install python-pip <- Debian/Ubuntu
	# yum install python-pip <- CentOS/Red Hat

Instalar la última versión del cliente de OpenStack nova es muy sencillo:

	# pip install python-novaclient

Para instalar una versión específica (por ejemplo la 2.16.0 para OpenStack
Havana):

	# pip install python-novaclient==2.16.0

Si quisiéramos tener instaladas más de una versión de uno de estos paquetes
podríamos utilizar <a
href="http://docs.python-guide.org/en/latest/dev/virtualenvs/">Python virtual
environments</a>.

De igual forma podemos instalar los clientes del resto de componentes de
OpenStack (versiones adecuadas para Havana):

	# pip install python-neutronclient==2.3.4
	# pip install python-cinderclient==1.0.7

### Credenciales de acceso

Para poder utilizar el cliente de OpenStack es necesario previamente
autenticarse frente a keystone, para lo cual necesitamos conocer algunos
parámetros como nombre de usario, contraseña, URL de acceso al servicio de
autenticación (Endpoint) o información sobre el proyecto al que pertenece el
usuario.

La forma más sencilla de conocer todos estos parámetros es descargarse desde
Horizon el fichero openrc.sh: "Acceso y Seguridad" > "Acceso a la API" >
"Descargar archivo RC de OpenStack". El contenido de este fichero es algo
similar a:

	#!/bin/bash

	# With the addition of Keystone, to use an openstack cloud you should
	# authenticate against keystone, which returns a **Token** and **Service
	# Catalog**.  The catalog contains the endpoint for all services the
	# user/tenant has access to - including nova, glance, keystone, swift.
	#
	# *NOTE*: Using the 2.0 *auth api* does not mean that compute api is 2.0.  We
	# will use the 1.1 *compute api*
	export OS_AUTH_URL=http://192.168.0.68:5000/v2.0
	
	# With the addition of Keystone we have standardized on the term **tenant**
	# as the entity that owns the resources.
	export OS_TENANT_ID=0912c80bef254d7b9352632793cf75b9
	export OS_TENANT_NAME="demo"
	
	# In addition to the owning entity (tenant), openstack stores the entity
	# performing the action as the **user**.
	export OS_USERNAME="demo"
	
	# With Keystone you pass the keystone password.
	echo "Please enter your OpenStack Password: "
	read -sr OS_PASSWORD_INPUT
	export OS_PASSWORD=$OS_PASSWORD_INPUT

Es conveniente ejecutarlo con source para evitar los problemas que podríamos
tener en Debian/Ubuntu con la utilización de la shell dash:

    $ source demo-openrc.sh

Y para probar que efectivamente estamos autenticados en OpenStack, podemos
simplemente obtener la lista de sabores de nova:

	$ nova flavor-list
	+----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
	| ID | Name      | Memory_MB | Disk | Ephemeral | Swap | VCPUs | RXTX_Factor | Is_Public |
	+----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
	| 1  | m1.tiny   | 512       | 1    | 0         |      | 1     | 1.0         | True      |
	| 2  | m1.small  | 2048      | 20   | 0         |      | 1     | 1.0         | True      |
	| 3  | m1.medium | 4096      | 40   | 0         |      | 2     | 1.0         | True      |
	| 4  | m1.large  | 8192      | 80   | 0         |      | 4     | 1.0         | True      |
	| 5  | m1.xlarge | 16384     | 160  | 0         |      | 8     | 1.0         | True      |
	+----+-----------+-----------+------+-----------+------+-------+-------------+-----------+

