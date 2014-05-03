---
layout: blog
title: Subir una imagen a OpenStack
menu:
  - Tema 2
---
# Subir una imagen

Como es lógico, solo es posible instanciar las imágenes que estén disponibles en
nuestra instalación de OpenStack. En una instalación limpia no se dispone de
ninguna imagen, por lo que uno de los primeros pasos a dar es agregar un
conjunto mínimo de imágenes con los sistemas operativos que vayamos a utilizar.

## Imagen de CentOS 6.5

En el propio repositorio de RDO existe una imagen mínima de CentOS 6.5 en
formato QCOW2 que podemos descargar para utilizar directamente en OpenStack:

* [RDO - Guest
  images](http://repos.fedorapeople.org/repos/openstack/guest-images/)

## Agregar la imagen a nuestro cloud

En el menú "Imágenes e instantáneas" pulsamos sobre el botón "Crear imagen" y
rellenamos apropiadamente los siguientes campos:

* Nombre
* Descripción (opcional)
* Origen de la imagen. En este caso podemos optar por poner la URL directamente,
  para lo que deberíamos elegir la opción "Ubicación de la imagen" o subir el
  fichero desde nuestro equipo, opción "Fichero de Imagen"
* De acuerdo a la opción anterior rellenamos el campo "Ubicación de la imagen" o
  seleccionamos la imagen desde nuestro equipo.
* Seleccionamos el formato.
* Disco mínimo. Para que no se permita iniciar la instancia con un disco raíz
  inferior al de la imagen
* Público. Seleccionamos esta opción si queremos permitir que otros usuarios la
  utilicen
* Protegida. Para evitar que se borre.

### Referencias

* [OpenStack -  Upload and manage images](http://docs.openstack.org/user-guide/content/dashboard_manage_images.html)
* [RDO - Image resources](http://openstack.redhat.com/Image_resources)
* [OpenStack Doc - Get Images](http://docs.openstack.org/image-guide/content/ch_obtaining_images.html)

