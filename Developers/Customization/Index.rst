.. ==================================================
.. FOR YOUR INFORMATION
.. --------------------------------------------------
.. -*- coding: utf-8 -*- with BOM.

.. include:: ../Includes.txt


.. _customization:

Customization
=============

TYPO3 extension developers can readily customize/extend the TYPO3 Forum by building a separate extension and overwriting certain aspects of the Forum extension. At the moment, two components support this approach.

**Editor**

   The *input* of posts. This typically an online text editor.

**TextParserOutput**

   The *output* of a single post.


Concept in General
------------------

Both components listed above have been decoupled from *normal* Fluid template files and are stored as partials:

* :code:`Resources/Private/Partials/Bootstrap/Editor.html`
* :code:`Resources/Private/Partials/Bootstrap/TextParserOutput.html`

This means other extensions can provide their own ``Editor.html`` and/or ``TextParserOutput.html`` partials, which can be configured with a higher priority in the loading order, and therefore take precedence over the partials of **EXT:typo3_forum**.

.. code-block:: typoscript

    plugin.tx_typo3forum {
       view {
          partialRootPaths {
             20 = EXT:myextension/Resources/Private/Partials/Bootstrap/
          }
       }
    }

For all partials, Fluid first checks if they exist in this path (priority: 20). If they don't, Fluid falls back to the original path (priority: 10).


Example
-------

Consider the following example extension:

.. code-block:: none

    myextension
    ├── Classes
    │   └── ViewHelpers
    │       └── ParserViewHelper.php
    ├── Configuration
    │   └── TypoScript
    │       └── setup.typoscript
    ├── Resources
    │   └── Private
    │       └── Partials
    │           └── Bootstrap
    │               └── TextParserOutput.html
    ├── ext_emconf.php
    └── ext_tables.php

The TypoScript configuration in file ``Configuration/TypoScript/setup.typoscript`` specifies the alternative path with priority 20 for example. As a result, whenever a single forum post is displayed, the partial ``TextParserOutput.html`` of **EXT:myextension** is used.

Instead of the original Fluid code, the content of this partial could pass the post to a ViewHelper:

.. code-block:: html

    {namespace myextension=Vendor\Myextension\ViewHelpers}
    <!-- Pass content to a custom ViewHelper named "Parser" -->
    <myextension:parser>
       {post.text}
    </myextension:parser>


Typical Use Cases
-----------------

A typical example for this type of customization is the post-processing of forum articles, e.g. additional styling or formatting, but also the implementation of an additional protection layer, e.g. anti-XSS or the detection and filtering of indecent and unwanted words, characters or expressions.

Given that the same approach is also supported for the **input** of forum posts, an extension could provide an alternative editor. For example `Summernote <https://summernote.org/>`__ or `CKEditor <http://ckeditor.com/>`__.

.. important::

   When implementing an alternative input or output solution, it is important to understand that you have to take of appropriate measures against cross site scripting and similar vulnerabilites. For example, it is **NOT** a good idea, to output post in its raw format: ``<f:format.raw>{post.text}</f:format.raw>``.
