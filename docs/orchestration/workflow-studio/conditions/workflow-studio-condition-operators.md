---
title: Condition Operators Reference
---

# Condition Operators Reference

When configuring rules inside a condition step, you choose an **operator** that defines how the field value is compared to the target value. This page lists every available operator, organized by category.

## Comparison operators

Comparison operators evaluate a single field against a target value. These are the operators you select when adding a rule to a branch.

| Operator | Description | Example use case |
|----------|-------------|------------------|
| `=` (equals) | Matches when the field value is exactly equal to the target value. | Currency equals `EUR` |
| `!=` (not equals) | Matches when the field value is anything other than the target value. | Currency is not `USD` |
| `>` (greater than) | Matches when the field value is greater than the target. Works with numeric fields. | Amount is greater than `1000` |
| `>=` (greater than or equal) | Matches when the field value is greater than or equal to the target. | Amount is at least `500` |
| `<` (less than) | Matches when the field value is less than the target. Works with numeric fields. | Amount is less than `100` |
| `<=` (less than or equal) | Matches when the field value is less than or equal to the target. | Amount is at most `5000` |
| `IN` | Matches when the field value is found within a specified set of values. | Country is in `DE, FR, NL` |
| `NOT IN` | Matches when the field value is not found within a specified set of values. | Country is not in `US, CA` |
| `CONTAINS` | Matches when the field value contains the target as a substring. | Description contains `subscription` |
| `NOT CONTAINS` | Matches when the field value does not contain the target as a substring. | Description does not contain `test` |
| `STARTS WITH` | Matches when the field value begins with the target string. | Card number starts with `4` (Visa) |
| `NOT STARTS WITH` | Matches when the field value does not begin with the target string. | Card number does not start with `3` |
| `ENDS WITH` | Matches when the field value ends with the target string. | Reference ends with `-recurring` |
| `NOT ENDS WITH` | Matches when the field value does not end with the target string. | Reference does not end with `-test` |
| `IS ONE OF` | Matches when the field value matches any item in a predefined list. | Card brand is one of `Visa, Mastercard, Amex` |
| `NOT IS ONE OF` | Matches when the field value does not match any item in a predefined list. | Card brand is not one of `Diners, JCB` |

### IN vs. IS ONE OF

Both **IN** and **IS ONE OF** check whether a value matches one of several options. The difference is in how you provide the list:

- **IN** -- You enter a comma-separated set of values directly in the rule.
- **IS ONE OF** -- You select from a predefined list of known values for that field (for example, a list of supported card brands or currencies).

Use **IS ONE OF** when the field has a known set of valid options. Use **IN** when you need to specify arbitrary values.

## Logical operators

Logical operators control how multiple rules within a branch are combined.

### AND

When rules are combined with **AND**, **all** rules in the group must be true for the branch to match.

**Example:** To match high-value EUR transactions, combine two rules with AND:

- Amount `>` `1000` **AND**
- Currency **=** `EUR`

Both conditions must be true for the branch to activate.

### OR

When rules are combined with **OR**, the branch matches if **at least one** rule in the group is true.

**Example:** To match transactions from any Nordic country, combine rules with OR:

- Country **=** `SE` **OR**
- Country **=** `NO` **OR**
- Country **=** `DK` **OR**
- Country **=** `FI`

The branch activates if the country matches any of these values.

> **Tip:** For this particular example, using a single rule with the **IN** or **IS ONE OF** operator would be simpler: Country **IN** `SE, NO, DK, FI`.

### NOT

The **NOT** operator negates an individual rule. When NOT is enabled on a rule, the rule matches when its comparison is *not* true.

**Example:** Amount **NOT >** `1000` matches transactions where the amount is 1000 or less.

You can combine NOT with any comparison operator. Enable it using the NOT toggle on the rule row in the settings panel.


## Building advanced rule groups

You can combine logical and comparison operators to create sophisticated branching logic:

1. **Add multiple rules** to a branch to test several fields at once.
2. **Choose AND or OR** between rules using the logic toggle.
3. **Enable NOT** on individual rules to negate specific comparisons.

**Example -- Route to a premium provider:**

A branch named "Premium EU card" could use the following rules combined with AND:

- Card brand **IS ONE OF** `Visa, Mastercard`
- Country **IN** `DE, FR, NL, BE, AT`
- Amount `>=` `500`

All three conditions must be satisfied for the branch to match.

## Choosing the right operator

| Goal | Recommended operator |
|------|---------------------|
| Exact match on a single value | **=** |
| Exclude a single value | **!=** |
| Numeric threshold (above) | `>` or `>=` |
| Numeric threshold (below) | `<` or `<=` |
| Match against a custom list | **IN** or **NOT IN** |
| Match against a known set of options | **IS ONE OF** or **NOT IS ONE OF** |
| Partial text match | **CONTAINS**, **STARTS WITH**, or **ENDS WITH** |
| Exclude partial text | **NOT CONTAINS**, **NOT STARTS WITH**, or **NOT ENDS WITH** |