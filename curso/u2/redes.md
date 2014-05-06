---
layout: blog
title: OpenStack Horizon
menu:
  - Tema 2
---
## Configuración básica de redes

De acuerdo a la configuración utilizada ([Per tenant routers with private
networks](http://docs.openstack.org/grizzly/openstack-network/admin/content/app_demo_routers_with_private_networks.html)),
cada usuario podrá crear las redes y routers que desee y conectarlos a la red
externa común.

### Definiciones previas:

* **Red** Dominio aislado de capa 2. Sería el equivalente a una VLAN.
* **Subred** Un bloque de direcciones IPv4 o IPv6 que se asignan a las máquinas
  virtuales que se conectan a ella.
* **Puerto** Es un puerto virtual de un switch

Partimos de una topología de red en la que existe una red externa previamente
creada por el administrador.

### Crear una red privada

Para crear una nueva red privada vamos al menú "Redes" y pulsamos sobre el botón
"Crear red", nos aparecerá una ventana en la que debemos rellenar los siguientes
campos:

* Pestaña "Red"
    * **Nombre de la red**
* Pestaña "Subred"
    * **Nombre de la subred** (Opcional)
    * **Direcciones de red** Dirección de red en formato CIDR
    * **Versión IP** (IPv4 o IPv6)
    * **IP de la puerta de enlace** Se recomienda dejar en blanco
* Pestaña "Detalle de subred"
    * **Habilitar DHCP** Si queremos o no que se cree un dispositivo DHCP
    encargado de asignar IPs fijas a las instancias en la red
    * **Pools de asignación** Por si no queremos que se asigne solo un rango de
    IPs dentro de las direcciones de red
    * **Servidores DNS** Para especificar los servidores DNS
    * **Rutas de host** Por si es necesario añadir rutas adicionales a los
    equipos de esta red

Supongamos que elegimos una red de nombre "privada" y con la dirección
10.0.0.0/24, dejando el resto de parámetros con los valores asumidos.

Tras especificar los parámetros necesarios se creará la red para este
proyecto. Ya sería posible iniciar instancias y asociarlas a esta red, pero de
momento esas instancias no serían accesibles desde fuera porque están en una red
privada aislada.

### Crear un router

Para crear un nuevo router vamos al menú "Routers" y pulsamos el botón "Crear
router", en la ventana que aparece simplemente tenemos que ponerle un nombre
distintivo al router.

Un router puede estar conectado a tantas redes privadas como queramos, pero como
es natural solo se puede definir una puerta de enlace.

Tras crear el router nos aparecerá el router creado y veremos el botón "Definir
puerta de enlace", procedemos a pulsar el mismo y seleccionar la red externa que
previamente ha creado el administrador de la red. Tras definir la puerta de
enlace se asignará una dirección IP flotante al router, aunque no aparecerá 

### Conectar la red privada con la red externa 

Ahora mismo tenemos un router con una interfaz de red conectada a la red externa
y una red privada, obviamente si queremos conectar la red privada con la red
externa, debemos hacerlo a través del router, por lo que seleccionamos el router
anteriormente creado y pulsamos en el botón "Añadir Interfaz", y podremos
seleccionar la subred a la que nos queremos conectar.

Una vez conectado el router a la red privada, podemos ir al detalle de la red
privada y comprobaremos que aparece un puerto asociado al dispositivo
"network:router_interface" con dirección IP fija la primera disponible, en este
caso la 10.0.0.1.

Cuando lancemos instancias, aparecerán aquí tantos puertos de tipo "compute"
como instancias haya conectadas a la red y un puerto de tipo "network:dhcp" para
el dispositivo DHCP, que no se crea hasta que se lanza la primera instancia
dentro de la red.

### Comprobación

Accedemos a "Topología de red" y comprobamos que todos los elementos están
correctamente definidos.

### Referencias

* [OpenStack -  Create and manage networks](http://docs.openstack.org/user-guide/content/dashboard_create_networks.html)