---
layout: blog
tittle: Utilización desde línea de comandos
menu:
  - Tema 3
---

## Redes

Podemos utilizar el cliente nova o neutron, en este caso hemos optado por
utilizar neutron.

### Creación de una red privada

En primer lugar creamos una red:

	$ neutron net-create privada
	Created a new network:
	+----------------+--------------------------------------+
	| Field          | Value                                |
	+----------------+--------------------------------------+
	| admin_state_up | True                                 |
	| id             | f84a328e-407e-4ca6-87bb-ec148853c585 |
	| name           | privada                              |
	| shared         | False                                |
	| status         | ACTIVE                               |
	| subnets        |                                      |
	| tenant_id      | 4beb810ce40f49659e0bca732e4f1a3c     |
	+----------------+--------------------------------------+

A continuación y dentro de esa red, definimos una subred, es decir un bloque de
direcciones IP que se asignarán a las instancias que se creen en la red.

	$ neutron subnet-create f84a328e-407e-4ca6-87bb-ec148853c585 10.0.0.0/24
	Created a new subnet:
	+------------------+--------------------------------------------+
	| Field            | Value                                      |
	+------------------+--------------------------------------------+
	| allocation_pools | {"start": "10.0.0.2", "end": "10.0.0.254"} |
	| cidr             | 10.0.0.0/24                                |
	| dns_nameservers  |                                            |
	| enable_dhcp      | True                                       |
	| gateway_ip       | 10.0.0.1                                   |
	| host_routes      |                                            |
	| id               | d4bb2d0e-2af7-44fe-9729-4e3b95766e28       |
	| ip_version       | 4                                          |
	| name             |                                            |
	| network_id       | f84a328e-407e-4ca6-87bb-ec148853c585       |
	| tenant_id        | 4beb810ce40f49659e0bca732e4f1a3c           |
	+------------------+--------------------------------------------+


	$ neutron security-group-rule-list
	+--------------------------------------+----------------+-----------+----------+------------------+--------------+
	| id                                   | security_group | direction | protocol | remote_ip_prefix | remote_group |
	+--------------------------------------+----------------+-----------+----------+------------------+--------------+
	| 68a447f4-95ef-4571-b209-17288bb17b2b | default        | ingress   |          |                  | default      |
	| 826d3068-9173-4477-be85-a304ab90a442 | default        | ingress   |          |                  | default      |
	| a008aa84-5543-4a24-8398-751b3aa3d1a8 | default        | egress    |          |                  |              |
	| ffa55db3-00ae-4e8f-8e03-92f76d998ba7 | default        | egress    |          |                  |              |
	+--------------------------------------+----------------+-----------+----------+------------------+--------------+
	
Añadimos una nueva regla para que puedan comunicarse por ICMP las máquinas de la red 10.0.0.0/24:

	$ neutron security-group-rule-create --direction ingress --protocol icmp --remote-ip-prefix 10.0.0.0/24 default
	
Para permitir de forma explícita la comunicación a algún puerto UDP o TCP se
utilizaría una regla de seguridad como la siguiente para el servicio ssh
(22/tcp):

	$ neutron security-group-rule-create --direction ingress --protocol tcp --port-range-min 22 --port-range-max 22 --remote-ip-prefix 10.0.0.0/24 default

##### Reglas permisivas en la red privada

Si queremos que las instancias tengan acceso a todos los puertos del resto de
instancias de su misma red privada, podemos definir reglas de seguridad
permisivas para esta situación, abriendo todo el rango de puertos UDP y TCP: 

	$ neutron security-group-rule-create --direction ingress --protocol tcp --port-range-min 1 --port-range-max 65535 --remote-ip-prefix 10.0.0.0/24 default
	$ neutron security-group-rule-create --direction ingress --protocol udp --port-range-min 1 --port-range-max 65535 --remote-ip-prefix 10.0.0.0/24 default

## Red externa

Creamos **como administrador** de neutron una red externa:

	# neutron net-create ext_net -- --router:external=True
	Created a new network:
	+---------------------------+--------------------------------------+
	| Field                     | Value                                |
	+---------------------------+--------------------------------------+
	| admin_state_up            | True                                 |
	| id                        | 3292768d-b916-45bc-8ce5-cbab121d6d01 |
	| name                      | ext_net                              |
	| provider:network_type     | gre                                  |
	| provider:physical_network |                                      |
	| provider:segmentation_id  | 3                                    |
	| router:external           | True                                 |
	| shared                    | False                                |
	| status                    | ACTIVE                               |
	| subnets                   |                                      |
	| tenant_id                 | 635dd2732ceb468e8518e556248c23d0     |
	+---------------------------+--------------------------------------+

Y asociamos una subred en la que especificamos el depósito de IP asignables:

	# neutron subnet-create --allocation-pool \
	start=192.168.0.15,end=192.168.0.45 ext_net 192.168.0.0/24 -- \
	--enable_dhcp=False
	Created a new subnet:
	+------------------+--------------------------------------------------+
	| Field            | Value                                            |
	+------------------+--------------------------------------------------+
	| allocation_pools | {"start": "192.168.0.15", "end": "192.168.0.45"} |
	| cidr             | 192.168.0.0/24                                   |
	| dns_nameservers  |                                                  |
	| enable_dhcp      | False                                            |
	| gateway_ip       | 192.168.0.1                                      |
	| host_routes      |                                                  |
	| id               | be85bfa6-6dd5-4b59-bdce-614e669c9f3e             |
	| ip_version       | 4                                                |
	| name             |                                                  |
	| network_id       | 3292768d-b916-45bc-8ce5-cbab121d6d01             |
	| tenant_id        | 635dd2732ceb468e8518e556248c23d0                 |
	+------------------+--------------------------------------------------+

