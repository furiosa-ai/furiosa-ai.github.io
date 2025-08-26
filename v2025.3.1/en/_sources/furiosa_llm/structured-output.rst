.. _StructuredOutput:

*****************
Structured Output
*****************

Furiosa-LLM supports structured output generation, enabling you to constrain
the model's output to follow specific formats, schemas, or patterns.
This feature is essential for applications requiring consistent,
parsable responses such as JSON data extraction and any-formatted text generation.

.. note::

  Furiosa-LLM uses `llguidance <https://github.com/guidance-ai/llguidance>`_
  as the default backend for guided decoding. This backend ensures efficient constraint
  enforcement during the token generation process. Additional backends
  like `XGrammar <https://github.com/mlc-ai/xgrammar>`_ will be supported soon,
  providing even more grammar format options and enhanced performance.

Supported Methods via OpenAI API
================================
Users can generate structured outputs using both OpenAI's `Completions API <https://platform.openai.com/docs/api-reference/completions>`_ and
`Chat Completions API <https://platform.openai.com/docs/api-reference/chat>`_.
Furiosa-LLM supports four main types of structured output constraints:

- Choices: Forces the output to be one of predefined options
- Regular Expressions: Generates text matching a specific regular expression pattern
- JSON Schema: Produces JSON output following a predefined schema
- Context-free Grammar: Generates text following a context-free grammar specification

.. _GuidedChoice:

Choices
-------

The ``guided_choice`` parameter constrains the model output to be one of a predefined set of choices. This is particularly useful for classification tasks or when you need the model to select from specific options.

**Usage Example:**

.. literalinclude:: ../../../examples/structured_output_guided_choice.py
   :language: python

**Use Cases:**

* Sentiment analysis
* Category classification
* Multiple choice questions
* Binary decisions (yes/no, true/false)

.. _GuidedRegex:

Regular Expressions
-------------------

The ``guided_regex`` parameter ensures the generated text matches a specific regular expression pattern. This is useful for generating structured text like email addresses, phone numbers, or custom formats.

**Usage Example:**

.. literalinclude:: ../../../examples/structured_output_guided_regex.py
   :language: python

**Use Cases:**

* Email address generation
* Phone number formatting
* ID or code generation
* Custom text patterns
* URL or file path generation

.. _GuidedJSON:

JSON Schema
-----------

The ``guided_json`` parameter is the most commonly used structured output method. It ensures the generated output is valid JSON that conforms to a specified schema. This is ideal for extracting structured data or creating consistent API responses.

**Usage Example:**

.. literalinclude:: ../../../examples/structured_output_guided_json.py
   :language: python

**Use Cases:**

* Data extraction from unstructured text
* API response formatting
* Database record creation
* Configuration file generation

.. _GuidedGrammar:

Context-free Grammar
--------------------

The ``guided_grammar`` parameter allows you to define a context-free grammar specification that the generated text must follow. This provides the most flexible control over output structure.

**Grammar Format Support:**

The llguidance backend currently supports multiple grammar formats:

- **EBNF (Extended Backus-Naur Form)**: Standard grammar notation with ``::=`` syntax
- **Lark**: a modern parsing library for Python (`Lark document <https://lark-parser.readthedocs.io/en/stable/>`_)

**Usage Example:**

.. literalinclude:: ../../../examples/structured_output_guided_grammar.py
   :language: python

The example above uses EBNF format with the ``::=`` notation. You can also use Lark format with different syntax rules depending on your grammar complexity requirements.

**Use Cases:**

* SQL query generation
* Code generation with specific syntax
* Mathematical expressions
* Custom domain-specific languages
* Complex structured formats
