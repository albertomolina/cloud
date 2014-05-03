---
layout: blog
title: Uso básico de horizon.
menu:
  - Tema 2
---
# Uso de OpenStack Horizon

##Objetivos

* Conocer los conceptos fundamentales para poder utilizar OpenStack.
* Utilizar OpenStack desde el panel web (Horizon)

## Conceptos Previos

* [Conceptos previos](conceptos_previos.html)

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

## Uso básico de OpenStack Horizon

* [Subir una imagen](subir_imagen)
* [Lanzar una instancia](lanzamiento)

### Enlaces relacionados

* [OpenStack End User Guide](http://docs.openstack.org/user-guide/content/index.html)
* [How to launch an instance on OpenStack(I): Horizon](http://albertomolina.wordpress.com/2013/11/20/how-to-launch-an-instance-on-openstack-i-horizon/)