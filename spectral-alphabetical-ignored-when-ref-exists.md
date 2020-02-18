# Alphabetical Ignored When a `$ref` Exists

I noticed an issue with getting the `alphabetical` function to work.
It seems when a `$ref` exists in the items to be checked for alphabetization
the rule is ignored.

## To Reproduce

Using the rules taken directly from
[this test scenario](https://github.com/stoplightio/spectral/blob/67faa2a608e794634eea4bc3130048514a5af91e/test-harness/scenarios/alphabetical-responses-order.oas3.scenario)
and a modified version of the OpenAPI document in the same test we are not
seeing the failed rule as we should.

my-rules.yaml:

```yaml
rules:
  response-order:
    message: Responses should be in alphabetical order
    recommended: true
    given: $.paths.*.*.responses
    then:
      function: alphabetical
```

test-spec.yaml:

```yaml
openapi: 3.0.2
info:
  title: Test Spec
  version: 0.0.0
paths:
  /foo:
    get:
      operationId: get-foo
      responses:
        '404':
          description: ''
        '200':
          description: ''
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/foo'
components:
  schemas:
    foo:
      title: More incorrect casing
      type: object
      properties:
        id:
          type: integer
        bar:
          type: string
```

CLI command:

```sh
spectral lint test-spec.yaml -r my-rules.yaml
```

We should see the following in our results:

```sh
 19:15  warning  response-order  Responses should be in alphabetical order
```

## Environment

* Library version: 5.0.0
* OS: Windows 10
* Node version: 12.14.1
