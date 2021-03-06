.. index::
   single: Formularios; Campos; choice

Tipo de campo ``entity``
========================

Un campo ``choice`` especial que está diseñado para cargar las opciones de una entidad Doctrine. Por ejemplo, si tiene una entidad ``Categoria``, puedes utilizar este campo para mostrar un campo ``select`` todos o algunos de los objetos ``Categoria`` de la base de datos.

+-------------+------------------------------------------------------------------+
| Reproducido | pueden ser varias etiquetas (ve                                  |
| como        | :ref:`forms-reference-choice-tags`)                              |
+-------------+------------------------------------------------------------------+
| Opciones    | - `class`_                                                       |
|             | - `property`_                                                    |
|             | - `query_builder`_                                               |
|             | - `em`_                                                          |
+-------------+------------------------------------------------------------------+
| Opciones    | - `required`_                                                    |
| heredadas   | - `label`_                                                       |
|             | - `multiple`_                                                    |
|             | - `expanded`_                                                    |
|             | - `preferred_choices`_                                           |
|             | - `read_only`_                                                   |
|             | - `error_bubbling`_                                              |
+-------------+------------------------------------------------------------------+
| Tipo del    | :doc:`choice</reference/forms/types/choice>`                     |
| padre       |                                                                  |
+-------------+------------------------------------------------------------------+
| Clase       | :class:`Symfony\\Bridge\\Doctrine\\Form\\Type\\EntityType`       |
+-------------+------------------------------------------------------------------+

Uso básico
----------

El tipo ``entidad`` tiene una sola opción obligatoria: la entidad que debe aparecer dentro del campo de elección::

    $builder->add('users', 'entity', array(
        'class' => 'Acme\\HolaBundle\\Entity\\User',
    ));

En este caso, todos los objetos ``Usuario`` serán cargados desde la base de datos y representados como etiquetas ``select``, botones de radio o una serie de casillas de verificación (esto depende del valor ``multiple`` y ``expanded``).

Usando una consulta personalizada para las Entidades
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Si es necesario especificar una consulta personalizada a utilizar al recuperar las entidades (por ejemplo, sólo deseas devolver algunas entidades, o necesitas ordenarlas), utiliza la opción ``query_builder``. La forma más fácil para utilizar la opción es la siguiente::

    use Doctrine\ORM\EntityRepository;
    // ...

    $builder->add('users', 'entity', array(
        'class' => 'Acme\\HolaBundle\\Entity\\User',
        'query_builder' => function(EntityRepository $er) {
            return $er->createQueryBuilder('u')
                ->orderBy('u.nombreusuario', 'ASC');
        },
    ));

.. include:: /reference/forms/types/options/select_how_rendered.rst.inc

.. include:: /reference/forms/types/options/empty_value.rst.inc

Opciones del campo
~~~~~~~~~~~~~~~~~~

``class``
~~~~~~~~~

**tipo**: ``string`` **requerido**

La clase de tu entidad (por ejemplo, ``Acme\GuardaBundle\Entity\Categoria``).

``property``
~~~~~~~~~~~~

**tipo**: ``string``

Esta es la propiedad que se debe utilizar para visualizar las entidades como texto en el elemento HTML. Si la dejas en blanco, el objeto entidad será convertido en una cadena y por lo tanto debe tener un método ``__toString()``.

``query_builder``
~~~~~~~~~~~~~~~~~

**tipo**: ``Doctrine\ORM\QueryBuilder`` o un Cierre

Si lo especificas, se utiliza para consultar el subconjunto de opciones (y su orden) que se debe utilizar para el campo. El valor de esta opción puede ser un objeto ``QueryBuilder`` o un Cierre. Si utilizas un Cierre, este debe tener un sólo argumento, el cual es el ``EntityRepository`` de la entidad.

``em``
~~~~~~

**tipo**: ``string`` **predeterminado**: el gestor de la entidad

Si lo especificas, el gestor de la entidad especificada se utiliza para cargar las opciones en lugar de las predeterminadas del gestor de la entidad.

Opciones heredadas
------------------

Estas opciones las hereda del tipo :doc:`choice</reference/forms/types/choice>`:

.. include:: /reference/forms/types/options/multiple.rst.inc

.. include:: /reference/forms/types/options/expanded.rst.inc

.. include:: /reference/forms/types/options/preferred_choices.rst.inc

Estas opciones las hereda del tipo :doc:`field </reference/forms/types/field>`:

.. include:: /reference/forms/types/options/required.rst.inc

.. include:: /reference/forms/types/options/label.rst.inc

.. include:: /reference/forms/types/options/read_only.rst.inc

.. include:: /reference/forms/types/options/error_bubbling.rst.inc
