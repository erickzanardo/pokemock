# Pokemock

A mock server generated from one or more arbitrary Swagger files.
Supports seeding, timeouts, response picking,
entity memory, semantic action inference, etc.


## Usage

```
Syntax:
  pokemock <swagger-urls-or-files> ... [-h] [-v] [-w] [-p <port>]

Options:
  -h, --help        Show help
  -v, --version     Show version
  -p, --port <port> Set server port, default is 8000
  -w, --watch       Watch mode: Restart on Swagger changes
  -k, --killable    Publish /kill endpoint to stop the service
      --memory      Enable memory module (experimental)
```


## Server

The mock server listens to the specified port and
mocks endpoints defined in the provided Swagger document.
Additionally, it publishes a Swagger UI under `/ui`,
the Swagger API under `/api-docs` and a `/kill` endpoint for shutdown.


## Request Headers

Using optional headers, clients can control the server's behavior:

- __X-Mock-Status__
  - Specifies the response status code
  - The correct response is inferred from the API if possible
  - Defaults to the first response code specified in the API
- __X-Mock-Seed__
  - Specifies a seed for data generation
  - If omitted, a random seed is generated
  - The current seed is always returned in a X-Mock-Seed response header
- __X-Mock-Time__
  - Specifies the minimum response time (milliseconds)
- __X-Mock-Size__
  - Specifies array size(s) in the response
  - Must be a valid JSON object of
    `<definitionName|attributeName>: <size>` pairs
  - If omitted, array sizes are randomly between 1 and 5
- __X-Mock-Depth__
  - Specifies the maximum JSON data depth
  - Defaults to 5
- __X-Mock-Override__
  - Specifies response data via [JSON Path](https://github.com/dchester/jsonpath)
  - Must be a valid JSON object of `<jsonPath>: <data>` pairs
  - `<data>` is arbitrary JSON
- __X-Mock-Replay__
  - Specifies the number of times the current X-Mock-* headers should be replayed
  - The next N requests to the requested URL will replay the current X-Mock-* headers
- __X-Mock-Replay-Pattern__
  - Specifies a regular expression to match for X-Mock-Replay
  - If omitted, the exact path is used for replaying


## Memory (experimental)

Use the `--memory` switch to enable the memory module.
When enabled, entities containing an ID are remembered by the server.
If the entity is requested again, the remembered data is returned.
This also applies to sub-entities across endpoints.

Additionally, the server tries to infer semantic actions from requests,
such as:

- Get by id
- Delete by id
- Update by id
- Create new entity

These actions are applied to known entities in memory.
For example, requesting a deleted entity will result in a 404 response.

## Customization

Pokemock provides a set of [Express](http://expressjs.com/de/) middlewares
which you can use independently.
The default app defined in `createDefaultApp.js` is an opinionated stack of
middlewares which you're encouraged to hack on.
By re-arranging and adding middlewares (especially generators)
you can tailor Pokemock to fit your APIs.

## Custom generators

Custom generators are a way to customize pokemock, without hack its middlewares, to use it you can pass a flag `--require-generators "ABSOLUTE_PATH_TO_YOUR_CUSTOM_GENERATORS_FILE", or if you are using `createDefaultApp` you should pass your custom generators on the option `customGenerators`.

Your custom generators module must export an array of custom generators which must has the following structure:

`matcher`: The field matcher, it can be a regexp or a function, the function will receive the name and schema of the current field, and you should return true or false
`fn`: The function that will generate the value for the field, it receives as args the name of the field, its schema and an pokemock option object.

An custom generators file example:

```
module.exports = [{
  matcher: /someField/,
  fn: () => generateSomeRandomValue(),
}, {
  matcher: (name, schema) name == "someOtherField" && schema.type == "array",
  fn: () => generateSomeRandomArray(),
}]
```
