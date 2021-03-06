.. index::
   pair: Doctrine; DBAL

Cómo utiliza Doctrine la capa DBAL
==================================

.. note::

    Este artículo es sobre la capa DBAL de Doctrine. Normalmente, vas a trabajar con el nivel superior de la capa ORM de Doctrine, la cual simplemente utiliza DBAL detrás del escenario para comunicarse realmente con la base de datos. Para leer más sobre el ORM de Doctrine, consulta ":doc:`/book/doctrine`".

`Doctrine`_ la capa de abstracción de base de datos (DataBase Abstraction Layer - DBAL) es una capa que se encuentra en la parte superior de `PDO`_ y ofrece una API intuitiva y flexible para comunicarse con las bases de datos relacionales más populares. En otras palabras, la biblioteca DBAL facilita la ejecución de consultas y realización de otras acciones de bases de datos.

.. tip::

    Lee la `documentación oficial de DBAL`_ para conocer todos los detalles y las habilidades de la biblioteca DBAL de Doctrine.

Para empezar, configura los parámetros de conexión a la base de datos:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        doctrine:
            dbal:
                driver:   pdo_mysql
                dbname:   Symfony2
                user:     root
                password: null
                charset:  UTF8

    .. code-block:: xml

        // app/config/config.xml
        <doctrine:config>
            <doctrine:dbal
                name="default"
                dbname="Symfony2"
                user="root"
                password="null"
                driver="pdo_mysql"
            />
        </doctrine:config>

    .. code-block:: php

        // app/config/config.php
        $contenedor->loadFromExtension('doctrine', array(
            'dbal' => array(
                'driver'    => 'pdo_mysql',
                'dbname'    => 'Symfony2',
                'user'      => 'root',
                'password'  => null,
            ),
        ));

Para ver todas las opciones de configuración DBAL, consulta :ref:`reference-dbal-configuration`.

A continuación, puedes acceder a la conexión Doctrine DBAL accediendo al servicio ``database_connection``:

.. code-block:: php

    class UserController extends Controller
    {
        public function indexAction()
        {
            $conn = $this->get('database_connection');
            $usuarios = $conn->fetchAll('SELECT * FROM users');

            // ...
        }
    }

Registrando tipos de asignación personalizados
----------------------------------------------

Puedes registrar tipos de asignación personalizados a través de la configuración de Symfony. Ellos se sumarán a todas las conexiones configuradas. Para más información sobre los tipos de asignación personalizados, lee la sección `Tipos de asignación personalizados`_ de la documentación de Doctrine.

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        doctrine:
            dbal:
                types:
                    custom_first: Acme\HolaBundle\Type\CustomFirst
                    custom_second: Acme\HolaBundle\Type\CustomSecond

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:doctrine="http://symfony.com/schema/dic/doctrine"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                                http://symfony.com/schema/dic/doctrine http://symfony.com/schema/dic/doctrine/doctrine-1.0.xsd">

            <doctrine:config>
                <doctrine:dbal>
                <doctrine:dbal default-connection="default">
                    <doctrine:connection>
                        <doctrine:mapping-type name="enum">string</doctrine:mapping-type>
                    </doctrine:connection>
                </doctrine:dbal>
            </doctrine:config>
        </container>

    .. code-block:: php

        // app/config/config.php
        $contenedor->loadFromExtension('doctrine', array(
            'dbal' => array(
                'connections' => array(
                    'default' => array(
                        'mapping_types' => array(
                            'enum'  => 'string',
                        ),
                    ),
                ),
            ),
        ));

Registrando tipos de asignación personalizados en SchemaTool
------------------------------------------------------------

La SchemaTool se utiliza al inspeccionar la base de datos para comparar el esquema. Para lograr esta tarea, es necesario saber qué tipo de asignación se debe utilizar para cada tipo de la base de datos. Por medio de la configuración puedes registrar nuevos tipos.

Vamos a asignar el tipo ENUM (no apoyado por DBAL de manera predeterminada) al tipo ``string``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        doctrine:
            dbal:
                connection:
                    default:
                        // otros parámetros de conexión
                        mapping_types:
                            enum: string

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:doctrine="http://symfony.com/schema/dic/doctrine"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                                http://symfony.com/schema/dic/doctrine http://symfony.com/schema/dic/doctrine/doctrine-1.0.xsd">

            <doctrine:config>
                <doctrine:dbal>
                    <doctrine:type name="custom_first" class="Acme\HolaBundle\Type\CustomFirst" />
                    <doctrine:type name="custom_second" class="Acme\HolaBundle\Type\CustomSecond" />
                </doctrine:dbal>
            </doctrine:config>
        </container>

    .. code-block:: php

        // app/config/config.php
        $contenedor->loadFromExtension('doctrine', array(
            'dbal' => array(
                'types' => array(
                    'custom_first'  => 'Acme\HolaBundle\Type\CustomFirst',
                    'custom_second' => 'Acme\HolaBundle\Type\CustomSecond',
                ),
            ),
        ));

.. _`PDO`:           http://www.php.net/pdo
.. _`Doctrine`:      http://www.doctrine-project.org/projects/dbal/2.0/docs/en
.. _`Documentación DBAL`: http://www.doctrine-project.org/projects/dbal/2.0/docs/en
.. _`Tipos de asignación personalizados`: http://www.doctrine-project.org/docs/dbal/2.0/en/reference/types.html#custom-mapping-types