Module development
==================

The basics
----------

An Annotator module is a function which can be passed to
:func:`~annotator.App.prototype.include` in order to extend the functionality of
an Annotator application. Modules can:

- be notified and make changes to annotations as they are created or modified
- provide alternative user interfaces
- provide alternative storage backends for annotations
- change how Annotator decides what permissions an annotating user has

and plenty more.

The simplest possible Annotator module looks like this::

    function myModule() {
        return {};
    }

This clearly won't do very much, but we can include it in an application::

    app.include(myModule);

If we want to do something more interesting, we have to provide some module
functionality. There are essentially two primary ways of doing this:

1. module hooks
2. component registration

Unless you are replacing core functionality of Annotator (writing a storage
component, for example) you probably want to use module hooks. Module hooks are
functions which you can expose from your module which will be run by the
:class:`~annotator.App` when important things happen. For example, here's a
module which will say ``Hello, world!`` to the user when the application
starts::

    function helloWorld() {
        return {
            start: function (app) {
                app.notify("Hello, world!");
            }
        };
    }

Just as before, we can include it in an application using
:func:`~annotator.App.prototype.include`::

    app.include(helloWorld);

Now, when you run ``app.start();``, this module will send a notification with
the words ``Hello, world!``.

Or, here's another example that uses the `HTML5 Audio API`_ to play a sound
every time a new annotation is made [#1]_::

    function fanfare(options) {
        options = options || {};
        options.url = options.url || 'trumpets.mp3';

        return {
            annotationCreated: function (annotation) {
                var audio = new Audio(options.url);
                audio.play();
            }
        };
    }

Here we've added an ``options`` argument to the module function so we can
configure the module when it's included in our application::

    app.include(fanfare, {
        url: "brass_band.wav"
    });

You might have noticed that the :func:`annotationCreated` module hook function
here receives one argument, ``annotation``. Similarly, the :func:`start` module
hook function in the previous example received an ``app`` argument. You can find
out which arguments are passed to which module hooks in :ref:`module-hooks`,
below.

.. _HTML5 Audio API: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API


Loading custom modules
----------------------

When you write a custom module, you'll end up with a JavaScript function which
you need to reference when you build your application. In the examples above
we've just defined a function and then used it straight away. This is probably
fine for small examples, but when things get a bit more complicated you might
want to put your modules in a namespace.

For example, if you were working on an application for annotating Shakespeare's
plays, you might put all your modules in a namespace called ``shakespeare``::

    var shakespeare = {};
    shakespeare.fanfare = function fanfare(options) {
        ...
    };
    shakespeare.addSceneData = function addSceneData(options) {
        ...
    };

You get the idea. You can now :func:`~annotator.App.prototype.include` these
modules directly from the namespace::

    app.include(shakespeare.fanfare, {
        url: "elizabethan_sackbuts.mp3"
    });
    app.include(shakespeare.addSceneData);

All the modules that ship with Annotator are placed within the ``annotator``
namespace. If you write and publish your own modules, be aware that you don't
need to put your modules in the ``annotator`` namespace for them to work.


.. _module-hooks:

Module hooks
------------

This is a list of module hooks, when they are called, and what arguments they
receive.


.. function:: configure(registry)

   Called when the plugin is included. If you are going to register components
   with the registry, you should do so in the `configure` module hook.

   :param Registry registry: The application registry.


.. function:: start(app)

   Called when :func:`~annotator.App.prototype.start` is called.

   :param App app: The configured application.


.. function:: destroy()

   Called when :func:`~annotator.App.prototype.destroy` is called. If your
   module needs to do any cleanup, such as unbind events or disposing of
   elements injected into the DOM, it should do so in the `destroy` hook.


.. function:: annotationsLoaded(annotations)

   Called with annotations retrieved from storage using
   :func:`~annotator.storage.StorageAdapter.load`.

   :param Array[Object] annotations: The annotation objects loaded.


.. function:: beforeAnnotationCreated(annotation)

   Called immediately before an annotation is created. Use if you need to modify
   the annotation before it is saved.

   :param Object annotation: The annotation object.


.. function:: annotationCreated(annotation)

   Called when a new annotation has been created.

   :param Object annotation: The annotation object.


.. function:: beforeAnnotationUpdated(annotation)

   Called immediately before an annotation is updated. Use if you need to modify
   the annotation before it is saved.

   :param Object annotation: The annotation object.


.. function:: annotationUpdated(annotation)

   Called when an annotation has been updated.

   :param Object annotation: The annotation object.


.. function:: beforeAnnotationDeleted(annotation)

   Called immediately before an annotation is deleted. Use if you need to
   conditionally cancel deletion, for example.

   :param Object annotation: The annotation object.


.. function:: annotationDeleted(annotation)

   Called when an annotation has been deleted.

   :param Object annotation: The annotation object.


.. rubric:: Footnotes

.. [#1] Yes, this might be quite annoying. Probably not an example to copy
        wholesale into your real application...
