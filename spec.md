# New Haystack JSON encoding specification

## Overview

JSON stands for JavaScript Object Notation. It is a plain text format commonly used for serialization of data.
It is specified in [RFC 4627](https://tools.ietf.org/html/rfc4627). The JSON format is designed to support 100% fidelity with with the full Haystack type system.

Hayson is the nickname given to this encoding scheme. It supersedes the [original Haystack JSON encoding](https://project-haystack.org/doc/Json).

## Format

### Kind

Why does `_kind` start with an underscore?

- An underscore is invalid as a tag name therefore making it a valid symbol to be used.
- `_kind` is unlikely to be used and hence will hopefully not clash with other data structures converted to Hayson.
- An underscore is a common prefix used fro system names. For example, MongoDB uses an underscore for a primary key.
- An underscore is explicit and has intention. It's less likely that it will be auto-corrected by a linting tool or developer who's not familiar with the standard.

The value text for kind is directly mapped back to the data type's [def used in Haystack 4.0](https://project-haystack.dev/doc/lib-ph/index).

A dict is the only object where `_kind` is optional. Therefore all JSON objects (with correctly formatted key names) without a `_kind` are dicts. This is very useful when a grid has to list its rows as dicts.

### Value

The `val` is the heart of the encoded value. It never requires further parsing to read the value.

### Type mapping

#### String

A JSON string.

    "a string"

#### Number (with units)

A JSON number. If the number has units, it’s an object with both val and unit.

The kind can be determined via the use of ‘unit’.

    123

    // or if the number has
    // units...

    {
      "_kind": "number",
      "val": 123,
      "unit": "m"
    }

    // Positive infinity...
    {
      "_kind": "number",
      "val": "INF"
    }

    // Negative infinity...
    {
      "_kind": "number",
      "val": "-INF"
    }

    // Not a number...
    {
      "_kind": "number",
      "val": "NaN"
    }

#### Boolean

A JSON boolean

    true
    // or
    false

#### List

A JSON array.

    []

    // A list with some values...
    [ true, 123, "a string" ]

#### Dict

A JSON object.

Dict's use of `_kind` is optional. If it’s an object and no kind is specified, it’s assumed to be a dict.

    {}

    // A dict with some values...
    {
      "site": "A site",
      "num": 123,
      "bool": true
    }

    // A dict with the optional kind specified...
    {
      "_kind": "dict",
      "site": "A site",
      "num": 123,
      "bool": true
    }

Keys with invalid tag names should be skipped. For example, 'Ignore' should be ignored since it starts with a capital letter...

    {
      "Ignore": "This should be ignored",
      "site": "A site"
    }

#### Grid

A grid object specifies a kind. The meta version `ver` should always be specified.

A column has a name and an optional meta dict.

Rows and columns are optional if the grid is empty.

Each row is encoded as a dict.

    {
      "_kind": "grid",
      "meta": { "ver":"3.0", "foo": "bar" },
      "cols": [
        {
          "name: "id",
          "meta": { "size": 123 }
        },
        {
          "name": "dis"
        }
      ],
      "rows": [
          { "id": 1, "dis": "Hall" },
          { "id": 2, "dis": "Bedroom" }
      ]
    }

#### Marker

A marker object.

    {
      "_kind": "marker"
    }

#### Null

A JSON `null`.

#### Remove

A Remove object.

    {
      "_kind": "remove"
    }

#### NA (not available)

A not available object.

    {
      "_kind": "na"
    }

#### Ref

An object for a reference with optional display name (`dis`).

    {
      "_kind": "ref",
      "val": "/foo",
      "dis": "Links to foo"
    }

#### Date

A date object.

    {
      "_kind": "date",
      "val": "2015-06-08"
    }

#### Time

A time object.

    {
      "_kind": "time",
      "val": "15:47:41"
    }

#### Date time

An object with a date, time and timezone value.

The tz parameter is optional if...

* The date time is UTC.
* The timezone name can be calculated from the value's GMT offset (i.e. GMT+4 from the example below).

The val is a standard ISO 8601 formatted date time with offset.

    {
      "_kind": "dateTime"
      "val": "2015-06-08T15:47:41-04:00",
      "tz": "New_York"
    }

#### Uri

A URI object.

    {
      "_kind": "uri",
      "val": "https://j2inn.com"
    }

#### Coordinate

A coordinate object.

    {
      "_kind": "coord",
      "lat": 51.019371,
      "lng": -0.453980
    }

#### XStr

An XStr object.

    {
      "_kind": "xstr",
      "type": "Type",
      "val": "value"
    }

#### Symbol

A symbol object.

    {
      "_kind": "symbol",
      "val": "foo"
    }

## Examples

    // Zinc
    ver:"3.0" projName:"test"
    dis dis:"Equip Name",equip,siteRef,installed
    "RTU-1",M,@153c-699a "HQ",2005-06-01
    "RTU-2",M,@153c-699a "HQ",1999-07-12

    // JSON
    {
      "_kind": "grid",
      "meta": {
        "ver": "3.0",
        "projName": "test"
      },
      "cols": [
        {
          "name": "dis",
          "meta": { "dis": "Equip Name" }
        },
        { "name": "equip" },
        { "name": "siteRef" },
        { "name": "installed" }
      ],
      "rows": [
        {
          "dis": "RTU-1",
          "equip": { "_kind": "marker" },
          "siteRef": {
            "_kind": "ref",
            "val": "153c-699a",
            "dis": "HQ"
          },
          "installed": {
            "_kind": "date",
            "val": "2005-06-01"
          }
        },
        {
          "dis": "RTU-2",
          "equip": { "_kind": "marker" },
          "siteRef": {
            "_kind": "ref",
            "val": "153c-699a",
            "dis": "HQ"
          },
          "installed": {
            "_kind": "date",
            "val": "1999-07-12"
          }
        }
      ]
    }

Here's another example using nested lists, dicts and grids...

    // Zinc
    ver:"3.0"
    type,val
    "list",[1,2,3]
    "dict",{dis:"Dict!" foo}
    "grid",<<
      ver:"3.0"
      a,b
      1,2
      3,4
      >>
    "scalar","simple string"

    // JSON
    {
      "_kind": "grid",
      "meta": { "ver": "3.0" },
      "cols": [
        { "name": "type" },
        { "name": "val" }
      ],
      "rows": [
        {
          "type": "list",
          "val": [1, 2, 3]
        },
        {
          "type": "dict",
          "val": {
            "dis": "Dict!",
            "foo": { "_kind": "marker" }
          }
        },
        {
          "type": "grid",
          "val": {
            "_kind": "grid",
            "meta": { "ver": "3.0" },
            "cols": [
              { "name": "a"},
              { "name": "b" }
            ],
            "rows": [
              {
                "a": 1,
                "b": 2
              },
              {
                "a": 3,
                "b": 4
              }
            ]
          }
        },
        {
          "type": "scalar",
          "val": "simple string"
        }
      ]
    }

## MIME type

Hayson has its own MIME type that can be used in HTTP requests and responses for content negotiation…

    application/vnd.haystack+json

An optional parameter can be used to specify the [older Haystack JSON encoding](https://project-haystack.org/doc/Json)...

    application/vnd.haystack+json;version=3

Ideally the default `application/json` MIME type should also return Hayson although it's not an explicit pre-requsite.
