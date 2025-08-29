==========
WizardHTML
==========


.. figure:: _static/img/WizardHTMLBanner.png
   :alt: WizardHTML Banner
   :width: 800
   :height: 300
   :align: center

.. image:: https://img.shields.io/pypi/v/wizardhtml.svg
   :target: https://pypi.org/project/wizardhtml/
   :alt: PyPI - Version

.. image:: https://img.shields.io/pypi/dm/wizardhtml.svg?label=PyPI%20downloads
   :target: https://pypistats.org/packages/wizardhtml
   :alt: PyPI - Downloads/month

.. image:: https://img.shields.io/pypi/l/wizardhtml.svg
   :target: https://github.com/textwizard-dev/wizardhtml/blob/main/LICENSE
   :alt: License



WHATWG-compliant HTML5 toolkit: DFA tokenizer, spec-guided tree builder, DOM,
configurable serializer, and high-level helpers for cleaning, pretty-printing, and HTML→Markdown.


Installation
============

Requires Python 3.9+.

.. code-block:: bash

   pip install wizardhtml

Quick start
===========

.. code-block:: python

   import wizardhtml as wh

   # Mode A: text-only extraction
   print(wh.clean_html("<div><p>Hello</p><script>x()</script></div>"))
   # -> "Hello"

   # Pretty print
   html = "<body><p>Hi <b>there</b></p><img src=x></body>"
   print(wh.beautiful_html(html, indent=2))

   # HTML → Markdown
   print(wh.html_to_markdown("<h1>T</h1><p>Body</p>"))

   # Parser and DOM
   doc = wh.parse("<!doctype html><html><body><p>Hi</p></body></html>")

Public API
=========================

.. list-table::
   :header-rows: 1
   :widths: 50 40

   * - Function
     - Purpose
   * - ``parse(html, fragment_context=None, return_errors=False)``
     - Parse into ``Document`` or ``DocumentFragment``; optional parse error list.
   * - ``clean_html(text, **flags)``
     - High-level HTML cleaning with A/B/C modes.
   * - ``beautiful_html(html, **opts)``
     - Non-destructive pretty-printer for HTML.
   * - ``html_to_markdown(html)``
     - Convert HTML → Markdown.
   * - ``serialize(node, **opts)``
     - Serialize DOM → HTML.
   * - ``to_text(html, separator="\\n", strip=True, collapse_ws=True)``
     - Extract readable text (internally uses Mode A, then normalizes whitespace).

DOM types
=========

``Node``, ``Document``, ``DocumentFragment``, ``Element``, ``Text``, ``Comment``.

Parsing
=======

Signature
---------

.. code-block:: python

   import wizardhtml as wh

   wh.parse(
       html: str,
       fragment_context: str | None = None,
       return_errors: bool = False,
   ) -> Document | DocumentFragment | tuple[Document | DocumentFragment, list[str]]

Behavior
--------

- **Full document** when ``fragment_context is None`` → returns ``Document``.
- **Fragment parsing** when ``fragment_context`` is an element name
  (e.g. ``"div"``, ``"template"``, ``"tbody"``, ``"svg"``, ``"math"``) → returns ``DocumentFragment``.
  Tokenizer state and insertion mode follow WHATWG rules for the context element.
- With ``return_errors=True`` returns ``(node, errors: list[str])`` where errors are informative.

Examples
--------

Full document:

.. code-block:: python

   import wizardhtml as wh
   doc = wh.parse("<!doctype html><html><body><p>Hi</p></body></html>")

Fragment:

.. code-block:: python

   import wizardhtml as wh
   frag = wh.parse("<li>item</li>", fragment_context="ul")

Collecting parse errors:

.. code-block:: python

   import wizardhtml as wh
   node, errors = wh.parse("<p><b>x</p>", return_errors=True)
   print(errors)



HTML cleaning
=============

HTML cleanup with granular switches for scripts, metadata, embedded media, interactive elements, headings, phrasing content, and more.  
Supports wildcard-based *tag* and *attribute* removal, selective content stripping, and empty-node pruning. Returns **text** or **HTML** depending on the mode.

Behavior
--------

Three explicit modes with different outputs:

