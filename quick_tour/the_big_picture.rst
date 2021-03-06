Un primer vistazo
=================

¡Empieza a usar Symfony2 en 10 minutos! Este capítulo te guiará a través de algunos de los conceptos más importantes detrás de Symfony2 y explica cómo puedes empezar a trabajar rápidamente, mostrándote un simple proyecto en acción.

Si ya haz usado una plataforma para desarrollo web, seguramente te sentirás a gusto con Symfony2. Si no es tu caso, ¡bienvenido a una nueva forma de desarrollar aplicaciones web!

.. tip::

    ¿Quieres saber por qué y cuándo es necesario utilizar una plataforma? Lee el documento "`Symfony en 5 minutos`_".

Descargando Symfony2
--------------------

En primer lugar, comprueba que tienes instalado y configurado un servidor web (como Apache) con PHP 5.3.2 o superior.

¿Listo? Empecemos descargando la "`Edición estándar de Symfony2`_", una :term:`distribución` de Symfony preconfigurada para la mayoría de los casos y que también contiene algún código de ejemplo que demuestra cómo utilizar Symfony2 (consigue el paquete que incluye *proveedores* para empezar aún más rápido).

Después de extraer el paquete bajo el directorio raíz del servidor web, deberías tener un directorio ``Symfony/`` con una estructura como esta:

.. code-block:: text

    www/ <- el directorio raíz de tu servidor web
        Symfony/ <- el archivo desempacado
            app/
                cache/
                config/
                logs/
                Resources/
            bin/
            src/
                Acme/
                    DemoBundle/
                        Controller/
                        Resources/
                        ...
            vendor/
                symfony/
                doctrine/
                ...
            web/
                app.php
                ...

.. note::

    Si descargaste la *Edición estándar* ``sin vendors``, basta con ejecutar la siguiente orden para descargar todas las bibliotecas de proveedores:

    .. code-block:: bash

        php bin/vendors install

Verificando tu configuración
----------------------------

Symfony2 integra una interfaz visual para probar la configuración del servidor, muy útil para solucionar problemas relacionados con el servidor Web o una incorrecta configuración de PHP. Usa la siguiente url para examinar el diagnóstico:

.. code-block:: text

    http://localhost/Symfony/web/config.php

Si se listan errores o aspectos de configuración pendientes, corrígelos; Puedes realizar los ajustes siguiendo las recomendaciones. Cuando todo esté bien, haz clic en "*Pospón la configuración y llévame a la página de bienvenida*" para solicitar tu primera página web "real" en Symfony2:

.. code-block:: text

    http://localhost/Symfony/web/app_dev.php/

¡Symfony2 debería felicitarte por tu arduo trabajo hasta el momento!

.. image:: /images/quick_tour/bienvenida.jpg
   :align: center

Comprendiendo los fundamentos
-----------------------------

Uno de los principales objetivos de una plataforma es garantizar la `separación de responsabilidades`_.
Esto mantiene tu código organizado y permite a tu aplicación evolucionar fácilmente en el tiempo, evitando mezclar llamadas a la base de datos, etiquetas HTML y el código de lógica del negocio en un mismo archivo. Para alcanzar este objetivo, debes aprender algunos conceptos y términos fundamentales.

.. tip::

    ¿Quieres más pruebas de que usar una plataforma es mucho mejor que mezclar todo en un mismo archivo? Lee el capítulo del libro ":doc:`/book/from_flat_php_to_symfony2`".

La distribución viene con algún código de ejemplo que puedes utilizar para aprender más sobre los principales conceptos de Symfony2. Ingresa a la siguiente URL para recibir un saludo de Symfony2 (reemplaza *Nacho* con tu nombre):

.. code-block:: text

    http://localhost/Symfony/web/app_dev.php/demo/hola/Nacho

.. image:: /images/quick_tour/hola_nacho.png
   :align: center

¿Qué sucedió? Bien, diseccionemos la url:

* ``app_dev.php``: Es un :term:`controlador frontal`. Es el único punto de entrada de la aplicación, mismo que responde a todas las peticiones del usuario;

* ``/demo/hola/Nacho``: Esta es la *ruta virtual* a los recursos que el usuario quiere acceder.

Tu responsabilidad como desarrollador es escribir el código que asigna la *petición* del usuario (``/demo/hola/Nacho``) al *recurso* asociado con ella (la página HTML ``¡Hola Nacho!``).