## Routers

En el despliegue de OpenStack que estamos utilizando cada proyecto puede definir
sus propias redes privadas y routers, estos routers son los dispositivos que
permiten conectar las redes privadas con la red exterior.

Como usuario normal, definimos un nuevo router para conectar la red privada con
el exterior:

	$ neutron router-create r1
	Created a new router:
	+-----------------------+--------------------------------------+
	| Field                 | Value                                |
	+-----------------------+--------------------------------------+
	| admin_state_up        | True                                 |
	| external_gateway_info |                                      |
	| id                    | cc5fd2f5-59d6-484d-a759-819917a5610c |
	| name                  | r1                                   |
	| status                | ACTIVE                               |
	| tenant_id             | 4beb810ce40f49659e0bca732e4f1a3c     |
	+-----------------------+--------------------------------------+

### Conexión a la red exterior

Conectamos el router a la red exterior, lo que se denomina en neutron configurar
la puerta de enlace:

	$ neutron router-gateway-set cc5fd2f5-59d6-484d-a759-819917a5610c \
	3292768d-b916-45bc-8ce5-cbab121d6d01
	Set gateway for router cc5fd2f5-59d6-484d-a759-819917a5610c

### Conexión a la red privada

Conectamos el router a la red privada (es necesario conocer el id de la subred)
mediante la instrucción:

    $ neutron router-interface-add cc5fd2f5-59d6-484d-a759-819917a5610c d4bb2d0e-2af7-44fe-9729-4e3b95766e28
	Added interface db992449-635f-4267-9d53-3ddb24a9acc1 to router cc5fd2f5-59d6-484d-a759-819917a5610c.

### IP flotante

Las IP flotantes se definen como las direcciones IPs asociadas a la red
exterior, que en un momento determinado se relacionan con una instancia y con su
dirección IP fija, pero que pueden relacionarse con otra instancia cuando se
desee. Las IPs flotantes permiten que las instancias sean accesibles desde el
exterior.

Solicitamos la asignación de una IP flotante al proyecto:

    $ neutron floatingip-create ext_net
	Created a new floatingip:
	+---------------------+--------------------------------------+
	| Field               | Value                                |
	+---------------------+--------------------------------------+
	| fixed_ip_address    |                                      |
	| floating_ip_address | 192.168.0.16                         |
	| floating_network_id | 3292768d-b916-45bc-8ce5-cbab121d6d01 |
	| id                  | bd31743b-79df-41a4-a40c-62756e868a88 |
	| port_id             |                                      |
	| router_id           |                                      |
	| tenant_id           | 4beb810ce40f49659e0bca732e4f1a3c     |
	+---------------------+--------------------------------------+

### Puertos

Cada usuario puede ver los puertos de sus redes privadas y el administrador
puede ver los de todas las redes. Si después de lanzar una instancia listamos
los puertos de nuestra red privada, vemos tres puertos:

    $ neutron port-list
	+--------------------------------------+------+-------------------+---------------------------------------------------------------------------------+
	| id                                   | name | mac_address       | fixed_ips
	|
	+--------------------------------------+------+-------------------+---------------------------------------------------------------------------------+
	| db992449-635f-4267-9d53-3ddb24a9acc1 |      | fa:16:3e:af:11:fe |
	{"subnet_id": "d4bb2d0e-2af7-44fe-9729-4e3b95766e28", "ip_address": "10.0.0.1"}
	|
	| 79712efb-320d-407d-82d7-eafd7eb51abe |      | fa:16:3e:dd:ff:a4 |
	{"subnet_id": "d4bb2d0e-2af7-44fe-9729-4e3b95766e28", "ip_address": "10.0.0.3"}
	|
	| b079c24c-8386-47db-a730-35b3a99419e8 |      | fa:16:3e:79:3b:ac |
	{"subnet_id": "d4bb2d0e-2af7-44fe-9729-4e3b95766e28", "ip_address": "10.0.0.2"}
	|
	+--------------------------------------+------+-------------------+---------------------------------------------------------------------------------+

Que se corresponden con la interfaz interna del router para el que se reserva la
primera dirección IP (10.0.0.1), el servidor DHCP (10.0.0.3) y la instancia
(10.0.0.2).

Lo último que queda es asociar la IP flotante al puerto de la instancia mediante:

    $ neutron floatingip-associate bd31743b-79df-41a4-a40c-62756e868a88 b079c24c-8386-47db-a730-35b3a99419e8
	Associated floatingip bd31743b-79df-41a4-a40c-62756e868a88

Y añadir las reglas de seguridad que sean necesarias, por ejemplo permitir
acceso por ssh a la red 192.168.0.0/24:

    $ neutron security-group-rule-create --direction ingress --protocol tcp --port-range-min 22 --port-range-max 22 --remote-ip-prefix 192.168.0.0/24 default
	
### Referencias

* <a
  href="http://docs.openstack.org/user-guide/content/neutron_client_sample_commands.html">OpenStack
  End User Guide</a>

