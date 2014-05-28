---
layout: blog
tittle: 
menu:
  - Tema 5
---

Este apartado final del curso trata simplemente de mostrar el abanico de
posibilidades que se abre en un proyecto tan vivo y con un desarrollo tan fuerte
como OpenStack y a modo de resumen mostrar otras muchas opciones a considerar.

## Otros componentes de OpenStack

En este curso nos hemos centrado en los componentes del *core* de OpenStack:
Keystone, Glance, Neutron, Cinder, Nova Swift y Horizon, aunque OpenStack va
incorporando paulatinamente más componentes que van complementando a los 
anteriores:

* <a href="https://wiki.openstack.org/wiki/Heat">Heat</a>: Orchestration
* <a href="https://wiki.openstack.org/wiki/Ceilometer">Ceilometer</a>: Metering
* <a href="https://wiki.openstack.org/wiki/Trove">Trove</a>: DBaaS
* <a href="https://wiki.openstack.org/wiki/Ironic">Ironic: Bare Metal
* <a https://wiki.openstack.org/wiki/Designate">Designate</a>: DNSaaS
* <a https://wiki.openstack.org/wiki/Sahara">Sahara</a>: Hadoop cluster
* <a https://wiki.openstack.org/wiki/TripleO">TripleO</a>: OpenStack on OpenStack

Esto en lo referente a aplicaciones oficiales, pero además se está creando un
ecosistema de aplicaciones que funcionan sobre OpenStack pero no forman parte
del proyecto oficial como <a
href="https://wiki.openstack.org/wiki/MagnetoDB">MagnetoDB</a>, <a
href="https://launchpad.net/kwapi">Kwapi</a> o <a
href="https://launchpad.net/stacksync">StackSync</a> entre otros o los muchos
proyectos que se ubican en <a
href="https://github.com/stackforge">StackForge</a>.

## Opciones de instalación de OpenStack

Hemos utilizado un entorno de pruebas con RDO sobre CentOS 6.5 creado de forma
automática con packstack y pequeñas modificaciones posteriores, pero hay muchos
aspectos adicionales a considerar en un despliegue real:

* Virtualización: Además de la opción de KVM vista en el curso, es posible
  utilizar otras opciones de virtualización en GNU/Linux como Xen, LXC o Docker
  o alternativamente utilizar como nodos de computación equipos con VMware o
  Hyper-V sobre Windows.
* Distribuciones GNU/Linux: Las que ofrecen mejor soporte actualmente son Ubuntu
  12.04 y CentOS/Red HAt 6.X. El soporte en Debian es mediante "backports" y
  está un poco reñido con la estabilidad de esta distribución.
* Opciones de almacenamiento:
    * Almacenamiento de Objetos: Utilizar o no Swift
	* Almacenamiento de bloques: Además de LVM se puede utilizar ZFS, GlusterFS,
      Sheepdog o Ceph RBD. Ceph además permite utlizar almacenamiento de objetos
      mediante APIs compatibles con Swift o Amazon S3
	* Almacenamiento centralizado: Pros y contras de utilizar almacenamiento
      centralizado para la ubicación de los discos de las imágenes o las
      imágenes base
	* Utilización de equipos específicos o servidores convencionales
* Virtualización de redes: Es recomendable utilizar el plugin ML2 (Modular Layer
  2) que permite utilizar simultáneamente diferentes tecnologías de red de capa
  2 en entorno heterogéneos. 
* Alta disponibilidad

## *Are Enterprises ready for the OpenStack Transformation?*

El cambio que puede propiciar OpenStack o en general las tecnologías de cloud
computing en el tratamiento de las infraestructuras es tan radical que es
importante esta pregunta y el análisis que hizo Nicolas Barcet en el último
*OpenStack Summit*:

* <a href="https://www.youtube.com/watch?v=pzWodEOshQ0">Are Enterprises Ready
for the OpenStack Transformation? - Nicolas Barcet (eNovance)</a>
