.. index::
   single: Formularios; Campos; repeated

Tipo de campo ``repeated``
==========================

Este es un campo "grupo" especial, el cual crea dos campos idénticos, cuyos valores deben coincidir (o lanza un error de validación). Se utiliza comúnmente cuando necesitas que el usuario repita su contraseña o correo electrónico para verificar su exactitud.

+-------------+------------------------------------------------------------------------+
| Reproducido | campo ``input`` ``text`` por omisión, pero ve la opción `type`_        |
| cómo        |                                                                        |
+-------------+------------------------------------------------------------------------+
| Opciones    | - `type`_                                                              |
|             | - `options`_                                                           |
|             | - `first_name`_                                                        |
|             | - `second_name`_                                                       |
+-------------+------------------------------------------------------------------------+
| Opciones    | - `invalid_message`_                                                   |
| heredadas   | - `invalid_message_parameters`_                                        |
|             | - `error_bubbling`_                                                    |
+-------------+------------------------------------------------------------------------+
| Tipo del    | :doc:`field</reference/forms/types/form>`                              |
| padre       |                                                                        |
+-------------+------------------------------------------------------------------------+
| Clase       | :class:`Symfony\\Component\\Form\\Extension\\Core\\Type\\RepeatedType` |
+-------------+------------------------------------------------------------------------+

Ejemplo de Uso
--------------

.. code-block:: php

    $builder->add('password', 'repeated', array(
        'type' => 'password',
        'invalid_message' => 'Los campos de contraseña deben coincidir.',
        'options' => array('label' => 'Contraseña'),
    ));

Al presentar satisfactoriamente un formulario, el valor ingresado en ambos campos "contraseña" se convierte en los datos de la clave ``password``. En otras palabras, a pesar de que ambos campos efectivamente son reproducidos, el dato final del formulario sólo es el único valor que necesitas (generalmente una cadena).

La opción más importante es ``type``, el cual puede ser cualquier tipo de campo y determina el tipo real de los dos campos subyacentes. La opción ``options`` se pasa a cada uno de los campos individuales, lo cual significa - en este ejemplo - que cualquier opción compatible con el tipo ``password`` la puedes pasar en esta matriz.

Validando
~~~~~~~~~

Una de las características clave del campo ``repeated`` es la validación interna (sin necesidad de hacer nada para configurar esto) el cual obliga a que los dos campos tengan un valor coincidente. Si los dos campos no coinciden, se mostrará un error al usuario.

El ``invalid_message`` se utiliza para personalizar el error que se mostrará cuando los dos campos no coinciden entre sí.

Opciones del campo
~~~~~~~~~~~~~~~~~~

``type``
========

**tipo**: ``string`` **predeterminado**: ``text``

Los dos campos subyacentes serán de este tipo de campo. Por ejemplo, pasando un tipo de ``password`` reproducirá dos campos de contraseña.

Opciones
~~~~~~~~

**tipo**: ``array`` **predeterminado**: ``array()``

Esta matriz de opciones se pasará a cada uno de los dos campos subyacentes. En otras palabras, estas son las opciones que personalizan los tipos de campo individualmente.
Por ejemplo, si la opción ``typo`` se establece en ``password``, esta matriz puede contener las opciones ``always_empty`` o ``required`` - ambas opciones son compatibles con el tipo de campo ``password``.

``first_name``
~~~~~~~~~~~~~~

**tipo**: ``string`` **predeterminado**: ``first``

Este es el nombre real del campo que se utilizará para el primer campo. Esto sobre todo no tiene sentido, sin embargo, puesto que los datos reales especificados en ambos campos disponibles bajo la clave asignada al campo ``repeated`` en sí mismo (por ejemplo, ``password``). Sin embargo, si no se especificas una etiqueta, el nombre de este campo se utiliza para "adivinar" la etiqueta por ti.

``second_name``
~~~~~~~~~~~~~~~

**tipo**: ``string`` **predeterminado**: ``second``

Al igual que ``nombre_de_pila``, pero para el segundo campo.

Opciones heredadas
------------------

Estas opciones las hereda del tipo :doc:`field </reference/forms/types/field>`:

.. include:: /reference/forms/types/options/invalid_message.rst.inc

.. include:: /reference/forms/types/options/invalid_message_parameters.rst.inc

.. include:: /reference/forms/types/options/error_bubbling.rst.inc
