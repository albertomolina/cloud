---
layout: blog
tittle: Utilización desde línea de comandos
menu:
  - Tema 3
---

## Volúmenes

Utilizamos los clientes de línea de comandos nova y cinder.

## Crear y asociar un volumen

Creamos un volumen de 1 GiB y lo asociamos a una de las instancias anteriores:

    $ cinder create 1
	+---------------------+--------------------------------------+
	|       Property      |                Value                 |
	+---------------------+--------------------------------------+
	|     attachments     |                  []                  |
	|  availability_zone  |                 nova                 |
	|       bootable      |                false                 |
	|      created_at     |      2014-05-22T18:35:38.256025      |
	| display_description |                 None                 |
	|     display_name    |                 None                 |
	|          id         | 70ba58bf-1201-4b27-8de3-a1f2f1bf49ed |
	|       metadata      |                  {}                  |
	|         size        |                  1                   |
	|     snapshot_id     |                 None                 |
	|     source_volid    |                 None                 |
	|        status       |               creating               |
	|     volume_type     |                 None                 |
	+---------------------+--------------------------------------+
	
Que sería equivalente a hacerlo con nova:

    $ nova volume-create 1
	+---------------------+--------------------------------------+
	| Property            | Value                                |
	+---------------------+--------------------------------------+
	| attachments         | []                                   |
	| availability_zone   | nova                                 |
	| bootable            | false                                |
	| created_at          | 2014-05-22T18:37:36.569571           |
	| display_description | -                                    |
	| display_name        | -                                    |
	| id                  | 4d624e6e-b866-44aa-b26b-ce12b7dff927 |
	| metadata            | {}                                   |
	| size                | 1                                    |
	| snapshot_id         | -                                    |
	| source_volid        | -                                    |
	| status              | creating                             |
	| volume_type         | None                                 |
	+---------------------+--------------------------------------+
	
Lo asociamos a la instancia test:

    $ nova volume-attach test 4d624e6e-b866-44aa-b26b-ce12b7dff927
	+----------+--------------------------------------+
	| Property | Value                                |
	+----------+--------------------------------------+
	| device   | /dev/vdb                             |
	| id       | 4d624e6e-b866-44aa-b26b-ce12b7dff927 |
	| serverId | e13793ab-2a2e-4279-8bb9-4364eaf0beaa |
	| volumeId | 4d624e6e-b866-44aa-b26b-ce12b7dff927 |
	+----------+--------------------------------------+
	
Si entramos en la instancia test, podemos comprobar que aparece un nuevo disco asociado al dispositivo /dev/vdb:

    # fdisk -l
    ...	   
	Disk /dev/vdb: 1073 MB, 1073741824 bytes
	16 heads, 63 sectors/track, 2080 cylinders, total 2097152 sectors
	Units = sectors of 1 * 512 = 512 bytes
	Sector size (logical/physical): 512 bytes / 512 bytes
	I/O size (minimum/optimal): 512 bytes / 512 bytes
	Disk identifier: 0x00000000
	...   
	Disk /dev/vdb doesn't contain a valid partition table

### Crear un volumen basado en una imagen

Además de utilizar los volúmenes para almacenar datos de aplicaciones o de usuarios, se puede pueden utilizar para almacenar el sistema operativo de la instancia. Vemos las imágenes disponibles y creamos un nuevo volumen basado en una de ellas (lógicamente el tamaño del nuevo volumen tiene que ser igual o mayor que la imagen en la que se basa):

    $ nova image-list
	+--------------------------------------+--------------+--------+--------+
	| ID                                   | Name         | Status | Server |
	+--------------------------------------+--------------+--------+--------+
	| f43a75f1-da13-4223-ac1f-cb0dac55399c | CentOS 6.5   | ACTIVE |        |
	| 4ddb27ab-b3cc-4a65-ac52-f4ce7894e4ed | Cirros 0.3.2 | ACTIVE |        |
	+--------------------------------------+--------------+--------+--------+
	
	$ cinder create --image-id 4ddb27ab-b3cc-4a65-ac52-f4ce7894e4ed --display-name "CirrOS 0.3.2" 1
	+---------------------+--------------------------------------+
	|       Property      |                Value                 |
	+---------------------+--------------------------------------+
	|     attachments     |                  []                  |
	|  availability_zone  |                 nova                 |
	|       bootable      |                false                 |
	|      created_at     |      2014-05-22T18:56:24.480790      |
	| display_description |                 None                 |
	|     display_name    |             CirrOS 0.3.2             |
	|          id         | 2869f3c1-5566-46ca-9ab2-8bec5d39497e |
	|       image_id      | 4ddb27ab-b3cc-4a65-ac52-f4ce7894e4ed |
	|       metadata      |                  {}                  |
	|         size        |                  1                   |
	|     snapshot_id     |                 None                 |
	|     source_volid    |                 None                 |
	|        status       |               creating               |
	|     volume_type     |                 None                 |
	+---------------------+--------------------------------------+

