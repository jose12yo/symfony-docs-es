Convenciones
============

El documento :doc:`estándares <standards>` describe las normas de codificación de los proyectos Symfony2 y los paquetes internos de terceros. Este documento describe los estándares de codificación y convenciones utilizadas en la plataforma básica para que sea más consistente y predecible. Los puedes seguir en tu propio código, pero no es necesario.

Nombres de método
-----------------

Cuando un objeto tiene una relación "principal" con muchas "cosas" relacionadas (objetos, parámetros, ...), los nombres de los métodos están normalizados:

  * ``get()``
  * ``set()``
  * ``has()``
  * ``all()``
  * ``replace()``
  * ``remove()``
  * ``clear()``
  * ``isEmpty()``
  * ``add()``
  * ``register()``
  * ``count()``
  * ``keys()``

El uso de estos métodos sólo se permite cuando es evidente que existe una relación principal:

* un ``CookieJar`` tiene muchos objetos ``Cookie``;

* un ``Container`` de servicio tiene muchos servicios y muchos parámetros (debido a que servicios
  es la relación principal, utilizamos la convención de nomenclatura para esta relación);

* una Consola ``Input`` tiene muchos argumentos y muchas opciones. No hay una relación "principal", por lo tanto no aplica la convención de nomenclatura.

Para muchas relaciones cuando la convención no aplica, en su lugar se deben utilizar los siguientes métodos (donde ``XXX`` es el nombre de aquello relacionado):

+----------------+-------------------+
| Relación       | Otras relaciones  |
| principal      |                   |
+================+===================+
| ``get()``      | ``getXXX()``      |
+----------------+-------------------+
| ``set()``      | ``setXXX()``      |
+----------------+-------------------+
| n/a            | ``replaceXXX()``  |
+----------------+-------------------+
| ``has()``      | ``hasXXX()``      |
+----------------+-------------------+
| ``all()``      | ``getXXXs()``     |
+----------------+-------------------+
| ``replace()``  | ``setXXXs()``     |
+----------------+-------------------+
| ``remove()``   | ``removeXXX()``   |
+----------------+-------------------+
| ``clear()``    | ``clearXXX()``    |
+----------------+-------------------+
| ``isEmpty()``  | ``isEmptyXXX()``  |
+----------------+-------------------+
| ``add()``      | ``addXXX()``      |
+----------------+-------------------+
| ``register()`` | ``registerXXX()`` |
+----------------+-------------------+
| ``count()``    | ``countXXX()``    |
+----------------+-------------------+
| ``keys()``     | n/a               |
+----------------+-------------------+

.. note::

    Si bien "setXXX" y "replaceXXX" son muy similares, hay una notable diferencia: "setXXX" puede sustituir o agregar nuevos elementos a la relación. 
    Por otra parte "replaceXXX" específicamente tiene prohibido añadir nuevos elementos, pero mayoritariamente lanza una excepción en estos casos.
