.. index::
   single: Seguridad; Conceptos ACL avanzados

Conceptos ACL avanzados
=======================

El objetivo de este capítulo es dar una visión en mayor profundidad del sistema ACL, y también explicar algunas de las decisiones de diseño detrás de él.

Conceptos de diseño
-------------------

La capacidad de la instancia del objeto seguridad de Symfony2 está basada en el concepto de una Lista de Control de Acceso. Cada **instancia** del objeto dominio tiene su propia ACL. La instancia de ACL contiene una detallada lista de las entradas de control de acceso (ACE) utilizada para tomar decisiones de acceso. El sistema ACL de Symfony2 se enfoca en dos objetivos principales:

- proporcionar una manera eficiente de recuperar una gran cantidad de ACL/ACE para tus objetos dominio, y para modificarlos;
- proporcionar una manera de facilitar las decisiones de si a una persona se le permite realizar una acción en un objeto dominio o no.

Según lo indicado por el primer punto, una de las principales capacidades del sistema ACL de Symfony2 es una forma de alto rendimiento de recuperar las ACL/ACE. Esto es muy importante ya que cada ACL puede tener varias ACE, y heredar de otra ACL anterior en una forma de árbol. Por lo tanto, específicamente no aprovechamos cualquier ORM, pero la implementación predeterminada interactúa con tu conexión directamente usando DBAL Doctrine.

Identidades de objeto
~~~~~~~~~~~~~~~~~~~~~

El sistema ACL está disociado completamente de los objetos de tu dominio. Ni siquiera se tienen que almacenar en la misma base de datos, o en el mismo servidor. Para lograr esta disociación, en el sistema ACL los objetos son representados a través de objetos identidad objeto. Cada vez, que desees recuperar la ACL para un objeto dominio, el sistema ACL en primer lugar crea un objeto identidad de tu objeto dominio y, a continuación pasa esta identidad de objeto al proveedor de ACL para su posterior procesamiento.


Identidad de seguridad
~~~~~~~~~~~~~~~~~~~~~~

Esto es análogo a la identidad de objeto, pero representa a un usuario o un rol en tu aplicación. Cada rol, o usuario tiene una identidad de seguridad propia.


Estructura de tabla en la base de datos
---------------------------------------

La implementación predeterminada usa cinco tablas de bases de datos enumeradas a continuación. Las tablas están ordenadas de menos filas a más filas en una aplicación típica:

- *acl_security_identities*: Esta tabla registra todas las identidades de seguridad (SID) de que dispone ACE. La implementación predeterminada viene con dos identidades de seguridad: ``RoleSecurityIdentity`` y ``UserSecurityIdentity``
- *acl_classes*: Esta tabla asigna los nombres de clase a un identificador único el cual puede hacer referencia a otras tablas.
- *acl_object_identities*: Cada fila de esta tabla representa una única instancia del objeto dominio.
- *acl_object_identity_ancestors*: Esta tabla nos permite determinar todos los antepasados​de una ACL de una manera muy eficiente.
- *acl_entries*: Esta tabla contiene todas las ACE. Esta suele ser la tabla con más filas. Puede contener decenas de millones sin impactar significativamente en el rendimiento.


Alcance de las entradas de control de acceso
--------------------------------------------

Las entradas del control de acceso pueden tener diferente ámbito en el cual se aplican. En Symfony2, básicamente tenemos dos diferentes ámbitos:

- Class-Scope: Estas entradas se aplican a todos los objetos con la misma clase.
- Object-Scope: Este fue el ámbito de aplicación utilizado únicamente en el capítulo anterior, y sólo se aplica a un objeto específico.

A veces, encontraras la necesidad de aplicar una ACE sólo a un campo específico del objeto. Digamos que deseas que el ID sólo sea visto por un administrador, pero no por tu servicio al cliente. Para resolver este problema común, hemos añadido dos subámbitos:

- Class-Field-Scope: Estas entradas se aplican a todos los objetos con la misma clase, pero sólo a un campo específico de los objetos.
- Object-Field-Scope: Estas entradas se aplican a un objeto específico, y sólo a un campo específico de ese objeto.

Decisiones de preautorización
-----------------------------

Para las decisiones de preautorización, es decir, las decisiones antes de que el método o la acción de seguridad se invoque, confiamos en el servicio provisto ``AccessDecisionManager`` que también se utiliza para tomar decisiones de autorización basadas en roles. Al igual que los roles, el sistema ACL añade varios nuevos atributos que se pueden utilizar para comprobar diferentes permisos.

