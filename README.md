<img src=".github/logo.png" width="190" alt="RulePilot" />

[![npm version](https://badge.fury.io/js/rulepilot.svg)](https://badge.fury.io/js/rulepilot)

| Statements                                                                                  | Functions | Lines                                                                                   |
|---------------------------------------------------------------------------------------------|-----------|-----------------------------------------------------------------------------------------|
| ![Statements](https://img.shields.io/badge/Coverage-100%25-brightgreen.svg) | ![Functions](https://img.shields.io/badge/Coverage-100%25-brightgreen.svg) | ![Lines](https://img.shields.io/badge/Coverage-100%25-brightgreen.svg) |


## Overview

`RulePilot` is a fast and lightweight rule engine for JavaScript. It is designed to be simple to use and easy to
integrate into your application.

The rule engine evaluates human-readable JSON rules against a set of criteria. The rules are evaluated in a top-down
fashion. 

Simple rules can be written to evaluate to a `boolean` value _(indicating whether the criteria tested passes the rule)_.

Otherwise, granular rules can be created, where each condition of the rule can evaluate to a `boolean`, `number`, 
`string`, `object` or `array`. This is particularly useful when you want to evaluate a rule and return a result based on 
each condition's evaluation.

## Features

- Simple to use
- Written in TypeScript
- Human-readable JSON rules
- Fluent rule builder tool (ORM style)
- Runs both in Node and in the browser
- Lightweight with **zero dependencies** & Fast _(10,000 rule evaluations in ~35-40ms)_
- Both Simple & Granular rule evaluation
- Infinite nesting of conditions in rules
- Supports `Any`, `All`, and `None` type conditions
- Supports `Equal`, `NotEqual`, `GreaterThan`, `GreaterThanOrEqual`, `LessThan`, `LessThanOrEqual`, `In`, and `NotIn` operators
- Validation & debug tools for rules

## Usage

### Installation

```bash
npm install rulepilot
```

```bash
yarn add rulepilot
```

### Importing

```typescript
import { RulePilot } from 'rulepilot';
```

For TypeScript users, you can import the `Rule` interface to get type definitions for the rule JSON.

```typescript
import { RulePilot, Rule } from 'rulepilot';

const rule: Rule = {
    // ...
}
```

### Basic Example

Here we are defining a rule which will evaluate to `true` or `false` based on the criteria provided. In this example, 
we are checking whether a user is allowed to benefit from a discount at checkout of not.

For the discount to be applied, the user must be from either the `UK or Finland`, have a `coupon` and the total checkout 
price must be greater than or equal to `120.00`.

```typescript
import { RulePilot, Rule } from 'rulepilot';

// Define a rule which caters for your needs
const rule: Rule = {
    "conditions": {
        "all": [
            {
                "field": "country",
                "operator": "in",
                "value": ["GB", "FI"]
            },
            {
                "field": "hasCoupon",
                "operator": "==",
                "value": true
            },
            {
                "field": "totalCheckoutPrice",
                "operator": ">=",
                "value": 120.00
            }
        ] 
    }
}

// Define the criteria which will be evaluated against the rule
const criteria = {
    country: "GB",
    totalCheckoutPrice: 125.00,
    hasCoupon: true,
}

/**
 * Evaluate the criteria against the rule
 * 
 * The result will be true
 */
let result = RulePilot.evaluate(rule, criteria);

// However, if any of the criteria do not pass the check, the result will be false
criteria.totalCheckoutPrice = 25.00

/**
 * The result will be false
 */
result = RulePilot.evaluate(rule, criteria);
```

We can add additional requirements to the rule, for example apart from the above-mentioned conditions, we can also 
require that the user is either `over 18` years old or `has a valid student card`.

Take note of how the `conditions` property is now an array of objects.

```typescript
import { RulePilot, Rule } from 'rulepilot';

// Define a rule which caters for your needs
const rule: Rule = {
    "conditions": [
        {
            "all": [
                {
                    "field": "country",
                    "operator": "in",
                    "value": ["GB", "FI"]
                },
                {
                    "field": "hasCoupon",
                    "operator": "==",
                    "value": true
                },
                {
                    "field": "totalCheckoutPrice",
                    "operator": ">=",
                    "value": 120.00
                }
            ]
        },
        {
            "any": [
                {
                    "field": "age",
                    "operator": ">=",
                    "value": 18
                },
                {
                    "field": "hasStudentCard",
                    "operator": "==",
                    "value": true
                }
            ]
        }
    ]
}

// Define the criteria which will be evaluated against the rule
const criteria = {
    country: "GB",
    totalCheckoutPrice: 20.00,
    hasCoupon: true,
}

/**
 * The result will be false
 */
let result = RulePilot.evaluate(rule, criteria);

/**
 * The result will be true
 */
criteria.hasStudentCard = true;
result = RulePilot.evaluate(rule, criteria);
```

If we want to add additional requirements to the rule, we can do so by adding another `any` or `all` condition. 

For example, we can add a requirement that a discount will also be given to all users from Sweden as long as they are 
18+ or have a valid student card _(irrelevant of any other conditions set)_.

```typescript
const rule: Rule = {
    "conditions": [
        {
            "any": [
                {
                    "all": [{
                        "field": "country",
                        "operator": "in",
                        "value": ["GB", "FI"]
                    },
                    {
                        "field": "hasCoupon",
                        "operator": "==",
                        "value": true
                    },
                    {
                        "field": "totalCheckoutPrice",
                        "operator": ">=",
                        "value": 120.00
                    }
                ]
            },
            {
                "field": "country",
                "operator": "==",
                "value": "SE"
            }
        ]
    },
    {
        "any": [
            {
                "field": "age",
                "operator": ">=",
                "value": 18
            },
            {
                "field": "hasStudentCard",
                "operator": "==",
                "value": true
            }
        ]
    }
]}
```

The criteria can be narrowed down further by specifying `Swedish` users cannot be from `Stockholm` or `Gothenburg` 
otherwise they must spend `more than 200.00` at checkout.

```typescript
const rule: Rule = {
    "conditions": [{
        "any": [
            {
                "all": [
                    {
                        "field": "country",
                        "operator": "in",
                        "value": ["GB", "FI"]
                    },
                    {
                        "field": "hasCoupon",
                        "operator": "==",
                        "value": true
                    },
                    {
                        "field": "totalCheckoutPrice",
                        "operator": ">=",
                        "value": 120.00
                    }
                ]
            },
            {
                "any": [
                    {
                        "all": [
                            {
                                "field": "country",
                                "operator": "==",
                                "value": "SE"
                            },
                            {
                                "field": "city",
                                "operator": "not in",
                                "value": ["Stockholm", "Gothenburg"]
                            }
                        ]
                    },
                    {
                        "all": [
                            {
                                "field": "country",
                                "operator": "==",
                                "value": "SE"
                            },
                            {
                                "field": "city",
                                "operator": "totalCheckoutPrice",
                                "value": 200.00
                            }
                        ]
                    }
                ]
            }
        ]
    },
    {
        "any": [
            {
                "field": "age",
                "operator": ">=",
                "value": 18
            },
            {
                "field": "hasStudentCard",
                "operator": "==",
                "value": true
            }
        ]
    }
]}
```

### Granular Example

It might be the case that we want to give different discounts to people based on the criteria they meet. For example,
we want to give a 10% discount to all users who `18+` or have a `student card` and a 5% discount to the rest of the
users who meet the other criteria.

To accomplish this, we can assign a `result` to each condition which will be used to calculate the discount.

```typescript
const rule: Rule = {
    "conditions": [{
        "any": [
            {
                "all": [
                    {
                        "field": "country",
                        "operator": "in",
                        "value": ["GB", "FI"]
                    },
                    {
                        "field": "hasCoupon",
                        "operator": "==",
                        "value": true
                    },
                    {
                        "field": "totalCheckoutPrice",
                        "operator": ">=",
                        "value": 120.00
                    }
                ]
            },
            {
                "any": [
                    {
                        "all": [
                            {
                                "field": "country",
                                "operator": "==",
                                "value": "SE"
                            },
                            {
                                "field": "city",
                                "operator": "not in",
                                "value": ["Stockholm", "Gothenburg"]
                            }
                        ]
                    },
                    {
                        "all": [
                            {
                                "field": "country",
                                "operator": "==",
                                "value": "SE"
                            },
                            {
                                "field": "city",
                                "operator": "totalCheckoutPrice",
                                "value": 200.00
                            }
                        ]
                    }
                ]
            }
        ],
        "result": 5
    },
    {
        "any": [
            {
                "field": "age",
                "operator": ">=",
                "value": 18
            },
            {
                "field": "hasStudentCard",
                "operator": "==",
                "value": true
            }
        ],
        "result": 10
    }
]}
```

In such a setup the result of our evaluation will be the value of the `result` property in condition which was met first.

```typescript
import { RulePilot } from 'rulepilot';

// Define the criteria which will be evaluated against the rule
const criteria = {
    country: "GB",
    totalCheckoutPrice: 340.22,
    hasCoupon: true,
}

/**
 * The result will be 5
 */
let result = RulePilot.evaluate(rule, criteria);

criteria.country = "SE";
criteria.city = "Linköping";

/**
 * The result will be 10
 */
result = RulePilot.evaluate(rule, criteria);

criteria.country = "IT";
criteria.age = 17;
criteria.hasStudentCard = false;

/**
 * The result will be false
 */
result = RulePilot.evaluate(rule, criteria);
```

**Important** When using granular rules, the order of rules matters. The first rule which is met will be the one which 
is used to calculate the discount.

#### Condition Types

There are three (3) types of conditions which can be used in a rule:

 - `all` - All conditions must be met
 - `any` - Any condition must be met
 - `none` - No conditions must be met

Each and all of these condition types can be mixed and matched or nested to create complex rules.

#### Defaulting A Rule Result

In granular rules, it is possible to set a default value which will be used if no conditions are met.

```typescript
import { RulePilot, Rule } from 'rulepilot';

const rule: Rule = {
    "conditions": [{
        // ..
    }],
    "default": 2.4
};

/**
 * The result will be 2.4
 */
let result = RulePilot.evaluate(rule, {});
```

In such a setup as seen above, if no conditions are met, the result will be `2.4`.

### Criteria With Nested Properties

In some cases, the criteria which is used to evaluate a rule might be more complex objects with nested properties. 

For example, we might want to evaluate a rule against a `User` object which has a `profile` property which contains
the user's profile information.

To do so, we can use the `.` (dot) notation to access nested properties in the criteria.

```typescript
import { RulePilot, Rule } from 'rulepilot';

const rule: Rule = {
    "conditions": [{
        "field": "profile.age",
        "operator": ">=",
        "value": 18
    }]
};

const criteria = {
    profile: {
        age: 20
    }
};

/**
 * The result will be true
 */
let result = RulePilot.evaluate(rule, criteria);
```

## Validating A Rule

of the rule to ensure it is valid and properly structured.

The `validate()` method will return `true` if the rule is valid, otherwise it will return an error message 
describing the problem along with the problem node from the rule for easy debugging.

```typescript
import { RulePilot, Rule } from 'rulepilot';

const rule: Rule = {
    // ...
}

const result = RulePilot.validate(rule);
```

For TypeScript users, the `ValidationResult` interface can be imported.

```typescript
import { RulePilot, Rule, ValidationResult } from 'rulepilot';

const rule: Rule = {
    // ...
}

const validationResult: ValidationResult = RulePilot.validate(rule);
```

## Fluent Rule Builder

Although creating rules in plain JSON is very straightforward, `RulePilot` comes with a `Builder` class which can be 
used to create rules in a fluent manner.

The `add()` method allows for the addition of a root condition to the rule. This condition can be then setup as required.

The `default()` method allows for the addition of a default value result for the rule.

```typescript
import { RulePilot, Rule } from 'rulepilot';

const b = RulePilot.builder();

const rule: Rule = b
  .add(
    b.condition(
      "all",
      [
        b.condition("any", [
          b.constraint("fieldA", "==", "bar"),
          b.constraint("fieldB", ">=", 2),
        ]),
        b.constraint("fieldC", "not in", [1, 2, 3]),
      ],
      3
    )
  )
  .add(b.condition("none", [], 5))
  .add(b.condition("any", [b.constraint("fieldA", "==", "value")]))
  .default(2)
  .build();
```

## Building The Library

The distribution can be built as follows, with the output being placed in a `dist` directory.

```bash
npm run build
```

```bash
yarn build
```

## Running Unit Tests

Tests are written in Jest and can be run with the following commands:

```bash
npm run jest --testPathPattern=test --detectOpenHandles --color --forceExit
```

```bash
yarn jest --testPathPattern=test --detectOpenHandles --color --forceExit
```