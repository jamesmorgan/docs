.. Connext documentation master file, created by
   sphinx-quickstart on Wed Apr  3 12:06:15 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Connext Documentation
===================================

Connext is an infrastructure layer on top of Ethereum that lets projects do instant, high volume transactions with arbitrarily complex conditions for settlement. Connext does this by batching signed Ethereum interactions which can all be settled later on the blockchain without needing to trust intermediaries.

These docs cover both the background of Connext and how you get started integrating Connext into your Ethereum wallet or application.

""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

*********
Community
*********

`Join the Connext Community <discord.gg/yKkzZZm>`_ to meet the team, discuss development, and hang out!

""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

*********
Contents
*********

.. toctree::
    :maxdepth: 2
    :caption: Background

    background/introduction
    background/faq


.. toctree::
    :maxdepth: 2
    :caption: User Documentation

    usage/gettingStarted
    develop/client


.. toctree::
    :maxdepth: 2
    :caption: Contributor Documentation

    background/architecture
    usage/coreConcepts
    advanced/runHub
    develop/contracts
    develop/hub
    develop/types
    CONTRIBUTING


