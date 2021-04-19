# Hayson Schemas

## JSON Schema

Hayson has a [JSON schema](hayson-json-schema.json). This can be used to validate Hayson documents in various web frameworks and IDEs.

A PR has been submitted to [schemastore](https://www.schemastore.org/json/) so Hayson is supported by default in all of the world's IDEs!

_Alternatively_ you can manually add support to [VSCode](https://code.visualstudio.com/)...

- Go to `File->Preferences->Settings`
- Type `JSON Schema`
- Under `JSON: Schemas` click `Edit in settings.json`.

Add the following entry...

    "json.schemas": [
      {
        "fileMatch": [
          "\*.hayson.json"
        ],
        "url": "https://raw.githubusercontent.com/j2inn/hayson/master/hayson-json-schema.json"
      }
    ]

Any files that end with `.hayson.json` will now have error checking and auto-complete for Hayson code.

For YAML support...

- Ensure you have the [YAML extension](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) added to VSCode.
- Go to `File->Preferences->Settings`
- Type `YAML Schema`
- Under `YAML: Schemas` click `Edit in settings.json`.

Add the following entry...

      "yaml.schemas": {
        "https://raw.githubusercontent.com/j2inn/hayson/master/hayson-json-schema.json": ".hayson.yaml"
      }

Any files that end with `.hayson.yaml` will now have error checking and auto-complete for Hayson code.

## OpenAPI Schema

[OpenAPI](https://www.openapis.org/) is a popular way to model REST APIs that provides JSON validation and documentation. It's similar to JSON schema in a lot of ways.

Hayson has an [OpenAPI schema](hayson-openapi-schema.yaml) that can be referenced externally by other OpenAPI documents. Here's an example...

    openapi: 3.0.2
    info:
      title: Hayson test
      version: 1.0.0
      description: A simple test document
    paths:
      /foo:
        get:
          summary: Request a foo grid
          responses:
            '200':
              description: Returns a grid!
              content:
                application/json:
                  schema:
                    $ref: "https://raw.githubusercontent.com/j2inn/hayson/master/hayson-openapi-schema.yaml#/components/schemas/grid"
                  example:
                    _kind: grid
                    meta:
                      ver: '3.0'
                    cols:
                      - name: id
                    rows:
                      - id:
                          _kind: ref
                          val: id1
                      - id:
                          _kind: ref
                          val: id2
