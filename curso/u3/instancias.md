---
layout: blog
tittle: Utilización desde línea de comandos
menu:
  - Tema 3
---

## Instancias

Utilizamos el cliente de línea de comandos nova.

### Pasos previos

Antes de lanzar una instancia vamos a crear un par de claves RSA para poder
acceder posteriormente por ssh (recordamos que habitualmente las imágenes que se
utilizan en OpenStack no tienen definidas contraseñas para los usuarios), hay
dos opciones: Subir la clave pública de una clave privada generada previamente o
generar directamente con el cliente nova un nuevo par de claves y almacenar la
clave privada.

#### Crear un nuevo par de claves

Ejecutamos la siguiente instruncción:

    $ nova keypair-add nueva-clave > ~/.ssh/nueva-clave.pem
	
Para que podamos utilizar posteriormente esta clave privada, es necesario
modificar los permisos, para que ningún otro usuario pueda leerla:

    $ chmod 400 ~/.ssh/nueva-clave.pem
	
#### Importar una clave

Si ya disponemos de una clave privada RSA en el equipo (por ejemplo
~/.ssh/miclave.pem) vamos a obtener la clave pública correspondiente:

    $ cd ~/.ssh
	$ ssh-keygen -y -f miclave.pem > miclave.pub

Y ahora subimos la clave pública a nuestro proyecto en OpenStack:

    $ nova keypair-add --pub_key ~/.ssh/miclave.pub miclave

### Recabar los parámetros necesarios

Para poder lanzar una instancia es necesario conocer previamente varios
parámetros como las imágenes disponibles, pares de clave, etc.

Listamos los sabores disponibles:

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

Listamos las imágenes disponibles:

    $ nova image-list
	+--------------------------------------+--------------+--------+--------+
	| ID                                   | Name         | Status | Server |
	+--------------------------------------+--------------+--------+--------+
	| f43a75f1-da13-4223-ac1f-cb0dac55399c | CentOS 6.5   | ACTIVE |        |
	| 4ddb27ab-b3cc-4a65-ac52-f4ce7894e4ed | Cirros 0.3.2 | ACTIVE |        |
	+--------------------------------------+--------------+--------+--------+

Listamos las redes disponibles:

    $ nova net-list
	+--------------------------------------+----------+------+
	| ID                                   | Label    | CIDR |
	+--------------------------------------+----------+------+
	| 0d43e5fe-99b9-46a8-b118-58136aa805b4 | privada  | -    |
	| 7481948f-3f80-4585-a91c-a85a9aba49b7 | ext_net  | -    |
	+--------------------------------------+----------+------+


### Lanzar una instancia

Una vez que conocemos todos los parámetros necesarios, lanzamos una instancia
con la instrucción:

    $ nova boot --image 4ddb27ab-b3cc-4a65-ac52-f4ce7894e4ed \
	--flavor 1 \
	--key_name miclave \
	--nic net-id=0d43e5fe-99b9-46a8-b118-58136aa805b4 \
	test


### Asociar una IP flotante

Una vez que la instancia se ha creado en la red privada, solicitamos una
dirección IP flotante y se la asociamos a la instancia:

    $ nova floating-ip-create ext_net
	+--------------+-----------+----------+---------+
	| Ip           | Server Id | Fixed Ip | Pool    |
	+--------------+-----------+----------+---------+
	| 192.168.0.16 |           | -        | ext_net |
	+--------------+-----------+----------+---------+
	$ nova floating-ip-associate test 192.168.0.16

### Acceder a la instancia

Para poder acceder por ssh a la instancia es necesario que esté permitido el
acceso al puerto 22/tcp, por lo que añadimos una regla al grupo de seguridad
*default*:

    $ nova secgroup-add-rule default tcp 22 22 0.0.0.0/0
	+-------------+-----------+---------+-----------+--------------+
	| IP Protocol | From Port | To Port | IP Range  | Source Group |
	-------------+-----------+---------+-----------+--------------+
	| tcp         | 22        | 22      | 0.0.0.0/0 |              |
	+-------------+-----------+---------+-----------+--------------+

