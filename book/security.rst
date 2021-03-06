Seguridad
=========

La seguridad es un proceso de dos etapas, cuyo objetivo es evitar que un usuario acceda a un recurso al cual él/ella no debería tener acceso.

En el primer paso del proceso, el sistema de seguridad identifica quién es el usuario obligándolo a presentar algún tipo de identificación. Esto se llama **autenticación**, y significa que el sistema está tratando de averiguar quién eres.

Una vez que el sistema sabe quien eres, el siguiente paso es determinar si deberías tener acceso a un determinado recurso. Esta parte del proceso se llama **autorización**, y significa que el sistema está comprobando para ver si tienes suficientes privilegios para realizar una determinada acción.

.. image:: /images/book/security_authentication_authorization.png
   :align: center

Puesto que la mejor manera de aprender es viendo un ejemplo, vamos a zambullirnos en este.

.. note::

    El `componente Security`_ de Symfony está disponible como una biblioteca PHP independiente para usarla dentro de cualquier proyecto PHP.

Ejemplo básico: Autenticación HTTP
----------------------------------

Puedes configurar el componente de seguridad a través de la configuración de tu aplicación.
De hecho, la mayoría de los ajustes de seguridad estándar son sólo cuestión de usar la configuración correcta. La siguiente configuración le dice a Symfony que proteja cualquier URL coincidente con ``/admin/*`` y pida al usuario sus credenciales mediante autenticación HTTP básica (es decir, el cuadro de dialogo a la vieja escuela de nombre de usuario/contraseña):

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        security:
            firewalls:
                secured_area:
                    pattern:    ^/
                    anonymous: ~
                    http_basic:
                        realm: "Demo de área protegida"

            access_control:
                - { path: ^/admin, roles: ROLE_ADMIN }

            providers:
                in_memory:
                    users:
                        ryan:  { password: ryanpass, roles: 'ROLE_USER' }
                        admin: { password: kitten, roles: 'ROLE_ADMIN' }

            encoders:
                Symfony\Component\Security\Core\User\User: plaintext

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

            <config>
                <firewall name="secured_area" pattern="^/">
                    <anonymous />
                    <http-basic realm="Demo de área protegida" />
                </firewall>

                <access-control>
                    <rule path="^/admin" role="ROLE_ADMIN" />
                </access-control>

                <provider name="in_memory">
                    <user name="ryan" password="ryanpass" roles="ROLE_USER" />
                    <user name="admin" password="kitten" roles="ROLE_ADMIN" />
                </provider>

                <encoder class="Symfony\Component\Security\Core\User\User" algorithm="plaintext" />
            </config>
        </srv:container>

    .. code-block:: php

        // app/config/config.php
        $contenedor->loadFromExtension('security', array(
            'firewalls' => array(
                'secured_area' => array(
                    'pattern' => '^/',
                    'anonymous' => array(),
                    'http_basic' => array(
                        'realm' => 'Demo de área protegida',
                    ),
                ),
            ),
            'access_control' => array(
                array('path' => '^/admin', 'role' => 'ROLE_ADMIN'),
            ),
            'providers' => array(
                'in_memory' => array(
                    'users' => array(
                        'ryan' => array('password' => 'ryanpass', 'roles' => 'ROLE_USER'),
                        'admin' => array('password' => 'kitten', 'roles' => 'ROLE_ADMIN'),
                    ),
                ),
            ),
            'encoders' => array(
                'Symfony\Component\Security\Core\User\User' => 'plaintext',
            ),
        ));

.. tip::

    Una distribución estándar de Symfony separa la configuración de seguridad en un archivo independiente (por ejemplo, ``app/config/security.yml``). Si no tienes un archivo de seguridad por separado, puedes poner la configuración directamente en el archivo de configuración principal (por ejemplo, ``app/config/config.yml``).

El resultado final de esta configuración es un sistema de seguridad totalmente funcional que tiene el siguiente aspecto:

* Hay dos usuarios en el sistema (``ryan`` y ``admin``);
* Los usuarios se autentican a través de la autenticación HTTP básica del sistema;
* Cualquier URL que coincida con ``/admin/*`` está protegida, y sólo el usuario ``admin`` puede acceder a ella;
* Todas las URL que *no* coincidan con ``/admin/*`` son accesibles por todos los usuarios (y nunca se pide al usuario que se registre).

Veamos brevemente cómo funciona la seguridad y cómo entra en juego cada parte de la configuración.

Cómo funciona la seguridad: autenticación y autorización
--------------------------------------------------------

El sistema de seguridad de Symfony trabaja identificando a un usuario (es decir, la autenticación) y comprobando si ese usuario debe tener acceso a un recurso o URL específico.

Cortafuegos (autenticación)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Cuando un usuario hace una petición a una URL que está protegida por un cortafuegos, se activa el sistema de seguridad. El trabajo del cortafuegos es determinar si el usuario necesita estar autenticado, y si lo hace, enviar una respuesta al usuario para iniciar el proceso de autenticación.

Un cortafuegos se activa cuando la URL de una petición entrante concuerda con el ``patrón`` de la expresión regular configurada en el valor 'config' del cortafuegos. En este ejemplo el ``patrón`` (``^/``) concordará con *cada* petición entrante. El hecho de que el cortafuegos esté activado *no* significa, sin embargo, que el nombre de usuario de autenticación HTTP y el cuadro de diálogo de la contraseña se muestre en cada URL. Por ejemplo, cualquier usuario puede acceder a ``/foo`` sin que se le pida se autentique.

.. image:: /images/book/security_anonymous_user_access.png
   :align: center

Esto funciona en primer lugar porque el cortafuegos permite *usuarios anónimos* a través del parámetro de configuración ``anonymous``. En otras palabras, el cortafuegos no requiere que el usuario se autentique plenamente de inmediato. Y puesto que no hay ``rol`` especial necesario para acceder a ``/foo`` (bajo la sección ``access_control``), la petición se puede llevar a cabo sin solicitar al usuario se autentique.

Si eliminas la clave ``anonymous``, el cortafuegos *siempre* hará que un usuario se autentique inmediatamente.

Control de acceso (autorización)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Si un usuario solicita ``/admin/foo``, sin embargo, el proceso se comporta de manera diferente.
Esto se debe a la sección de configuración ``access_control`` la cual dice que cualquier URL coincidente con el patrón de la expresión regular ``^/admin`` (es decir, ``/admin`` o cualquier cosa coincidente con ``/admin/*``) requiere el rol ``ROLE_ADMIN``. Los roles son la base para la mayor parte de la autorización: el usuario puede acceder a ``/admin/foo`` sólo si cuenta con el rol ``ROLE_ADMIN``.

.. image:: /images/book/security_anonymous_user_denied_authorization.png
   :align: center

Como antes, cuando el usuario hace la petición originalmente, el cortafuegos no solicita ningún tipo de identificación. Sin embargo, tan pronto como la capa de control de acceso niega el acceso a los usuarios (ya que el usuario anónimo no tiene el rol ``ROLE_ADMIN``), el servidor de seguridad entra en acción e inicia el proceso de autenticación).
El proceso de autenticación depende del mecanismo de autenticación que utilice. Por ejemplo, si estás utilizando el método de autenticación con formulario de acceso, el usuario será redirigido a la página de inicio de sesión. Si estás utilizando autenticación HTTP, se enviará al usuario una respuesta HTTP 401 para que el usuario vea el cuadro de diálogo de nombre de usuario y contraseña.

Ahora el usuario de nuevo tiene la posibilidad de presentar sus credenciales a la aplicación.
Si las credenciales son válidas, se puede intentar de nuevo la petición original.

.. image:: /images/book/security_ryan_no_role_admin_access.png
   :align: center

En este ejemplo, el usuario ``ryan`` se autentica correctamente con el cortafuegos.
Pero como ``ryan`` no cuenta con el rol ``ROLE_ADMIN``, se le sigue negando el acceso a ``/admin/foo``. En última instancia, esto significa que el usuario debe ver algún tipo de mensaje indicándole que se le ha denegado el acceso.

.. tip::

    Cuando Symfony niega el acceso a los usuarios, el usuario ve una pantalla de error y recibe un código de estado HTTP 403 (``Prohibido``). Puedes personalizar la pantalla de error, acceso denegado, siguiendo las instrucciones de las :ref:`Páginas de error <cookbook-error-pages-by-status-code>` en la entrada del recetario para personalizar la página de error 403.

Por último, si el usuario ``admin`` solicita ``/admin/foo``, se lleva a cabo un proceso similar, excepto que ahora, después de haberse autenticado, la capa de control de acceso le permitirá pasar a través de la petición:

.. image:: /images/book/security_admin_role_access.png
   :align: center

El flujo de la petición cuando un usuario solicita un recurso protegido es sencillo, pero increíblemente flexible. Como verás más adelante, la autenticación se puede realizar de varias maneras, incluyendo a través de un formulario de acceso, certificados X.509 o la autenticación del usuario a través de *Twitter*. Independientemente del método de autenticación, el flujo de la petición siempre es el mismo:

#. Un usuario accede a un recurso protegido;
#. La aplicación redirige al usuario al formulario de acceso;
#. El usuario presenta sus credenciales (por ejemplo nombre de usuario/contraseña);
#. El cortafuegos autentica al usuario;
#. El nuevo usuario autenticado intenta de nuevo la petición original.

.. note::

    El proceso *exacto* realmente depende un poco en el mecanismo de autenticación utilizado. Por ejemplo, cuando utilizas el formulario de acceso, el usuario presenta sus credenciales a una URL que procesa el formulario (por ejemplo ``/login_check``) y luego es redirigido a la dirección solicitada originalmente (por ejemplo ``/admin/foo``).
    Pero con la autenticación HTTP, el usuario envía sus credenciales directamente a la URL original (por ejemplo ``/admin/foo``) y luego la página se devuelve al usuario en la misma petición (es decir, sin redirección).

    Este tipo de idiosincrasia no debería causar ningún problema, pero es bueno tenerla en cuenta.

.. tip::

    También aprenderás más adelante cómo puedes proteger *cualquier cosa* en Symfony2, incluidos controladores específicos, objetos, e incluso métodos PHP.

.. _book-security-form-login:

Usando un formulario de acceso tradicional
------------------------------------------

Hasta ahora, hemos visto cómo cubrir tu aplicación bajo un cortafuegos y proteger el acceso a determinadas zonas con roles. Al usar la autenticación HTTP, sin esfuerzo puedes aprovechar el cuadro de diálogo nativo nombre de usuario/contraseña que ofrecen todos los navegadores. Sin embargo, fuera de la caja, Symfony permite múltiples mecanismos de autenticación. Para información detallada sobre todos ellos, consulta la :doc:`Referencia en configurando Security </reference/configuration/security>`.

En esta sección, vamos a mejorar este proceso permitiendo la autenticación del usuario a través de un formulario de acceso HTML tradicional.

En primer lugar, activa el formulario de acceso en el cortafuegos:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        security:
            firewalls:
                secured_area:
                    pattern:    ^/
                    anonymous: ~
                    form_login:
                        login_path:  /login
                        check_path:  /login_check

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

            <config>
                <firewall name="secured_area" pattern="^/">
                    <anonymous />
                    <form-login login_path="/login" check_path="/login_check" />
                </firewall>
            </config>
        </srv:container>

    .. code-block:: php

        // app/config/config.php
        $contenedor->loadFromExtension('security', array(
            'firewalls' => array(
                'secured_area' => array(
                    'pattern' => '^/',
                    'anonymous' => array(),
                    'form_login' => array(
                        'login_path' => '/login',
                        'check_path' => '/login_check',
                    ),
                ),
            ),
        ));

.. tip::

    Si no necesitas personalizar tus valores ``login_path`` o ``check_path`` (los valores utilizados aquí son los valores predeterminados), puedes acortar tu configuración:

    .. configuration-block::

        .. code-block:: yaml

            form_login: ~

        .. code-block:: xml

            <form-login />

        .. code-block:: php

            'form_login' => array(),

Ahora, cuando el sistema de seguridad inicia el proceso de autenticación, redirige al usuario al formulario de acceso (``/login`` predeterminado). La implementación visual de este formulario de acceso es tu trabajo. En primer lugar, crea dos rutas: una que muestre el formulario de acceso (es decir, ``/login``) y una que maneje el envío del formulario de acceso (es decir, ``/login_check``):

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        login:
            pattern:   /login
            defaults:  { _controller: AcmeSecurityBundle:Security:login }
        login_check:
            pattern:   /login_check

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="login" pattern="/login">
                <default key="_controller">AcmeSecurityBundle:Security:login</default>
            </route>
            <route id="login_check" pattern="/login_check" />

        </routes>

    ..  code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $coleccion = new RouteCollection();
        $coleccion->add('login', new Route('/login', array(
            '_controller' => 'AcmeDemoBundle:Security:login',
        )));
        $coleccion->add('login_check', new Route('/login_check', array()));

        return $coleccion;

.. note::

    *No* necesitas implementar un controlador para la URL ``/login_check`` ya que el cortafuegos automáticamente captura y procesa cualquier formulario enviado a esta URL. Es opcional, pero útil, crear una ruta para que puedas usarla al generar la URL de envío del formulario en la plantilla de entrada, a continuación.

Observa que el nombre de la ruta ``login`` no es importante. Lo importante es que la URL de la ruta (``/login``) coincida con el valor ``login_path`` configurado, ya que es donde el sistema de seguridad redirige a los usuarios que necesitan acceder.

A continuación, crea el controlador que mostrará el formulario de acceso:

.. code-block:: php

    // src/Acme/SecurityBundle/Controller/Main;
    namespace Acme\SecurityBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Symfony\Component\Security\Core\SecurityContext;

    class SecurityController extends Controller
    {
        public function loginAction()
        {
            $peticion = $this->getRequest();
            $sesion = $peticion->getSession();

            // obtiene el error de inicio de sesión si lo hay
            if ($peticion->attributes->has(SecurityContext::AUTHENTICATION_ERROR)) {
                $error = $peticion->attributes->get(SecurityContext::AUTHENTICATION_ERROR);
            } else {
                $error = $sesion->get(SecurityContext::AUTHENTICATION_ERROR);
            }

            return $this->render('AcmeSecurityBundle:Security:login.html.twig', array(
                // el último nombre de usuario ingresado por el usuario
                'ultimo_nombreusuario' => $sesion->get(SecurityContext::LAST_USERNAME),
                'error'         => $error,
            ));
        }
    }

No dejes que este controlador te confunda. Como veremos en un momento, cuando el usuario envía el formulario, el sistema de seguridad se encarga de automatizar el envío del formulario por ti. Si el usuario ha presentado un nombre de usuario o contraseña no válidos, este controlador lee el error del formulario enviado desde el sistema de seguridad de modo que se pueda mostrar al usuario.

En otras palabras, su trabajo es mostrar el formulario al usuario y los errores de ingreso que puedan haber ocurrido, pero, el propio sistema de seguridad se encarga de verificar el nombre de usuario y contraseña y la autenticación del usuario.

Por último, crea la plantilla correspondiente:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/SecurityBundle/Resources/views/Security/login.html.twig #}
        {% if error %}
            <div>{{ error.message }}</div>
        {% endif %}

        <form action="{{ path('login_check') }}" method="post">
            <label for="nombreusuario">Nombreusuario:</label>
            <input type="text" id="nombreusuario" name="_username" value="{{ last_username }}" />

            <label for="password">Password:</label>
            <input type="password" id="password" name="_password" />

            {#
                Si deseas controlar la URL a la que rediriges al usuario en caso de éxito (más detalles abajo)
                <input type="hidden" name="_target_path" value="/account" />
            #}

            <input type="submit" name="login" />
        </form>

    .. code-block:: html+php

        <?php // src/Acme/SecurityBundle/Resources/views/Security/login.html.php ?>
        <?php if ($error): ?>
            <div><?php echo $error->getMessage() ?></div>
        <?php endif; ?>

        <form action="<?php echo $view['router']->generate('login_check') ?>" method="post">
            <label for="nombreusuario">Nombreusuario:</label>
            <input type="text" id="nombreusuario" name="_username" value="<?php echo $last_username ?>" />

            <label for="password">Password:</label>
            <input type="password" id="password" name="_password" />

            <!--
                Si deseas controlar la URL a la que rediriges al usuario en caso de éxito (más detalles abajo)
                <input type="hidden" name="_target_path" value="/account" />
            -->

            <input type="submit" name="login" />
        </form>

.. tip::

    La variable ``error`` pasada a la plantilla es una instancia de :class:`Symfony\\Component\\Security\\Core\\Exception\\AuthenticationException`.
    Esta puede contener más información - o incluso información confidencial - sobre el fallo de autenticación, ¡por lo tanto utilízalo prudentemente!

El formulario tiene muy pocos requisitos. En primer lugar, presentando el formulario a ``/login_check`` (a ​​través de la ruta ``login_check``), el sistema de seguridad debe interceptar el envío del formulario y procesarlo automáticamente. En segundo lugar, el sistema de seguridad espera que los campos presentados se llamen ``_username`` y ``_password`` (estos nombres de campo se 
pueden :ref:`configurar <reference-security-firewall-form-login>`).

¡Y eso es todo! Cuando envías el formulario, el sistema de seguridad automáticamente comprobará las credenciales del usuario y, o bien autenticará al usuario o enviará al usuario al formulario de acceso donde se puede mostrar el error.

Vamos a revisar todo el proceso:

#. El usuario intenta acceder a un recurso que está protegido;
#. El cortafuegos inicia el proceso de autenticación redirigiendo al usuario al formulario de acceso (``/login``);
#. La página ``/login`` reproduce el formulario de acceso a través de la ruta y el controlador creado en este ejemplo;
#. El usuario envía el formulario de acceso a ``/login_check``;
#. El sistema de seguridad intercepta la petición, comprueba las credenciales presentadas por el usuario, autentica al usuario si todo está correcto, y si no, envía al usuario de nuevo al formulario de acceso.

Por omisión, si las credenciales presentadas son correctas, el usuario será redirigido a la página solicitada originalmente (por ejemplo ``/admin/foo``). Si originalmente el usuario fue directo a la página de inicio de sesión, será redirigido a la página principal.
Esto puede ser altamente personalizado, lo cual te permite, por ejemplo, redirigir al usuario a una URL específica.

Para más detalles sobre esto y cómo personalizar el proceso de entrada en general, consulta :doc:`/cookbook/security/form_login`.

.. _book-security-common-pitfalls:

.. sidebar:: Evitando errores comunes

    Cuando prepares tu formulario de acceso, ten cuidado con unas cuantas trampas muy comunes.

    **1. Crea las rutas correctas**

    En primer lugar, asegúrate de que haz definido las rutas ``/login`` y ``/login_check`` correctamente y que correspondan a los valores de configuración ``login_path`` y ``check_path``. Una mala configuración aquí puede significar que serás redirigido a una página de error 404 en lugar de la página de acceso, o que al presentar el formulario de acceso no haga nada (sólo verás el formulario de acceso una y otra vez).

    **2. Asegúrate de que la página de inicio de sesión no es segura**

    Además, asegúrate de que la página de entrada *no* requiere ningún rol para verla. Por ejemplo, la siguiente configuración - la cual requiere el rol ``ROLE_ADMIN`` para todas las URL (incluyendo la URL ``/login`` ), provocará un bucle de redirección:

    .. configuration-block::

        .. code-block:: yaml

            access_control:
                - { path: ^/, roles: ROLE_ADMIN }

        .. code-block:: xml

            <access-control>
                <rule path="^/" role="ROLE_ADMIN" />
            </access-control>

        .. code-block:: php

            'access_control' => array(
                array('path' => '^/', 'role' => 'ROLE_ADMIN'),
            ),

    Quitar el control de acceso en la URL ``/login`` soluciona el problema:

    .. configuration-block::

        .. code-block:: yaml

            access_control:
                - { path: ^/login, roles: IS_AUTHENTICATED_ANONYMOUSLY }
                - { path: ^/, roles: ROLE_ADMIN }

        .. code-block:: xml

            <access-control>
                <rule path="^/login" role="IS_AUTHENTICATED_ANONYMOUSLY" />
                <rule path="^/" role="ROLE_ADMIN" />
            </access-control>

        .. code-block:: php

            'access_control' => array(
                array('path' => '^/login', 'role' => 'IS_AUTHENTICATED_ANONYMOUSLY'),
                array('path' => '^/', 'role' => 'ROLE_ADMIN'),
            ),

    Además, si el cortafuegos *no* permite usuarios anónimos, necesitas crear un cortafuegos especial que permita usuarios anónimos en la página de acceso:

    .. configuration-block::

        .. code-block:: yaml

            firewalls:
                login_firewall:
                    pattern:    ^/login$
                    anonymous:  ~
                secured_area:
                    pattern:    ^/
                    form_login: ~

        .. code-block:: xml

            <firewall name="login_firewall" pattern="^/login$">
                <anonymous />
            </firewall>
            <firewall name="secured_area" pattern="^/">
                <form_login />
            </firewall>

        .. code-block:: php

            'firewalls' => array(
                'login_firewall' => array(
                    'pattern' => '^/login$',
                    'anonymous' => array(),
                ),
                'secured_area' => array(
                    'pattern' => '^/',
                    'form_login' => array(),
                ),
            ),

    **3. Asegúrate de que** ``/login_check`` **está detrás de un cortafuegos**

    A continuación, asegúrate de que tu ruta URL ``check_path`` (por ejemplo ``/login_check``) está detrás del cortafuegos que estás utilizando para tu formulario de acceso (en este ejemplo, el único cortafuegos coincide con *todas* las URL, incluyendo ``/login_check``). Si ``/login_check`` no coincide con ningún cortafuegos, recibirás una excepción ``No se puede encontrar el controlador para la ruta "/login_check"``.

    **4. Múltiples cortafuegos no comparten el contexto de seguridad**

    Si estás utilizando múltiples cortafuegos y autenticas contra un cortafuegos, *no* serás autenticado contra ningún otro cortafuegos automáticamente.
    Diferentes cortafuegos, son como diferentes sistemas de seguridad. Es por eso que, para la mayoría de las aplicaciones, tener un cortafuegos principal es suficiente.

Autorizando
-----------

El primer paso en la seguridad siempre es la autenticación: el proceso de verificar quién es el usuario. Con Symfony, la autenticación se puede hacer de cualquier manera - a través de un formulario de acceso, autenticación básica HTTP, e incluso a través de Facebook.

Una vez que el usuario se ha autenticado, comienza la autorización. La autorización proporciona una forma estándar y potente para decidir si un usuario puede acceder a algún recurso (una URL, un modelo de objetos, una llamada a un método, ...). Esto funciona asignando roles específicos a cada usuario y, a continuación, requiriendo diferentes roles para diferentes recursos.

El proceso de autorización tiene dos lados diferentes:

#. El usuario tiene un conjunto de roles específico;
#. Un recurso requiere un rol específico a fin de tener acceso.

En esta sección, nos centraremos en cómo proteger diferentes recursos (por ejemplo, direcciones URL, llamadas a métodos, etc.) con diferentes roles. Más tarde, aprenderás más sobre cómo crear y asignar roles a los usuarios.

Protegiendo patrones de URL específicas
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

La forma más básica para proteger parte de tu aplicación es asegurar un patrón de direcciones URL completo. Ya haz visto en el primer ejemplo de este capítulo, donde algo que coincide con el patrón de la expresión regular ``^/admin`` requiere el rol ``ROLE_ADMIN``.

Puedes definir tantos patrones URL como necesites - cada uno es una expresión regular.

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        security:
            # ...
            access_control:
                - { path: ^/admin/users, roles: ROLE_SUPER_ADMIN }
                - { path: ^/admin, roles: ROLE_ADMIN }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <config>
            <!-- ... -->
            <access-control>
                <rule path="^/admin/users" role="ROLE_SUPER_ADMIN" />
                <rule path="^/admin" role="ROLE_ADMIN" />
            </access-control>
        </config>

    .. code-block:: php

        // app/config/config.php
        $contenedor->loadFromExtension('security', array(
            // ...
            'access_control' => array(
                array('path' => '^/admin/users', 'role' => 'ROLE_SUPER_ADMIN'),
                array('path' => '^/admin', 'role' => 'ROLE_ADMIN'),
            ),
        ));

.. tip::

    Al prefijar la ruta con ``^`` te aseguras que sólo coinciden las URL que *comienzan* con ese patrón. Por ejemplo, una ruta de simplemente ``/admin`` coincidirá con ``/admin/foo``, pero también con ``/foo/admin``.

Para cada petición entrante, Symfony2 trata de encontrar una regla de control de acceso coincidente (la primera gana). Si el usuario no está autenticado, sin embargo, se inicia el proceso de autenticación (es decir, se le da al usuario una oportunidad de ingresar). Sin embargo, si el usuario *está* autenticado, pero no tiene el rol necesario, se lanza una excepción :class:`Symfony\\Component\Security\\Core\\Exception\\AccessDeniedException`, que puedes manejar y convertir en una bonita página de error "acceso denegado" para el usuario. Consulta :doc:`/cookbook/controller/error_pages` para más información.

Debido a que Symfony utiliza la primera regla de control de acceso coincidente, una URL como ``/admin/usuarios/nuevo`` coincidirá con la primera regla y sólo requiere el rol ``ROLE_SUPER_ADMIN``.
Cualquier URL como ``/admin/blog`` coincidirá con la segunda regla y requiere un ``ROLE_ADMIN``.

También puedes forzar ``HTTP`` o ``HTTPS`` a través de una entrada ``access_control``.
Para más información, consulta :doc:`/cookbook/security/force_https`.

.. _book-security-securing-controller:

Protegiendo un controlador
~~~~~~~~~~~~~~~~~~~~~~~~~~

Proteger tu aplicación basándote en los patrones URL es fácil, pero, en algunos casos, puede no estar suficientemente bien ajustado. Cuando sea necesario, fácilmente puedes forzar la autorización al interior de un controlador:

.. code-block:: php

    use Symfony\Component\Security\Core\Exception\AccessDeniedException
    // ...

    public function holaAction($nombre)
    {
        if (false === $this->get('security.context')->isGranted('ROLE_ADMIN')) {
            throw new AccessDeniedException();
        }

        // ...
    }

.. _book-security-securing-controller-annotations:

También puedes optar por instalar y utilizar el opcional ``JMSSecurityExtraBundle``, el cual puede asegurar tu controlador usando anotaciones:

.. code-block:: php

    use JMS\SecurityExtraBundle\Annotation\Secure;

    /**
     * @Secure(roles="ROLE_ADMIN")
     */
    public function holaAction($nombre)
    {
        // ...
    }

