En qué difiere Symfony2 de symfony1
===================================

La plataforma Symfony2 representa una evolución significativa en comparación con la primera versión. Afortunadamente, con la arquitectura MVC en su núcleo, las habilidades para dominar un proyecto symfony1 siguen siendo muy relevantes para el desarrollo de Symfony2. Claro, ``app.yml`` se ha ido, pero el enrutado, los controladores y las plantillas permanecen.

En este capítulo, vamos a recorrer las diferencias entre symfony1 y Symfony2.
Como verás, se abordan muchas tareas de una manera ligeramente diferente. Llegarás a apreciar estas diferencias menores ya que promueven código estable, predecible, verificable y disociado de tus aplicaciones Symfony2.

Por lo tanto, siéntate y relájate mientras te llevamos de "entonces" a "ahora".

Estructura del directorio
-------------------------

Al examinar un proyecto Symfony2 - por ejemplo, la `Edición estándar de Symfony2`_ - te darás cuenta que la estructura de directorios es muy diferente a la de Symfony1. Las diferencias, sin embargo, son un tanto superficiales.

El directorio ``app/``
~~~~~~~~~~~~~~~~~~~~~~

En symfony1, tu proyecto tiene una o más aplicaciones, y cada una vive dentro del directorio ``apps/`` (por ejemplo, ``apps/frontend``). De forma predeterminada en Symfony2, tienes una sola aplicación representada por el directorio ``app/``. Al igual que en Symfony1, el directorio ``app/`` contiene configuración específica a esa aplicación. Este también contiene directorios caché, registro y plantillas específicas de tu aplicación, así como una clase ``Kernel`` (``AppKernel``), la cual es el objeto base que representa la aplicación.

A diferencia de Symfony1, casi no vive código PHP en el directorio ``app/``. Este directorio no está destinado a ser el hogar de módulos o archivos de biblioteca como lo hizo en Symfony1.
En cambio, simplemente es el hogar de la configuración y otros recursos (plantillas, archivos de traducción).

El directorio ``src/``
~~~~~~~~~~~~~~~~~~~~~~

En pocas palabras, tu verdadero código va aquí. En Symfony2, todo el código real de tu aplicación, vive dentro de un paquete (aproximadamente equivalente a un complemento de Symfony1) y, por omisión, cada paquete vive dentro del directorio ``src``. De esta manera, el directorio ``src`` es un poco como el directorio ``plugins`` en Symfony1, pero mucho más flexible. Además, mientras que *tus* paquetes deben vivir en el directorio ``src/``, los paquetes de otros fabricantes pueden vivir en el directorio ``vendor/bundles``.

Para obtener una mejor imagen del directorio ``src/``, primero vamos a pensar en una aplicación Symfony1. En primer lugar, parte de tu código probablemente viva dentro de una o más aplicaciones. Comúnmente son módulos, pero también podrían incluir otras clases PHP que pones en tu aplicación. Es posible que también crees un archivo ``schema.yml`` en el directorio ``config`` de tu proyecto y construyas varios archivos de modelo. Por último, para ayudar con alguna funcionalidad común, estarás usando varios complementos de terceros que viven en el directorio ``plugins/``.
En otras palabras, el código que impulsa tu aplicación vive en muchos lugares diferentes.

En Symfony2, la vida es mucho más simple porque *todo* el código Symfony2 debe vivir en un paquete. En nuestro pretendido proyecto Symfony1, todo el código se *podría* trasladar a uno o más complementos (lo cual es, de hecho, una muy buena práctica). Suponiendo que todos los módulos, clases PHP, esquema, configuración de enrutado, etc. fueran trasladados a un complemento, el directorio ``plugins/`` de symfony1 sería muy similar al ``src/`` de Symfony2.

En pocas palabras de nuevo, el directorio ``src/`` es donde vive el código, activos, plantillas y la mayoría de cualquier otra cosa específica a tu proyecto.

El directorio ``vendor/``
~~~~~~~~~~~~~~~~~~~~~~~~~

El directorio ``vendor/`` básicamente es el equivalente al directorio ``lib/vendor/`` en Symfony1, que fue el directorio convencional para todas las bibliotecas y paquetes de los proveedores. De manera predeterminada, encontrarás los archivos de la biblioteca Symfony2 en este directorio, junto con varias otras bibliotecas dependientes, como Doctrine2, Twig y SwiftMailer. La 3ª parte de los paquetes de Symfony2 generalmente vive en ``vendor/bundles/``.

El Directorio ``web/``
~~~~~~~~~~~~~~~~~~~~~~

