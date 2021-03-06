===============
ctk-cli-indexer
===============

The files in this repository allow you to create an Elasticsearch_ database containing
information on available CLI modules.  The idea is that we have a public Kibana_ dashboard
listing CLI modules from multiple sources, so there is a script with two modes:

'extract' mode
  Extracts JSON descriptions from a set of CLI modules (in one or more common directories). ::

    # ./ctk_cli_indexer.py extract --help
    usage: ctk_cli_indexer.py extract [-h] [--json_filename JSON_FILENAME]
                                      base_directory [base_directory ...]

    positional arguments:
      base_directory        directories (at least one) in which to search for CLI
                            module executables, or direct paths to executables

    optional arguments:
      -h, --help            show this help message and exit
      --json_filename JSON_FILENAME, -o JSON_FILENAME

  This is to be run by the administrators of sites that offer CLI modules, and the idea is
  that the resulting .json files are published on some website.

'index' mode
  Takes a JSON file (or a list of CLI executables) and updates an
  Elasticsearch_ database.  An identifier for the source of the CLI
  modules is passed as first parameter, and the script takes care to
  delete old documents in the database (CLIs that got removed), and
  will also maintain timestamps of the last change of each CLI
  (i.e. not re-upload stuff that did not change, as well as mark each
  change with the modification time of the CLI executable that
  introduced the change). Instead of passing a JSON file, you may also
  pass a list of directories or CLI executables directly. ::

    # ./ctk_cli_indexer.py index --help
    usage: ctk_cli_indexer.py index [-h] [--host HOST] [--port PORT]
                                    source_name path [path ...]

    positional arguments:
      source_name  identifier for the source (e.g. 'slicer' or 'nifty-reg') of
                   this set of CLI modules (will be used to remove old documents
                   from this source from the Elasticsearch index if they are no
                   longer present)
      path         one or more directories in which to search for CLI module
                   executables, paths to CLI executables, or (exactly one) JSON
                   file as created by `extract` subcommand

    optional arguments:
      -h, --help   show this help message and exit
      --host HOST  hostname elasticsearch is listening on (default: localhost)
      --port PORT  port elasticsearch is listening on (default: 9200)

  This script should be run by a cron job (i.e. setup by a CTK administrator), from a script
  that pulls the above-mentioned .json URLs regularly and updates a central database.
  A Kibana_ dashboard will then give interested people an overview over the available modules
  from multiple sites.

.. _Elasticsearch: http://www.elasticsearch.org/overview/elasticsearch/
.. _Kibana: http://www.elasticsearch.org/overview/kibana/
  
System Prerequisites
====================

The following software packages are required to be installed on your system:

* `Python <http://python.org>`_
* `pip <https://pypi.python.org/pypi/pi>`_ (recommended for installation)
* `Git <http://git-scm.com/>`_ (for developer only)

The following python packages will be automatically installed if not present (see
`requirements.txt`, listed here in case you prefer to install them via your system's
package manager):

* `simplejson <https://pypi.python.org/pypi/simplejson/>`_
* `elasticsearch-py <https://pypi.python.org/pypi/elasticsearch>`_
* `ctk-cli <https://pypi.python.org/pypi/ctk-cli>`_

Installation for user
=====================

Use ``pip`` (or ``easy_install``) for installation from pypi_::

    pip install ctk-cli-indexer

.. _pypi: https://pypi.python.org/pypi
    
Installation for developer
==========================

First download the source::

    git clone git://github.com/commontk/ctk-cli-indexer.git

To use the module, you must install some external python package
dependencies: ::

    cd ctk-cli-indexer
    pip install -r requirements.txt

Elasticsearch Setup
===================

In order to use this code, you must have access to a running Elasticsearch_ server.  This
section shall give just the basic instructions for getting started.  First, download_ the
latest stable elasticsearch and kibana tarballs (logstash is not necessary / used here).

Elasticsearch_ is written in Java, so you can basically unpack the tarball and run
``bin/elasticsearch``, and the server should be running on http://localhost:9200/ (yes,
you can just try that URL in the browser, and you should get some status JSON).  This
default location is also built into ``index_from_json.py``, so you may immediately start
indexing.  One may use http://localhost:9200/cli/cli/_search?pretty=1 to check whether
there is data in the index.

Kibana_ is a purely browser-based web application (based on client-side HTML and JS), so
you can serve the files using any kind of HTTP server, e.g.::

  cd kibana-3.1.1/
  python -m SimpleHTTPServer

which will serve Kibana on http://localhost:8000/ You may even be able to use Kibana
without any HTTP server, just by opening ``kibana-x.y.z/index.html`` within your browser.
In that case, you may want to edit ``config.js`` to point to the server like this::

  elasticsearch: "http://localhost:9200",

That's it!  If you see the welcome dashboard in the browser, you're all set.  Note that
you can even store dashboards within Kibana; by default, they will be stored within
Elasticsearch, so you don't even have to care about filesystem access.

.. _download: http://www.elasticsearch.org/overview/elkdownloads/

First Steps with Kibana
=======================

I suggest to start with a blank dashboard (link at the bottom of the default dashboard).
Start by going to the dashboard settings (cog in the upper right corner) and under
"Index", select 'cli' as default index and enable autocompletion under "Preload Fields".

Next, add rows ("Rows" tab in the dashboard settings), for instance, one with 200px
height, one with 300px, and a third with 500px.  Don't forget to press "Create Row" for
each row (in particular, also for the last one), then press "Save".

Within each row, there is an (invisible) 12-column layout, so you want to add "widgets"
now that span either 3 or 4 such columns.  Start with "Terms" widgets only, try different
fields (e.g. "license"), and different view options (in particular, the bar/pie/table
styles).

The widgets allow interactive filtering, e.g. click on a specific term to filter by
license / author / source / category; active filters will be listed and can be cleared at
the top (make sure that line is not collapsed).  There is also a search row where you can
try entering keywords.

The last row (which we made particularly high) was intended for a "Table" widget (like on
the sample dashboard), which can be used to list all matching documents.

Now play around with the various options, and don't forget to save your dashboard (floppy
symbol near the upper right corner).  If you enable "Save to > Export" and "Load from >
Local file" under "Controls" in the dashboard settings, you can also download/upload the
dashboard as JSON.  Furthermore, you can make the dashboard your default/home dashboard.
Within this repository, you also find an example dashboard in the file `cli_dashboard.json`.
