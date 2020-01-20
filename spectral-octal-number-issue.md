# Non-Octal Numbers In Unquoted Strings

## Issue Description

When linting OpenAPI 3.x.x specifications, Spectral lint is not appropriately
parsing strings per the YAML specification. This issue presents itself when
strings that are made up of numbers that begin with the digit `0`, and contain
the digits `8` and `9`, are not quoted in the OpenAPI specification.

Per the [Integer Language-Independent Type for YAML Version 1.1](https://yaml.org/type/int.html),
octal integers can be represented by numbers that begin with the digit `0`.
However, integers can not have any digit greater than `7` in their
representation.
YAML specification dictates that any integer that does not match valid integer
values, as defined in the link above, should be treated as strings.

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

Given the above OpenAPI specification, when linting we will get the following
error from Spectral:

```sh
OpenAPI 3.x detected

c:/Source/enterprise-apis/apis/test/test.yaml
 35:15  warning  example-value-or-externalValue     Example should have either a `value` or `externalValue` field.
 44:13    error  oas3-valid-content-schema-example  `7` property type should be string
 44:13    error  oas3-valid-schema-example          `7` property type should be string

✖ 3 problems (2 errors, 1 warning, 0 infos, 0 hints)
```

> The warning identified at location 35:15 ("Example should have either a
> `value` or `externalValue` field") is related to
> [Issue 883](https://github.com/stoplightio/spectral/issues/883).

In the example above, the items `08`, `09`, and `01981` are not parsed by
Spectral as strings.
However, these are not valid number representations, as they begin with the `0`
digit, indicating an octal number, but contain invalid octal digits.
Quoted strings in the list are valid representations of octal numbers and thus
must be quoted in order to be read in as strings.

## Expected Behavior

Per the YAML specification, any number that begins with the digit `0` could be
an octal representation. However, any non-octal digits present in the number
(e.g. `8` and `9`), should invalidate the octal representation and thus be
treated like any other non-numeric representation and parsed as a string.

## Environment

* Spectral Version: 5.0.0
  * This issue is also present in 4.2.0
* Windows 10
  * Version: 1909
  * Build: 18363.592
* Node Version: 10.16.2
* NPM Version: 6.13.6
