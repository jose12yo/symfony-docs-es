.. index::
   single: Formularios; Campos; search

Tipo de campo search
====================

Esto reproduce un campo ``<input type="search" />``, el cual es un cuadro de texto con funcionalidad especial apoyada por algunos navegadores.

Lee sobre el campo de de entrada búsqueda en `DiveIntoHTML5.org`_

+-------------+----------------------------------------------------------------------+
| Reproducido | campo ``input search``                                               |
| como        |                                                                      |
+-------------+----------------------------------------------------------------------+
| Opciones    | - `max_length`_                                                      |
| heredadas   | - `required`_                                                        |
|             | - `label`_                                                           |
|             | - `trim`_                                                            |
|             | - `read_only`_                                                       |
|             | - `error_bubbling`_                                                  |
+-------------+----------------------------------------------------------------------+
| Tipo del    | :doc:`text</reference/forms/types/text>`                             |
| padre       |                                                                      |
+-------------+----------------------------------------------------------------------+
| Clase       | :class:`Symfony\\Component\\Form\\Extension\\Core\\Type\\SearchType` |
+-------------+----------------------------------------------------------------------+

Opciones heredadas
------------------

Estas opciones las hereda del tipo :doc:`field </reference/forms/types/field>`:

.. include:: /reference/forms/types/options/max_length.rst.inc

.. include:: /reference/forms/types/options/required.rst.inc

.. include:: /reference/forms/types/options/label.rst.inc

.. include:: /reference/forms/types/options/trim.rst.inc

.. include:: /reference/forms/types/options/read_only.rst.inc

.. include:: /reference/forms/types/options/error_bubbling.rst.inc

.. _`DiveIntoHTML5.org`: http://diveintohtml5.org/forms.html#type-search
