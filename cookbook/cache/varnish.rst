.. index::
    single: Caché; Varnish

Cómo utilizar Varnish para acelerar mi sitio web
================================================

Debido a que la caché de Symfony2 utiliza las cabeceras de caché HTTP estándar, el :ref:`symfony-gateway-cache` se puede sustituir fácilmente por cualquier otro sustituto inverso. Varnish es un potente acelerador HTTP, de código abierto, capaz de servir contenido almacenado en caché de forma rápida y es compatible con :ref:`Inclusión del borde lateral <edge-side-includes>`.

.. index::
    single: Varnish; configurando

Configurando
------------

Como vimos anteriormente, Symfony2 es lo suficientemente inteligente como para detectar si está hablando con un sustituto inverso que entiende |ESI| o no. Funciona fuera de la caja cuando utilizas el sustituto inverso de Symfony2, pero necesita una configuración especial para que funcione con Varnish. Afortunadamente, Symfony2 se basa en otra norma escrita por Akamaï (`Arquitectura límite`_), por lo que el consejo de configuración en este capítulo te puede ser útil incluso si no utilizas Symfony2.

.. note::

    Varnish sólo admite el atributo ``src`` para las etiquetas |ESI| (los atributos ``onerror`` y ``alt`` se omiten).

En primer lugar, configura Varnish para que anuncie su apoyo a |ESI| añadiendo una cabecera ``Surrogate-Capability`` a las peticiones remitidas a la interfaz administrativa de tu aplicación:

.. code-block:: text

    sub vcl_recv {
        set req.http.Surrogate-Capability = "abc=ESI/1.0";
    }

Entonces, optimiza Varnish para que sólo analice el contenido de la respuesta cuando al menos hay una etiqueta |ESI| comprobando la cabecera ``Surrogate-Control`` que Symfony2 añade automáticamente:

.. code-block:: text

    sub vcl_fetch {
        if (beresp.http.Surrogate-Control ~ "ESI/1.0") {
            unset beresp.http.Surrogate-Control;
            esi;
        }
    }

.. caution::

    No utilices compresión con |ESI| ya que Varnish no será capaz de analizar el contenido de la respuesta. Si deseas utilizar compresión, pon un servidor web frente a Varnish para hacer el trabajo.

.. index::
    single: Varnish; Invalidación

Invalidando la caché
--------------------

Nunca debería ser necesario invalidar los datos almacenados en caché porque la invalidación ya se tiene en cuenta de forma nativa en los modelos de caché HTTP (consulta :ref:`http-cache-invalidation`).

Sin embargo, Varnish se puede configurar para aceptar un método HTTP especial ``PURGE`` que invalida la caché para un determinado recurso:

.. code-block:: text

    sub vcl_hit {
        if (req.request == "PURGE") {
            set obj.ttl = 0s;
            error 200 "Purged";
        }
    }

    sub vcl_miss {
        if (req.request == "PURGE") {
            error 404 "Not purged";
        }
    }

.. caution::

    De alguna manera, debes proteger el método ``PURGE`` HTTP para evitar que alguien aleatoriamente purgue los datos memorizados.

.. _`Arquitectura Límite`: http://www.w3.org/TR/edge-arch
..  |ESI| replace:: :abbr:`ESI (Edge Side Includes o Inclusión del borde lateral)`