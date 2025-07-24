.. image:: https://img.shields.io/badge/sqr--059-lsst.io-brightgreen.svg
   :target: https://sqr-059.lsst.io/
.. image:: https://github.com/lsst-sqre/sqr-059/workflows/CI/badge.svg
   :target: https://github.com/lsst-sqre/sqr-059/actions/

######################################
RSP Notebook container tag conventions
######################################

SQR-059
=======

We rely on particular tag formats to order and present Lab images to users.  This Technote is intended to formalize and document those formats.

**Links:**

- Publication URL: https://sqr-059.lsst.io/
- Alternative editions: https://sqr-059.lsst.io/v
- GitHub repository: https://github.com/lsst-sqre/sqr-059
- Build system: https://github.com/lsst-sqre/sqr-059/actions/

Build this technical note
=========================

You can clone this repository and build the technote locally if your system has Python 3.11 or later:

.. code-block:: bash

   git clone https://github.com/lsst-sqre/sqr-059
   cd sqr-059
   make init
   make html

Repeat the ``make html`` command to rebuild the technote after making changes.
If you need to delete any intermediate files for a clean build, run ``make clean``.

The built technote is located at ``_build/html/index.html``.

Publishing changes to the web
=============================

This technote is published to https://sqr-059.lsst.io/ whenever you push changes to the ``main`` branch on GitHub.
When you push changes to a another branch, a preview of the technote is published to https://sqr-059.lsst.io/v.

Editing this technical note
===========================

The main content of this technote is in ``index.rst`` (a reStructuredText file).
Metadata and configuration is in the ``technote.toml`` file.
For guidance on creating content and information about specifying metadata and configuration, see the Documenteer documentation: https://documenteer.lsst.io/technotes.