Si miramos las propiedades de este último volumen, podemos comprobar que se ha etiquetado como *bootable*, por lo que podemos iniciar una nueva instancia que lo utilice como unidad de almacenamiento primaria con el sistema:

    $ nova boot --flavor 1 --key-name clave-openstack --nic net-id=0d43e5fe-99b9-46a8-b118-58136aa805b4 --block-device-mapping vda=2869f3c1-5566-46ca-9ab2-8bec5d39497e::: test4
	+--------------------------------------+--------------------------------------------------+
	| Property                             | Value                                            |
	+--------------------------------------+--------------------------------------------------+
	| OS-DCF:diskConfig                    | MANUAL                                           |
	| OS-EXT-AZ:availability_zone          | nova                                             |
	| OS-EXT-STS:power_state               | 0                                                |
	| OS-EXT-STS:task_state                | scheduling                                       |
	| OS-EXT-STS:vm_state                  | building                                         |
	| OS-SRV-USG:launched_at               | -                                                |
	| OS-SRV-USG:terminated_at             | -                                                |
	| accessIPv4                           |                                                  |
	| accessIPv6                           |                                                  |
	| adminPass                            | 6ukwwaBAVHB3                                     |
	| config_drive                         |                                                  |
	| created                              | 2014-05-22T19:10:51Z                             |
	| flavor                               | m1.tiny (1)                                      |
	| hostId                               |                                                  |
	| id                                   | 789f01b0-ac20-40a2-a916-7732f70d00be             |
	| image                                | Attempt to boot from volume - no image supplied  |
	| key_name                             | clave-openstack                                  |
	| metadata                             | {}                                               |
	| name                                 | test4                                            |
	| os-extended-volumes:volumes_attached | [{"id": "2869f3c1-5566-46ca-9ab2-8bec5d39497e"}] |
	| progress                             | 0                                                |
	| security_groups                      | default                                          |
	| status                               | BUILD                                            |
	| tenant_id                            | 0912c80bef254d7b9352632793cf75b9                 |
	| updated                              | 2014-05-22T19:10:51Z                             |
	| user_id                              | 0f94f0bbcdee4715ba2750502bb0d63f                 |
	+--------------------------------------+--------------------------------------------------+

El sistema ahora es independiente de la vida de la instancia y tenemos un mecanismo muy sencillo de escalar verticalmente la instancia, ya que podemos asociar el volumen al sabor que más nos convenga en cada momento y redimensionar el volumen a medida que nos haga falta.