Para más información, consulta la documentación de `JMSSecurityExtraBundle`_. Si estás usando la distribución estándar de Symfony, este paquete está disponible de forma predeterminada.
Si no es así, lo puedes descargar e instalar.

Protegiendo otros servicios
~~~~~~~~~~~~~~~~~~~~~~~~~~~

De hecho, en Symfony puedes proteger cualquier cosa utilizando una estrategia similar a la observada en la sección anterior. Por ejemplo, supongamos que tienes un servicio (es decir, una clase PHP), cuyo trabajo consiste en enviar mensajes de correo electrónico de un usuario a otro.
Puedes restringir el uso de esta clase - no importa dónde se esté utilizando - a los usuarios que tienen un rol específico.

Para más información sobre cómo utilizar el componente de seguridad para proteger diferentes servicios y métodos en tu aplicación, consulta :doc:`/cookbook/security/securing_services`.

Listas de control de acceso (ACL): Protegiendo objetos individuales de base de datos
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Imagina que estás diseñando un sistema de blog donde los usuarios pueden comentar tus mensajes. Ahora, deseas que un usuario pueda editar sus propios comentarios, pero no los de otros usuarios. Además, como usuario ``admin``, quieres tener la posibilidad de editar *todos* los comentarios.

El componente de seguridad viene con un sistema opcional de lista de control de acceso (ACL) que puedes utilizar cuando sea necesario para controlar el acceso a instancias individuales de un objeto en el sistema. *Sin* ACL, puedes proteger tu sistema para que sólo determinados usuarios puedan editar los comentarios del blog en general. Pero *con* ACL, puedes restringir o permitir el acceso en base a comentario por comentario.

