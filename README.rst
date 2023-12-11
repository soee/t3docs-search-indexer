TYPO3 Documentation Search
==========================

Install locally
---------------

* Clone this repo ``git clone https://github.com/TYPO3-Documentation/t3docs-search-indexer.git``

* Enter the ``t3docs-search-indexer``

* Put rendered documentation inside the ``_docs`` folder. You should have a structure like ``_docs/docs.typo3.org/Web``.
  If you are using some IDE which indexes all files from the project folder, you should exclude the ``_docs`` folder from indexing.

* Run ``ddev start`` while been in the ``t3docs-search-indexer`` folder

* Run ``ddev exec composer global require t3g/elasticorn:^7.0`` to install Elasticorn

* Create elasticsearch index via Elasticorn:

  ``ddev exec  ~/.composer/vendor/bin/elasticorn.php index:init -c config/Elasticorn``

* If necessary adapt ``DOCS_ROOT_PATH`` in your ``.env`` file if needed (see .env.dist for examples).
  DDEV environment has ``DOCS_ROOT_PATH=../-docs/docs.typo3.org/Web`` set up by default, so usually
  you don't need to change it if you followed the folder structure.

* Index documents as described below in "Usage" section

* Enjoy the local search under ``https://t3docs-search-indexer.ddev.site/``

Configuration
-------------

Configure assets
^^^^^^^^^^^^^^^^

* Assets configuration is located in ``services.yml`` file in ``assets`` section

* For rendering assets in template use Twig function ``{{ render_assets($assetType, $assetLocation) | raw }}`` where $assetType is ``js`` or ``css``
and ``$assetLocation`` is ``header`` or ``footer``

* You can define assets for ``header`` and ``footer`` parts of templates in in ``services.yml`` file in ``assets``

Usage
-----

Common instructions for docsearch indexer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Docsearch indexer configuration is keep in the ``services.yml`` file in ``docsearch`` section.

* You can configure 2 kinds of directories:

    * allowed_paths - regular expressions for paths which should be indexed by Indexer

    * excluded_directories - directories which should be ignored by Indexer

Index docs
^^^^^^^^^^

* Start elasticsearch.

* Run ``./bin/console docsearch:import`` to index all documentations from configured
  root path (DOCS_ROOT_PATH) folder (taking into account configured ``allowed_paths``
  and ``excluded_directories``).

* Open `https://t3docs-search-indexer.ddev.site:9201/docsearch/_search?q=*:*` to see indexed
  documents.

* enter `https://t3docs-search-indexer.ddev.site` to see application

Index single manual
^^^^^^^^^^^^^^^^^^^

* Run ``ddev exec ./bin/console docsearch:import <packagePath>`` where ``packagePath`` is
   a path to manual (or manuals) you want to import, relative to ``DOCS_ROOT_PATH``.
   This command doesn't check ``allowed_paths``, to ease usage when indexing single
   documentation folder from custom location (so you don't have to recreate folder
   structure from docs server).
   e.g. ``ddev exec ./bin/console docsearch:import c/typo3/cms-felogin/12.4``
   to import EXT:felogin documentation for v12

* Open `https://t3docs-search-indexer.ddev.site:9200/docsearch_english_a/_search?q=*:*` to see indexed
  documents.

Removing index to start fresh
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you want to start with fresh Elasticsearch index locally, you can use chrome extensions
like `Elasticvue` to clear/drop Elasticsearch index if necessary.

As a uri you can simply enter ``https://t3docs-search-indexer.ddev.site:9200`` or ``http://localhost:XXXX`` with any of the
host ports as listed by ``ddev describe``.


Indexing Core changelog
^^^^^^^^^^^^^^^^^^^^^^^

Core changelog is treated as a "sub manual" of the core manual. To index it, just run indexing for `cms-core` manual.

To avoid duplicates search is indexing Core changelog only from "main" version/branch of the core documentation.
E.g. when you run ``./bin/console docsearch:import c/typo3/cms-core/main/`` then the changelog for all versions will be indexed,
but if you run `./bin/console docsearch:import c/typo3/cms-core/12.4/` the changelog will NOT be indexed.