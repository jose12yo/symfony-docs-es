La arquitectura
===============

¡Eres mi héroe! ¿Quién habría pensado que todavía estarías aquí después de las tres primeras partes? Tu esfuerzo ​​pronto será bien recompensado. En las tres primeras partes no vimos en demasiada profundidad la arquitectura de la
plataforma. Porque esta hace que Symfony2 esté al margen de la multitud de plataformas, ahora vamos a
profundizar en la arquitectura.

Comprendiendo la estructura de directorios
------------------------------------------

La estructura de directorios de una :term:`aplicación` Symfony2 es bastante flexible, pero la estructura de directorios de la distribución de la *edición estándar* refleja la estructura típica y recomendada de una aplicación Symfony2:

* ``app/``:    La configuración de la aplicación;
* ``src/``:    El código PHP del proyecto;
* ``vendor/``: Las dependencias de terceros;
* ``web/``:    La raíz del directorio web.

El Directorio ``web/``
~~~~~~~~~~~~~~~~~~~~~~

El directorio web raíz, es el hogar de todos los archivos públicos y estáticos tales como imágenes, hojas de estilo y archivos JavaScript. También es el lugar donde vive cada :term:`controlador frontal`::

    // web/app.php
    require_once __DIR__.'/../app/bootstrap.php.cache';
    require_once __DIR__.'/../app/AppKernel.php';

    use Symfony\Component\HttpFoundation\Request;

    $nucleo = new AppKernel('prod', false);
    $nucleo->loadClassCache();
    $nucleo->handle(Request::createFromGlobals())->send();

El núcleo requiere en primer lugar el archivo ``bootstrap.php.cache``, el cual arranca la plataforma y registra el cargador automático (ve más abajo).

Al igual que cualquier controlador frontal, ``app.php`` utiliza una clase del núcleo, ``AppKernel``, para arrancar la aplicación.

.. _the-app-dir:

El directorio ``app/``
~~~~~~~~~~~~~~~~~~~~~~

La clase ``AppKernel`` es el punto de entrada principal para la configuración de la aplicación y, como tal, se almacena en el directorio ``app/``.

Esta clase debe implementar dos métodos:

* ``registerBundles()`` debe devolver una matriz de todos los paquetes necesarios para ejecutar la aplicación;

* ``registerContainerConfiguration()`` carga la configuración de la aplicación (más sobre esto más adelante).

La carga automática de clases PHP se puede configurar a través de ``app/autoload.php``::

    // app/autoload.php
    use Symfony\Component\ClassLoader\UniversalClassLoader;

    $loader = new UniversalClassLoader();
    $loader->registerNamespaces(array(
        'Symfony'          => array(__DIR__.'/../vendor/symfony/src', __DIR__.'/../vendor/bundles'),
        'Sensio'           => __DIR__.'/../vendor/bundles',
        'JMS'              => __DIR__.'/../vendor/bundles',
        'Doctrine\\Common' => __DIR__.'/../vendor/doctrine-common/lib',
        'Doctrine\\DBAL'   => __DIR__.'/../vendor/doctrine-dbal/lib',
        'Doctrine'         => __DIR__.'/../vendor/doctrine/lib',
        'Monolog'          => __DIR__.'/../vendor/monolog/src',
        'Assetic'          => __DIR__.'/../vendor/assetic/src',
        'Metadata'         => __DIR__.'/../vendor/metadata/src',
    ));
    $loader->registerPrefixes(array(
        'Twig_Extensions_' => __DIR__.'/../vendor/twig-extensions/lib',
        'Twig_'            => __DIR__.'/../vendor/twig/lib',
    ));

    // ...

    $loader->registerNamespaceFallbacks(array(
        __DIR__.'/../src',
    ));
    $loader->register();

La :class:`Symfony\\Component\\ClassLoader\\UniversalClassLoader` se usa para cargar automáticamente archivos que respetan tanto los `estándares`_ de interoperabilidad técnica de los espacios de nombres de PHP 5.3 como la `convención`_ de nomenclatura de las clases PEAR. Como puedes ver aquí, todas las dependencias se guardan bajo el directorio ``vendor/``, pero esto es sólo una convención. Puedes guardarlas donde quieras, a nivel global en el servidor o localmente en tus proyectos.

.. note::

    Si deseas obtener más información sobre la flexibilidad del autocargador de Symfony2, lee la fórmula ":doc:`/cookbook/tools/autoloader`" en el recetario.

Comprendiendo el sistema de paquetes
------------------------------------

Esta sección introduce una de las más importantes y poderosas características de Symfony2, el sistema de :term:`paquetes <paquete>`.

Un paquete es un poco como un complemento en otros programas. Así que ¿por qué se llama *paquete* y no *complemento*? Esto se debe a que en Symfony2 *todo* es un paquete, desde las características del núcleo de la plataforma hasta el código que escribes para tu aplicación. Los paquetes son ciudadanos de primera clase en Symfony2. Esto te proporciona la flexibilidad para utilizar las características preconstruidas envasadas en paquetes de terceros o para distribuir tus propios paquetes. Además, facilita la selección y elección de las características por habilitar en tu aplicación y optimizarlas en la forma que desees.
Y al final del día, el código de tu aplicación es tan *importante* como el mismo núcleo de la plataforma.