Enrutando
~~~~~~~~~

Symfony2 encamina la petición al código que la maneja tratando de hacer coincidir la URL solicitada contra algunos patrones configurados. De forma predeterminada, estos patrones (llamados rutas) se definen en el archivo de configuración ``app/config/routing.yml``: Cuando estás en el :ref:`entorno <quick-tour-big-picture-environments>` ``dev`` - indicado por el controlador frontal **app_dev.php** - el archivo de configuración ``app/config/routing_dev.yml`` también es cargado. En la edición estándar, las rutas a estas páginas de "demostración" se encuentran en ese archivo:

.. code-block:: yaml

    # app/config/routing_dev.yml
    _bienvenida:
        pattern:  /
        defaults: { _controller: AcmeDemoBundle:Bienvenida:index }

    _demo:
        resource: "@AcmeDemoBundle/Controller/DemoController.php"
        type:     annotation
        prefix:   /demo

    # ...

Las primeras tres líneas (después del comentario) definen el código que se ejecuta cuando el usuario solicita el recurso "``/``" (es decir, la página de bienvenida que viste anteriormente). Cuando así lo solicite, el controlador ``AcmeDemoBundle:Bienvenida:index`` será ejecutado. En la siguiente sección, aprenderás exactamente lo que eso significa.

.. tip::

    La edición estándar de Symfony2 utiliza `YAML`_ para sus archivos de configuración, pero Symfony2 también es compatible con XML, PHP y anotaciones nativas. Los diferentes formatos son compatibles y se pueden utilizar indistintamente en una aplicación. Además, el rendimiento de tu aplicación no depende del formato de configuración que elijas, ya que todo se memoriza en caché en la primer petición.

Controladores
~~~~~~~~~~~~~

Un controlador es un nombre elegante para una función o método PHP que se encarga de las *peticiones* entrantes y devuelve las *respuestas* (a menudo código HTML). En lugar de utilizar variables globales y funciones PHP (como ``$_GET`` o ``header()``) para manejar estos mensajes HTTP, Symfony utiliza objetos: :class:`Symfony\\Component\\HttpFoundation\\Request` y :class:`Symfony\\Component\\HttpFoundation\\Response`. El controlador más simple posible es crear la respuesta a mano, basándote en la petición::

    use Symfony\Component\HttpFoundation\Response;

    $nombre = $peticion->query->get('nombre');

    return new Response('Hola '.$nombre, 200, array('Content-Type' => 'text/plain'));

.. note::

    Symfony2 abarca la especificación HTTP, esta contiene las reglas que gobiernan todas las comunicaciones en la web. Lee el capítulo ":doc:`/book/http_fundamentals`" del libro para aprender más acerca de esto y la potencia que ello conlleva.

Symfony2 elige el controlador basándose en el valor de ``_controller`` de la configuración de enrutado: ``AcmeDemoBundle:Bienvenida:index``. Esta cadena es el nombre lógico del *controlador*, y hace referencia al método ``indexAction`` de la clase ``Acme\DemoBundle\Controller\BienvenidaController``::

    // src/Acme/DemoBundle/Controller/BienvenidaController.php
    namespace Acme\DemoBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class BienvenidaController extends Controller
    {
        public function indexAction()
        {
            return $this->render('AcmeDemoBundle:Bienvenida:index.html.twig');
        }
    }

.. tip::

    Podrías haber usado el nombre completo de la clase y método - ``Acme\DemoBundle\Controller\BienvenidaController::indexAction`` - para el valor del ``_controller``. Pero si sigues algunas simples convenciones, el nombre lógico es más conciso y te permite mayor flexibilidad.

La clase ``BienvenidaController`` extiende la clase integrada ``Controller``, la cual proporciona útiles atajos a métodos, como el :method:`Symfony\\Bundle\\FrameworkBundle\\Controller\\Controller::render` que carga y reproduce una plantilla (``AcmeDemoBundle:Bienvenida:index.html.twing``). El valor devuelto es un objeto ``Respuesta`` poblado con el contenido reproducido. Por lo tanto, si surge la necesidad, la ``Respuesta`` se puede ajustar antes de enviarla al navegador::

    public function indexAction()
    {
        $respuesta = $this->render('AcmeDemoBundle:Bienvenida:index.txt.twig');
        $respuesta->headers->set('Content-Type', 'text/plain');

        return $respuesta;
    }

