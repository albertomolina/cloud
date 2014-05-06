---
layout: blog
title: Manejo de volúmenes
menu:
  - Tema 2
---
# Manejo de volúmenes

Los volúmenes son dispositivos de bloques que se pueden asociar o desasociar de
las instancias y son independientes de la vida de éstas, por lo que se pueden
utilizar para almacenamiento "permanente", es decir, para almacenar datos
después de terminar la instancia.

## Crear volúmenes

Simplemente pulsando en el botón "Crear volúmen" dentro de "Volúmenes". Veremos
una ventana en la que podremos crear un nuevo volumen definiendo un nombre y un
tamaño (en GiB).

## Asociar volumen

Una vez creado el volumen, podremos asociarlo a cualquiera de las instancias
activas del proyecto.

Si accedemos a la instancia podemos comprobar que existe un nuevo dispositivo en
el sistema:

	# fdisk -l 
	Disk /dev/vdb: 1073 MB, 1073741824 bytes
	16 heads, 63 sectors/track, 2080 cylinders, total 2097152 sectors
	Units = sectors of 1 * 512 = 512 bytes
	Sector size (logical/physical): 512 bytes / 512 bytes
	I/O size (minimum/optimal): 512 bytes / 512 bytes
	Disk identifier: 0x00000000

	Disk /dev/vdb doesn't contain a valid partition table

Podemos particionarlo, formatearlo y montarlo si queremos.

## Desasociar volumen

Cuando no sea necesario tener conectado el volumen a la instancia, podemos
desasociarlo (previamente habría que desmontarlo de la instancia) y utilizarlo
en la misma instancia o en cualquier otra cuando sea preciso.

## Limitaciones

No es posible asociar el mismo volumen a más de una instancia simultáneamente
para prevenir corrupción del sistema de ficheros del volumen, a pesar de que el 
protocolo de redes de almacenamiento subyacente lo permite y la utilización de
un sistema de ficheros de tipo cluster evitaría la corrupción de datos.
