Corriendo las pruebas de Symfony2
=================================

Antes de presentar un :doc:`parche <patches>` para su inclusión, es necesario ejecutar el banco de pruebas Symfony2 para comprobar que no ha roto nada.

PHPUnit
-------

Para ejecutar el banco de pruebas de Symfony2, primero `instala PHPUnit`_ 3.5.0 o superior:

.. code-block:: bash

    $ pear channel-discover pear.phpunit.de
    $ pear channel-discover components.ez.no
    $ pear channel-discover pear.symfony-project.com
    $ pear install phpunit/PHPUnit

Dependencias (opcional)
-----------------------

Para ejecutar el banco de pruebas completo, incluyendo las pruebas supeditadas con dependencias externas, Symfony2 tiene que ser capaz de cargarlas automáticamente. De forma predeterminada, se cargan automáticamente desde `vendor/` en la raíz del directorio principal (consulta la sección `autoload.php.dist`).

El banco de pruebas necesita las siguientes bibliotecas de terceros:

* Doctrine
* Doctrine Migrations
* Swiftmailer
* Twig

Para instalarlas todas, ejecuta el archivo `vendors.php`:

.. code-block:: bash

    $ php vendors install

.. note::

    Ten en cuenta que el guión toma algún tiempo para terminar.

Después de la instalación, puedes actualizar los proveedores en cualquier momento con la siguiente orden.

.. code-block:: bash

    $ php vendors update

Ejecutando
----------

En primer lugar, actualiza los proveedores (consulta más arriba).

A continuación, ejecuta el banco de pruebas desde el directorio raíz de Symfony2 con la siguiente orden:

.. code-block:: bash

    $ phpunit

La salida debe mostrar `OK`. Si no es así, es necesario averiguar qué está pasando y si las pruebas se rompen a causa de tus modificaciones.

.. tip::

    Ejecuta el banco de pruebas antes de aplicar las modificaciones para comprobar que funcionan bien en tu configuración.

Cobertura de código
-------------------

Si agregas una nueva característica, también necesitas comprobar la cobertura de código usando la opción `coverage-html`:

.. code-block:: bash

    $ phpunit --coverage-html=cov/

Verifica la cobertura de código abriendo en un navegador la página generada `cov/index.html`.

.. tip::

    La cobertura de código sólo funciona si tienes activado XDebug e instaladas todas las dependencias.

.. _`instala PHPUnit`: http://www.phpunit.de/manual/current/en/installation.html