+-----------------------------------------------+--------------------------------------------+-------------------------+--------------------------------------------------------------+
| **Mode**                                      | **How to trigger**                         | **Returns**             | **Description**                                              |
+===============================================+============================================+=========================+==============================================================+
| **A) text-only**                              | No parameters provided (all ``None``)      | ``str`` (plain text)    | Extracts text, skips script-supporting tags, inserts spaces. |
+-----------------------------------------------+--------------------------------------------+-------------------------+--------------------------------------------------------------+
| **B) structural clean**                       | At least one flag is ``True``              | ``str`` (HTML)          | Removes/unwraps per flags and serializes sanitized HTML.     |
+-----------------------------------------------+--------------------------------------------+-------------------------+--------------------------------------------------------------+
| **C) text+preserve**                          | Parameters present and all are ``False``   | ``str`` (text+markup)   | Extracts text but **preserves** groups explicitly set False. |
+-----------------------------------------------+--------------------------------------------+-------------------------+--------------------------------------------------------------+

.. note::
   When deleting nodes between adjacent text nodes, the cleaner inserts **one space** to avoid word concatenation.  
   In Mode B the serializer uses ``quote_attr_values="always"`` for stable diffs.

Parameters 
----------

+-------------------------------+--------------------------------------------------------------------------+
| **Parameter**                 | **Description**                                                          |
+===============================+==========================================================================+
| ``text``                      | (*str*) Raw HTML input.                                                  |
+-------------------------------+--------------------------------------------------------------------------+
| ``remove_script``             | (*bool | None*) Drop executable tags (``<script>``, ``<template>``).     |
+-------------------------------+--------------------------------------------------------------------------+
| ``remove_metadata_tags``      | (*bool | None*) Drop metadata (``<link>``, ``<meta>``, ``<base>``,       |
|                               | ``<noscript>``, ``<style>``, ``<title>``).                               |
+-------------------------------+--------------------------------------------------------------------------+
| ``remove_flow_tags``          | (*bool | None*) Drop flow content (layout + phrasing, e.g. ``<div>``,    |
|                               | ``<p>``, ``<span>``, ``<input>``).                                       |
+-------------------------------+--------------------------------------------------------------------------+
| ``remove_sectioning_tags``    | (*bool | None*) Drop sectioning (``<article>``, ``<aside>``, ``<nav>``,  |
|                               | ``<section>``).                                                          |
+-------------------------------+--------------------------------------------------------------------------+
| ``remove_heading_tags``       | (*bool | None*) Drop headings ``<h1>``–``<h6>``.                         |
+-------------------------------+--------------------------------------------------------------------------+
| ``remove_phrasing_tags``      | (*bool | None*) Drop phrasing (inline) elements, e.g. ``<span>``,        |
|                               | ``<strong>``, ``<img>``, ``<code>``, ``<svg>``, ``<textarea>``.          |
+-------------------------------+--------------------------------------------------------------------------+
| ``remove_embedded_tags``      | (*bool | None*) Drop embedded content (``<img>``, ``<video>``,           |
|                               | ``<iframe>``, ``<embed>``, ``<object>``, ``<svg>``, ``<math>``).         |
+-------------------------------+--------------------------------------------------------------------------+
| ``remove_interactive_tags``   | (*bool | None*) Drop interactive elements (``<button>``, ``<input>``,    |
|                               | ``<select>``, ``<label>``, ``<textarea>``, interactive media).           |
+-------------------------------+--------------------------------------------------------------------------+
| ``remove_palpable``           | (*bool | None*) Drop palpable elements (broad set incl. ``<table>``,     |
|                               | ``<section>``, ``<p>``, ``<ul>``, etc.).                                 |
+-------------------------------+--------------------------------------------------------------------------+
| ``remove_doctype``            | (*bool | None*) Remove ``<!DOCTYPE html>``.                              |
+-------------------------------+--------------------------------------------------------------------------+
| ``remove_comments``           | (*bool | None*) Remove ``<!-- ... -->`` comments.                        |
+-------------------------------+--------------------------------------------------------------------------+
| ``remove_specific_attributes``| (*str | list | None*) Remove attributes by name or wildcard              |
|                               | (e.g. ``"id"``, ``"data-*"``, ``"on*"``).                                |
+-------------------------------+--------------------------------------------------------------------------+
| ``remove_specific_tags``      | (*str | list | None*) **Unwrap** tags by name or wildcard                |
|                               | (children are lifted into parent).                                       |
+-------------------------------+--------------------------------------------------------------------------+
| ``remove_empty_tags``         | (*bool | None*) Prune empty nodes after edits.                           |
+-------------------------------+--------------------------------------------------------------------------+
| ``remove_content_tags``       | (*str | list | None*) Keep tag but drop inner content.                   |
+-------------------------------+--------------------------------------------------------------------------+
| ``remove_tags_and_contents``  | (*str | list | None*) Remove tag **and** its entire content.             |
+-------------------------------+--------------------------------------------------------------------------+



