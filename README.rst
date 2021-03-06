Walter
======

Walter is a configuration library, inspired by `python-decouple <https://pypi.python.org/pypi/python-decouple>`_, and intended to replace direct access to ``os.environ`` in Django ``settings.py`` files (although it is by no means Django-specific). It currently supports Python 3.6+.

It differs from other, similar libraries for one reason: when your users try to start up your app with invalid configuration, the error message they get shows a list of **all of the errors** with every configuration parameter, not just the first one.

Installation
------------

.. code-block:: shell

    pip install walter
    # or
    poetry add walter

Usage
-----

Here's an example of a Python file that uses Walter to define its configuration.

::

    from walter import Config

    with Config(author="Acme Inc.", name="My Awesome App") as config:

        # Read a configuration value with config()
        SECRET_KEY = config('SECRET_KEY')

        # Convert the returned value to something other than a string with cast.
        DEBUG = config('DEBUG', cast=bool)

        # You can pass any function that takes a string to `cast`.
        # Here, we're using a third party function to parse a database URL
        # string into a Django-compatible dictionary.
        import dj_database_url
        DATABASES = {
            'default': config('DATABASE_URL', cast=dj_database_url.parse),
        }

        # You can also make a parameter optional by giving it a default.
        SENTRY_DSN = config('SENTRY_DSN', default=None)

    print(f"Here, you can use values like {SITE_NAME}!")

If we run that code without setting anything, Walter throws an error at the end of the ``with`` block.

.. code-block:: pytb

    Traceback (most recent call last):
    File "<stdin>", line 27, in <module>
    File "/Users/leigh/Projects/walter/walter/config.py", line 90, in __exit__
        raise ConfigErrors(errors=self.errors)
    walter.config.ConfigErrors: 4 configuration values not set, 0 invalid

    SECRET_KEY not set
    DEBUG not set
    DATABASE_URL not set
    SITE_NAME not set

Note that Walter lists out all of the errors in our configuration, not just the first one! If we set all of those settings as environment variables and run the code again, the code runs to completion:

.. code-block:: text

    Here, you can use values like MyAwesomeApp!
