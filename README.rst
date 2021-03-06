==============
pyramid_upwork
==============

``pyramid_upwork`` allows your users to authorize via `upwork`_
on your `pyramid`_ project.

You can find it on `pypi`_ as ``pyramid_upwork``. Also don't forget to check the `documentation`_.

.. _`upwork`: https://upwork.com/
.. _`pyramid`: http://www.pylonsproject.org/
.. _`pypi`: http://pypi.python.org/pypi/pyramid_upwork
.. _`documentation`: http://pythonhosted.org/pyramid_upwork/


Requirements
------------
`pyramid-redis-sessions`_ is used to store session in `Redis`_ database, so you need to install
`Redis`_ and configure it to listen to port ``127.0.0.1``.

Other packages are installed automatically::

    pyramid
    pyramid_redis_sessions
    python-upwork

To activate ``jinja2`` renderer, install::

    pip install pyramid_jinja2

.. _`pyramid-redis-sessions`: https://github.com/ericrasmussen/pyramid_redis_sessions
.. _`Redis`: http://redis.io/


Installation
------------
Install with pip::

    pip install pyramid_upwork

or using ``easy_install``::

    easy_install pyramid_upwork

You need to create `Upwork API keys`_ of the type ``Web`` and set appropriate permissions to the generated API key.

.. _`Upwork API keys`: https://www.upwork.com/services/api/keys


Usage
-----
You can take a look at the `pyramid_upwork_example`_ application or use
the instructions below.

Include following settings in your ``*.ini`` file::

    [app:main]

    ...

    # Redis session settings
    redis.sessions.secret = FILL ME

    # upwork settings
    upwork.api.key = FILL ME
    upwork.api.secret = FILL ME

Then in your project's ``__init__.py`` define the following function::

    def get_acl_group(user_uid, request):
        """Here goes your ACL logic."""
        # All authenticated users have ``view`` permission
        return 'view'

This function should return list of ACL group `principals`_ or None if user
is not allowed to have any access groups. See pyramid documentation for `security`_ and `tutorial`_.

Define a RootFactory in your ``models.py``::

    class RootFactory(object):
        """This object sets the security for our application."""
        __acl__ = [
            (Allow, Authenticated, 'view'),
            (Deny, Authenticated, 'login'),
            (Allow, Everyone, 'login'),
        ]

        def __init__(self, request):
            pass

Now register ``get_acl_group()`` function in the config registry to make authorization work. Put in your main method::

    def get_acl_group(request):
        return ('view',)

    def main(global_config, **settings):
        """Main app configuration binding."""

        config = Configurator(settings=settings,
                              root_factory="myapp.models.RootFactory")

        # ACL authorization callback for pyramid-upwork
        config.registry.get_acl_group = get_acl_group

        # External includes
        config.include('pyramid_upwork')

        # Views and routing goes here
        # ...
        #
        config.add_view('myapp.views.MainPage',
                        renderer='templates/main.jinja2',
                        permission='view')

        return config.make_wsgi_app()

.. _`principals`: http://docs.pylonsproject.org/projects/pyramid/en/1.5-branch/glossary.html#term-principal
.. _`security`: http://docs.pylonsproject.org/projects/pyramid/en/1.5-branch/narr/security.html
.. _`tutorial`: http://docs.pylonsproject.org/projects/pyramid/en/1.5-branch/tutorials/wiki2/authorization.html
.. _`pyramid_upwork_example`: https://github.com/kipanshi/pyramid_upwork_example

You can provide custom ``forbidden.jinja2`` template by overriding asset in your ``__init__.py``::

    # Override forbidden template                                                                                                                                                                   config.override_asset(
        to_override='pyramid_upwork:templates/forbidden.jinja2',
        override_with='myapp:templates/forbidden.jinja2')

See template example in `pyramid_upwork/templates/forbidden.jinja2`_.

The "Logout" action is done also via POST request with CSRF protection,
see example of "Logout" buttion in `pyramid_upwork_example/templates/layout.jinja2`_.

.. _`pyramid_upwork/templates/forbidden.jinja2`: https://github.com/kipanshi/pyramid_upwork/tree/master/pyramid_upwork/templates/forbidden.jinja2
.. _`pyramid_upwork_example/templates/layout.jinja2`: https://github.com/kipanshi/pyramid_upwork_example/blob/master/pyramid_upwork_example/templates/layout.jinja2


Contacts
--------
The project is made by Cyril Panshine (`@CyrilPanshine`_). Bug reports and pull requests are very much welcomed!

.. _`@CyrilPanshine`: https://twitter.com/CyrilPanshine
