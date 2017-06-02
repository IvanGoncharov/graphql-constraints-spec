# GraphQL Constraints Directives RFC (draft 1)
*Working Draft - June 2017*

## Introduction
GraphQL Constraints Directives is a set of directives that allows you to annotate [GraphQL types](http://facebook.github.io/graphql/#sec-Types). The primary use case for these annotations is the validation of input parameters. However, it may be used by any tooling including documentation, GraphiQL, form-generation, etc.
It is inspired by [JSON Schema](http://json-schema.org/latest/json-schema-validation.html) and reuses as much semantics as possible from it.

**Note: Reference implementation is coming**

## Notes for readers
The issues list for this draft can be found at https://github.com/apis-guru/graphql-constraints-spec/issues
To provide feedback, use this issue tracker, or email the document editors.

All the examples below will be based on [GraphQL IDL](https://github.com/facebook/graphql/pull/90).

## Notation

- **Constraints Directive** - is a group of related constraints represented as a GraphQL directive
- **Constraint** - Atomic assertion which is represented as arguments and input values of Constraints Directive
- **Instance** - actual **non-null** value of argument, field, input field that is interpreted by the directive.

**NOTE**: null values are handled by GraphQL natively. All the constraints don't affect nullability.

## Multiple constraints
When a Constraint Directive has more than one constraint in it, they should be treated as logical AND between individual constraints.

## Kinds of Constraints Directives

In GraphQL there are types (`String`, `Int`, `Float`, `ID`, `Boolean` and various user-defined types) and type wrappers (`List` and `Not-Null`).

Accordingly, this document describes two kinds of Constraints Directives: `Type Constraints Directives` and `Wrapper Constraints Directives`.

## Type Constraints Directives

## Coercion
**TBD**
If applied before coercion, possible issue with `ID` type since it accepts both integers and strings.

### Applicability
Type Constraints Directives can be applied to the following GraphQL entities:
- Object Field Definition
- Input Object Field Definition
- Object Field Arguments Definition
- Directive Argument Definition
- Scalar Definition

If Type Constraint is applied to an entity of `List` wrapper type it describes innermost values of the lists.

### Multiple Type Constraints Directives
Applying more than one Type Constraints Directives is allowed only for Scalar Definitions. Doing that for other entities should result in an error.  If there are more than one Type Constraints Directive applied to one entity they should be treated as a logical OR operator between each directive. For example:

```graphql
scalar IntOrFalse @numberValue(multipleOf: 1) @booleanValue(const: false)
scalar FloatOrBoolean @numberValue @booleanValue
```

### Relation to GraphQL scalars

Type Constraints Directives do not override GraphQL standard scalars semantic and runtime behavior. Moreover, each Type Constraints Directive is compatible only with specific standard scalars

|Directive\Type | Float | Int | Boolean | String | ID | Non-standard scalar |
| ------------- | ----- | --- | ------- | ------ | -- | ------------------- |
| @numberValue  | +     | +   | -       | -      | +  | +                   |
| @stringValue  | -     | -   | -       | +      | +  | +                   |
| @booleanValue | -     | -   | +       | -      | -  | +                   |


Applying a directive to a field definition, an argument definition or an input field definition of an incompatible standard type should result in an error.

### @numberValue

`@numberValue` directive is used to describe possible numeric values.

**NOTE**: the decision to have a single directive for all numeric values (instead of `@floatValue` and `@intValue`) was made in order to avoid the edge cases resulting in undefined behavior. For example, a custom scalar with applied `@floatValue` and `@intValue` with incompatible constraints.

Instance is valid if it is a numeric value according to the Serialization Format (e.g. JSON)

#### Constraints

##### multipleOf
The value of `multipleOf` MUST be a number, strictly greater than `0`. A numeric instance is valid only if division by this constraint's value results in an integer.

##### max
The value of `max` MUST be a number, representing an inclusive upper limit for a numeric instance.  A numeric instance is valid only if the instance is less than or exactly equal to `max`.

##### min
The value of `min` MUST be a number, representing an inclusive upper limit for a numeric instance. A numeric instance is valid only if the instance is greater than or exactly equal to `min`.

##### exclusiveMax
The value of `exclusiveMax` MUST be a number, representing an exclusive upper limit for a numeric instance. A numeric instance is valid only if it is strictly less than (not equal to) `exclusiveMax`.

##### exclusiveMin
The value of `exclusiveMin` MUST be a number, representing an exclusive upper limit for a numeric instance. A numeric instance is valid only if it has a value strictly greater than (not equal to) `exclusiveMin`.

##### oneOf
The value of this argument MUST be an array. This array SHOULD have at least one element. Elements in the array SHOULD be unique.
An instance is valid only if its value is equal to one of the elements in this constraint's array value.

##### equals
A numeric instance is valid only if its value is equal to the value of the constrain.

#### Examples
```graphql
type Foo {
  byte:Integer @numberValue(
    min: 0
    max: 255
  )
}
```
*Examples of valid values for `byte` field*: `155`, `255`, `0`

*Examples of invalid values for `byte` field*: `"string"`, `256`, `-1`

```graphql
type Foo {
  bitMask: Integer @numberValue(
    oneOf: [ 1, 2, 4, 8, 16, 32, 64, 128 ]
  )
}
```
*Examples of valid values for `bitMask` field*: `1`, `16`, `128`

*Examples of invalid values for `bitMask` field*: `"string"`, `3`, `5`


### @stringValue

`@stringValue` directive is used to describe possible string values.
Instance is valid if it is a string value according to the Serialization Format (e.g. JSON)

#### Constraints

##### maxLength
The value of this constraint MUST be a non-negative integer.
A string instance is valid against this constraint if its length is less than, or equal to `maxLength`. The length of a string instance is defined as the number of its characters.

##### minLength
The value of this constraint MUST be a non-negative integer.
A string instance is valid against this constraint if its length is greater than, or equal to `minLength`. The length of a string instance is defined as the number of its characters.

##### startsWith
The value of this constraint MUST be a string. An instance is valid if it begins with the characters of the constraint's string.

##### endsWith
The value of this constraint MUST be a string. An instance is valid if it ends with the characters of the constraint's string.

##### includes
The value of this constraint MUST be a string. An instance is valid if constraint's value may be found within the instance string.

##### regex
The value of this constraint MUST be a string. This string SHOULD be a valid regular expression, according to the ECMA 262 regular expression dialect. An instance is valid if the regular expression matches the instance successfully. Recall: regular expressions are not implicitly anchored.

##### oneOf
The same as for [oneOf for @numberValue](#numbervalue)

##### equals
The same as for [equals for @numberValue](#numbervalue)

#### Examples

```graphql
scalar AlphaNumeric @stringValue(
  regex: "^[0-9a-zA-Z]*$"
)
```
*Examples of valid values*: `"foo1"`, `"Apollo13"`, `123test`

*Examples of invalid values*: `3`, `"dash-dash"`, `admin@example.com`


### @booleanValue

`@booleanValue` directive is used to describe possible boolean values.
Instance is valid if it is a boolean value according to the Serialization Format (e.g. JSON)

#### Constraints

##### equals
Same as for [equals for @numberValue](#numbervalue)

## Wrapper Constraints Directives

### Applicability

Wrapper Constraints Directives can be applied to the following GraphQL entities:
- Object Field Definition
- Input Object Field Definition
- Object Field Arguments Definition
- Directive Argument Definition

**NOTE**: Wrapper Constraints Directives can't be applied to Scalar Definitions

Wrapper Constraints Directives do not override GraphQL standard wrappers semantics and runtime behavior. Moreover, each Wrapper Constraints Directive is compatible only with specific GraphQL wrapper.

Applying a directive to a field definition, an argument definition or an input field definition without matching a wrapper type should result in an error.

### @list
`@list` directive is used to describe list values. Instance is valid if it is a list according to the Serialization Format (e.g. JSON)

#### Constraints

##### maxItems
The value of this constraint MUST be a non-negative integer. An instance is valid if only its size is less than, or equal to, the value of this directive.

##### minItems
The value of this constraint MUST be a non-negative integer.
An instance is valid against `minItems` if its size is greater than, or equal to, the value of this constraint.
Omitting this constraint has the same behavior as a value of 0.

##### uniqueItems
The value of this constraint MUST be a boolean.
If it has boolean value `true`, the instance is valid if all of its elements are unique.

##### innerList
This constraint is used to describe constraints of the nested List. The value of this constraint MUST be an object. This may contain fields with the same name as arguments of `@list` directive. Semantic of these fields is the same but applied to the inner list.

#### Examples

```graphql
type Foo {
  point3D: [Float] @list(
    maxItems: 3,
    minItems: 3
  )
}
```
*Examples of valid values for `point3D` field*: `[1, 2, 3]`, `[-10, 2.5, 100]`

*Examples of invalid values for `point3D` field*: `[-1, 0]`, `[-1, 0, 100, 0]`



```graphql
type Foo {
  pointOnScreen: [Float] @list(
    maxItems: 2,
    minItems: 2
  ) @numberValue(min: 0.0)
}
```

*Examples of valid values for `pointOnScreen` field*: `[1, 2.5]`, `[0, 100]`

*Examples of invalid values for `pointOnScreen` field*: `[-10, 100]`, `[100, -100]`, `[0, 0, 0]`

```graphql
type ticTacToe {
  board: [[String!]!] @list(
    minItems: 3,
    maxItems: 3
    innerList: {
      minItems: 3,
      maxItems: 3
    }
  ) @stringValue(oneOf: [" ","X", "O"])
}
```

*Examples of valid value for `board` field*:
``` json
[
  [" ", " ", " "],
  [" ", "X", " "],
  ["O", " ", " "]
]
```

*Examples of invalid values*: `[]`, `[[],[],[]]`, `"Empty board"`,
``` json
[
  [" ", " ", " "],
  [" ", "Y", " "],
  ["N", " ", " "]
]
```

## Error messages
**TBD**

Some ideas:

**Variant1**:
Allows to provide a single error for multiple constraints but not intuitive structure
```
age: Float @numberValue(
  min: 0,
  max: 100,
  errors: [{
    constraints: [min, max]
    message: "Invalid age ${value}, should be between ${min} and ${max} years"
    code: "AGE_OUT_OF_RANGE"
  }]
)
```

**Variant2**:
Simpler in terms of DX but some messages duplication
```
age: Integer @numberValue(
  min: 0,
  max: 100,
  errors: {
    min: {
      message: Invalid age ${value}, should be greater than ${min} years",
      code: "AGE_LT_MINIMUM"
    },
    max: {
      message: "Invalid age ${value}, should be between less than {max} years",
      code: "AGE_GT_MAXIMUM"
    }
  }
)
```

## Appendix A: Some Examples

```graphql
scalar IntOrFalse @numberValue(multipleOf: 1) @booleanValue(equals: false)
```

*Examples of valid values*: `2`, `50`, `false`

*Examples of invalid values*: `2.5`, `true`, `"string"`

```graphql
scalar FloatOrBoolean @numberValue @booleanValue
```

*Examples of valid values*: `2`, `50.3`, `false`, `true`

*Examples of invalid values*: `"string"`, `[]`


```
type Foo {
  bar: [Int] @numberValue(
    multipleOf: 0.01
  ) @list({
    minItems: 1,
    maxItems: 3,
    uniqueItems: true
  })
}
```

*Examples of valid values for `bar` field*: `[1, 2, 3]`, `[0.01, 0.02]`, `[0.99]`

*Examples of invalid values for `bar` field*: `[0.999]`, `[]`, `[1, 2, 3, 4]`, `[1.001, 2]`, `[1, 1]`

```
type Query {
 allPersons(
   first: Integer @numberValue(min: 1, max: 25)
   after: String
   last: Integer @numberValue(min: 1, max: 25)
   before: String
 ): [Foo!]
}
```

*Examples of valid values for `first` and `last` input arguments*: `1`, `25`, `10`

*Examples of invalid values for `first` and `last` input arguments*: `0`, `30`

# Appendix B: IDL
**TBD**