Para más información, consulta el artículo del recetario: :doc:`/book/security/acl`.

Usuarios
--------

En las secciones anteriores, aprendiste cómo puedes proteger diferentes recursos que requieren un conjunto de *roles* para un recurso. En esta sección vamos a explorar el otro lado de la autorización: los usuarios.

¿De dónde provienen los usuarios? (*Proveedores de usuarios*)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Durante la autenticación, el usuario envía un conjunto de credenciales (por lo general un nombre de usuario y contraseña). El trabajo del sistema de autenticación es concordar esas credenciales contra una piscina de usuarios. Entonces, ¿de dónde viene esta lista de usuarios?

En Symfony2, los usuarios pueden venir de cualquier parte - un archivo de configuración, una tabla de base de datos, un servicio web, o cualquier otra cosa que se te ocurra. Todo lo que proporcione uno o más usuarios al sistema de autenticación se conoce como "proveedor de usuario".
Symfony2 de serie viene con los dos proveedores de usuario más comunes: uno que carga los usuarios de un archivo de configuración y otro que carga usuarios de una tabla de base de datos.

Especificando usuarios en un archivo de configuración
.....................................................

La forma más fácil para especificar usuarios es directamente en un archivo de configuración.
De hecho, ya lo haz visto en algunos ejemplos de este capítulo.

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        security:
            # ...
            providers:
                default_provider:
                    users:
                        ryan:  { password: ryanpass, roles: 'ROLE_USER' }
                        admin: { password: kitten, roles: 'ROLE_ADMIN' }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <config>
            <!-- ... -->
            <provider name="default_provider">
                <user name="ryan" password="ryanpass" roles="ROLE_USER" />
                <user name="admin" password="kitten" roles="ROLE_ADMIN" />
            </provider>
        </config>

    .. code-block:: php

        // app/config/config.php
        $contenedor->loadFromExtension('security', array(
            // ...
            'providers' => array(
                'default_provider' => array(
                    'users' => array(
                        'ryan' => array('password' => 'ryanpass', 'roles' => 'ROLE_USER'),
                        'admin' => array('password' => 'kitten', 'roles' => 'ROLE_ADMIN'),
                    ),
                ),
            ),
        ));

