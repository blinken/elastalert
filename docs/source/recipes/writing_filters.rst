.. _writingfilters:

Writing Filters For Rules
=========================

This document describes how to create a filter section for your rule config file.

The filters used in rules are part of the Elasticsearch query DSL, further documentation for which can be found at
https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html
This document contains a small subset of particularly useful filters.

The filter section is converted from YAML to JSON and passed to Elasticsearch exactly as follows::

    query:
      bool:
        must:
          - [filters from rule.yaml]

Every result that matches these filters will be passed to the rule for processing.

Common Filter Types:
--------------------

query_string
************

The query_string type follows the Lucene query format and can be used for partial or full matches to multiple fields.
See http://lucene.apache.org/core/2_9_4/queryparsersyntax.html for more information::

    filter:
    - query:
        query_string:
          query: "username: bob"
    - query:
        query_string:
          query: "_type: login_logs"
    - query:
        query_string:
          query: "field: value OR otherfield: othervalue"
    - query:
        query_string:
          query: "this: that AND these: those"

term
****

The term type allows for exact field matches::

    filter:
    - term:
        name_field: "bob"
    - term:
        _type: "login_logs"

Note that a term query may not behave as expected if a field is analyzed. By default, many string fields will be tokenized by whitespace, and a term query for "foo bar" may not match
a field that appears to have the value "foo bar", unless it is not analyzed. Conversely, a term query for "foo" will match analyzed strings "foo bar" and "foo baz". For full text
matching on analyzed fields, use query_string. See https://www.elastic.co/guide/en/elasticsearch/guide/current/term-vs-full-text.html

terms
*****

Terms allows for easy combination of multiple term filters::

    filter:
    - terms:
        field: ["value1", "value2"]

Using the minimum_should_match option, you can define a set of term filters of which a certain number must match::

    - terms:
        fieldX: ["value1", "value2"]
        fieldY: ["something", "something_else"]
        fieldZ: ["foo", "bar", "baz"]
        minimum_should_match: 2

wildcard
********

For wildcard matches::

    filter:
    - query:
        wildcard:
          field: "foo*bar"

range
*****

For ranges on fields::

    filter:
    - range:
        status_code:
          from: 500
          to: 599

Negation, and, or
*****************

Any of the filters can be embedded in ``must_not`` (boolean NOT), ``must`` (boolean AND), and ``should`` (boolean OR)::

    filter:
    - bool:
        should:
          - term:
              field: "value"
          - wildcard:
              field: "foo*bar"
          - bool:
              must:
                - term:
                    field: "value"
                - bool:
                    must_not:
                      - term:
                          _type: "something"

See https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html for more examples.


Loading Filters Directly From Kibana 3
--------------------------------------

There are two ways to load filters directly from a Kibana 3 dashboard. You can set your filter to::

    filter:
      download_dashboard: "My Dashboard Name"

and when ElastAlert starts, it will download the dashboard schema from Elasticsearch and use the filters from that.
However, if the dashboard name changes or if there is connectivity problems when ElastAlert starts, the rule will not load and
ElastAlert will exit with an error like "Could not download filters for .."

The second way is to generate a config file once using the Kibana dashboard. To do this, run ``elastalert-rule-from-kibana``.

.. code-block:: console

    $ elastalert-rule-from-kibana
    Elasticsearch host: elasticsearch.example.com
    Elasticsearch port: 14900
    Dashboard name: My Dashboard

    Partial Config file
    -----------

    name: My Dashboard
    es_host: elasticsearch.example.com
    es_port: 14900
    filter:
    - query:
        query_string: {query: '_exists_:log.message'}
    - query:
        query_string: {query: 'some_field:12345'}
