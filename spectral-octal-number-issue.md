# Falsely Identifying Strings as Octals

## Issue Description

When linting OpenAPI 3.x.x specifications, Spectral lint is not appropriately
parsing strings per the YAML specification. This issue seems to present itself
when strings that are made up of numbers that begin with the digit `0`, and
contain the digits `8` or `9`, are not quoted in the OpenAPI specification.

Since these are not valid octal strings, Spectral should be parsing these as
the YAML [!!str](https://yaml.org/type/str.html) type as opposed to the
[!!int](https://yaml.org/type/int.html) type.

## Steps to Reproduce

```yaml
openapi: 3.0.2
tags:
  - name: Test
    description: Test tag
info:
  title: Test API
  version: '0.0.1'
  description: An API for testing
  contact:
    name: Test Contact
servers:
  - url: https://foo-bar-dev.nonprod.jbhunt.com/test
paths:
  /things:
    get:
      operationId: get-things
      description: Fetches some things
      tags: [Test]
      responses:
        '200':
          description: Fetched some things
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ThingList'
components:
  schemas:
    ThingList:
      type: object
      properties:
        things:
          type: array
          items:
            type: string
      example:
        things:
          - '01'
          - '02'
          - '03'
          - '04'
          - '05'
          - '06'
          - '07'
          - 08
          - 09
          - '010'
          - 01981
```

When linting the above OpenAPI specification, we get the following error.

```sh
OpenAPI 3.x detected

c:/Source/enterprise-apis/apis/test/test.yaml
 35:15  warning  example-value-or-externalValue     Example should have either a `value` or `externalValue` field.
 44:13    error  oas3-valid-content-schema-example  `7` property type should be string
 44:13    error  oas3-valid-schema-example          `7` property type should be string

✖ 3 problems (2 errors, 1 warning, 0 infos, 0 hints)
```

In the example above, the error at location `44:13` references the array example
item `08`.
Spectral seem to be incorrectly interpereting these values as octals rather than
strings.

## Expected Behavior

Per the YAML specification, the given example should be valid.

## Environment

* Spectral Version: 5.0.0
* Windows 10
  * Version: 1909
  * Build: 18363.592
* Node Version: 10.16.2
* NPM Version: 6.13.6