Parameter semantics
-------------------

- **None** → flag **unset**. If all are None ⇒ **Mode A**.  
- **True** → request removal/operation ⇒ **Mode B**.  
- **False** → request preservation ⇒ **Mode C** (text output that preserves those groups; ``remove_comments=False`` and ``remove_doctype=False`` also preserve them).

Tag groups reference
--------------------

.. list-table::
   :header-rows: 1
   :widths: 22 78

   * - **Flag**
     - **Tags affected**
   * - ``remove_script``
     - ``script``, ``template``
   * - ``remove_metadata_tags``
     - ``base``, ``link``, ``meta``, ``noscript``, ``script``, ``style``, ``title``
   * - ``remove_flow_tags``
     - ``address``, ``article``, ``aside``, ``blockquote``, ``div``, ``dl``, ``fieldset``, ``figcaption``, ``figure``, ``footer``, ``form``, ``h1``, ``h2``, ``h3``, ``h4``, ``h5``, ``h6``, 
       ``header``, ``hgroup``, ``hr``, ``main``, ``nav``, ``ol``, ``p``, ``pre``, ``section``, ``ul``, 
       ``a``, ``abbr``, ``b``, ``bdi``, ``bdo``, ``br``, ``button``, ``cite``, ``code``, ``data``, ``dfn``, ``em``, 
       ``i``, ``img``, ``input``, ``kbd``, ``label``, ``mark``, ``q``, ``ruby``, ``s``, ``samp``, ``small``, ``span``, 
       ``strong``, ``sub``, ``sup``, ``time``, ``u``, ``var``, ``wbr``
   * - ``remove_sectioning_tags``
     - ``article``, ``aside``, ``nav``, ``section``
   * - ``remove_heading_tags``
     - ``h1``, ``h2``, ``h3``, ``h4``, ``h5``, ``h6``
   * - ``remove_phrasing_tags``
     - ``abbr``, ``audio``, ``b``, ``bdi``, ``bdo``, ``br``, ``button``, ``cite``, ``code``, ``data``, ``dfn``, ``em``, 
       ``i``, ``img``, ``input``, ``kbd``, ``label``, ``mark``, ``math``, ``meter``, ``noscript``, ``object``, ``output``, 
       ``progress``, ``q``, ``ruby``, ``s``, ``samp``, ``script``, ``select``, ``small``, ``span``, ``strong``, 
       ``sub``, ``sup``, ``svg``, ``template``, ``textarea``, ``time``, ``u``, ``var``, ``wbr``
   * - ``remove_embedded_tags``
     - ``audio``, ``canvas``, ``embed``, ``iframe``, ``img``, ``map``, ``object``, ``picture``, ``svg``, ``video``, ``math``
   * - ``remove_interactive_tags``
     - ``a``, ``audio``, ``button``, ``details``, ``embed``, ``iframe``, ``img``, ``input``, ``keygen``, ``label``, ``select``, ``textarea``, ``video``
   * - ``remove_palpable``
     - ``a``, ``abbr``, ``address``, ``article``, ``aside``, ``audio``, ``b``, ``bdi``, ``bdo``, ``blockquote``, ``button``, 
       ``canvas``, ``cite``, ``code``, ``data``, ``del``, ``details``, ``dfn``, ``div``, ``dl``, ``em``, ``embed``, 
       ``fieldset``, ``figure``, ``footer``, ``form``, ``h1``, ``h2``, ``h3``, ``h4``, ``h5``, ``h6``, ``header``, ``hgroup``, 
       ``i``, ``iframe``, ``img``, ``input``, ``kbd``, ``label``, ``main``, ``map``, ``mark``, ``math``, ``menu``, ``meter``, 
       ``nav``, ``object``, ``ol``, ``output``, ``p``, ``picture``, ``pre``, ``progress``, ``q``, ``ruby``, ``s``, ``samp``, 
       ``search``, ``section``, ``select``, ``small``, ``span``, ``strong``, ``sub``, ``sup``, ``svg``, ``table``, 
       ``textarea``, ``time``, ``u``, ``ul``, ``var``, ``video``

