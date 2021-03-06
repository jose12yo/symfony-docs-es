.. index::
   single: Pruebas

Rendimiento
===========

Symfony2 es rápido, desde que lo sacas de la caja. Por supuesto, si realmente necesitas velocidad, hay muchas maneras en las cuales puedes hacer que Symfony sea aún más rápido. En este capítulo, podrás explorar muchas de las formas más comunes y potentes para hacer que tu aplicación Symfony sea aún más rápida.

.. index::
   single: Rendimiento; caché en código de bits

Utilizando una caché de código de bytes (p. ej. APC)
----------------------------------------------------

Una de las mejores (y más fáciles) cosas que debes hacer para mejorar tu rendimiento es utilizar una "caché de código de bytes". La idea de una caché de código de bytes es eliminar la necesidad de constantemente tener que volver a compilar el código fuente PHP. Hay disponible una serie de `cachés de código de bytes`_, algunas de las cuales son de código abierto. Probablemente, la caché de código de bytes más utilizada sea `APC`_

Usar una caché de código de bytes realmente no tiene ningún inconveniente, y Symfony2 se ha diseñado para llevar a cabo muy bien en este tipo de entorno.

Optimización adicional
~~~~~~~~~~~~~~~~~~~~~~

La caché de código de bytes, por lo general, comprueba los cambios de los archivos fuente. Esto asegura que si cambia un archivo fuente, el código de bytes se vuelve a compilar automáticamente.
Esto es muy conveniente, pero, obviamente, implica una sobrecarga.

Por esta razón, algunas cachés de código de bytes ofrecen una opción para desactivar esa comprobación.
Obviamente, cuando desactivas esta comprobación, será responsabilidad del administrador del servidor asegurarse de que la caché se borra cada vez que cambia un archivo fuente. De lo contrario, no se verán los cambios realizados.

Por ejemplo, para desactivar estos controles en APC, sólo tienes que añadir la opción ``apc.stat=0`` en tu archivo de configuración ``php.ini``.

.. index::
   single: Rendimiento; Autocargador

Usa un autocargador con caché (p. ej. ``ApcUniversalClassLoader``)
------------------------------------------------------------------

De manera predeterminada, la edición estándar de Symfony2 utiliza el ``UniversalClassLoader`` del archivo `autoloader.php`_. Este autocargador es fácil de usar, ya que automáticamente encontrará cualquier nueva clase que hayas colocado en los directorios registrados.

Desafortunadamente, esto tiene un costo, puesto que el cargador itera en todos los espacios de nombres configurados para encontrar un archivo, haciendo llamadas a ``file_exists`` hasta que finalmente encuentra el archivo que está buscando.

La solución más sencilla es que la caché memorice la ubicación de cada clase después de encontrarla por primera vez. Symfony incluye una clase cargadora - ``ApcUniversalClassLoader`` - que extiende al ``UniversalClassLoader`` y almacena la ubicación de las clases en APC.

Para utilizar este cargador de clases, sólo tienes que adaptar tu ``autoloader.php`` como sigue:

.. code-block:: php

    // app/autoload.php
    require __DIR__.'/../vendor/symfony/src/Symfony/Component/ClassLoader/UniversalClassLoader.php';
    require __DIR__.'/../vendor/symfony/src/Symfony/Component/ClassLoader/ApcUniversalClassLoader.php';

    use Symfony\Component\ClassLoader\ApcUniversalClassLoader;

    $loader = new ApcUniversalClassLoader('algún prefijo de caché único');
    // ...

.. note::

    Al utilizar el autocargador APC, si agregas nuevas clases, estas serán encontradas automáticamente y todo funcionará igual que antes (es decir, no hay razón para "limpiar" la caché). Sin embargo, si cambias la ubicación de un determinado espacio de nombres o prefijo, tendrás que limpiar tu caché APC. De lo contrario, el cargador aún buscará en la ubicación anterior todas las clases dentro de ese espacio de nombres.

.. index::
   single: Rendimiento; Archivos de arranque

Utilizando archivos de arranque
-------------------------------

Para garantizar una óptima flexibilidad y reutilización de código, las aplicaciones de Symfony2 aprovechan una variedad de clases y componentes de terceros. Pero cargar todas estas clases desde archivos separados en cada petición puede dar lugar a alguna sobrecarga. Para reducir esta sobrecarga, la edición estándar de Symfony2 proporciona un guión para generar lo que se conoce como `archivo de arranque`_, el cual contiene la definición de múltiples clases en un solo archivo. Al incluir este archivo (el cual contiene una copia de muchas de las clases del núcleo), Symfony ya no tiene que incluir algunos de los archivos de código fuente que contienen las clases. Esto reducirá bastante la E/S del disco.

Si estás utilizando la edición estándar de Symfony2, entonces probablemente ya estás utilizando el archivo de arranque. Para estar seguro, abre el controlador frontal (por lo general ``app.php``) y asegúrate de que existe una de las siguientes líneas y no tiene comentarios (exactamente la que necesitas depende de si estás utilizando la :doc:`capa de almacenamiento en caché HTTP </book/http_cache>` de Symfony)::

    require_once __DIR__.'/../app/bootstrap.php.cache';
    require_once __DIR__.'/../app/bootstrap_cache.php.cache';

Ten en cuenta que hay dos desventajas cuando utilizas un archivo de arranque:

* El archivo se tiene que regenerar cada vez que cambia alguna de las fuentes original (es decir, cuando actualizas el código fuente de Symfony2 o de las bibliotecas de proveedores);

* En la depuración, será necesario colocar puntos de interrupción dentro del archivo de arranque.

Si estás utilizando la Edición estándar de Symfony2, los archivos de arranque se reconstruyen automáticamente después de actualizar las bibliotecas de proveedores a través de la orden ``php bin/vendors install``.

Archivos de arranque y caché de código de bytes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Incluso cuando utilizas código de bytes en caché, el rendimiento mejorará cuando utilices un archivo de arranque ya que habrá menos archivos en los cuales supervisar los cambios. Por supuesto, si esta función está desactivada en la caché del código de bytes (por ejemplo, ``apc.stat = 0`` en APC), ya no hay una razón para utilizar un archivo de arranque.

.. _`cachés de código de bytes`: http://en.wikipedia.org/wiki/List_of_PHP_accelerators
.. _`APC`: http://php.net/manual/es/book.apc.php
.. _`autoloader.php`: https://github.com/symfony/symfony-standard/blob/master/app/autoload.php
.. _`archivo de arranque`: https://github.com/sensio/SensioDistributionBundle/blob/master/Resources/bin/build_bootstrap.php