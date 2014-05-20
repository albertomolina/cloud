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
| ID | Name      | Memory_MB | Disk | Ephemeral | Swap | VCPUs | RXTX_Factor |
Is_Public |
+----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
| 1  | m1.tiny   | 512       | 1    | 0         |      | 1     | 1.0         |
True      |
| 2  | m1.small  | 2048      | 20   | 0         |      | 1     | 1.0         |
True      |
| 3  | m1.medium | 4096      | 40   | 0         |      | 2     | 1.0         |
True      |
| 4  | m1.large  | 8192      | 80   | 0         |      | 4     | 1.0         |
True      |
| 5  | m1.xlarge | 16384     | 160  | 0         |      | 8     | 1.0         |
True      |
+----+-----------+-----------+------+-----------+------+-------+-------------+-----------+

Listamos las imágenes disponibles:

    $ nova image-list
	+--------------------------------------+---------------------------------+--------+--------+
	| ID                                   | Name                            |
	Status | Server |
	+--------------------------------------+---------------------------------+--------+--------+
	Completar !!!
	
## Lanzar una instancia

Una vez que conocemos todos los parámetros necesarios, lanzamos una instancia
con la instrucción:

        $ nova boot --image 4ddb27ab-b3cc-4a65-ac52-f4ce7894e4ed \
		--flavor 1 \
		--key_name miclave \
		--nic net-id=f84a328e-407e-4ca6-87bb-ec148853c585 \
		test
										

### Referencias

* <a
  href="http://docs.openstack.org/user-guide/content/cli_launch_instances.html">OpenStack
  End User Guide</a>