No ha cambiado mucho el directorio ``web/``. La diferencia más notable es la ausencia de los directorios ``css/``, ``js/`` e ``images/``. Esto es intencional. Al igual que con tu código PHP, todos los activos también deben vivir dentro de un paquete. Con la ayuda de una consola de ordenes, el directorio ``Resources/public/`` de cada paquete se copia o enlaza simbólicamente al directorio ``web/bundles/``. Esto nos permite mantener los activos organizados dentro de tu paquete, pero estando disponibles al público. Para asegurarte que todos los paquetes están disponibles, ejecuta la siguiente orden::

    php app/console assets:install web

.. note::

   Esta orden es el equivalente Symfony2 a la orden ``plugin:publish-assets`` de Symfony1.

Carga automática
----------------

Una de las ventajas de las plataformas modernas es nunca tener que preocuparte de requerir archivos manualmente. Al utilizar un cargador automático, puedes referirte a cualquier clase en tu proyecto y confiar en que esté disponible. La carga automática de clases ha cambiado en Symfony2 para ser más universal, más rápida e independiente de la necesidad de vaciar la caché.

En Symfony1, la carga automática de clases se llevó a cabo mediante la búsqueda en todo el proyecto de la presencia de archivos de clases PHP y almacenamiento en caché de esta información en una matriz gigante.
Esa matriz decía a Symfony1 exactamente qué archivo contenía cada clase. En el entorno de producción, esto causó que necesitaras borrar la memoria caché cuando añadías o movías clases.

En Symfony2, una nueva clase - ``UniversalClassLoader`` - se encarga de este proceso.
La idea detrás del cargador automático es simple: el nombre de tu clase (incluyendo el espacio de nombres) debe coincidir con la ruta al archivo que contiene esa clase.
Tomemos como ejemplo el ``FrameworkExtraBundle`` de la Edición estándar de Symfony2::

    namespace Sensio\Bundle\FrameworkExtraBundle;

    use Symfony\Component\HttpKernel\Bundle\Bundle;
    // ...

    class SensioFrameworkExtraBundle extends Bundle
    {
        // ...

El archivo en sí mismo vive en ``vendor/bundle/Sensio/Bundle/FrameworkExtraBundle/SensioFrameworkExtraBundle.php``.
Como puedes ver, la ubicación del archivo sigue el espacio de nombres de la clase.
Específicamente, el espacio de nombres, ``Sensio\Bundle\FrameworkExtraBundle``, explica el directorio en que el archivo debe vivir (``vendor/bundle/Sensio/Bundle/FrameworkExtraBundle``). Esto es así porque, en el archivo ``app/autoload.php``, debes configurar a Symfony para buscar el espacio de nombres ``Sensio`` en el directorio ``vendor/bundle``:

.. code-block:: php

    // app/autoload.php

    // ...
    $loader->registerNamespaces(array(
        // ...
        'Sensio'           => __DIR__.'/../vendor/bundles',
    ));

Si el archivo *no* vive en ese lugar exacto, recibirás un error ``La clase "Acme\DemoBundle\Controller\DemoController" no existe.`` En Symfony2, una "clase no existe" significa que el espacio de nombres de la clase sospechosa y la ubicación física no coinciden. Básicamente, Symfony2 está buscando en una ubicación exacta dicha clase, pero ese lugar no existe (o contiene una clase diferente). Para que una clase se cargue automáticamente, en Symfony2 **nunca necesitas vaciar la caché**.

Como se mencionó anteriormente, para que trabaje el cargador automático, es necesario saber que el espacio de nombres ``Sensio`` vive en el directorio ``vendor/bundles`` y que, por ejemplo, el espacio de nombres ``Doctrine`` vive en el directorio ``vendor/doctrine/lib/``. Esta asignación es controlada totalmente por ti a través del archivo ``app/autoload.php``.

Si nos fijamos en el ``HolaController`` de la edición estándar de Symfony2 puedes ver que este vive en el espacio de nombres ``Acme\DemoBundle\Controller``. Sin embargo, el espacio de nombres ``Acme`` no está definido en el ``app/autoload.php``. Por omisión, no es necesario configurar explícitamente la ubicación de los paquetes que viven en el directorio ``src/``. El ``UniversalClassLoader`` está configurado para retroceder al directorio ``src/`` usando el método ``registerNamespaceFallbacks``:

.. code-block:: php

    // app/autoload.php

    // ...
    $loader->registerNamespaceFallbacks(array(
        __DIR__.'/../src',
    ));

Usando la consola
-----------------

En symfony1, la consola se encuentra en el directorio raíz de tu proyecto y se llama ``symfony``:

.. code-block:: text

    php symfony

En Symfony2, la consola se encuentra ahora en el subdirectorio ``app`` y se llama ``console``:

.. code-block:: text

    php app/console

Aplicaciones
------------

En un proyecto symfony1, es común tener varias aplicaciones: una para la interfaz de usuario y otra para la interfaz de administración, por ejemplo.

En un proyecto Symfony2, sólo tienes que crear una aplicación (una aplicación de blog, una aplicación de intranet, ...). La mayoría de las veces, si deseas crear una segunda aplicación, en su lugar podrías crear otro proyecto y compartir algunos paquetes entre ellos.

Y si tienes que separar la interfaz y las funciones de interfaz de administración de algunos paquetes, puedes crear subespacios de nombres para los controladores, subdirectorios de plantillas, diferentes configuraciones semánticas, configuración de enrutado separada, y así sucesivamente.

Por supuesto, no hay nada malo en tener varias aplicaciones en el proyecto, lo cual es una elección totalmente tuya. Una segunda aplicación significaría un nuevo directorio, por ejemplo, ``mi_aplic/``, con la misma configuración básica que el directorio ``app/``.

.. tip::

    Lee la definición de un :term:`Proyecto`, una :term:`Aplicación`, y un :term:`Paquete` en el glosario.

Paquetes y complementos
-----------------------

En un proyecto symfony1, un complemento puede contener configuración, módulos, bibliotecas PHP, activos y cualquier otra cosa relacionada con tu proyecto. En Symfony2, la idea de un complemento es reemplazada por el "paquete". Un paquete es aún más poderoso que un complemento porque el núcleo de la plataforma Symfony2 consta de una serie de paquetes. En Symfony2, los paquetes son ciudadanos de primera clase y son tan flexibles que incluso el código del núcleo en sí es un paquete.

En symfony1, un complemento se debe activar dentro de la clase ``ProjectConfiguration``::

    // config/ProjectConfiguration.class.php
    public function setup()
    {
        $this->enableAllPluginsExcept(array(/* algunos complementos aquí */));
    }

En Symfony2, los paquetes se activan en el interior del núcleo de la aplicación::

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
            new Symfony\Bundle\TwigBundle\TwigBundle(),
            // ...
            new Acme\DemoBundle\AcmeDemoBundle(),
        );

        return $bundles;
    }