Este proveedor de usuario se denomina proveedor de usuario "en memoria", ya que los usuarios no se almacenan en alguna parte de una base de datos. El objeto usuario en realidad es provisto por Symfony (:class:`Symfony\\Component\\Security\\Core\\User\\User`).

.. tip::
    Cualquier proveedor de usuario puede cargar usuarios directamente desde la configuración especificando el parámetro de configuración ``users`` y la lista de usuarios debajo de él.

.. caution::

    Si tu nombre de usuario es completamente numérico (por ejemplo, ``77``) o contiene un guión (por ejemplo, ``nombre-usuario``), debes utilizar la sintaxis alterna al especificar usuarios en YAML:

    .. code-block:: yaml

        users:
            - { name: 77, password: pass, roles: 'ROLE_USER' }
            - { name: user-name, password: pass, roles: 'ROLE_USER' }

Para sitios pequeños, este método es rápido y fácil de configurar. Para sistemas más complejos, querrás cargar usuarios desde la base de datos.

.. _book-security-user-entity:

Cargando usuarios de la base de datos
.....................................

Si deseas cargar tus usuarios a través del ORM de Doctrine, lo puedes hacer creando una clase ``Usuario`` y configurando el proveedor ``entity``.

.. tip:

    Hay disponible un paquete de código abierto de alta calidad, el cual permite a los almacenar a los usuarios a través del ORM u ODM de Doctrine. Lee más acerca del `FOSUserBundle`_ en GitHub.

