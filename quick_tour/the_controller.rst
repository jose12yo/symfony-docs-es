El controlador
==============

¿Todavía con nosotros después de las dos primeras partes? ¡Ya te estás volviendo adicto a Symfony2! Sin más preámbulos, vamos a descubrir lo que los controladores pueden hacer por ti.

Usando Formatos
---------------

Hoy día, una aplicación web debe ser capaz de ofrecer algo más que solo páginas HTML. Desde XML para alimentadores RSS o Servicios Web, hasta JSON para peticiones Ajax, hay un montón de formatos diferentes a elegir. Apoyar estos formatos en Symfony2 es sencillo. Modifica la ruta añadiendo un valor predeterminado de ``xml`` a la variable ``_format``::

    // src/Acme/DemoBundle/Controller/DemoController.php
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

    /**
     * @Route("/hola/{nombre}", defaults={"_format"="xml"}, name="_demo_hola")
     * @Template()
     */
    public function holaAction($nombre)
    {
        return array('nombre' => $nombre);
    }

Al utilizar el formato de la petición (como lo define el valor ``_format``), Symfony2 automáticamente selecciona la plantilla adecuada, aquí ``hola.xml.twig``:

.. code-block:: xml+php

    <!-- src/Acme/DemoBundle/Resources/views/Demo/hola.xml.twig -->
    <hola>
        <nombre>{{ nombre }}</nombre>
    </hola>

Eso es todo lo que hay que hacer. Para los formatos estándar, Symfony2 también elije automáticamente la mejor cabecera ``Content-Type`` para la respuesta. Si quieres apoyar diferentes formatos para una sola acción, en su lugar, usa el marcador de posición ``{_format}`` en el patrón de la ruta::

    // src/Acme/DemoBundle/Controller/DemoController.php
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

    /**
     * @Route("/hola/{nombre}.{_format}", defaults={"_format"="html"}, requirements={"_format"="html|xml|json"}, name="_demo_hola")
     * @Template()
     */
    public function holaAction($nombre)
    {
        return array('nombre' => $nombre);
    }

El controlador ahora será llamado por la URL como ``/demo/hola/Fabien.xml`` o ``/demo/hola/Fabien.json``.

La entrada ``requirements`` define las expresiones regulares con las cuales los marcadores de posición deben coincidir. En este ejemplo, si tratas de solicitar el recurso ``/demo/hola/Fabien.js``, obtendrás un error HTTP 404, ya que no coincide con el requisito de ``_format``.

Redirigiendo y reenviando
-------------------------

Si deseas redirigir al usuario a otra página, utiliza el método ``redirect()``::

    return $this->redirect($this->generateUrl('_demo_hola', array('nombre' => 'Lucas')));

El método ``generateUrl()`` es el mismo que la función ``path()`` que utilizamos en las plantillas. Este toma el nombre de la ruta y una serie de parámetros como argumentos y devuelve la URL amigable asociada.

Además, fácilmente puedes reenviar a otra acción con el método ``forward()``. Internamente, Symfony hace una "subpetición", y devuelve el objeto ``Respuesta`` desde la subpetición::

    $respuesta = $this->forward('AcmeDemoBundle:Hola:maravillosa', array('nombre' => $nombre, 'color' => 'verde'));

    // hace algo con la respuesta o la devuelve directamente

Obteniendo información de la petición
-------------------------------------

Además del valor de los marcadores de posición de enrutado, el controlador también tienes acceso al objeto ``Petición``::

    $peticion = $this->getRequest();

    $peticion->isXmlHttpRequest(); // ¿es una petición Ajax?

    $peticion->getPreferredLanguage(array('en', 'es'));

    $peticion->query->get('pag'); // consigue un parámetro $_GET

    $peticion->request->get('pag'); // consigue un parámetro $_POST

En una plantilla, también puedes acceder al objeto ``Petición`` por medio de la variable ``app.request``:

.. code-block:: html+jinja

    {{ app.request.query.get('pag') }}

    {{ app.request.parameter('pag') }}

Persistiendo datos en la sesión
-------------------------------

Aunque el protocolo HTTP es sin estado, Symfony2 proporciona un agradable objeto sesión que representa al cliente (sea una persona real usando un navegador, un robot o un servicio web). Entre dos peticiones, Symfony2 almacena los atributos en una ``cookie`` usando las sesiones nativas de PHP.

Almacenar y recuperar información de la sesión se puede conseguir fácilmente desde cualquier controlador::

    $sesion = $this->getRequest()->getSession();

    // guarda un atributo para reutilizarlo durante una posterior petición del usuario
    $sesion->set('foo', 'bar');

    // en otro controlador por otra petición
    $foo = $sesion->get('foo');

    // fija la configuración regional del usuario
    $sesion->setLocale('es');

También puedes almacenar pequeños mensajes que sólo estarán disponibles para la siguiente petición::

    // guarda un mensaje para la siguiente petición (en un controlador)
    $sesion->setFlash('aviso', '¡Felicidades, tu acción fue exitosa!');

    // muestra el mensaje de nuevo en la siguiente petición (en una plantilla)
    {{ app.session.flash('aviso') }}