Si esta misma regla de seguridad la definiésemos con neutron sería:

    $ neutron security-group-rule-create --direction ingress \
	--protocol tcp --port-range-min 22 --port-range-max 22 \
	--remote-ip-prefix 192.168.0.0/24 default
	Created a new security_group_rule:
	+-------------------+--------------------------------------+
	| Field             | Value                                |
	+-------------------+--------------------------------------+
	| direction         | ingress                              |
	| ethertype         | IPv4                                 |
	| id                | 185580b9-45d8-4e29-9019-22400873efd1 |
	| port_range_max    | 22                                   |
	| port_range_min    | 22                                   |
	| protocol          | tcp                                  |
	| remote_group_id   |                                      |
	| remote_ip_prefix  | 192.168.0.0/24                       |
	| security_group_id | d1a8eb3a-3335-48ec-90d6-e6608e75aaef |
	| tenant_id         | 0912c80bef254d7b9352632793cf75b9     |
	+-------------------+--------------------------------------+

Una vez permitido el tráfico ssh, podemos acceder utilizando la clave privada
RSA asociada a la clave pública que se ha inyectado en la cuenta del usuario
cirros de la instancia:

    $ ssh cirros@192.168.0.16
	The authenticity of host '192.168.0.16 (192.168.0.16)' can't be established.
	RSA key fingerprint is db:2d:84:f7:56:49:69:3f:50:f1:96:c1:75:1c:cd:b9.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added '192.168.0.16' (RSA) to the list of known hosts.
	$ 
	
### Acciones sobre las instancias

En primer lugar es importante entender que una instancia de cloud no suele tener el mismo comportamiento ni tratamiento que un servidor físico o virtual que normalmente están funcionando en régimen 24x7 y se hace siempre todo lo posible porque sea así. De forma muy simpática a alguien se le ocurrió hacer la analogía "Pets vs Cattle" que aparece en la siguiente imagen:

<img src="http://www.it20.info/misc/pictures/vCloudOpenStackPetsandCattle1.JPG" alt="pets_vs_cattle"/>

Fuente: <a href="http://www.slideshare.net/gmccance/cern-data-centre-evolution">Gavin McCance - CERN</a>

Es por tanto es bastantes ocasiones necesario realizar alguna de las siguientes acciones sobre las instancias:

* Pausar una instancia activa: 

    nova pause <server>

* Despausar una instancia pausada: 

    nova unpause <server>

* Suspender una instancia activa: <pre>nova suspend</pre>
* Reanudar una instancia suspendida: <pre>nova resume</pre>
* Parar una instancia activa: <pre>nova stop</pre>
* Arrancar una instancia parada: <pre>nova start</pre>
* Reiniciar una instancia activa: <pre>nova [--hard] reboot</pre>
* Terminar una instancia (irreversible): <pre>nova delete</pre>

### Redimensionar la instancia

Para poder utilizar esta funcionalidad es necesario realizar algunas configuraciones en los nodos de computación que normalmente no se realizan en instalaciones automáticas como la utilizada en este curso. Básicamente hay que garantizar que los usuarios "nova" de los nodos de computación pueden acceder de un nodo a otro con ssh sin contraseña utilizando pares de claves pública/privada sin frase de paso.

Una vez configurado el escenario se puede redimensionar la instancia con la instrucción <pre>nova resize</pre>

### Instantáneas de instancias

Es posible en cualquier momento tomar una instantánea de la instancia, que se almacenará como una nueva imagen que luego pueda instanciarse:

    $ nova image-create --poll test test-snap`date +%s`
	Server snapshotting... 100% complete
	Finished
	
	$ nova image-list
	+--------------------------------------+----------------------+--------+--------------------------------------+
	| ID                                   | Name                 | Status | Server                               |
	+--------------------------------------+----------------------+--------+--------------------------------------+
	| f43a75f1-da13-4223-ac1f-cb0dac55399c | CentOS 6.5           | ACTIVE |                                      |
	| 4ddb27ab-b3cc-4a65-ac52-f4ce7894e4ed | Cirros 0.3.2         | ACTIVE |                                      |
	| 40d7cedb-27d1-44c6-9777-7ea252094865 | test-snap1400773026  | ACTIVE | e13793ab-2a2e-4279-8bb9-4364eaf0beaa |
	+--------------------------------------+----------------------+--------+--------------------------------------+
	
### Referencias

* <a
  href="http://docs.openstack.org/user-guide/content/cli_launch_instances.html">OpenStack
  End User Guide</a>

