Cómo personalizar el formulario de acceso
=========================================

Usar un :ref:`formulario de acceso <book-security-form-login>` para autenticación es
un método común y flexible para gestionar la autenticación en Symfony2. Casi todos los aspectos del formulario de acceso se pueden personalizar. La configuración predeterminada completa se muestra en la siguiente sección.

Referencia de configuración del formulario de acceso
----------------------------------------------------

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                main:
                    form_login:
                        # el usuario es redirigido aquí cuando él/ella necesita ingresar
                        login_path:                     /login

                        # si es ``true``, reenvía al usuario al formulario de acceso en lugar de redirigirlo
                        use_forward:                    false

                        # presenta el formulario de acceso aquí
                        check_path:                     /login_check

                        # por omisión, el formulario de acceso DEBE ser POST, no GET
                        post_only:                      true

                        # opciones para redirigir en ingreso satisfactorio (lee más adelante)
                        always_use_default_target_path: false
                        default_target_path:            /
                        target_path_parameter:          _target_path
                        use_referer:                    false

                        # opciones para redirigir en ingreso fallido (lee más adelante)
                        failure_path:                   null
                        failure_forward:                false

                        # nombres de campo para el nombre de usuario y contraseña
                        username_parameter:             _username
                        password_parameter:             _password

                        # opciones del elemento csrf
                        csrf_parameter:                 _csrf_token
                        intention:                      authenticate

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <form-login
                    check_path="/login_check"
                    login_path="/login"
                    use_forward="false"
                    always_use_default_target_path="false"
                    default_target_path="/"
                    target_path_parameter="_target_path"
                    use_referer="false"
                    failure_path="null"
                    failure_forward="false"
                    username_parameter="_username"
                    password_parameter="_password"
                    csrf_parameter="_csrf_token"
                    intention="authenticate"
                    post_only="true"
                />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $contenedor->loadFromExtension('security', array(
            'firewalls' => array(
                'main' => array('form_login' => array(
                    'check_path'                     => '/login_check',
                    'login_path'                     => '/login',
                    'user_forward'                   => false,
                    'always_use_default_target_path' => false,
                    'default_target_path'            => '/',
                    'target_path_parameter'          => _target_path,
                    'use_referer'                    => false,
                    'failure_path'                   => null,
                    'failure_forward'                => false,
                    'username_parameter'             => '_username',
                    'password_parameter'             => '_password',
                    'csrf_parameter'                 => '_csrf_token',
                    'intention'                      => 'authenticate',
                    'post_only'                      => true,
                )),
            ),
        ));

Redirigiendo después del ingreso
--------------------------------

Puedes cambiar el lugar al cual el formulario de acceso redirige después de un inicio de sesión satisfactorio utilizando diferentes opciones de configuración. Por omisión, el formulario redirigirá a la URL que el usuario solicitó (por ejemplo, la URL que provocó la exhibición del formulario de acceso).
Por ejemplo, si el usuario solicitó ``http://www.ejemplo.com/admin/post/18/editar`` entonces después de que él/ella haya superado el inicio de sesión, finalmente será devuelto a ``http://www.ejemplo.com/admin/post/18/editar``. Esto se consigue almacenando la URL solicitada en la sesión. Si no hay presente una URL en la sesión (tal vez el usuario se dirigió directamente a la página de inicio de sesión), entonces el usuario es redirigido a la página predeterminada, la cual es ``/`` (es decir, la página predeterminada del formulario de acceso). Puedes cambiar este comportamiento de varias maneras.

Cambiando la página predeterminada
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

En primer lugar, la página predeterminada se puede configurar (es decir, la página a la cual el usuario es redirigido si no se almacenó una página previa en la sesión). Para establecer esta a ``/admin`` usa la siguiente configuración:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                main:
                    form_login:
                        # ...
                        default_target_path: /admin

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <form-login
                    default_target_path="/admin"                    
                />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $contenedor->loadFromExtension('security', array(
            'firewalls' => array(
                'main' => array('form_login' => array(
                    // ...
                    'default_target_path' => '/admin',
                )),
            ),
        ));

Ahora, cuando no se encuentre una URL en la sesión del usuario, será enviado a ``/admin``.

Redirigiendo siempre a la página predeterminada
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Puedes hacer que los usuarios siempre sean redirigidos a la página predeterminada, independientemente de la URL que hayan solicitado con anterioridad, estableciendo la opción ``always_use_default_target_path`` a ``true``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                main:
                    form_login:
                        # ...
                        always_use_default_target_path: true
                        
    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <form-login
                    always_use_default_target_path="true"
                />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $contenedor->loadFromExtension('security', array(
            'firewalls' => array(
                'main' => array('form_login' => array(
                    // ...
                    'always_use_default_target_path' => true,
                )),
            ),
        ));

Utilizando la URL referente
~~~~~~~~~~~~~~~~~~~~~~~~~~~

En caso de no haber almacenado una URL anterior en la sesión, posiblemente desees intentar usar el ``HTTP_REFERER`` en su lugar, ya que a menudo será el mismo. Lo puedes hacer estableciendo ``use_referer`` a ``true`` (el valor predeterminado es ``false``): 

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                main:
                    form_login:
                        # ...
                        use_referer:        true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <form-login
                    use_referer="true"
                />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $contenedor->loadFromExtension('security', array(
            'firewalls' => array(
                'main' => array('form_login' => array(
                    // ...
                    'use_referer' => true,
                )),
            ),
        ));

Controlando la URL a redirigir desde el interior del formulario
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

También puedes sustituir a dónde es redirigido el usuario a través del formulario en sí mismo incluyendo un campo oculto con el nombre ``_target_path``. Por ejemplo, para redirigir a la URL definida por alguna ruta ``cuenta``, utiliza lo siguiente:

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

            <input type="hidden" name="_target_path" value="cuenta" />

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

            <input type="hidden" name="_target_path" value="cuenta" />

            <input type="submit" name="login" />
        </form>

Ahora, el usuario será redirigido al valor del campo oculto del formulario. El valor del atributo puede ser una ruta relativa, una URL absoluta, o un nombre de ruta. Incluso, puedes cambiar el nombre del campo oculto en el formulario cambiando la opción ``target_path_parameter`` a otro valor.

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                main:
                    form_login:
                        target_path_parameter: redirect_url

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <form-login
                    target_path_parameter="redirect_url"
                />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $contenedor->loadFromExtension('security', array(
            'firewalls' => array(
                'main' => array('form_login' => array(
                    'target_path_parameter' => redirect_url,
                )),
            ),
        ));

Redirigiendo en ingreso fallido
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Además de redirigir al usuario después de un inicio de sesión, también puedes definir la URL a que el usuario debe ser redirigido después de un ingreso fallido (por ejemplo, si presentó un nombre de usuario o contraseña no válidos). Por omisión, el usuario es redirigido de nuevo al formulario de acceso. Puedes establecer este a una URL diferente con la siguiente configuración:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                main:
                    form_login:
                        # ...
                        failure_path: /login_failure
                        
    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <form-login
                    failure_path="login_failure"
                />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $contenedor->loadFromExtension('security', array(
            'firewalls' => array(
                'main' => array('form_login' => array(
                    // ...
                    'failure_path' => login_failure,
                )),
            ),
        ));