Pero en todos los casos, el trabajo final del controlador siempre es devolver el objeto ``Respuesta`` que será entregado al usuario. Este objeto ``Respuesta`` se puede poblar con código HTML, representar una redirección al cliente, e incluso devolver el contenido de una imagen JPG con una cabecera ``Content-Type`` de ``image/jpg``.

.. tip::

    Derivar de la clase base ``Controlller`` es opcional. De hecho, un controlador puede ser una simple función PHP e incluso un cierre PHP.
    El capítulo ":doc:`Controlador </book/controller>`" del libro abarca todo sobre los controladores de Symfony2.

El nombre de la plantilla, ``AcmeDemoBundle:Bienvenida:index.html.twig``, es el *nombre lógico* de la plantilla y hace referencia al archivo ``Resources/views/Bienvenida/index.html.twig`` dentro del ``AcmeDemoBundle`` (ubicado en ``src/Acme/DemoBundle``). En la sección paquetes, a continuación, explicaré por qué esto es útil.

Ahora, de nuevo echa un vistazo a la configuración de enrutado y encuentra la clave ``_demo``:

.. code-block:: yaml

    # app/config/routing_dev.yml
    _demo:
        resource: "@AcmeDemoBundle/Controller/DemoController.php"
        type:     annotation
        prefix:   /demo

Symfony2 puede leer/importar la información de enrutado desde diferentes archivos escritos en YAML, XML, PHP o, incluso, incorporada en anotaciones PHP. En este caso, el *nombre lógico* del recurso es ``@AcmeDemoBundle/Controller/DemoController.php`` y se refiere al archivo ``src/Acme/DemoBundle/Controller/DemoController.php``. En este archivo, las rutas se definen como anotaciones sobre los métodos de acción::

    // src/Acme/DemoBundle/Controller/DemoController.php
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

    class DemoController extends Controller
    {
        /**
         * @Route("/hola/{nombre}", name="_demo_hola")
         * @Template()
         */
        public function holaAction($nombre)
        {
            return array('nombre' => $nombre);
        }

        // ...
    }

La anotación ``@Route()`` define una nueva ruta con un patrón de ``/hola/{nombre}`` que ejecuta el método ``holaAction`` cuando concuerda. Una cadena encerrada entre llaves como ``{nombre}`` se conoce como marcador de posición. Como puedes ver, su valor se puede recuperar a través del argumento ``$nombre`` del método.

.. note::

    Incluso si las anotaciones no son compatibles nativamente en PHP, las utilizamos ampliamente en Symfony2 como una conveniente manera de configurar el comportamiento de la plataforma y mantener la configuración del lado del código.

Si echas un vistazo más de cerca al código de la acción del controlador, puedes ver que en lugar de reproducir una plantilla y devolver un objeto ``Respuesta`` como antes, sólo devuelve una matriz de parámetros. La anotación ``@Template()`` le dice a Symfony que reproduzca la plantilla por ti, pasando cada  variable del arreglo a la plantilla. El nombre de la plantilla reproducida sigue al nombre del controlador. Por lo tanto, en este ejemplo, se reproduce la plantilla ``AcmeDemoBundle:Demo:hola.html.twig`` (ubicada en ``src/Acme/DemoBundle/Resources/views/Demo/hola.html.twig``).

.. tip::

    Las anotaciones ``@Route()`` y ``@Template()`` son más poderosas que lo mostrado en el ejemplo simple de esta guía. Aprende más sobre las "`anotaciones en controladores`_" en la documentación oficial.

Plantillas
~~~~~~~~~~

El controlador procesa la plantilla ``src/Acme/DemoBundle/Resources/views/Demo/hola.html.twig`` (o ``AcmeDemoBundle:Demo:hola.html.twig`` si utilizas el nombre lógico):

