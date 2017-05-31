# graphql-constraints-spec
RFC for GraphQL Constraints

## GraphQL Constraints

Semantics from as much as possible reused from:
http://json-schema.org/latest/json-schema-validation.html

** Reference implementation is comming **

@numberValue
multipleOf
maximum
minimum
exclusiveMaximum
exclusiveMinimum
enum
const

@stringValue
maxLength
minLength
pattern
enum
const

@booleanValue
const

@list
additionalItems
maxItems
minItems
uniqueItems

byte:Integer @numberValue(
  minimum: 0
  maximum: 256
)

byteMask: Integer @numberValue(
  enum: [ 1, 2, 4, 8, 16, 32, 64, 128, 256]
)

price: Float @numberValue(
  multipleOf: 0.01
  errors: [{
    constraints: [multipleOf]
    message: "Value '${value}' isn't valid price since it doesn't much xx.yy format"
    code: "PRICE_FORMAT"
  }]
)

@numberValue(
  multipleOf: 0.01
) @list({
  minItems: 1,
  maxItems: 10,
  uniqueItems: true,
  innerList: {
    maxItems: 5
    errors: [{
      constrains: [minItems, maxItems]
      message: "Array size is ${value} but it should be from ${minItems} to ${maxItems}"
    }]
  }
})

scalar IntOrFalse @numberValue(multipleOf: 1) @booleanValue(const: false)
scalar FloatOrBoolean @numberValue @booleanValue

scalar AlphaNumeric @stringValue(
  pattern: "^[0-9a-zA-Z]*$",
  errors: [{
    constrains: [pattern],
    message: "Got '${value}' with invalid symbol only letters and digits are allowed"
  }]
)

personConnection(
  first: Integer @numberValue(minimum: 1, maximum: 25)
  after: String
  last: Integer @numberValue(minimum: 1, maximum: 25)
  before: String
)
