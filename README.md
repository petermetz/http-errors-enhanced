# http-errors-enhanced-cjs

[![Version](https://img.shields.io/npm/v/http-errors-enhanced-cjs.svg)](https://npm.im/http-errors-enhanced-cjs)
[![Dependencies](https://img.shields.io/librariesio/release/npm/http-errors-enhanced-cjs)](https://libraries.io/npm/http-errors-enhanced-cjs)
[![Build](https://github.com/petermetz/http-errors-enhanced-cjs/workflows/CI/badge.svg)](https://github.com/petermetz/http-errors-enhanced-cjs/actions?query=workflow%3ACI)
[![Coverage](https://img.shields.io/codecov/c/gh/petermetz/http-errors-enhanced-cjs?token=jxElZm8DEK)](https://codecov.io/gh/petermetz/http-errors-enhanced-cjs)

## Wait, is this a fork!?

**YES! All credit goes to https://github.com/ShogunPanda**

TLDR: I needed a CJS build of the original library.

1. This is a fork of the excellent https://github.com/ShogunPanda/http-errors-enhanced/ repository whose author deserves all the credit.
2. The reason for the fork is explained here in the comments: https://github.com/ShogunPanda/http-errors-enhanced/pull/4
3. The intention for the fork is to keep up with upstream changes as much as possible, so the changes here
will be kept to a minimum to ensure that there we don't end up in a situation where certain changes
can no longer be pulled from upstream.
4. If you have a bug to report, please try to reproduce it with the upstream repo first. If it is also present there, then it should be reported there and we'll backport it here as the fixes come in.
5. The only time you should report issues here if you can demonstrate that it is working fine in the upstream repo and it does not work here - most likely due to something with the CJS/EJS differences.

### Couldn't you just use the ESM-only lib with dynamic imports?

Nope. Jest crashes with segmentation faults (!!!) when we try to do that... ;-(

## Summary

Create HTTP errors with additional properties for any framework.

## Installation

Just run:

```bash
npm install http-errors-enhanced-cjs --save
```

## Usage

You can create an error by using the generic base class.

```javascript
import { HttpError } from 'http-errors-enhanced'

// Any of the following will return the same error
const error1 = new HttpError(404, 'Page not found.', { key1: 'prop1' })
const error2 = new HttpError(404, { key1: 'prop1', message: 'Page not found.' })
const error3 = new HttpError('NotFound', 'Page not found.')
const error4 = new HttpError('NotFound', { key1: 'prop1', message: 'Page not found.' })
```

Or you can import a specific error class and instantiate that, to have some status and messages set up for you automatically.

```javascript
import { NotFoundError } from 'http-errors-enhanced'

// Any of the following will return the same error
const error1 = new NotFoundError('Page not found.', { key1: 'prop1' })
const error2 = new NotFoundError({ key1: 'prop1', message: 'Page not found.' })
```

Or, if you don't want to use the `new` operator, you can use the `createError` function:

```javascript
import { createError } from 'http-errors-enhanced'

// Any of the following will return the same error
const error1 = createError(404, 'Page not found.', { key1: 'prop1' })
const error2 = createError(404, { key1: 'prop1', message: 'Page not found.' })
const error3 = createError('NotFound', 'Page not found.')
const error4 = createError('NotFound', { key1: 'prop1', message: 'Page not found.' })
```

## API

### HttpError

The main error class.

The constructor signature is the following:

```
constructor(status, [message], [properties])
```

Where:

- `status`: A HTTP status code as number or a identifier (like: `NotFound`). If the identifier is not valid or the number outside the validity range (`400 <= status <= 599`), it defaults to 500.
- `message`: The error message.
- `properties`: A list of additional properties to attach to the error. With the exception of `code`, `message`, `stack`, `expose` and `headers`, all properties that already exist in the object are ignored.

The returned instance is a descendant class of `Error` with the following base properties, plus all the other properties specified in the constructor:

- `status`: The HTTP status as number.
- `statusCode`: It always mirrors `status`. It exists for compatibility reasons.
- `statusClass`: The HTTP status class, which is its first digit multiplied by 100. For instance, the status class of `404` is `400`.
- `code`: A string identifier for the error. If not ovewritten using properties, it always starts with `HTTP_ERROR_` (this prefix is also exported as `HttpError.standardErrorPrefix`). If the status code is know, it will be a string representation of it, like `HTTP_ERROR_NOT_FOUND`, otherwise it will contain the status number, like `HTTP_ERROR_570`.
- `error`: The HTTP status code description, if found. Example: `Not Found`.
- `errorPhrase`: The HTTP status code as phrase, if found. Example: `Not found.`.
- `expose`: A boolean to notify frameworks if errors internals (like message and stack) should be sent to client. Unless overwritten, it is set to `true` for errors whose statusClass is 400.
- `headers`: An object of HTTP headers to send to the client.
- `isClientError`: This is set to `true` if the HTTP status class is 400.
- `isServerError`: This is set to `true` if the HTTP status class is 500.

Each instance has a `serialize(extended, omitStack)` function.

If called without argument, it will return a object with the properties `statusCode`, `error` and `message` copied from the instance.

If called with `extended` set to `true`, it will call `serializeError` (see below) on the instance.

### Named error classes

For each known HTTP error, taken from Node.js [http.STATUS_CODES](https://nodejs.org/api/http.html#http_http_status_codes), there is an exported named class.

For instance, for the HTTP error 404, there is the `NotFoundError` which inherits from `HttpError`.

The constructor of a named class is the following:

```javascript
constructor([message], [properties])
```

### createError(status, [message], [properties])

The function can be used to create an `HttpError` without using the `new` operator.

It accepts the same arguments of the `HttpError` constructor.

### isHttpError(val)

The function can be used to check if an value is a `HttpError` or it has compatible properties. It follows the same semantic of the hononym function of [http-errors](https://github.com/jshttp/http-errors).

The function returns `true` if the value is an instance of `HttpError` or if the value is an object with a `status` property which is a number (and it is mirrored by the `statusCode` property) and a `expose` property which is a boolean.

### addAdditionalProperties(target, source)

Copies all enumerable properties from `source` to `target`, skipping the ones which are already defined.

### serializeError(val, [omitStack])

The function extracts the `message`, `code` (or `name` if no code is present) and `stack` properties from an object (typically a descendant of `Error`).

The extract properties are formatted and returned in a plain object containing `message` (string) and `stack` (array of strings) properties.

The object also contains all the other properties of the original object.

If `omitStack` is `true`, the `stack` property is omitted.

### Constants

The package exports several utility constants to make HTTP status handling easier

#### identifierByCodes and codesByIdentifier

A map of statuses identifiers indexed by HTTP status code and its reverse. This also exports HTTP classes 100, 200 and 300.

```javascript
console.log(identifierByCodes[404])
// => NotFound

console.log(codesByIdentifier['NotFound'])
// => 404
```

#### messagesByCodes and phrasesByCodes

A map of statuses descriptions and phrases indexed by HTTP status code. This also exports HTTP classes 100, 200 and 300.

```javascript
console.log(messagesByCodes[404])
// => Not Found

console.log(phrasesByCodes[404])
// => Not found.
```

#### Named status constants

For each HTTP status there is a upper snake case constant exported.

```javascript
console.log(CREATED)
// => 201

console.log(NOT_FOUND)
// => 404
```

### JSON Schemas definitions

The package defines a schema for commonly used error codes, in order to easily include them in [JSON Schema](https://json-schema.org/) and [OpenAPI](https://www.openapis.org/) based validation frameworks.

```javascript
console.log(JSON.stringify(notFoundErrorSchema))
/*
=> {
  "type": "object",
  "$id": "errors/404",
  "description": "Error returned when the requested resource is not found.",
  "properties": {
    "statusCode": {
      "type": "number",
      "description": "The error code",
      "enum": [
        404
      ],
      "example": 404
    },
    "error": {
      "type": "string",
      "description": "The error title",
      "enum": [
        "NotFound"
      ],
      "example": "NotFound"
    },
    "message": {
      "type": "string",
      "description": "The error message",
      "pattern": ".+",
      "example": "NotFound."
    }
  },
  "required": [
    "statusCode",
    "error",
    "message"
  ],
  additionalProperties: false
}
*/
```

## Credits

This project has been heavily inspired by [http-errors](https://github.com/jshttp/http-errors), of which is a indipendent and unrelated project.

This is a fork of https://github.com/ShogunPanda/http-errors-enhanced/

## Contributing to http-errors-enhanced-cjs

- Report bugs on the upstream repository unless you can demonstrate with a reproduction code repository/gist that your bug is **only** present in this fork and not in the upstream repository.

## Copyright

Copyright (C) 2020 and above Shogun (shogun@cowtech.it).

Licensed under the ISC license, which can be found at https://choosealicense.com/licenses/isc.