.. code-block:: jinja

    {# src/Acme/DemoBundle/Resources/views/Demo/hola.html.twig #}
    {% extends "AcmeDemoBundle::base.html.twig" %}

    {% block titulo "Hola " ~ nombre %}

    {% block contenido %}
        <h1>Hola {{ nombre }}!</h1>
    {% endblock %}

Por omisión, Symfony2 utiliza `Twig`_ como motor de plantillas, pero también puede utilizar plantillas PHP tradicionales si lo deseas. El siguiente capítulo es una introducción a cómo trabajan las plantillas en Symfony2.

Paquetes
~~~~~~~~

Posiblemente te hayas preguntado por qué la palabra bundle (:term:`paquete` en adelante), se utiliza en muchos de los nombres que hemos visto hasta ahora. Todo el código que escribas para tu aplicación está organizado en paquetes. Hablando en Symfony2, un paquete es un conjunto estructurado de archivos (archivos PHP, hojas de estilo, JavaScript, imágenes, ...) que implementa una sola característica (un blog, un foro, ...) y que fácilmente se puede compartir con otros desarrolladores. Hasta ahora, hemos manipulado un paquete, ``AcmeDemoBundle``. Aprenderás más acerca de los paquetes en el último capítulo de esta guía.

.. _quick-tour-big-picture-environments:

Trabajando con entornos
-----------------------

Ahora que tienes una mejor comprensión de cómo funciona Symfony2, dale una mirada más atenta a la parte inferior de cualquier página reproducida por Symfony2. Deberás notar una pequeña barra con el logotipo de Symfony2. Esta se conoce como la "barra de depuración web" y es la mejor amiga del desarrollador.

.. image:: /images/quick_tour/web_debug_toolbar.png
   :align: center

Pero lo que ves al principio es sólo la punta del iceberg; haz clic en el extraño número hexadecimal para revelar otra muy útil herramienta de depuración de Symfony2: el generador de perfiles.

.. image:: /images/quick_tour/profiler.png
   :align: center

Por supuesto, no querrás mostrar estas herramientas al desplegar tu aplicación en producción. Es por eso que encontrarás otro controlador frontal en el directorio ``web/`` (``app.php``), el cual está optimizado para el entorno de producción:

.. code-block:: text

    http://localhost/Symfony/web/app.php/demo/hola/Nacho

Y si utilizas Apache con ``mod_rewrite`` habilitado, incluso puedes omitir la parte ``app.php`` de la URL:

.. code-block:: text

    http://localhost/Symfony/web/demo/hola/Nacho

Por último pero no menos importante, en los servidores en producción, debes apuntar tu directorio web raíz al directorio ``web/`` para proteger tu instalación e incluso, para que tus direcciones URL tengan un mejor aspecto:

.. code-block:: text

    http://localhost/demo/hola/Nacho

Para hacer que la aplicación responda más rápido, Symfony2 mantiene una caché en el directorio ``app/cache/``. En el entorno de desarrollo (``app_dev.php``), esta caché se vacía automáticamente cada vez que realizas cambios en cualquier código o configuración. Pero ese no es el caso en el entorno de producción (``app.php``) donde el rendimiento es clave. Es por eso que siempre debes utilizar el entorno de desarrollo al estar desarrollando tu aplicación.

Diferentes :term:`entornos <entorno>` de una determinada aplicación sólo se diferencian en su configuración. De hecho, una configuración puede heredar de otra:

.. code-block:: yaml

    # app/config/config_dev.yml
    imports:
        - { resource: config.yml }

    web_profiler:
        toolbar: true
        intercept_redirects: false

El entorno ``dev`` (el cual carga el archivo de configuración ``config_dev.yml``) importa el archivo global ``config.yml`` y luego lo modifica, en este ejemplo, activando la barra de herramientas para depuración web.

Consideraciones finales
-----------------------

¡Enhorabuena! Haz tenido tu primera experiencia codificando en Symfony2. No fue tan difícil, ¿cierto? Hay mucho más por explorar, pero ya debes tener una idea de cómo Symfony2 facilita la implementación de mejores y más rápidos sitios web. Si estás interesado en aprender más acerca de Symfony2, sumérgete en la siguiente sección: ":doc:`La vista<the_view>`".

.. _Edición estándar de Symfony2:          http://symfony.com/download
.. _Symfony en 5 minutos:               http://symfony.com/symfony-in-five-minutes
.. _separación de responsabilidades:             http://en.wikipedia.org/wiki/Separation_of_concerns
.. _YAML:                               http://www.yaml.org/
.. _anotaciones en controladores:         http://bundles.symfony-reloaded.org/frameworkextrabundle/
.. _Twig:                               http://www.twig-project.org/
