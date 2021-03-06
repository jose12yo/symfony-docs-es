.. index::
   single: Registrando

Cómo utilizar ``Monolog`` para escribir Registros
=================================================

`Monolog`_ es una biblioteca de registro para PHP 5.3 utilizada por Symfony2. Inspirada en la biblioteca LogBook de Python.

Utilizando
----------

En ``Monolog`` cada registrador define un canal de registro. Cada canal tiene una pila de controladores para escribir los registros (los controladores se pueden compartir).

.. tip::

    Cuando inyectas el registrador en un servicio puedes :ref:`utilizar un canal personalizado <dic_tags-monolog>` para ver fácilmente qué parte de la aplicación registró el mensaje.

El encargado básico es el ``StreamHandler`` el cual escribe los registros en un flujo (por omisión en ``app/logs/prod.log`` en el entorno de producción y ``app/logs/dev.log`` en el entorno de desarrollo).

Monólogo también viene con una potente capacidad integrada para encargarse del registro en el entorno de producción: ``FingersCrossedHandler``. Esta te permite almacenar los mensajes en la memoria intermedia y registrarla sólo si un mensaje llega al nivel de acción (ERROR en la configuración prevista en la edición estándar) transmitiendo los mensajes a otro controlador.

Para registrar un mensaje, simplemente obtén el servicio registrador del contenedor en tu controlador::

    $registrador = $this->get('logger');
    $registrador->info('acabamos de recibir el registrador');
    $registrador->err('Ha ocurrido un error');

.. tip::

    Usar sólo los métodos de la interfaz :class:`Symfony\Component\HttpKernel\Log\LoggerInterface` te permite cambiar la implementación del anotador sin cambiar el código.

Utilizando varios controladores
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

El registrador utiliza una pila de controladores que se llaman sucesivamente. Esto te permite registrar los mensajes de varias formas fácilmente.

.. configuration-block::

    .. code-block:: yaml

        monolog:
            handlers:
                syslog:
                    type: stream
                    path: /var/log/symfony.log
                    level: error
                main:
                    type: fingerscrossed
                    action_level: warning
                    handler: file
                file:
                    type: stream
                    level: debug

    .. code-block:: xml

        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:monolog="http://symfony.com/schema/dic/monolog"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                                http://symfony.com/schema/dic/monolog http://symfony.com/schema/dic/monolog/monolog-1.0.xsd">

            <monolog:config>
                <monolog:handler
                    name="syslog"
                    type="stream"
                    path="/var/log/symfony.log"
                    level="error"
                />
                <monolog:handler
                    name="main"
                    type="fingerscrossed"
                    action-level="warning"
                    handler="file"
                />
                <monolog:handler
                    name="file"
                    type="stream"
                    level="debug"
                />
            </monolog:config>
        </container>

La configuración anterior define una pila de controladores que se llamarán en el orden en el cual se definen.

.. tip::

    El controlador denominado "file" no se incluirá en la misma pila que se utiliza como un controlador anidado del controlador `fingerscrossed`.

.. note::

    Si deseas cambiar la configuración de `MonologBundle` en otro archivo de configuración necesitas redefinir toda la pila. No se pueden combinar, ya que el orden es importante y una combinación no te permite controlar el orden.

Cambiando el formateador
~~~~~~~~~~~~~~~~~~~~~~~~

El controlador utiliza un ``formateador`` para dar formato al registro antes de ingresarlo. Todos los manipuladores ``Monolog`` utilizan una instancia de ``Monolog\Formatter\LineFormatter`` por omisión, pero la puedes reemplazar fácilmente. Tu formateador debe implementar ``Monolog\Formatter\FormatterInterface``.

.. configuration-block::

    .. code-block:: yaml

        services:
            my_formatter:
                class: Monolog\Formatter\JsonFormatter
        monolog:
            handlers:
                file:
                    type: stream
                    level: debug
                    formatter: my_formatter

    .. code-block:: xml

        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:monolog="http://symfony.com/schema/dic/monolog"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                                http://symfony.com/schema/dic/monolog http://symfony.com/schema/dic/monolog/monolog-1.0.xsd">

            <services>
                <service id="my_formatter" class="Monolog\Formatter\JsonFormatter" />
            </services>
            <monolog:config>
                <monolog:handler
                    name="file"
                    type="stream"
                    level="debug"
                    formatter="my_formatter"
                />
            </monolog:config>
        </container>

Agregando algunos datos adicionales en los mensajes de registro
---------------------------------------------------------------

Monolog te permite procesar el registro antes de ingresarlo añadiendo algunos datos adicionales. Un procesador se puede aplicar al manipulador de toda la pila o sólo para un manipulador específico.

Un procesador simplemente es un ejecutable que recibe el registro como primer argumento.

Los procesadores se configuran usando la etiqueta DIC ``monolog.processor``. Consulta la
:ref:`referencia sobre esto <dic_tags-monolog-processor>`.

Agregando un segmento Sesión/Petición
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A veces es difícil saber cuál de las entradas en el registro pertenece a cada sesión y/o petición. En el siguiente ejemplo agregaremos una ficha única para cada petición usando un procesador.

.. code-block:: php

    namespace Acme\MyBundle;

    use Symfony\Component\HttpFoundation\Session;

    class SessionRequestProcessor
    {
        private $sesion;
        private $muestra;

        public function __construct(Session $sesion)
        {
            $this->sesion = $sesion;
        }

        public function processRecord(array $record)
        {
            if (null === $this->muestra) {
                try {
                    $this->muestra = substr($this->session->getId(), 0, 8);
                } catch (\RuntimeException $e) {
                    $this->muestra = '????????';
                }
                $this->muestra .= '-' . substr(uniqid(), -8);
            }
            $record['extra']['token'] = $this->muestra;

            return $record;
        }
    }

.. configuration-block::

    .. code-block:: yaml

        services:
            monolog.formatter.session_request:
                class: Monolog\Formatter\LineFormatter
                arguments:
                    - "[%%datetime%%] [%%extra.token%%] %%channel%%.%%level_name%%: %%message%%\n"

            monolog.processor.session_request:
                class: Acme\MyBundle\SessionRequestProcessor
                arguments:  [ @session ]
                tags:
                    - { name: monolog.processor, method: processRecord }

        monolog:
            handlers:
                main:
                    type: stream
                    path: %kernel.logs_dir%/%kernel.environment%.log
                    level: debug
                    formatter: monolog.formatter.session_request

.. note::

    Si utilizas varios manipuladores, también puedes registrar el procesador a nivel del manipulador en lugar de globalmente.

.. _Monolog: https://github.com/Seldaek/monolog
