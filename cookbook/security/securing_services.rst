Cómo proteger cualquier servicio o método de tu aplicación
==========================================================

En el capítulo sobre seguridad, puedes ver cómo :ref:`proteger un controlador <book-security-securing-controller>` requiriendo el servicio ``security.context`` desde el Contenedor de servicios y comprobando el rol del usuario actual::

    use Symfony\Component\Security\Core\Exception\AccessDeniedException
    // ...

    public function holaAction($nombre)
    {
        if (false === $this->get('security.context')->isGranted('ROLE_ADMIN')) {
            throw new AccessDeniedException();
        }

        // ...
    }

También puedes proteger *cualquier* servicio de manera similar, inyectándole el servicio ``security.context``. Para una introducción general a la inyección de dependencias en servicios consulta el capítulo :doc:`/book/service_container` del libro. Por ejemplo, supongamos que tienes una clase ``BoletinGestor`` que envía mensajes de correo electrónico y deseas restringir su uso a únicamente los usuarios que tienen algún rol ``ROLE_BOLETIN_ADMIN``. Antes de agregar la protección, la clase se ve algo parecida a esta:

.. code-block:: php

    namespace Acme\HolaBundle\Boletin;

    class BoletinGestor
    {

        public function sendNewsletter()
        {
            // donde realmente haces el trabajo
        }

        // ...
    }

Nuestro objetivo es comprobar el rol del usuario cuando se llama al método ``sendNewsletter()``. El primer paso para esto es inyectar el servicio ``security.context`` en el objeto. Dado que no tiene sentido *no* realizar la comprobación de seguridad, este es un candidato ideal para el constructor de inyección, lo cual garantiza que el objeto del contexto de seguridad estará disponible dentro de la clase ``BoletinGestor``::

    namespace Acme\HolaBundle\Boletin;

    use Symfony\Component\Security\Core\SecurityContextInterface;

    class BoletinGestor
    {
        protected $securityContext;

        public function __construct(SecurityContextInterface $securityContext)
        {
            $this->securityContext = $securityContext;
        }

        // ...
    }

Luego, en tu configuración del servicio, puedes inyectar el servicio:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HolaBundle/Resources/config/services.yml
        parameters:
            boletin_gestor.class: Acme\HolaBundle\Boletin\BoletinGestor

        services:
            boletin_gestor:
                class:     %boletin_gestor.class%
                arguments: [@security.context]

    .. code-block:: xml

        <!-- src/Acme/HolaBundle/Resources/config/services.xml -->
        <parameters>
            <parameter key="boletin_gestor.class">Acme\HolaBundle\Boletin\BoletinGestor</parameter>
        </parameters>

        <services>
            <service id="boletin_gestor" class="%boletin_gestor.class%">
                <argument type="service" id="security.context"/>
            </service>
        </services>

    .. code-block:: php

        // src/Acme/HolaBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;
        use Symfony\Component\DependencyInjection\Reference;

        $contenedor->setParameter('boletin_gestor.class', 'Acme\HolaBundle\Boletin\BoletinGestor');

        $contenedor->setDefinition('boletin_gestor', new Definition(
            '%boletin_gestor.class%',
            array(new Reference('security.context'))
        ));

El servicio inyectado se puede utilizar para realizar la comprobación de seguridad cuando se llama al método ``sendNewsletter()``::

    namespace Acme\HolaBundle\Boletin;

    use Symfony\Component\Security\Core\Exception\AccessDeniedException
    use Symfony\Component\Security\Core\SecurityContextInterface;
    // ...

    class BoletinGestor
    {
        protected $securityContext;

        public function __construct(SecurityContextInterface $securityContext)
        {
            $this->securityContext = $securityContext;
        }

        public function sendNewsletter()
        {
            if (false === $this->securityContext->isGranted('ROLE_BOLETIN_ADMIN')) {
                throw new AccessDeniedException();
            }

            //--
        }

        // ...
    }

Si el usuario actual no tiene el rol ``ROLE_BOLETIN_ADMIN``, se le pedirá que inicie sesión.

Protegiendo métodos usando anotaciones
--------------------------------------

También puedes proteger las llamadas a métodos en cualquier servicio con anotaciones usando el paquete opcional `JMSSecurityExtraBundle`_. Este paquete está incluido en la Edición estándar de Symfony2.

Para habilitar la funcionalidad de las anotaciones, :ref:`etiqueta <book-service-container-tags>` el servicio que deseas proteger con la etiqueta ``security.secure_service`` (también puedes habilitar esta funcionalidad automáticamente para todos los servicios, consulta la :ref:`barra lateral <securing-services-annotations-sidebar>` más adelante):

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HolaBundle/Resources/config/services.yml
        # ...

        services:
            boletin_gestor:
                # ...
                tags:
                    -  { name: security.secure_service }

    .. code-block:: xml

        <!-- src/Acme/HolaBundle/Resources/config/services.xml -->
        <!-- ... -->

        <services>
            <service id="boletin_gestor" class="%boletin_gestor.class%">
                <!-- ... -->
                <tag name="security.secure_service" />
            </service>
        </services>

    .. code-block:: php

        // src/Acme/HolaBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;
        use Symfony\Component\DependencyInjection\Reference;

        $definicion = new Definition(
            '%boletin_gestor.class%',
            array(new Reference('security.context'))
        ));
        $definicion->addTag('security.secure_service');
        $contenedor->setDefinition('boletin_gestor', $definicion);

Entonces puedes obtener los mismos resultados que el anterior usando una anotación::

    namespace Acme\HolaBundle\Boletin;

    use JMS\SecurityExtraBundle\Annotation\Secure;
    // ...

    class BoletinGestor
    {

        /**
         * @Secure(roles="ROLE_BOLETIN_ADMIN")
         */
        public function sendNewsletter()
        {
            //--
        }

        // ...
    }

.. note::

    Las anotaciones trabajan debido a que se crea una clase sustituta para la clase que realiza las comprobaciones de seguridad. Esto significa que, si bien puedes utilizar las anotaciones sobre métodos públicos y protegidos, no las puedes utilizar con los métodos privados o los métodos marcados como finales.

El ``JMSSecurityExtraBundle`` también te permite proteger los parámetros y valores devueltos de los métodos. Para más información, consulta la documentación de `JMSSecurityExtraBundle`_.

.. _securing-services-annotations-sidebar:

.. sidebar:: Activando la funcionalidad de anotaciones para todos los servicios

    Cuando proteges el método de un servicio (como se muestra arriba), puedes etiquetar cada servicio individualmente, o activar la funcionalidad para *todos* los servicios a la vez. Para ello, establece la opción de configuración ``secure_all_services`` a ``true``:

    .. configuration-block::

        .. code-block:: yaml

            # app/config/config.yml
            jms_security_extra:
                # ...
                secure_all_services: true

        .. code-block:: xml

            <!-- app/config/config.xml -->
            <srv:container xmlns="http://symfony.com/schema/dic/security"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xmlns:srv="http://symfony.com/schema/dic/services"
                xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

                <jms_security_extra secure_controllers="true" secure_all_services="true" />

            </srv:container>

        .. code-block:: php

            // app/config/config.php
            $contenedor->loadFromExtension('jms_security_extra', array(
                // ...
                'secure_all_services' => true,
            ));

    La desventaja de este método es que, si está activada, la carga de la página inicial puede ser muy lenta dependiendo de cuántos servicios hayas definido.

.. _`JMSSecurityExtraBundle`: https://github.com/schmittjoh/JMSSecurityExtraBundle