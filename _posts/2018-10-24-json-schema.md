---
title: "JSON Schema Reference"
categories: write-up
tags: json
---

Schemas validate against JSON files (or, *instances*). Schemas are themselves written in JSON.

A Schema must be either a JSON object, or a boolean. If it's a boolean, then it simply passes or fails the JSON validation based on the value.

For example, a schema that passes everything can either be an empty schema, or the boolean value `true`.
```json
{}
```

```json
true
```

✔️:
```json
28
```
```json
"testing"
```
```json
null
```

On the other hand, a schema that fails everything can be implemented using the boolean value `false`.
```json
false
```
Which is equivalent to the `"not"` assertion:
```json
{ "not":{} }
```

❌:
```json
19.3
```
```json
"words, words, words"
```
```json
{
    "property": [ "test", "testing", "tester" ]
}
```

If it's an object, then it has properties that are applied to the instance, or *keywords*. Keywords fall into one or both of two categories:

- **Assertions** produce a boolean result when applied to an instance
- **Annotations** attach information to an instance for application use

The RFC states that:

> Validation is a process of checking assertions. Each assertion adds constraints that an instance must satisfy in order to successfully validate.

---

- [Declaring a JSON Schema](#declaring-a-json-schema)
- [Writing a JSON Schema](#writing-a-json-schema)
    - [The `type` keyword](#the-type-keyword)
    - [Matching integers](#matching-integers)
    - [Validating a `string`](#validating-a-string)
        - [Length](#length)
        - [Regular expressions](#regular-expressions)
        - [Common string formats](#common-string-formats)
    - [Validating a `number`](#validating-a-number)
        - [Multiples](#multiples)
        - [Ranges](#ranges)
    - [Validating an `object`](#validating-an-object)
        - [Validating object properties](#validating-object-properties)
        - [Restricting additional properties](#restricting-additional-properties)
        - [Required properties](#required-properties)
        - [Restrict property names by pattern](#restrict-property-names-by-pattern)
        - [Restricting number of properties](#restricting-number-of-properties)
        - [Dependencies](#dependencies)
            - [Property dependencies](#property-dependencies)
            - [Schema dependencies](#schema-dependencies)
        - [Pattern properties](#pattern-properties)
    - [Validating an `array`](#validating-an-array)
        - [Validating `array` items](#validating-array-items)
            - [List validation](#list-validation)
            - [Tuple validation](#tuple-validation)
        - [Validating `array` length](#validating-array-length)
        - [Validating `array` uniqueness](#validating-array-uniqueness)
    - [The `enum` keyword](#the-enum-keyword)
    - [The `const` keyword](#the-const-keyword)
    - [Schema metadata](#schema-metadata)
    - [Combining schemas](#combining-schemas)
        - [`allOf`](#allof)
        - [`anyOf`](#anyof)
        - [`oneOf`](#oneof)
        - [`not`](#not)
- [Schema reuse](#schema-reuse)
    - [References to other schemas](#references-to-other-schemas)
    - [Schema identifiers](#schema-identifiers)
    - [Using `$id` with `$ref`](#using-id-with-ref)
- [Bibliography](#bibliography)

---

# Declaring a JSON Schema
Since a schema is itself JSON, it's not easy to tell whether something is a schema or just regular old JSON. The `$schema` keyword is used at the root of the schema to declare that something is a schema. The value of the `$schema` keyword should be a URL to the standard that the schema was written against.

```json
{
    "$schema": "http://json-schema.org/draft-06/schema#"
}
```

You can also just use this link if you want the latest version of the schema:
```json
{
    "$schema": "http://json-schema.org/schema#"
}
```
# Writing a JSON Schema

## The `type` keyword
The `type` keyword is an assertion used to validate the type of the JSON element.
```json
{ "type": "string" }
```
✔️:
```json
"This is a string"
```
❌:
```json
63
```

The value can be one of the six primitive types allowed in JSON:
- `"null"`
- `"boolean"`
- `"object"`
- `"array"`
- `"number"`
- `"string"`

Or `"integer"`, which matches any number without a fractional part.

Additionally, the value of `type` can be an array containing any combination of the above values with no repeats. It will match to any of the given types.

```json
{ "type": [ "string", "boolean" ] }
```
✔️:
```json
"This is a string"
```
```json
true
```
❌:
```json
63
```

## Matching integers
The `"integer"` type is used for matching integral numbers.
```json
{ "type": "integer" }
```
✔️:
```json
3
```
```json
-3
```
❌:
```json
3.141592654
```
```json
"3"
```

## Validating a `string`

### Length
String length can be validated using `minLength` and `maxLength` keywords. The value of each must be a non-negative number.

```json
{
    "type": "string",
    "minLength": 3,
    "maxLength": 5
}
```
✔️:
```json
"Hello"
```
```json
"Car"
```
❌:
```json
"Hi"
```
```json
"Vehicle"
```

### Regular expressions
The `pattern` keyword can be used to restrict a string to a regex.

Example for matching a http(s) URL:
```json
{
    "type": "string",
    "pattern": "/^(https?:\/\/)?([\da-z\.-]+)\.([a-z\.]{2,6})([\/\w \.-]*)*\/?$/"
}
```
✔️:
```json
"https://www.google.com"
```
❌:
```json
"Hello world!"
```

### Common string formats
The `format` keyword can be used for basic validation of common string formats:
- `"date-time"` for ISO8601 dates
- `"email"` for [email addresses](https://tools.ietf.org/html/rfc5322#section-3.4.1)
- `"hostname"` for [internet host names](https://tools.ietf.org/html/rfc1034#section-3.1)
- `"ipv4"` for dotted-quad IPv4 addresses
- `"ipv6"` for IPv6 addresses
- `"uri"` for a [universal resource identifier](https://tools.ietf.org/html/rfc3986)

## Validating a `number`

### Multiples
Numbers can be restricted to a multiple of a given number using the `multipleOf` keyword. The given number can be any positive number.

```json
{
    "type": "number",
    "multipleOf": 15
}
```
✔️:
```json
0
```
```json
15
```
```json
30
```
❌:
```json
14
```

### Ranges
Numbers can be restricted to a certain inclusive range using a combination of the `minimum` and `maximum` keywords. You can also use `exclusiveMinimum` and `exlusiveMaximum` for exclusive ranges.
If *x* is the value being validated, then the following must be true:
- `x >= minimum`
- `x > exclusiveMinimum`
- `x <= maximum`
- `x < exclusiveMaximum`

```json
{
    "type": "number",
    "minimum": 3,
    "exclusiveMaximum": 22
}
```
✔️:
```json
3
```
```json
8
```
```json
21
```
❌:
```json
2
```
```json
22
```

## Validating an `object`
You can use the `object` type to match JSON objects. It will match to any valid JSON object.

```json
{ "type": "object" }
```
✔️:
```json
{
    "key": "value",
    "anotherKey": "anotherValue"
}
```
❌:

Invalid JSON won't be matched, all keys must be of type string.
```json
{
    3.141: "Pi",
    Number: "Blah blah blah..."
}
```
```json
["This", "is", "not", "an", "object"]
```

### Validating object properties
The properties of an object can be validated using the `properties` keyword.
```json
{
    "type": "object",
    "properties": {
        "x": { "type": "number" },
        "y": { "type": "number" }
    }
}
```
✔️:
```json
{
    "x": 12.4,
    "y": 18.6
}
```
Leaving out properties is valid, as long as they are not `required`.
```json
{
    "x": 12.4
}
```
To this end, an empty object can even be valid.
```json
{ }
```
Additional properties can be valid too.
```json
{
    "x": 12.4,
    "y": 18.6,
    "additional": "an additional property"
}
```
❌:
```json
{
    "x": 12.4,
    "y": "shouldn't this be a number?"
}
```

### Restricting additional properties
Additional properties can be restricted through the use of the `additionalProperties` keyword. Its value can be either a boolean or an object. If it's a boolean and set to `false` then any additional properties will make an object invalid.
```json
{
    "type": "object",
    "properties": {
        "x": { "type": "number" },
        "y": { "type": "number" }
    },
    "additionalProperties": false
}
```
✔️:
```json
{
    "x": 12.4,
    "y": 18.6
}
```
❌:
```json
{
    "x": 12.4,
    "y": 18.6,
    "additional": "an additional property"
}
```
If `additionalProperties` is an object, it functions as a schema that will be used to evaluate any properties not listed in `properties`. For example, you could allow additional properties, but only if they are a `number`.
```json
{
    "type": "object",
    "properties": {
        "x": { "type": "number" },
        "y": { "type": "number" }
    },
    "additionalProperties": { "type": "number" }
}
```
✔️:
```json
{
    "x": 12.4,
    "y": 18.6
}
```
```json
{
    "x": 12.4,
    "y": 18.6,
    "additional": 26.8
}
```
❌:
```json
{
    "x": 12.4,
    "y": 18.6,
    "additional": "an additional property"
}
```

### Required properties
By default the properties in the `properties` keyword are not required, but you can provide a list of properties which must be present using the `required` keyword. The keyword takes an array of zero or more strings, all of which must be unique.
```json
{
    "type": "object",
    "properties": {
        "username": { "type": "string" },
        "password": { "type": "string" },
        "location": { "type": "string" },
        "phoneNumber": { "type": "string" }
    },
    "required": [ "username", "password" ]
}
```
✔️:
```json
{
    "username": "AUser",
    "password": "YXdkYWRhZHNkYWdodGhzZmdzZGZnc2FlcmZlYWY="
}
```
```json
{
    "username": "AUser",
    "password": "YXdkYWRhZHNkYWdodGhzZmdzZGZnc2FlcmZlYWY=",
    "location": "United Kingdom"
}
```
❌:
```json
{
    "username": "AUser",
    "location": "United Kingdom"
}
```

### Restrict property names by pattern
The names of properties can be validated against a pattern. This can be useful if you want to make sure all of the properties in an object follow a specific convention, for something like programmatic access.

```json
{
    "type": "object",
    "propertyNames": {
        "pattern": "^[a-zA-Z_$][a-zA-Z_$0-9]*$"
    }
}
```
✔️:
```json
{
    "test": 123,
    "$alsoTest": 1234,
    "_testFurther": 12345
}
```
❌:
```json
{
    "123Test": "test",
    " test": "these aren't permitted"
}
```

### Restricting number of properties
The number of properties of an object can be restricted using the `minProperties` and `maxProperties` keywords. Each must be a non-negative integer.

```json
{
    "type": "object",
    "minProperties": 1,
    "maxProperties": 2
}
```
✔️:
```json
{
    "property1": 1
}
```
```json
{
    "property1": 1,
    "property2": 2
}
```
❌:
```json
{
    "property1": 1,
    "property2": 2,
    "property3": 3
}
```
```json
{ }
```

### Dependencies

The `dependencies` keyword allows the schema of an `object` to change based on certain properties.

There are two forms of dependencies:
- *Property dependencies* whereby certain properties must be present if a given property is present
- *Schema dependencies* whereby the schema changes when a given property is present

#### Property dependencies
In this example of customer data, when a payment card is provided a billing address must also be present:
```json
{
    "type": "object",
    "properties": {
        "name": { "type": "string" },
        "payment_card": { "type": "string" },
        "billing_address": { "type": "string" }
    },
    "required": [ "name" ],
    "dependencies": {
        "payment_card": [ "billing_address" ]
    }
}
```
✔️:
```json
{
    "name": "Firstname Lastname",
    "payment_card": "1234567812345678",
    "billing_address": "123 Fake Avenue"
}
```
This is all good because we don't have a payment card so no billing address is required.
```json
{
    "name": "Firstname Lastname"
}
```
This works because the dependency doesn't work both ways, so the billing address is just treated as an additional field.
```json
{
    "name": "Firstname Lastname",
    "billing_address": "123 Fake Avenue"
}
```
To make this invalid, you'd have to specify the two way dependency explicitly like this:
```json
{
    "dependencies": {
        "payment_card": [ "billing_address" ],
        "billing_address": [ "payment_card" ]
    }
}
```
❌:

Payment card and no billing address:
```json
{
    "name": "Firstname Lastname",
    "payment_card": "1234567812345678"
}
```

#### Schema dependencies
Schema dependencies work like property dependencies but they can extend the schema to have more constraints.

For example here's another way you can write the schema above:
```json
{
    "type": "object",
    "properties": {
        "name": { "type": "string" },
        "payment_card": { "type": "string" },
        "billing_address": { "type": "string" }
    },
    "required": [ "name" ],
    "dependencies": {
        "payment_card": {
            "properties": {
                "billing_address": { "type": "string" }
            },
            "required": [ "billing_address" ]
        }
    }
}
```
✔️:
```json
{
    "name": "Firstname Lastname",
    "payment_card": "1234567812345678",
    "billing_address": "123 Fake Avenue"
}
```
```json
{
    "name": "Firstname Lastname"
}
```
```json
{
    "name": "Firstname Lastname",
    "billing_address": "123 Fake Avenue"
}
```
❌:
```json
{
    "name": "Firstname Lastname",
    "payment_card": "1234567812345678"
}
```

### Pattern properties
`additionalProperties` can restrict an object so that it has no additional properties than the ones originally listed, or it can apply a schema for any additional properties. It turns out this isn't powerful enough, and that you might want to restrict the names of these properties too, or you might want to say that all properties of a certain name should conform to a particular schema. This is where the `patternProperties` keyword is useful, as it's a keyword that maps from regular expressions to schemas. If a property matches a given regex then it must also validate against the corresponding schema.

In this example, any properties that start with the prefix `"str_"` must be strings and `"bool_"` must be booleans:
```json
{
    "type": "object",
    "patternProperties": {
        "^str_": { "type": "string" },
        "^bool_": { "type": "boolean" }
    },
    "additionalProperties": false
}
```
✔️:
```json
{
    "str_test": "String!!!"
}
```
```json
{
    "bool_notAString": true
}
```
❌:
```json
{
    "bool_MaybeAString": "testing..."
}
```
This one doesn't match any of the regexes:
```json
{
    "completely_different": 3.141
}
```

## Validating an `array`

### Validating `array` items
There are two ways in which arrays are generally used in JSON:
- *List* - A sequence of arbitrary length where each item matches the same schema
- *Tuple* - A sequence of fixed length where each item may have a different schema

#### List validation
List validation is used for if the array is required to be of arbitrary length where each item matches the same schema. The `items` keyword should be set to a schema that will be used to match all of the items in the array.

```json
{
    "type": "array",
    "items": {
        "type": "string"
    }
}
```
✔️:
```json
[ "one", "two", "three" ]
```
An empty array is always valid:
```json
[]
```
❌:
```json
[ "one", 2, "three", 4, 5 ]
```

The `contains` keyword can be used to validate against arrays containing at least one of an element:
```json
{
    "type": "array",
    "contains": {
        "type": "string"
    }
}
```
✔️:
```json
[ "one", 2, 3 ]
```
❌:
```json
[ 1, 2, 3 ]
```

#### Tuple validation
Tuple validation is used for when an array is a collection of items where each has a different schema, and the index of each item can be meaningful. For example, a sandwich `Cheese and salami, Plain bread, Toasted` could be represented as a 3-tuple of form `[ filling, bread, cooking ]`. Each of these fields can have a different schema:
- `filling`: The filling is a string
- `bread`: The bread is a string from a fixed set of values
- `cooking`: A string from a fixed set of values

Here's a schema that will validate against sandwiches:
```json
{
    "type": "array",
    "items": [
        {
            "type": "string"
        },
        {
            "type": "string",
            "enum": [ "Plain", "Seeded", "Brown", "Wholegrain" ]
        },
        {
            "type": "string",
            "enum": [ "On its own", "Toasted", "Fried", "Grilled" ]
        }
    ]
}
```
✔️:
```json
[ "Cheese and salami", "Wholegrain", "Grilled" ]
```
It's okay to miss an item:
```json
[ "Cheese and salami", "Wholegrain" ]
```
It's okay to add an item too, but the sandwich shop might get annoyed!
```json
[ "Cheese and salami", "Wholegrain", "Grilled", "With lettuce" ]
```
❌:

This sandwich doesn't have a filling (weird!!):
```json
[ "Wholegrain", "Grilled" ]
```
Hearty Italian is not one of the bread types!
```json
[ "Cheese and salami", "Hearty Italian", "Grilled" ]
```

You can use the `additionalItems` keyword to control whether it's valid to have more items in the array beyond what's defined. Additionally, the `additionalItems` keyword may be a schema to validate against additional items in the array.

### Validating `array` length
Array length can be specified using `minItems` and `maxItems` keywords. Value of each must be a non-negative number. These keywords work in both list and tuple validation.

```json
{
    "type": "array",
    "minItems": 1,
    "maxItems": 3
}
```
✔️:
```json
[ 1, 2 ]
```
❌:
```json
[]
```
```json
[ 1, 2, 3, 4 ]
```

### Validating `array` uniqueness
Using the `uniqueItems` keyword you can ensure that all items in an array are unique. Simple set it to `true`.

```json
{
    "type": "array",
    "uniqueItems": true
}
```
✔️:
```json
[ 1, 2, 3, 4 ]
```
```json
[]
```
❌:

```json
[ 1, 2, 2, 3 ]
```

## The `enum` keyword
The `enum` keyword restricts a value to a fixed set of values. Its format is an array with at least one value where each element is unique.

Here is an example of using `enum` to validate paint colours:
```json
{
    "type": "string",
    "enum": [ "red", "blue", "yellow", "green" ]
}
```
✔️:
```json
"red"
```
❌:
```json
"orange"
```

You can use the `enum` keyword without a type too. Here `null` is used to indicate no paint, and a number is added too for no particular reason.
```json
{
    "type": "string",
    "enum": [ "red", "blue", "yellow", "green", null, 86.3 ]
}
```
✔️:
```json
"red"
```
```json
null
```
```json
86.3
```
❌:
```json
86.4
```


## The `const` keyword
The `const` keyword restricts a value to only a single value.

```json
{
    "properties": {
        "wallColour": {
            "const": "red"
        }
    }
}
```
✔️:
```json
{ "wallColour": "red" }
```
❌:
```json
{ "wallColour": "pink" }
```

The `const` keyword is syntactic sugar for an `enum` with only one element.

## Schema metadata
JSON Schema includes keywords `title`, `description`, `default`, and `examples` that aren't strictly used for validation but are used to describe parts of a schema.

`title` and `description` must be strings used to describe the schema.

The `default` keyword specified a default value for an item. JSON processing tools might use this to provide a default value for a missing JSON key, though many validators ignore the `default` keyword.

The `examples` keyword is a place to give an array of examples that validate against the schema. It's not required for validation but it might help explain the schema to a reader.

```json
{
    "title": "Match everything",
    "description": "This schema will literally match anything",
    "default": "This is a default value",
    "examples": [
        "Anything",
        false,
        1337
    ]
}
```

## Combining schemas
There are a few keywords for combining schemas together. These are:
- `allOf`: Must be valid against **all** of the subschemas
- `anyOf`: Must be valid against **any** of the subschemas
- `oneOf`: Must be valid against **one** of the subschemas

All of these keywords must be set to an array where each item is a schema. There is also the `not` keyword, whereby to validate, the JSON must **not** be valid against the given schema.

### `allOf`
To validate against `allOf`, the given data must be valid against all of the given subschemas.
```json
{
    "allOf": [
        { "type": "string" },
        { "maxLength": 10 }
    ]
}
```
✔️:
```json
"This works"
```
❌:
```json
"This does not"
```

You can create schemas that are impossible to validate against with `allOf`, for example this is a schema that won't validate against anything:
```json
{
    "allOf": [
        { "type": "string" },
        { "type": "number" }
    ]
}
```

### `anyOf`
To validate against `anyOf`, the given data must be valid against one or more of the given subschemas:
```json
{
    "anyOf": [
        { "type": "string" },
        { "type": "number" }
    ]
}
```
✔️:
```json
"String"
```
```json
78
```
❌:
```json
{ "this is an object": "so it won't match" }
```

### `oneOf`
To validate against `oneOf`, the given data must validate against only one of the given subschemas.
```json
{
    "oneOf": [
        { "type": "number", "multipleOf": 5 },
        { "type": "number", "multipleOf": 3 }
    ]
}
```
✔️:
```json
10
```
```json
9
```
❌:

Not a multiple of 5 or 3:
```json
2
```
A multiple of both 5 and 3:
```json
15
```

### `not`
`not` doesn't combine schemas, but it's similar to the other keywords in that its value is a subschema that affects validation. The following schema validates against anything that is not a boolean:
```json
{
    "not": { "type": "boolean" }
}
```
✔️:
```json
10
```
```json
"true"
```
```json
{ "an": "object" }
```
❌:
```json
false
```

# Schema reuse
## References to other schemas
Because some schemas can be generic it can be useful to make schemas that can be used in a variety of other schemas. This can be achieved through the use of the `definitions` keyword.
Say you had a schema for an address:
```json
{
    "type": "object",
    "properties": {
        "street_address": { "type": "string" },
        "city": { "type": "string" },
        "state": { "type": "string" }
    },
    "required": ["street_address", "city", "state"]
}
```
To reuse this schema you can put it in the definitions section of the parent schema:
```json
{
    "definitions": {
        "address": {
            "type": "object",
            "properties": {
                "street_address": { "type": "string" },
                "city": { "type": "string" },
                "state": { "type": "string" }
            },
            "required": ["street_address", "city", "state"]
        }
    }
}
```
Then this schema can be referred to from elsewhere in the parent schema using the `$ref` keyword. `$ref` gets logically replaced with the thing that it points to, so to reuse the address schema we would do:
```json
{ "$ref": "#/definitions/address" }
```
This can be used anywhere a schema is expected. The value of `$ref` is a URI, and the part after the `#` (*fragment*, or *named anchor*) is a [JSON Pointer](https://tools.ietf.org/html/rfc6901).

If the reference is in the same document, then `$ref` begins with `#`, after which the slash-separated items traverse the keys in the document.

`$ref` can also be a relative or absolute URI, so if you want to include your definitions in separate files then that's possible:
```json
{ "$ref": "definitions.json#/address" }
```

Even though the value of `$ref` is a URI, it's not necessarily a network locator. This means that the schema doesn't need to be accessible at that URI, but it can be. Whether network resources are fetched or not depends on the validator implementation.

`$ref` elements can refer to themselves, and this can be useful. However you need to be careful not to cause a loop of `$ref` schemas referring to each other, as this can cause crashes depending on the implementation.

## Schema identifiers
The `$id` property is a URI that serves two purposes:
- It declares a unique ID for the schema
- It declares a base URI against which `$ref` URIs are resolved

It's best practise that every top-level schema should set `$id` to an absolute URI with a domain that you control:
```json
{
    "$id": "http://www.example.com/schemas/schema.json"
}
```
This provides a unique ID for the schema, as well as where it can be downloaded. The second purpose of the `$id` property also means that any relative `$ref` URLs elsewhere in the schema resolve with this domain as the base URL, for example:
```json
{
    "$ref": "person.json"
}
```
Resolves to `http://www.example.com/schemas/person.json`.

`$id` should never be the empty string or an empty fragment `#`, as that doesn't make sense.

## Using `$id` with `$ref`
`$id` also provides a way to refer to a subschema without using JSON pointers. This means you can refer to them by a unique name rather than by where they appear in the JSON file.
For example:
```json
{
    "definitions": {
        "address": {
            "$id": "#address",
            "type": "object",
            "properties": {
                "street_address": { "type": "string" },
                "city": { "type": "string" },
                "state": { "type": "string" }
            },
            "required": ["street_address", "city", "state"]
        }
    },

    "type": "object",

    "properties": {
        "billing_address": { "$ref": "#address" },
        "shipping_address": { "$ref": "#address" }
    }
}
```

# Bibliography
- [Understanding JSON Schema](https://json-schema.org/understanding-json-schema/) 
- [JSON Schema RFC](https://json-schema.org/latest/json-schema-core.html#rfc.section.8)
- [JSON Schema Validation RFC](https://json-schema.org/latest/json-schema-validation.html)
- [JSON Pointer RFC](https://tools.ietf.org/html/rfc6901)