Con este enfoque, primero crea tu propia clase ``User``, la cual se almacenará en la base de datos.

.. code-block:: php

    // src/Acme/UserBundle/Entity/Usuario.php
    namespace Acme\UserBundle\Entity;

    use Symfony\Component\Security\Core\User\UserInterface;
    use Doctrine\ORM\Mapping as ORM;

    /**
     * @ORM\Entity
     */
    class Usuario implements UserInterface
    {
        /**
         * @ORM\Column(type="string", length="255")
         */
        protected $nombreusuario;

        // ...
    }

En cuanto al sistema de seguridad se refiere, el único requisito para tu clase Usuario personalizada es que implemente la interfaz :class:`Symfony\\Component\\Security\\Core\\User\\UserInterface`. Esto significa que el concepto de un "usuario" puede ser cualquier cosa, siempre y cuando implemente esta interfaz.

.. note::

    El objeto de usuario se debe serializar y guardar en la sesión durante las peticiones, por lo tanto se recomienda que `implementes la interfaz \Serializable`_ en tu objeto usuario. Esto es especialmente importante si tu clase ``Usuario`` tiene una clase padre con propiedades privadas.

A continuación, configura una ``entidad`` proveedora de usuario, y apuntala a tu clase ``Usuario``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            providers:
                main:
                    entity: { class: Acme\UserBundle\Entity\User, property: nombreusuario }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <provider name="main">
                <entity class="Acme\UserBundle\Entity\User" property="nombreusuario" />
            </provider>
        </config>

    .. code-block:: php

        // app/config/security.php
        $contenedor->loadFromExtension('security', array(
            'providers' => array(
                'main' => array(
                    'entity' => array('class' => 'Acme\UserBundle\Entity\User', 'property' => 'nombreusuario'),
                ),
            ),
        ));