Mapa de permisos incorporados
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

+------------------+----------------------------+-----------------------------+
| Atributo         | Significado previsto       | Máscara de Bits             |
+==================+============================+=============================+
| VIEW             | Cuando le es permitido a   | VIEW, EDIT, OPERATOR,       |
|                  | alguien ver el objeto      | MASTER u OWNER              |
|                  | dominio.                   |                             |
+------------------+----------------------------+-----------------------------+
| EDIT             | Cuando le es permitido a   | EDIT, OPERATOR, MASTER,     |
|                  | alguien hacer cambios al   | u OWNER                     |
|                  | objeto dominio.            |                             |             
+------------------+----------------------------+-----------------------------+
| DELETE           | Cuando le es permitido a   | DELETE, OPERATOR, MASTER    |
|                  | alguien eliminar el objeto | u OWNER                     |
|                  | dominio.                   |                             |                    
+------------------+----------------------------+-----------------------------+
| UNDELETE         | Cuando le es permitido a   | UNDELETE, OPERATOR, MASTER  |
|                  | alguien restaurar un       | u OWNER                     |
|                  | objeto dominio previamente |                             |
|                  | eliminado.                 |                             |     
+------------------+----------------------------+-----------------------------+
| OPERATOR         | Cuando le es permitido a   | OPERATOR, MASTER u OWNER    |
|                  | alguien realizar todas las |                             |
|                  | acciones anteriores.       |                             |                   
+------------------+----------------------------+-----------------------------+
| MASTER           | Cuando le es permitido a   | MASTER u OWNER              |
|                  | alguien realizar todas las |                             |
|                  | acciones anteriores, y     |                             |
|                  | además tiene permitido     |                             |
|                  | conceder cualquiera de los |                             |
|                  | permisos anteriores a      |                             |
|                  | otros.                     |                             |     
+------------------+----------------------------+-----------------------------+
| OWNER            | Cuando alguien es dueño    | OWNER                       |
|                  | del objeto dominio.        |                             | 
|                  | un propietario puede       |                             |
|                  | realizar cualquiera de las |                             |
|                  | acciones anteriores.       |                             |                    
+------------------+----------------------------+-----------------------------+

Atributos de permisos frente a máscaras de bits de permisos
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Los atributos los utiliza el ``AccessDecisionManager``, al igual que los roles son los atributos utilizados por ``AccessDecisionManager``. A menudo, estos atributos en realidad representan un conjunto de enteros como máscaras de bits. Las máscaras de bits de enteros en cambio, las utiliza el sistema ACL interno para almacenar de manera eficiente los permisos de los usuarios en la base de datos, y realizar comprobaciones de acceso mediante las operaciones muy rápidas de las máscaras de bits.

Extensibilidad
~~~~~~~~~~~~~~

El mapa de permisos citado más arriba no es estático, y, teóricamente, lo podrías reemplazar a voluntad por completo. Sin embargo, debería abarcar la mayoría de los problemas que encuentres, y para interoperabilidad con otros paquetes, te animamos a que le adhieras el significado que tienes previsto para ellos.

Decisiones de postautorización
------------------------------

Las decisiones de postautorización se realizan después de haber invocado a un método seguro, y por lo general implican que el objeto dominio es devuelto por este método.
Después de invocar a los proveedores también te permite modificar o filtrar el objeto dominio antes de devolverlo.

Debido a las limitaciones actuales del lenguaje PHP, no hay capacidad de postautorización integradas en el núcleo del componente seguridad.
Sin embargo, hay un JMSSecurityExtraBundle_ experimental que añade estas capacidades. Consulta su documentación para más información sobre cómo se logra esto.

Proceso para conseguir decisiones de autorización
-------------------------------------------------

La clase ACL proporciona dos métodos para determinar si una identidad de seguridad tiene la máscara de bits necesaria, ``isGranted`` y ``isFieldGranted``. Cuando la ACL recibe una petición de autorización a través de uno de estos métodos, delega esta petición a una implementación de ``PermissionGrantingStrategy``. Esto te permite reemplazar la forma en que se tomen decisiones de acceso sin tener que modificar la clase ACL misma.

``PermissionGrantingStrategy`` primero verifica todo su ámbito de aplicación ACE a objetos si no es aplicable, comprobará el ámbito de la clase ACE, si no es aplicable, entonces el proceso se repetirá con las ACE de la ACL padre. Si no existe la ACL padre, será lanzada una excepción.

.. _JMSSecurityExtraBundle: https://github.com/schmittjoh/JMSSecurityExtraBundle