Examples
--------

Mode A — text only
~~~~~~~~~~~~~~~~~~

.. code-block:: python

   import wizardhtml as wh
   txt = wh.clean_html("<div><p>Hello</p><script>x()</script></div>")
   print(txt)

**Output**

.. code-block:: text

   Hello

Mode B — structural clean (HTML out)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Drop scripts, metadata, embeds; strip attributes; prune empties.

.. code-block:: python

   import wizardhtml as wh

   html = """
   <html><head>
     <title>x</title><meta charset="utf-8">
     <link rel="preload" href="x.css"><script>evil()</script>
   </head>
   <body>
     <article><h1>Title</h1><img src="a.png"><p id="k" onclick="x()">hello</p></article>
     <!-- comment -->
   </body></html>
   """
   out = wh.clean_html(
       html,
       remove_script=True,
       remove_metadata_tags=True,
       remove_embedded_tags=True,
       remove_specific_attributes=["id", "on*"],
       remove_empty_tags=True,
       remove_comments=True,
       remove_doctype=True,
   )
   print(out)

**Output**

.. code-block:: html

   <html>
     <body>
       <article><h1>Title</h1><p>hello</p></article>
     </body>
   </html>

Wildcards and unwrap vs hard remove:

.. code-block:: python

   import wizardhtml as wh

   html = """
   <div id="hero" data-track="x">
     <svg viewBox="0 0 10 10"><circle r="5"/></svg>
     <p class="k" onclick="hack()">Hello</p>
     <iframe src="a.html"></iframe>
   </div>
   """
   out = wh.clean_html(
       html,
       remove_tags_and_contents=["iframe", "template"],
       remove_specific_attributes=["id", "data-*", "on*"],
       remove_empty_tags=True,
   )
   print(out)

**Output**

.. code-block:: html

   <html><body><div>
     <p class="k">Hello</p>
   </div></body></html>

Content stripping vs tag deletion:

.. code-block:: python

   import wizardhtml as wh

   html = """
   <article>
     <script>track()</script>
     <style>p{}</style>
     <pre>code stays</pre>
     <noscript>fallback</noscript>
   </article>
   """
   keep_tags_drop_content = wh.clean_html(
       html,
       remove_content_tags=["script","style"],     # keep <script>/<style> but empty them
   )
   print(keep_tags_drop_content)

**Output**

.. code-block:: html

   <html><head></head><body><article>
     <script></script>
     <style></style>
     <pre>code stays</pre>
     <noscript>fallback</noscript>
   </article></body></html>

Sectioning, headings, flow:

.. code-block:: python

   import wizardhtml as wh

   html = "<section><h1>T</h1><div><address>X</address><p>Body</p></div></section>"
   out = wh.clean_html(
       html,
       remove_sectioning_tags=True,  # drop <section>/<article>/<aside>/<nav>
       remove_heading_tags=True,     # drop <h1>-<h6>
   )
   print(out)

**Output**

.. code-block:: html

   <html><head></head><body></body></html>

Interactive and embedded:

.. code-block:: python

   import wizardhtml as wh

   html = """
   <button id="b" disabled>Click</button>
   <img src="logo.png" alt="Logo">
   <video src="v.mp4"></video>
   """
   out = wh.clean_html(
       html,
       remove_interactive_tags=True,  # button, input, select
       remove_embedded_tags=True,     # img, iframe, embed, video, audio
       remove_specific_attributes=["id"],
       remove_empty_tags=True
   )
   print(out)  # empty string if everything got removed

Mode C — text with preservation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Preserve sectioning + headings + comments:

.. code-block:: python

   import wizardhtml as wh

   html = "<article><h1>T</h1><p>Body</p><!-- c --></article>"
   txt = wh.clean_html(
       html,
       remove_sectioning_tags=False,
       remove_heading_tags=False,
       remove_comments=False,
   )
   print(txt)

**Output**

.. code-block:: html

   <article><h1>T</h1>Body<!-- c --></article>

Preserve images but text-only elsewhere:

.. code-block:: python

   import wizardhtml as wh

   html = '<p>A<img src="a.png" alt="A">B</p>'
   txt = wh.clean_html(
       html,
       remove_embedded_tags=False,   # keep <img>
   )
   print(txt)

**Output**

.. code-block:: html

   A<img src="a.png" alt="A">B

Operational notes
-----------------

- When deleting nodes between adjacent text nodes, the cleaner inserts **one space** to avoid word concatenation.
- In Mode B the serializer prefers stable quoting for diff-friendly output.
- If the DOM becomes empty after removals, returns ``""``.


Text helper
===========

Extract readable text with whitespace normalization.

.. code-block:: python

   import wizardhtml as wh
   txt = wh.to_text("<div> A <b> B </b>\n\n <i>C</i></div>", separator=" ")
   print(txt)  # "A B C"
   
   


Beautiful HTML
==============

Pretty-print raw HTML **without changing semantics**. The formatter parses *html*,
serializes a normalized DOM, and indents nodes by a configurable amount. It never
reflows RCData content (``<script>``, ``<style>``, ``<textarea>``) and avoids introducing
visible whitespace unless explicitly requested.


Parameters
----------

- ``html`` (str): Raw HTML input.
- ``indent`` (int, default ``2``): Spaces per indentation level.
- ``quote_attr_values`` ({``"always"``, ``"spec"``, ``"legacy"``}, default ``"spec"``):
  Quoting policy for attribute values.
  - ``"always"`` → always quote.
  - ``"spec"``  → quote only when required by the HTML5 spec (space, quotes, ``=``, ``<``, ``>``, backtick).
  - ``"legacy"`` → legacy behavior; quote only for whitespace or quotes.
- ``quote_char`` ({``'"'``, ``"'"``}, default ``'"'``): Preferred quote character when quoting.
- ``use_best_quote_char`` (bool, default ``True``): Choose the quote character that minimizes escaping per attribute.
- ``minimize_boolean_attributes`` (bool, default ``False``): Render compact boolean attributes (e.g., ``disabled`` instead of ``disabled="disabled"``).
- ``use_trailing_solidus`` (bool, default ``False``): Emit a trailing solidus on void elements (``<br />``). Cosmetic in HTML5.
- ``space_before_trailing_solidus`` (bool, default ``True``): Insert a space before the trailing solidus if it is used.
- ``escape_lt_in_attrs`` (bool, default ``False``): Escape ``<``/``>`` inside attribute values.
- ``escape_rcdata`` (bool, default ``False``): Escape characters inside RCData elements (usually keep ``False``).
- ``resolve_entities`` (bool, default ``True``): Prefer named entities where available during serialization.
- ``alphabetical_attributes`` (bool, default ``True``): Sort attributes alphabetically (useful for diff-friendly output).
- ``strip_whitespace`` (bool, default ``False``): Trim leading/trailing whitespace in text nodes and collapse runs of spaces.
- ``include_doctype`` (bool, default ``True``): Prepend ``<!DOCTYPE html>`` if missing.
- ``expand_mixed_content`` (bool, default ``True``): For elements that contain both text and child elements, place each child on its own indented line (may introduce visible whitespace in inline contexts).
- ``expand_empty_elements`` (bool, default ``True``): Render empty non-void elements on two lines (open/close on separate lines).


Examples
--------

Basic pretty-print:

.. code-block:: python

   import wizardhtml as wh

   html = """
   <body>
     <button id='btn1' class="primary" disabled="disabled">
       Click   <b>me</b>
     </button>
     <img alt="Logo" src="/static/logo.png">
   </body>
   """
   pretty = wh.beautiful_html(
       html=html,
       indent=4,
       alphabetical_attributes=True,
       minimize_boolean_attributes=True,
       quote_attr_values="always",
       strip_whitespace=True,
       include_doctype=True,
       expand_mixed_content=True,
       expand_empty_elements=True,
   )
   print(pretty)

**Output**

.. code-block:: html

    <!DOCTYPE html>
    <html>
        <head>
        </head>
        <body>
    
            <button class="primary" disabled id="btn1">
                Click
                <b>
                    me
                </b>
    
            </button>
    
            <img alt="Logo" src="/static/logo.png">
    
        </body>
    </html>

Quote policies & best quote char
--------------------------------