Con la introducción de este nuevo proveedor, el sistema de autenticación intenta cargar un objeto ``Usuario`` de la base de datos utilizando el campo ``nombreusuario`` de esa clase.

.. note:
    Este ejemplo sólo intenta mostrar la idea básica detrás del proveedor de la ``entidad``. Para obtener un ejemplo completo trabajando, consulta :doc:`/cookbook/security/entity_provider`.

Para más información sobre cómo crear tu propio proveedor personalizado (por ejemplo, si es necesario cargar los usuarios a través de un servicio Web), consulta :doc:`/cookbook/security/custom_provider`.

Codificando la contraseña del usuario
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hasta ahora, por simplicidad, todos los ejemplos tienen las contraseñas de los usuarios almacenadas en texto plano (si los usuarios se almacenan en un archivo de configuración o en alguna base de datos). Por supuesto, en una aplicación real, por razones de seguridad, desearás codificar las contraseñas de los usuarios. Esto se logra fácilmente asignando la clase Usuario a una de las varias integradas en ``encoders``. Por ejemplo, para almacenar los usuarios en memoria, pero ocultar sus contraseñas a través de ``sha1``, haz lo siguiente:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        security:
            # ...
            providers:
                in_memory:
                    users:
                        ryan:  { password: bb87a29949f3a1ee0559f8a57357487151281386, roles: 'ROLE_USER' }
                        admin: { password: 74913f5cd5f61ec0bcfdb775414c2fb3d161b620, roles: 'ROLE_ADMIN' }

            encoders:
                Symfony\Component\Security\Core\User\User:
                    algorithm:   sha1
                    iterations: 1
                    encode_as_base64: false

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <config>
            <!-- ... -->
            <provider name="in_memory">
                <user name="ryan" password="bb87a29949f3a1ee0559f8a57357487151281386" roles="ROLE_USER" />
                <user name="admin" password="74913f5cd5f61ec0bcfdb775414c2fb3d161b620" roles="ROLE_ADMIN" />
            </provider>

            <encoder class="Symfony\Component\Security\Core\User\User" algorithm="sha1" iterations="1" encode_as_base64="false" />
        </config>

    .. code-block:: php

        // app/config/config.php
        $contenedor->loadFromExtension('security', array(
            // ...
            'providers' => array(
                'in_memory' => array(
                    'users' => array(
                        'ryan' => array('password' => 'bb87a29949f3a1ee0559f8a57357487151281386', 'roles' => 'ROLE_USER'),
                        'admin' => array('password' => '74913f5cd5f61ec0bcfdb775414c2fb3d161b620', 'roles' => 'ROLE_ADMIN'),
                    ),
                ),
            ),
            'encoders' => array(
                'Symfony\Component\Security\Core\User\User' => array(
                    'algorithm'         => 'sha1',
                    'iterations'        => 1,
                    'encode_as_base64'  => false,
                ),
            ),
        ));

Al establecer las ``iterations`` a ``1`` y ``encode_as_base64`` en ``false``, la contraseña simplemente se corre una vez a través del algoritmo ``sha1`` y sin ninguna codificación adicional. Ahora puedes calcular el |hash| de la contraseña mediante programación (por ejemplo, ``hash('sha1', 'ryanpass')``) o a través de alguna herramienta en línea como `functions-online.com`_

Si vas a crear dinámicamente a tus usuarios (y almacenarlos en una base de datos), puedes utilizar algoritmos hash aún más difíciles y, luego confiar en un objeto codificador de clave real para ayudarte a codificar las contraseñas. Por ejemplo, supongamos que tu objeto usuario es ``Acme\UserBundle\Entity\User`` (como en el ejemplo anterior). Primero, configura el codificador para ese usuario:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        security:
            # ...

            encoders:
                Acme\UserBundle\Entity\User: sha512

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <config>
            <!-- ... -->

            <encoder class="Acme\UserBundle\Entity\User" algorithm="sha512" />
        </config>

    .. code-block:: php

        // app/config/config.php
        $contenedor->loadFromExtension('security', array(
            // ...

            'encoders' => array(
                'Acme\UserBundle\Entity\User' => 'sha512',
            ),
        ));

En este caso, estás utilizando el fuerte algoritmo ``SHA512``. Además, puesto que hemos especificado simplemente el algoritmo (``sha512``) como una cadena, el sistema de manera predeterminada revuelve tu contraseña 5000 veces en una fila y luego la codifica como base64. En otras palabras, la contraseña ha sido fuertemente ofuscada por lo tanto la contraseña revuelta no se puede decodificar (es decir, no se puede determinar la contraseña desde la contraseña ofuscada).

Si tienes algún formulario de registro para los usuarios, tendrás que poder determinar la contraseña con el algoritmo hash, para que puedas ponerla en tu usuario. No importa qué algoritmo configures para el objeto usuario, la contraseña con algoritmo hash siempre la puedes determinar de la siguiente manera desde un controlador:

.. code-block:: php

    $factory = $this->get('security.encoder_factory');
    $user = new Acme\UserBundle\Entity\User();

    $encoder = $factory->getEncoder($user);
    $pase = $encoder->encodePassword('ryanpass', $user->getSalt());
    $user->setPassword($pase);

Recuperando el objeto usuario
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Después de la autenticación, el objeto ``Usuario`` del usuario actual se puede acceder a través del servicio ``security.context``. Desde el interior de un controlador, este se verá así:

.. code-block:: php

    public function indexAction()
    {
        $user = $this->get('security.context')->getToken()->getUser();
    }

.. note::

    Los usuarios anónimos técnicamente están autenticados, lo cual significa que el método ``isAuthenticated()`` de un objeto usuario anónimo devolverá ``true``. Para comprobar si el usuario está autenticado realmente, verifica el rol ``IS_AUTHENTICATED_FULLY``.

Usando múltiples proveedores de usuario
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Cada mecanismo de autenticación (por ejemplo, la autenticación HTTP, formulario de acceso, etc.) utiliza exactamente un proveedor de usuario, y de forma predeterminada utilizará el primer proveedor de usuario declarado. Pero, si deseas especificar unos cuantos usuarios a través de la configuración y el resto de los usuarios en la base de datos? Esto es posible creando un nuevo proveedor que encadene los dos:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            providers:
                chain_provider:
                    providers: [in_memory, user_db]
                in_memory:
                    users:
                        foo: { password: test }
                user_db:
                    entity: { class: Acme\UserBundle\Entity\User, property: nombreusuario }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <config>
            <provider name="chain_provider">
                <provider>in_memory</provider>
                <provider>user_db</provider>
            </provider>
            <provider name="in_memory">
                <user name="foo" password="test" />
            </provider>
            <provider name="user_db">
                <entity class="Acme\UserBundle\Entity\User" property="nombreusuario" />
            </provider>
        </config>

    .. code-block:: php

        // app/config/config.php
        $contenedor->loadFromExtension('security', array(
            'providers' => array(
                'chain_provider' => array(
                    'providers' => array('in_memory', 'user_db'),
                ),
                'in_memory' => array(
                    'users' => array(
                        'foo' => array('password' => 'test'),
                    ),
                ),
                'user_db' => array(
                    'entity' => array('class' => 'Acme\UserBundle\Entity\User', 'property' => 'nombreusuario'),
                ),
            ),
        ));

