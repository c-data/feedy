=====
Feedy
=====

.. image:: https://raw.githubusercontent.com/c-bata/feedy/master/feedy.png
   :alt: feedy's logo

Simple RSS Feed Fetching Framework.

* Decorator based feed fetcher.
* Fast fetching each entries using asyncio and aiohttp
* Flexible Command line interface.
* Simple plugin system and useful standard plugin.


Installation
============

Supported python version is 3.6 or later.

.. code-block:: console

    pip install feedy


Tutorials
=========

1. Getting feed entry's title and link URL
------------------------------------------

Creating ``main.py`` like:

.. code-block:: python

    from feedy import Feedy

    app = Feedy('feedy.dat')

    @app.add('https://www.djangopackages.com/feeds/packages/latest/rss/')
    def djangopackages(info, body):
        print("- [%s](%s)" % info['article_title'], info['article_url'])

    if __name__ == '__main__':
        app.run()


And running:

.. code-block:: console

    $ python main.py
    - [django-dynamic-views](http://www.djangopackages.com/packages/p/django-dynamic-views/)
    - [django-simple-address](http://www.djangopackages.com/packages/p/django-simple-address/)
    - [django-db-sanitizer](http://www.djangopackages.com/packages/p/django-db-sanitizer/)
    :

After that please execute again.

.. code-block:: console

    $ python main.py
    # No output

It displays only newest feed entries.



2. Add feed patterns
--------------------

Add CNN news feed for collecting images in each articles.

.. code-block:: python

    from feedy import Feedy
    from bs4 import BeautifulSoup  # You can select your favorite html parser.

    app = Feedy(store='feedy.dat', ignore_fetched=True)

    @app.add('http://rss.cnn.com/rss/edition.rss')
    def cnn(info, body):
        soup = BeautifulSoup(body, "html.parser")
        for x in soup.find_all('img'):
            print(x['src'])

    @app.add('https://www.djangopackages.com/feeds/packages/latest/rss/')
    def djangopackages(info, body):
        """Get the latest django library information."""
        print("- [{pkgname}]({link})".format(pkgname=info['article_title'],
                                             link=info['article_link']))

    if __name__ == '__main__':
        app.run()

And running:

.. code-block:: console

    $ python main.py
    - [django-dynamic-views](http://www.djangopackages.com/packages/p/django-dynamic-views/)
    - [django-simple-address](http://www.djangopackages.com/packages/p/django-simple-address/)
    - [django-db-sanitizer](http://www.djangopackages.com/packages/p/django-db-sanitizer/)
    :
    :
    http://i.cdn.turner.com/cnn/.e1mo/img/4.0/logos/menu_money.png
    http://i.cdn.turner.com/cnn/.e1mo/img/4.0/logos/menu_style.png
    http://edition.i.cdn.cnn.com/.a/1.269.4/assets/logo_cnn_nav_bottom.png
    :
    :


3. Command line interface
-------------------------

Feedy offers command line interface. It's useful for debugging

**help messages**

.. code-block:: console

   $ feedy --help
   Usage: feedy [OPTIONS] FILEPATH

     Run your feedy's project flexibly.

   Options:
     -v, --verbose                   Set log level
     -t, --targets TEXT              The target function names.
     -m, --max-entries INTEGER       The maximum length for fetching entries
                                     every RSS feed
     --ignore-fetched / --no-ignore-fetched
                                     The maximum length for fetching entries
                                     every RSS feed
     --help                          Show this message and exit.


If you want to get specified entry for debugging, please execute following command:

.. code-block:: console

    $ feedy main.py -m 2 --no-ignore-fetched
    - [django-dynamic-views](http://www.djangopackages.com/packages/p/django-dynamic-views/)
    - [django-simple-address](http://www.djangopackages.com/packages/p/django-simple-address/)
    http://i.cdn.turner.com/cnn/.e1mo/img/4.0/logos/menu_money.png
    http://i.cdn.turner.com/cnn/.e1mo/img/4.0/logos/menu_style.png

And if you want to run only a cnn function, please execute:

.. code-block:: console

    $ feedy main.py -m 2 --no-ignore-fetched --target cnn
    http://i.cdn.turner.com/cnn/.e1mo/img/4.0/logos/menu_money.png
    http://i.cdn.turner.com/cnn/.e1mo/img/4.0/logos/menu_style.png


After that, please execute a following command:


4. Using plugins
----------------

You can easy developing by using feedy's plugins.
For example, you can get shared count in social sns like facebook and pocket.
There are two ways for applying the plugin.

**Apply specified function using decorator**

.. code-block:: python

    from feedy_plugins import social_share_plugin

    @app.add('http://rss.cnn.com/rss/edition.rss')
    @social_share_plugin
    def cnn_shared(info, body, social_count):
        article = {
            'title': info['article_title'],
            'pocket': social_count['pocket_count'],
            'facebook': social_count['facebook_count'],
        }
        print(article)


And running:

.. code-block:: console

    $ feedy main.py -t cnn_shared -m 2
    {'title': 'Searchers locate Flight 804, EgyptAir vice chairman says', 'pocket': 4, 'facebook': 25}
    {'title': 'Security fears over French airports', 'pocket': 2, 'facebook': 9}


**Apply all functions with ``.install()``**

.. code-block:: python

    from feedy import Feedy
    from feedy_plugins import social_share_plugin
    from bs4 import BeautifulSoup

    app = Feedy(store='feedy.dat', ignore_fetched=True)
    app.install(social_shared_plugin)  # apply each patterns.

    @app.add('http://rss.cnn.com/rss/edition.rss')
    def cnn_shared(info, body, social_count):
        article = {
            'title': info['article_title'],
            'pocket': social_count['pocket_count'],
            'facebook': social_count['facebook_count'],
        }
        print(article)

    @app.add('https://www.djangopackages.com/feeds/packages/latest/rss/')
    def djangopackages(info, body, social_count):
        print("- [{pkgname}]({link})".format(pkgname=info['article_title'],
                                             link=info['article_link']))
        print(social_count['pocket_count'])

    if __name__ == '__main__':
        app.run()


5. Using feedy_utils
--------------------

Download images using feedy_utils.

.. code-block:: python

    from feedy import Feedy
    from feedy_utils import download_image

    IMG_DIR = os.path.join(os.path.dirname('.'), 'img')

    app = Feedy(store='feedy.dat', ignore_fetched=True)

    @app.add('http://rss.cnn.com/rss/edition.rss')
    def cnn(info, body):
        download_image(body, info['site_url'],
                       filename="{title}-{i}".format(info['article_title']),
                       directory=IMG_DIR)

    if __name__ == '__main__':
        app.run()


Create Plugins
==============

To write a new plugin, simply create decorator like:

.. code-block:: python

    def add_information_plugin(callback):
        @wraps(callback)
        def wrapper(*args, **kwargs):
            additional_info = "This is custom plugin."
            kwargs['additional_info'] = additional_info
            callback(*args, **kwargs)
        return wrapper


Happy hacking :)


Development
===========

- type checking: `mypy --fast-parser --python-version 3.6 feedy.py feedy_utils feedy_plugins`
- lint: `flake8 feedy.py feedy_utils feedy_plugins`

Resources
=========

* `Github <https://github.com/c-bata/feedy>`_
* `PyPI <https://pypi.python.org/pypi/feedy>`_