Es posible también hacer esto mismo directamente desde nova en un solo paso con la siguiente instrucción (utilizamos uno de los volúmenes vacíos previamente creados):

    $ nova boot --flavor 1 --image 4ddb27ab-b3cc-4a65-ac52-f4ce7894e4ed \
	--block-device source=volume,id=70ba58bf-1201-4b27-8de3-a1f2f1bf49ed,\
	dest=volume,shutdown=preserve --nic net-id=0d43e5fe-99b9-46a8-b118-58136aa805b4 test5
	+--------------------------------------+-----------------------------------------------------+
	| Property                             | Value                                               |
	+--------------------------------------+-----------------------------------------------------+
	| OS-DCF:diskConfig                    | MANUAL                                              |
	| OS-EXT-AZ:availability_zone          | nova                                                |
	| OS-EXT-STS:power_state               | 0                                                   |
	| OS-EXT-STS:task_state                | scheduling                                          |
	| OS-EXT-STS:vm_state                  | building                                            |
	| OS-SRV-USG:launched_at               | -                                                   |
	| OS-SRV-USG:terminated_at             | -                                                   |
	| accessIPv4                           |                                                     |
	| accessIPv6                           |                                                     |
	| adminPass                            | 6JeDCcLCRnuy                                        |
	| config_drive                         |                                                     |
	| created                              | 2014-05-22T19:22:58Z                                |
	| flavor                               | m1.tiny (1)                                         |
	| hostId                               |                                                     |
	| id                                   | 53fc4ec0-9ec1-4989-891d-961df26177e2                |
	| image                                | Cirros 0.3.2 (4ddb27ab-b3cc-4a65-ac52-f4ce7894e4ed) |
	| key_name                             | -                                                   |
	| metadata                             | {}                                                  |
	| name                                 | test5                                               |
	| os-extended-volumes:volumes_attached | [{"id": "70ba58bf-1201-4b27-8de3-a1f2f1bf49ed"}]    |
	| progress                             | 0                                                   |
	| security_groups                      | default                                             |
	| status                               | BUILD                                               |
	| tenant_id                            | 0912c80bef254d7b9352632793cf75b9                    |
	| updated                              | 2014-05-22T19:22:59Z                                |
	| user_id                              | 0f94f0bbcdee4715ba2750502bb0d63f                    |
	+--------------------------------------+-----------------------------------------------------+

O incluso sería posible también crear un nuevo volumen al vuelo sin necesidad de utilizar un volumen previamente creado.

### Instantáneas de volúmenes

Es posible hacer instantáneas también a los volúmenes:

    $ cinder list
	+--------------------------------------+-----------+--------------+------+-------------+----------+-------------+
	|                  ID                  |   Status  | Display Name | Size | Volume Type | Bootable | Attached to |
	+--------------------------------------+-----------+--------------+------+-------------+----------+-------------+
	| 2869f3c1-5566-46ca-9ab2-8bec5d39497e | available | CirrOS 0.3.2 |  1   |     None    |   true   |             |
	| 4d624e6e-b866-44aa-b26b-ce12b7dff927 | available |     None     |  1   |     None    |  false   |             |
	| 70ba58bf-1201-4b27-8de3-a1f2f1bf49ed | available |     None     |  1   |     None    |  false   |             |
	+--------------------------------------+-----------+--------------+------+-------------+----------+-------------+
	
	$ cinder snapshot-create --display-name snap-`date +%s` 70ba58bf-1201-4b27-8de3-a1f2f1bf49ed
	+---------------------+--------------------------------------+
	|       Property      |                Value                 |
	+---------------------+--------------------------------------+
	|      created_at     |      2014-05-22T19:28:54.774496      |
	| display_description |                 None                 |
	|     display_name    |           snap-1400787127            |
	|          id         | 3546a39f-fc32-42ea-9c51-2d6959f9798a |
	|       metadata      |                  {}                  |
	|         size        |                  1                   |
	|        status       |               creating               |
	|      volume_id      | 70ba58bf-1201-4b27-8de3-a1f2f1bf49ed |
	+---------------------+--------------------------------------+
	
Las instantáneas de volumen no se pueden utilizar directamente, se utilizan para crear nuevos volúmenes a partir de ellas:

    $ cinder create --snapshot-id 3546a39f-fc32-42ea-9c51-2d6959f9798a 1
	+---------------------+--------------------------------------+
	|       Property      |                Value                 |
	+---------------------+--------------------------------------+
	|     attachments     |                  []                  |
	|  availability_zone  |                 nova                 |
	|       bootable      |                false                 |
	|      created_at     |      2014-05-22T19:30:23.997947      |
	| display_description |                 None                 |
	|     display_name    |                 None                 |
	|          id         | f9447fd0-2bad-4b7d-b703-6fa58b838448 |
	|       metadata      |                  {}                  |
	|         size        |                  1                   |
	|     snapshot_id     | 3546a39f-fc32-42ea-9c51-2d6959f9798a |
	|     source_volid    |                 None                 |
	|        status       |               creating               |
	|     volume_type     |                 None                 |
	+---------------------+--------------------------------------+

### Referencias

* <a
  href="http://docs.openstack.org/user-guide/content/cli_manage_volumes.html">OpenStack
  End User Guide</a>