Registrando un paquete
~~~~~~~~~~~~~~~~~~~~~~

Una aplicación se compone de paquetes tal como está definido en el método ``registerBundles()`` de la clase ``AppKernel``. Cada paquete vive en un directorio que contiene una única clase ``Paquete`` que lo describe::

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
            new Symfony\Bundle\SecurityBundle\SecurityBundle(),
            new Symfony\Bundle\TwigBundle\TwigBundle(),
            new Symfony\Bundle\MonologBundle\MonologBundle(),
            new Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle(),
            new Symfony\Bundle\DoctrineBundle\DoctrineBundle(),
            new Symfony\Bundle\AsseticBundle\AsseticBundle(),
            new Sensio\Bundle\FrameworkExtraBundle\SensioFrameworkExtraBundle(),
            new JMS\SecurityExtraBundle\JMSSecurityExtraBundle(),
        );

        if (in_array($this->getEnvironment(), array('dev', 'test'))) {
            $bundles[] = new Acme\DemoBundle\AcmeDemoBundle();
            $bundles[] = new Symfony\Bundle\WebProfilerBundle\WebProfilerBundle();
            $bundles[] = new Sensio\Bundle\DistributionBundle\SensioDistributionBundle();
            $bundles[] = new Sensio\Bundle\GeneratorBundle\SensioGeneratorBundle();
        }

        return $bundles;
    }

Además de ``AcmeDemoBundle`` del cual ya hemos hablado, observa que el núcleo también habilita otros paquetes como ``FrameworkBundle``, ``DoctrineBundle``, ``SwiftmailerBundle`` y ``AsseticBundle``.
Todos ellos son parte del núcleo de la plataforma.

Configurando un paquete
~~~~~~~~~~~~~~~~~~~~~~~

Cada paquete se puede personalizar a través de los archivos de configuración escritos en YAML, XML o PHP. Echa un vistazo a la configuración predeterminada:

.. code-block:: yaml

    # app/config/config.yml
    imports:
        - { resource: parameters.ini }
        - { resource: security.yml }

    framework:
        secret:          %secret%
        charset:         UTF-8
        router:          { resource: "%kernel.root_dir%/config/routing.yml" }
        form:            true
        csrf_protection: true
        validation:      { enable_annotations: true }
        templating:      { engines: ['twig'] } #assets_version: SomeVersionScheme
        session:
            default_locale: %locale%
            auto_start:     true

    # Configuración de Twig
    twig:
        debug:            %kernel.debug%
        strict_variables: %kernel.debug%

    # Configuración de Assetic
    assetic:
        debug:          %kernel.debug%
        use_controller: false
        filters:
            cssrewrite: ~
            # closure:
            #     jar: %kernel.root_dir%/java/compiler.jar
            # yui_css:
            #     jar: %kernel.root_dir%/java/yuicompressor-2.4.2.jar

    # Configuración de Doctrine
    doctrine:
        dbal:
            driver:   %database_driver%
            host:     %database_host%
            dbname:   %database_name%
            user:     %database_user%
            password: %database_password%
            charset:  UTF8

        orm:
            auto_generate_proxy_classes: %kernel.debug%
            auto_mapping: true

    # Configuración de Swiftmailer
    swiftmailer:
        transport: %mailer_transport%
        host:      %mailer_host%
        username:  %mailer_user%
        password:  %mailer_password%

    jms_security_extra:
        secure_controllers:  true
        secure_all_services: false

Cada entrada como ``framework`` define la configuración de un paquete específico.
Por ejemplo, ``framework`` configura el ``FrameworkBundle`` mientras que ``swiftmailer`` configura el ``SwiftmailerBundle``.

Cada :term:`entorno` puede reemplazar la configuración predeterminada proporcionando un archivo de configuración específico. Por ejemplo, el entorno ``dev`` carga el archivo ``config_dev.yml``, el cual carga la configuración principal (es decir, ``config.yml``) y luego la modifica agregando algunas herramientas de depuración:

.. code-block:: yaml

    # app/config/config_dev.yml
    imports:
        - { resource: config.yml }

    framework:
        router:   { resource: "%kernel.root_dir%/config/routing_dev.yml" }
        profiler: { only_exceptions: false }

    web_profiler:
        toolbar: true
        intercept_redirects: false

    monolog:
        handlers:
            main:
                type:  stream
                path:  %kernel.logs_dir%/%kernel.environment%.log
                level: debug
            firephp:
                type:  firephp
                level: info

    assetic:
        use_controller: true

Extendiendo un paquete
~~~~~~~~~~~~~~~~~~~~~~