Esto es útil cuando es necesario configurar un mensaje de éxito antes de redirigir al usuario a otra página (la cual entonces mostrará el mensaje).

Protegiendo recursos
--------------------

La edición estándar de Symfony viene con una configuración de seguridad sencilla adaptada a las necesidades más comunes:

.. code-block:: yaml

    # app/config/security.yml
    security:
        encoders:
            Symfony\Component\Security\Core\User\User: plaintext

        role_hierarchy:
            ROLE_ADMIN:       ROLE_USER
            ROLE_SUPER_ADMIN: [ROLE_USER, ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH]

        providers:
            in_memory:
                users:
                    user:  { password: userpass, roles: [ 'ROLE_USER' ] }
                    admin: { password: adminpass, roles: [ 'ROLE_ADMIN' ] }

        firewalls:
            dev:
                pattern:  ^/(_(profiler|wdt)|css|images|js)/
                security: false

            login:
                pattern:  ^/demo/secured/login$
                security: false

            secured_area:
                pattern:    ^/demo/secured/
                form_login:
                    check_path: /demo/secured/login_check
                    login_path: /demo/secured/login
                logout:
                    path:   /demo/secured/logout
                    target: /demo/

Esta configuración requiere que los usuarios inicien sesión para cualquier URL que comienza con ``/demo/secured/`` y define dos usuarios válidos: ``user`` y ``admin``.
Por otra parte, el usuario ``admin`` tiene un rol ``ROLE_ADMIN``, el cual incluye el rol ``ROLE_USER`` también (consulta el ajuste ``role_hierarchy``).

.. tip::

    Para facilitar la lectura, las contraseñas se almacenan en texto plano en esta configuración simple, pero puedes usar cualquier algoritmo de codificación ajustando la sección ``encoders``.

Al ir a la dirección ``http://localhost/Symfony/web/app_dev.php/demo/secured/hola``
automáticamente redirigirá al formulario de acceso, porque el recurso está
protegido por un ``cortafuegos``.

También puedes forzar la acción para exigir un determinado rol usando la anotación ``@Secure`` en el controlador::

    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;
    use JMS\SecurityExtraBundle\Annotation\Secure;

    /**
     * @Route("/hola/admin/{nombre}", name="_demo_secured_hola_admin")
     * @Secure(roles="ROLE_ADMIN")
     * @Template()
     */
    public function holaAdminAction($nombre)
    {
        return array('nombre' => $nombre);
    }

Ahora, inicia sesión como ``user`` (el cual no *tiene* el rol ``ROLE_ADMIN``) y desde la página protegida ``hola``, haz clic en el enlace "Hola recurso protegido".
Symfony2 debe devolver un código de estado HTTP 403, el cual indica que el usuario tiene "prohibido" el acceso a ese recurso.

.. note::

    La capa de seguridad de Symfony2 es muy flexible y viene con muchos proveedores de usuario diferentes (por ejemplo, uno para el ORM de Doctrine) y proveedores de autenticación (como HTTP básica, HTTP digest o certificados X509). Lee el capítulo ":doc:`/book/security`" del libro para más información en cómo se usa y configura.

Memorizando recursos en caché
-----------------------------

Tan pronto como tu sitio web comience a generar más tráfico, tendrás que evitar se genere el mismo recurso una y otra vez. Symfony2 utiliza cabeceras de caché HTTP para administrar los recursos en caché. Para estrategias de memorización en caché simples, utiliza la conveniente anotación ``@Cache()``::

    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Cache;

    /**
     * @Route("/hola/{nombre}", name="_demo_hola")
     * @Template()
     * @Cache(maxage="86400")
     */
    public function holaAction($nombre)
    {
        return array('nombre' => $nombre);
    }

En este ejemplo, el recurso se mantiene en caché por un día. Pero también puedes utilizar validación en lugar de caducidad o una combinación de ambas, si se ajusta mejor a tus necesidades.

El recurso memorizado en caché es gestionado por el sustituto inverso integrado en Symfony2. Pero debido a que la memorización en caché se gestiona usando cabeceras de caché HTTP, puedes reemplazar el sustituto inverso integrado, con Varnish o Squid y escalar tu aplicación fácilmente.

.. note::

    Pero ¿qué pasa si no puedes guardar en caché todas las páginas? Symfony2 todavía tiene la solución vía ESI (Edge Side Includes o Inclusión de borde lateral), con la cual es compatible nativamente. Consigue más información leyendo el capítulo ":doc:`/book/http_cache`" del libro.

Consideraciones finales
-----------------------

Eso es todo lo que hay que hacer, y ni siquiera estoy seguro de que hayan pasado los 10 minutos completos. Presentamos brevemente los paquetes en la primera parte, y todas las características que hemos explorado hasta ahora son parte del paquete básico de la plataforma.
Pero gracias a los paquetes, todo en Symfony2 se puede ampliar o sustituir.
Ese, es el tema de la :doc:`siguiente parte de esta guía <the_architecture>`.