Enrutando (``routing.yml``) y configurando (``config.yml``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

En symfony1, los archivos de configuración ``routing.yml`` y ``app.yml`` cargan cualquier complemento automáticamente. En Symfony2, la configuración de enrutado y de la aplicación dentro de un paquete se debe incluir manualmente. Por ejemplo, para incluir la ruta a un recurso de un paquete, puedes hacer lo siguiente::

    # app/config/routing.yml
    _hola:
        resource: "@AcmeDemoBundle/Resources/config/routing.yml"

Esto cargará las rutas que se encuentren en el archivo ``Resources/config/routing.yml`` del ``AcmeDemoBundle``. La sintaxis especial ``@AcmeDemoBundle`` es un atajo que, internamente, resuelve la ruta al directorio del paquete.

Puedes utilizar esta misma estrategia en la configuración de un paquete:

.. code-block:: yaml

    # app/config/config.yml
    imports:
        - { resource: "@AcmeDemoBundle/Resources/config/config.yml" }

En Symfony2, la configuración es un poco como ``app.yml`` en  Symfony1, salvo que mucho más sistemática. Con ``app.yml``, simplemente puedes crear las claves que quieras.
De forma predeterminada, estas entradas no tenían sentido y dependían totalmente de cómo se utilizaban en tu aplicación:

.. code-block:: yaml

    # algún archivo app.yml de symfony1
    all:
      correo:
        from_domicilio:  foo.bar@ejemplo.com

En Symfony2, también puedes crear entradas arbitrarias bajo la clave ``parameters`` de tu configuración:

.. code-block:: yaml

    parameters:
        email.from_domicilio: foo.bar@ejemplo.com

Ahora puedes acceder a ella desde un controlador, por ejemplo::

    public function holaAction($nombre)
    {
        $fromAddress = $this->contenedor->getParameter('email.from_domicilio');
    }

En realidad, la configuración de Symfony2 es mucho más poderosa y se utiliza principalmente para configurar los objetos que puedes utilizar. Para más información, consulta el capítulo titulado ":doc:`/book/service_container`".

.. _`Edición estándar de Symfony2`: https://github.com/symfony/symfony-standard
