Normalization Rules
===================

Normalization rules are applied to fields, also in ``schema`` for mappings, as
well when defined as a bulk operation by ``schema`` (for sequences),
``allow_unknown``, ``propertyschema`` and ``valueschema``.  Normalization rules
in definitions for testing variants like with ``anyof`` are not processed.


Renaming Of Fields
------------------
You can define a field to be renamed before any further processing.

.. doctest::

   >>> v = Validator({'foo': {'rename': 'bar'}})
   >>> v.normalized({'foo': 0})
   {'bar': 0}

To let a callable rename a field or arbitrary fields, you can define a handler
for renaming. If the constraint is a string, it points to a
:doc:`custom method <customize>`. If the constraint is an iterable, the value
is processed through that chain.

.. doctest::

   >>> v = Validator({}, allow_unknown={'rename_handler': int})
   >>> v.normalized({'0': 'foo'})
   {0: 'foo'}

.. doctest::

   >>> even_digits = lambda x: '0' + x if len(x) % 2 else x
   >>> v = Validator({}, allow_unknown={'rename_handler': [str, even_digits]})
   >>> v.normalized({1: 'foo'})
   {'01': 'foo'}


.. versionadded:: 0.10

Purging Unknown Fields
----------------------
After renaming, unknown fields will be purged if the
:attr:`~cerberus.Validator.purge_unknown` property of a
:class:`~cerberus.Validator` instance is ``True``; it defaults to ``False``.
You can set the property per keyword-argument upon initialization or as rule for
subdocuments like ``allow_unknown`` (see :ref:`allowing-the-unknown`). The default is
``False``.

.. doctest::

   >>> v = Validator({'foo': {'type': 'string'}}, purge_unknown=True)
   >>> v.normalized({'bar': 'foo'})
   {}

.. versionadded:: 0.10

Default Values
--------------
You can set default values for missing fields in the document by using the ``default`` rule.

.. doctest::

   >>> v.schema = {'amount': {'type': 'integer'}, 'kind': {'type': 'string', 'default': 'purchase'}}
   >>> v.normalized({'amount': 1})
   {'amount': 1, 'kind': 'purchase'}

   >>> v.normalized({'amount': 1, 'kind': None})
   {'amount': 1, 'kind': 'purchase'}

   >>> v.normalized({'amount': 1, 'kind': 'other'})
   {'amount': 1, 'kind': 'other'}

.. versionadded:: 0.10

.. _type-coercion:

Value Coercion
--------------
Coercion allows you to apply a callable (given as object or the name of a
:doc:`custom normalization method <customize>`) to a value before the document
is validated. The return value of the callable replaces the new value in the
document. This can be used to convert values or sanitize data before it is
validated.  If the constraint is an iterable, the value is processed through
that chain.

.. doctest::

   >>> v.schema = {'amount': {'type': 'integer'}}
   >>> v.validate({'amount': '1'})
   False

   >>> v.schema = {'amount': {'type': 'integer', 'coerce': int}}
   >>> v.validate({'amount': '1'})
   True
   >>> v.document
   {'amount': 1}

   >>> to_bool = lambda v: v.lower() in ['true', '1']
   >>> v.schema = {'flag': {'type': 'boolean', 'coerce': to_bool}}
   >>> v.validate({'flag': 'true'})
   True
   >>> v.document
   {'flag': True}

.. versionadded:: 0.9
