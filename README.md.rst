Xeno: The Python dependency injector from outer space.
======================================================

Xeno is a simple Python dependency injection framework. Use it when you
need to manage complex inter-object dependencies in a clean way. For the
merits of dependency injection and IOC, see
https://en.wikipedia.org/wiki/Dependency\_injection.

Xeno should feel pretty familiar to users of Google Guice in Java, as it
is somewhat similar, although it is less focused on type names and more
on named resources and parameter injection.

Installation
============

Installation is simple. With python3-pip, do the following:

::

    $ sudo pip install -e .

Or, to install the latest version available on PyPI:

::

    $ sudo pip install xeno

Usage
=====

To use Xeno as a dependency injection framework, you need to create a
xeno.Injector and provide it with modules. These modules are regular
Python objects with methods marked with the ``@xeno.provider``
annotation. This annotation tells the ``Injector`` that this method
provides a named resource, the same name as the method marked with
``@provider``. These methods should either take no parameters (other
than ``self``), or take named parameters which refer to other resources
by name, i.e. the providers can also be injected with other resources in
order to build a dependency chain.

Once you have an ``Injector`` full of resources, you can use it to
inject instances, functions, or methods with resources.

To create a new object instance by injecting resources into its
constructor, use ``Injector.create(clazz)``, where ``clazz`` is the
class which you would like to instantiate. The constructor of this class
is called, and all named parameters in the constructor are treated as
resource references. Once the object is instantiated, any methods marked
with ``@inject`` are invoked with named resources provided.

Resources can be injected into normal functions, bound methods, or
existing object instances via ``Injector.inject(obj)``. If the parameter
is an object instance, it is scanned for methods marked with ``@inject``
and these methods are invoked with named resources provided.

Example
-------

In this simple example, we inject an output stream into an object.

::

    import sys
    from xeno import *

    class OutputStreamModule:
       @provide
       def output_stream(self):
          return sys.stdout

    class VersionWriter:
       def __init__(self, output_stream):
          self.output_stream = output_stream

       def write_version(self):
          print('The python version is %s' % sys.version_info,
                file=self.output_stream)

    injector = Injector(OutputStreamModule())
    writer = injector.create(VersionWriter)
    writer.write_version()

Checkout ``test.py`` in the git repo for more usage examples.

Change Log
----------

Version 2.0.0: July 25th, 2017
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Change the default namespace separator and breakout symbol to '/'
   (backwards incompatible change)

Code using the old namespace separator can be made to work by overriding
the value of xeno.Namespace.SEP:

::

    import xeno
    xeno.Namespace.SEP = '::'

Version 1.10: July 25th, 2017
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Allow names prefixed with ``::`` to escape their module's namespace,
   e.g. ``::top_level_item``

Version 1.9: May 23rd, 2017
~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Add ``@const()`` module annotation for value-based resources
-  Add ``Injector.get_dependency_tree()`` to fetch a tree of dependency
   names for a given resource name.

Version 1.8: May 16th, 2017
~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Add ``MissingResourceError`` and ``MissingDependencyError`` exception
   types.

Version 1.7: May 16th, 2017
~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Major update, adding support for namespaces, aliases, and inline
   resource parameter aliases. See the unit tests in test.py for
   examples.
-  Added ``@namespace('Name')`` decorator for modules to specify that
   all resources defined in the module should be scoped within 'Name::'.
-  Added ``@name('alt-name')`` to allow resources to be named something
   other than the name of the function that defines them.
-  Added ``@alias('alt-name', 'name')`` to allow a resource to be
   renamed within either the scope of a single resource or a whole
   module.
-  Added ``@using('NamespaceName')`` to allow the contents of the given
   namespace to be automatically aliases into either the scope of a
   single resource or a whole module.
-  Added support for resource function annotations via PEP 3107 to allow
   inline aliases, e.g.
   ``def my_resource(name: 'Name::something-important'):``

Version 1.6: April 26th, 2017
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Changed how ``xeno.MethodAttributes`` works: it now holds a map of
   attributes and provides methods ``get()``, ``put()``, and ``check()``

Version 1.5: April 26th, 2017
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Added injection interceptors
-  Refactored method tagging to use ``xeno.MethodAttributes`` instead of
   named object attributes to make attribute tagging more flexible and
   usable by the outside world, e.g. for the new injectors.

Version 1.4: August 30th, 2016
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Added cycle detection.

Version 1.3: August 29th, 2016
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Have the injector offer itself as a named resource named 'injector'.