Además de ser una buena manera de organizar y configurar tu código, un paquete puede extender otro paquete. La herencia de paquetes te permite sustituir cualquier paquete existente con el fin de personalizar sus controladores, plantillas, o cualquiera de sus archivos.
Aquí es donde son útiles los nombres lógicos (por ejemplo, ``@AcmeDemoBundle/Controller/SecuredController.php``): estos abstraen en dónde se almacena el recurso.

Nombres lógicos de archivo
..........................

Cuando quieras hacer referencia a un archivo de un paquete, utiliza esta notación: ``@NOMBRE_PAQUETE/ruta/al/archivo``; Symfony2 resolverá ``@NOMBRE_PAQUETE`` a la ruta real del paquete. Por ejemplo, la ruta lógica ``@AcmeDemoBundle/Controller/DemoController.php`` se convierte en ``src/Acme/DemoBundle/Controller/DemoController.php``, ya que Symfony conoce la ubicación del ``AcmeDemoBundle`` .

Nombres lógicos de Controlador
..............................

Para los controladores, necesitas hacer referencia a los nombres de método usando el formato ``NOMBRE_PAQUETE:NOMBRE_CONTROLADOR:NOMBRE_ACCIÓN``. Por ejemplo, ``AcmeDemoBundle:Bienvenida:index`` representa al método ``indexAction`` de la clase ``Acme\DemoBundle\Controller\BienvenidaController``.

Nombres lógicos de plantilla
............................

Para las plantillas, el nombre lógico ``AcmeDemoBundle:Bienvenida:index.html.twig`` se convierte en la ruta del archivo ``src/Acme/DemoBundle/Resources/views/Bienvenida/index.html.twig``.
Incluso las plantillas son más interesantes cuando te das cuenta que no es necesario almacenarlas en el sistema de archivos. Puedes guardarlas fácilmente en una tabla de la base de datos, por ejemplo.

Extendiendo paquetes
....................

Si sigues estas convenciones, entonces puedes utilizar :doc:`herencia de paquetes </cookbook/bundles/inheritance>` para "redefinir" archivos, controladores o plantillas. Por ejemplo, si un nuevo paquete llamado ``AcmeNuevoBundle`` extiende el ``AcmeDemoBundle``, entonces Symfony primero intenta cargar el controlador ``AcmeDemoBundle:Bienvenida:index`` de ``AcmeNuevoBundle``, y luego ve dentro de ``AcmeDemoBundle``.

¿Entiendes ahora por qué Symfony2 es tan flexible? Comparte tus paquetes entre aplicaciones, guárdalas local o globalmente, tú eliges.

.. _using-vendors:

Usando vendors
--------------

Lo más probable es que tu aplicación dependerá de bibliotecas de terceros. Estas se deberían guardar en el directorio ``vendor/``. Este directorio ya contiene las bibliotecas Symfony2, la biblioteca SwiftMailer, el ORM de Doctrine, el sistema de plantillas Twig y algunas otras bibliotecas y paquetes de terceros.

Comprendiendo la caché y los registros
--------------------------------------

Symfony2 probablemente es una de las plataformas más rápidas hoy día. Pero ¿cómo puede ser tan rápida si analiza e interpreta decenas de archivos YAML y XML por cada petición? La velocidad, en parte, se debe a su sistema de caché. La configuración de la aplicación sólo se analiza en la primer petición y luego se compila hasta código PHP simple y se guarda en el directorio ``app/cache/``. En el entorno de desarrollo, Symfony2 es lo suficientemente inteligente como para vaciar la caché cuando se cambia un archivo. Pero en el entorno de producción, es tu responsabilidad borrar la caché cuando se actualiza o cambia tu código o configuración.

Al desarrollar una aplicación web, las cosas pueden salir mal de muchas formas. Los archivos de registro en el directorio ``app/logs/`` dicen todo acerca de las peticiones y ayudan a solucionar rápidamente el problema.

Usando la interfaz de línea de ordenes
--------------------------------------

Cada aplicación incluye una herramienta de interfaz de línea de ordenes (``app/console``) que te ayuda a mantener la aplicación. Esta proporciona ordenes que aumentan tu productividad automatizando tediosas y repetitivas tareas.

Ejecútalo sin argumentos para obtener más información sobre sus posibilidades:

.. code-block:: bash

    php app/console

La opción ``--help`` te ayuda a descubrir el uso de una orden:

.. code-block:: bash

    php app/console router:debug --help

Consideraciones finales
-----------------------

Llámame loco, pero después de leer esta parte, debes sentirte cómodo moviendo cosas y haciendo que Symfony2 trabaje por ti. Todo en Symfony2 está diseñado para allanar tu camino. Por lo tanto, no dudes en renombrar y mover directorios como mejor te parezca.

Y eso es todo para el inicio rápido. Desde probar hasta enviar mensajes de correo electrónico, todavía tienes que aprender mucho para convertirte en gurú de Symfony2. ¿Listo para zambullirte en estos temas ahora? No busques más - revisa el :doc:`/book/index` oficial y elije cualquier tema que desees.

.. _estándares: http://groups.google.com/group/php-standards/web/psr-0-final-proposal
.. _convención: http://pear.php.net/manual/en/standards.naming.php