Ahora, todos los mecanismos de autenticación utilizan el ``chain_provider``, puesto que es el primero especificado. El ``chain_provider``, a su vez, intenta cargar el usuario, tanto el proveedor ``in_memory`` cómo ``USER_DB``.

.. tip::

    Si no tienes razones para separar a tus usuarios ``in_memory`` de tus usuarios ``user_db``, lo puedes hacer aún más fácil combinando las dos fuentes en un único proveedor:

    .. configuration-block::

        .. code-block:: yaml

            # app/config/security.yml
            security:
                providers:
                    main_provider:
                        users:
                            foo: { password: test }
                        entity: { class: Acme\UserBundle\Entity\User, property: nombreusuario }

        .. code-block:: xml

            <!-- app/config/config.xml -->
            <config>
                <provider name=="main_provider">
                    <user name="foo" password="test" />
                    <entity class="Acme\UserBundle\Entity\User" property="nombreusuario" />
                </provider>
            </config>

        .. code-block:: php

            // app/config/config.php
            $contenedor->loadFromExtension('security', array(
                'providers' => array(
                    'main_provider' => array(
                        'users' => array(
                            'foo' => array('password' => 'test'),
                        ),
                        'entity' => array('class' => 'Acme\UserBundle\Entity\User', 'property' => 'nombreusuario'),
                    ),
                ),
            ));

También puedes configurar el cortafuegos o mecanismos de autenticación individuales para utilizar un proveedor específico. Una vez más, a menos que especifiques un proveedor explícitamente, siempre se utiliza el primer proveedor:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        security:
            firewalls:
                secured_area:
                    # ...
                    provider: user_db
                    http_basic:
                        realm: "Demo de área protegida"
                        provider: in_memory
                    form_login: ~

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <config>
            <firewall name="secured_area" pattern="^/" provider="user_db">
                <!-- ... -->
                <http-basic realm="Demo de área protegida" provider="in_memory" />
                <form-login />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/config.php
        $contenedor->loadFromExtension('security', array(
            'firewalls' => array(
                'secured_area' => array(
                    // ...
                    'provider' => 'user_db',
                    'http_basic' => array(
                        // ...
                        'provider' => 'in_memory',
                    ),
                    'form_login' => array(),
                ),
            ),
        ));

En este ejemplo, si un usuario intenta acceder a través de autenticación HTTP, el sistema de autenticación debe utilizar el proveedor de usuario ``in_memory``. Pero si el usuario intenta acceder a través del formulario de acceso, se utilizará el proveedor  ``USER_DB`` (ya que es el valor predeterminado para el servidor de seguridad en su conjunto).

Para más información acerca de los proveedores de usuario y la configuración del cortafuegos, consulta la :doc:`/reference/configuration/security`.

Roles
-----

La idea de un "rol" es clave para el proceso de autorización. Cada usuario tiene asignado un conjunto de roles y cada recurso requiere uno o más roles. Si el usuario tiene los roles necesarios, se le concede acceso. En caso contrario se deniega el acceso.

Los roles son bastante simples, y básicamente son cadenas que puedes inventar y utilizar cuando sea necesario (aunque los roles son objetos internos). Por ejemplo, si necesitas comenzar a limitar el acceso a la sección admin del blog de tu sitio web, puedes proteger esa sección con un rol llamado ``ROLE_BLOG_ADMIN``. Este rol no necesita estar definido en ningún lugar - puedes comenzar a usarlo.

.. note::

    Todos los roles **deben** comenzar con el prefijo ``ROLE_`` el cual será gestionado por Symfony2. Si defines tu propios roles con una clase ``Rol`` dedicada (más avanzada), no utilices el prefijo ``ROLE_``.

Roles jerárquicos
~~~~~~~~~~~~~~~~~

En lugar de asociar muchos roles a los usuarios, puedes definir reglas de herencia creando una jerarquía de roles:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            role_hierarchy:
                ROLE_ADMIN:       ROLE_USER
                ROLE_SUPER_ADMIN: [ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH]

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <role-hierarchy>
                <role id="ROLE_ADMIN">ROLE_USER</role>
                <role id="ROLE_SUPER_ADMIN">ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH</role>
            </role-hierarchy>
        </config>

    .. code-block:: php

        // app/config/security.php
        $contenedor->loadFromExtension('security', array(
            'role_hierarchy' => array(
                'ROLE_ADMIN'       => 'ROLE_USER',
                'ROLE_SUPER_ADMIN' => array('ROLE_ADMIN', 'ROLE_ALLOWED_TO_SWITCH'),
            ),
        ));

En la configuración anterior, los usuarios con rol ``ROLE_ADMIN`` también tendrán el rol de ``ROLE_USER``. El rol ``ROLE_SUPER_ADMIN`` tiene ``ROLE_ADMIN``, ``ROLE_ALLOWED_TO_SWITCH`` y ``ROLE_USER`` (heredado de ``ROLE_ADMIN``).

Cerrando sesión
---------------

Por lo general, también quieres que tus usuarios puedan salir. Afortunadamente, el cortafuegos puede manejar esto automáticamente cuando activas el parámetro de configuración ``logout``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        security:
            firewalls:
                secured_area:
                    # ...
                    logout:
                        path:   /logout
                        target: /
            # ...

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <config>
            <firewall name="secured_area" pattern="^/">
                <!-- ... -->
                <logout path="/logout" target="/" />
            </firewall>
            <!-- ... -->
        </config>

    .. code-block:: php

        // app/config/config.php
        $contenedor->loadFromExtension('security', array(
            'firewalls' => array(
                'secured_area' => array(
                    // ...
                    'logout' => array('path' => 'logout', 'target' => '/'),
                ),
            ),
            // ...
        ));

Una vez que este está configurado en tu cortafuegos, enviar a un usuario a ``/logout`` (o cualquiera que sea tu ``path`` configurada), debes desautenticar al usuario actual. El usuario será enviado a la página de inicio (el valor definido por el parámetro ``target``). Ambos parámetros ``path`` y ``target`` por omisión se configuran a lo que esté especificado aquí. En otras palabras, a menos que necesites personalizarlos, los puedes omitir por completo y acortar tu configuración:

.. configuration-block::

    .. code-block:: yaml

        logout: ~

    .. code-block:: xml

        <logout />

    .. code-block:: php

        'logout' => array(),

