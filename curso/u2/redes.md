---
layout: blog
title: Configuración básica de redes
menu:
  - Tema 2
---
# Configuración básica de redes

De acuerdo a la configuración utilizada ([Per tenant routers with private
networks](http://docs.openstack.org/grizzly/openstack-network/admin/content/app_demo_routers_with_private_networks.html)),
cada usuario podrá crear las redes y routers que desee y conectarlos a la red
externa común.

## Definiciones previas:

* **Red** Dominio aislado de capa 2. Sería el equivalente a una VLAN.
* **Subred** Un bloque de direcciones IPv4 o IPv6 que se asignan a las máquinas
  virtuales que se conectan a ella.
* **Puerto** Es un puerto virtual de un switch

## Crear una red privada y conectarla a una red externa

Para crear una nueva red privada 