.. code-block:: python

   import wizardhtml as wh

   html = '<a data-title=\'He said "hi"\'>x</a>'
   out = wh.beautiful_html(
       html,
       quote_attr_values="always",
       quote_char='"',
       use_best_quote_char=True,  # picks ' to minimize escaping
   )
   print(out)

**Output**

.. code-block:: html

   <!DOCTYPE html>
   <html>
     <head></head>
     <body>
       <a data-title='He said "hi"'>
         x
       </a>
     </body>
   </html>


Void elements and trailing solidus
----------------------------------

.. code-block:: python

   import wizardhtml as wh

   html = "<br><img src=x>"
   out = wh.beautiful_html(
       html,
       use_trailing_solidus=True,
       space_before_trailing_solidus=False,
   )
   print(out)

**Output**

.. code-block:: html

   <!DOCTYPE html>
   <html>
     <head></head>
     <body>
       <br/>
       <img src=x/>
     </body>
   </html>

Whitespace & mixed content
--------------------------

.. code-block:: python

   import wizardhtml as wh

   html = "<p>Hello <b>world</b>!</p>"
   out = wh.beautiful_html(
       html,
       expand_mixed_content=True,   # puts <b> on its own line
       strip_whitespace=False,
   )
   print(out)

**Output**

.. code-block:: html

   <!DOCTYPE html>
   <html>
     <head></head>
     <body>
       <p>
         Hello
         <b>
           world
         </b>
         !
       </p>
     </body>
   </html>


Serialization
=============

Signature
---------

.. code-block:: python

   import wizardhtml as wh

   wh.serialize(
       node,
       *,
       quote_attr_values: str = "spec",     # "spec" | "legacy" | "always"
       quote_char: str = '"',
       use_best_quote_char: bool = True,
       minimize_boolean_attributes: bool = False,
       resolve_entities: bool = True,
       alphabetical_attributes: bool = False,
       strip_whitespace: bool = False,
       include_doctype: bool = True,
   ) -> str

Notes
-----

- ``include_doctype`` is effective only when ``node`` is a ``Document``.
- ``alphabetical_attributes=True`` is useful for diff-friendly output.
- Does not alter DOM structure.



HTML → Markdown
===============

Best-effort conversion of common HTML structures to Markdown (headings, paragraphs,
inline emphasis/code, lists, links, images, blockquotes, code blocks, horizontal rules).
Attributes and presentational markup are ignored. When the input cannot be converted
safely, the original HTML is returned unchanged.

Parameters
----------

- ``html`` (str): Raw HTML input.

Return value
------------

- ``str`` — Markdown representation of the input HTML (or the original HTML if conversion is not applicable).

Examples
--------

Basic
-----

.. code-block:: python

   import wizardhtml as wh

   md = wh.html_to_markdown("<h1>Hello</h1><p>World</p>")
   print(md)

**Output**

.. code-block:: markdown

   # Hello

   World

Links, lists, code
------------------

.. code-block:: python

   import wizardhtml as wh

   html = """
   <h2>Quick links</h2>
   <p>Visit <a href="https://example.com">our site</a>.</p>
   <ul>
     <li><strong>One</strong></li>
     <li>Two</li>
   </ul>
   <pre><code>print("hi")</code></pre>
   <hr>
   """
   print(wh.html_to_markdown(html))

**Output**

.. code-block:: markdown

    ## Quick links
    
    Visit [our site](https://example.com)\.
    
    - **One**
    - Two
    
    ```
    print("hi")
    ```
    
---

Errors & validation
===================

- ``TypeError`` on non-string input for ``parse``/``to_text``/``clean_html``/``beautiful_html``.
- Malformed markup is normalized whenever possible following WHATWG rules.
- When the DOM becomes empty after removals, ``clean_html`` returns ``""``.

License
=======

`AGPL-3.0-or-later <_static/LICENSE>`_.

Resources
=========

- `PyPI Package <https://pypi.org/project/wizardhtml/>`_
- `Documentation <https://wizardhtml.readthedocs.io/en/latest/>`_
- `GitHub Repository <https://github.com/textwizard-dev/wizardhtml>`_

.. _contact_author:

Contact & Author
================

:Author: Mattia Rubino
:Email: `textwizard.dev@gmail.com <mailto:textwizard.dev@gmail.com>`_