Ten en cuenta que *no* es necesario implementar un controlador para la URL ``/logout`` porque el cortafuegos se encarga de todo. Puedes, sin embargo, desear crear una ruta para que puedas utilizarla para generar la URL:

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        logout:
            pattern:   /logout

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="logout" pattern="/logout" />

        </routes>

    ..  code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $coleccion = new RouteCollection();
        $coleccion->add('logout', new Route('/logout', array()));

        return $coleccion;

Una vez que el usuario ha cerrado la sesión, será redirigido a cualquier ruta definida por el parámetro ``target`` anterior (por ejemplo, la página ``principal``). Para más información sobre cómo configurar el cierre de sesión, consulta :doc:`Referencia en configurando Security </reference/configuration/security>`.

Controlando el acceso en plantillas
-----------------------------------

Si deseas comprobar si el usuario actual tiene un rol dentro de una plantilla, utiliza la función ayudante incorporada:

.. configuration-block::

    .. code-block:: html+jinja

        {% if is_granted('ROLE_ADMIN') %}
            <a href="...">Delete</a>
        {% endif %}

    .. code-block:: html+php

        <?php if ($view['security']->isGranted('ROLE_ADMIN')): ?>
            <a href="...">Delete</a>
        <?php endif; ?>

.. note::

    Si utilizas esta función y *no* estás en una URL donde haya un cortafuegos activo, se lanzará una excepción. Una vez más, casi siempre es buena idea tener un cortafuegos principal que cubra todas las URL (como hemos mostrado en este capítulo).

Controlando el acceso en controladores
--------------------------------------

Si deseas comprobar en tu controlador si el usuario actual tiene un rol, utiliza el método ``isGranted`` del contexto de seguridad:

.. code-block:: php

    public function indexAction()
    {
        // muestra diferente contenido para los usuarios admin
        if($this->get('security.context')->isGranted('ADMIN')) {
            // carga el contenido admin aquí
        }
        // carga el contenido regular aquí
    }

.. note::

    Un cortafuegos debe estar activo o cuando se llame al método ``isGranted`` se producirá una excepción. Ve la nota anterior acerca de las plantillas para más detalles.

Suplantando a un usuario
------------------------

A veces, es útil poder cambiar de un usuario a otro sin tener que iniciar sesión de nuevo (por ejemplo, cuando depuras o tratas de entender un error que un usuario ve y que no se puede reproducir). Esto se puede hacer fácilmente activando el escucha ``switch_user`` del cortafuegos:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                main:
                    # ...
                    switch_user: true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <!-- ... -->
                <switch-user />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $contenedor->loadFromExtension('security', array(
            'firewalls' => array(
                'main'=> array(
                    // ...
                    'switch_user' => true
                ),
            ),
        ));

Para cambiar a otro usuario, sólo tienes que añadir una cadena de consulta con el parámetro ``_switch_user`` y el nombre de usuario como el valor de la dirección actual::

    http://ejemplo.com/somewhere?_switch_user=thomas

Para volver al usuario original, utiliza el nombre de usuario especial ``_exit``::

    http://ejemplo.com/somewhere?_switch_user=_exit

Por supuesto, esta función se debe poner a disposición de un pequeño grupo de usuarios.
De forma predeterminada, el acceso está restringido a usuarios que tienen el rol ``ROLE_ALLOWED_TO_SWITCH``. El nombre de esta función se puede modificar a través de la configuración ``role``. Para mayor seguridad, también puedes cambiar el nombre del parámetro de consulta a través de la configuración ``parameter``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                main:
                    // ...
                    switch_user: { role: ROLE_ADMIN, parameter: _want_to_be_this_user }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <!-- ... -->
                <switch-user role="ROLE_ADMIN" parameter="_want_to_be_this_user" />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $contenedor->loadFromExtension('security', array(
            'firewalls' => array(
                'main'=> array(
                    // ...
                    'switch_user' => array('role' => 'ROLE_ADMIN', 'parameter' => '_want_to_be_this_user'),
                ),
            ),
        ));

Autenticación apátrida
----------------------

De forma predeterminada, Symfony2 confía en una ``cookie`` (la Sesión) para persistir el contexto de seguridad del usuario. Pero si utilizas certificados o autenticación HTTP, por ejemplo, la persistencia no es necesaria ya que están disponibles las credenciales para cada petición. En ese caso, y si no es necesario almacenar cualquier otra cosa entre peticiones, puedes activar la autenticación apátrida (lo cual significa que Symfony2 jamás creará una ``cookie``):

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                main:
                    http_basic: ~
                    stateless:  true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall stateless="true">
                <http-basic />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $contenedor->loadFromExtension('security', array(
            'firewalls' => array(
                'main' => array('http_basic' => array(), 'stateless' => true),
            ),
        ));

.. note::

    Si utilizas un formulario de acceso, Symfony2 creará una ``cookie``, incluso si estableces ``stateless`` a ``true``.

Palabras finales
----------------

La seguridad puede ser un tema profundo y complejo de resolver correctamente en tu aplicación.
Afortunadamente, el componente de seguridad de Symfony sigue un modelo de seguridad bien probado en torno a la *autenticación* y *autorización*. Autenticación, siempre sucede en primer lugar, está a cargo de un cortafuegos, cuyo trabajo es determinar la identidad del usuario a través de varios métodos diferentes (por ejemplo, la autenticación HTTP, formulario de acceso, etc.) En el recetario, encontrarás ejemplos de otros métodos para manejar la autenticación, incluyendo la manera de implementar una funcionalidad "recuérdame" por medio de ``cookie``.

Una vez que un usuario se autentica, la capa de autorización puede determinar si el usuario debe tener acceso a un recurso específico. Por lo general, los *roles* se aplican a las direcciones URL, clases o métodos y si el usuario actual no tiene ese rol, se le niega el acceso. La capa de autorización, sin embargo, es mucho más profunda, y sigue un sistema de "voto" para que varias partes puedan determinar si el usuario actual debe tener acceso a un determinado recurso.
Para saber más sobre este y otros temas busca en el recetario.

Aprende más en el recetario
---------------------------

* :doc:`Forzando HTTP/HTTPS </cookbook/security/force_https>`
* :doc:`Lista negra de usuarios por dirección IP con votante personalizado </cookbook/security/voters>`
* :doc:`Listas de control de acceso (ACLs) </cookbook/security/acl>`
* :doc:`/cookbook/security/remember_me`

.. _`componente Security`: https://github.com/symfony/Security
.. _`JMSSecurityExtraBundle`: https://github.com/schmittjoh/JMSSecurityExtraBundle
.. _`FOSUserBundle`: https://github.com/FriendsOfSymfony/FOSUserBundle
.. _`implementes la interfaz \Serializable`: http://www.php.net/manual/es/class.serializable.php
.. _`functions-online.com`: http://www.functions-online.com/sha1.html
..  |hash| replace:: *hash*