---
title: Conditions and Branching
---

# Conditions and Branching

Condition steps let you route your workflow down different paths based on rules you define. When a workflow reaches a condition step, it evaluates each branch's rules in order and follows the first branch that matches. If no branch matches, the workflow follows the **Default** branch.


## How conditions work

A condition step acts as a decision point in your workflow. You define one or more **branches**, and each branch contains a set of **rules** that determine when that branch should be followed. During execution:

1. The platform evaluates branches from top to bottom.
2. For each branch, it checks whether the rules are satisfied.
3. As soon as a branch's rules match, the workflow follows that branch and skips the rest.
4. If no branch matches, the workflow follows the **Default** branch.

Every condition step includes a Default branch that cannot be removed. This guarantees that the workflow always has a path forward, even when none of your custom rules apply.

## Adding a condition step

To add a condition to your workflow:

1. Click the **+** button on the canvas where you want to insert a decision point.
2. Select **Condition** from the step menu.
3. The condition step appears on the canvas with a single Default branch.


## Adding and managing branches

Each condition step starts with only a Default branch. You add custom branches to define the paths you need:

1. Select the condition step on the canvas to open its settings panel.
2. Click **Add branch** to create a new branch.
3. Give the branch a descriptive name (for example, "High-value transaction" or "EUR currency").
4. Configure the rules for that branch (see below).
5. Repeat to add as many branches as you need.


You can reorder branches by dragging them in the settings panel. The order matters because branches are evaluated from top to bottom, and the first match wins.

To remove a branch, click the delete icon next to the branch name. The Default branch cannot be removed.

## Configuring rules

Each branch contains one or more rules that define when the branch should be followed. A rule compares a field value against a target using an operator.

To add a rule to a branch:

1. Select the condition step and open its settings panel.
2. Expand the branch you want to configure.
3. Click **Add rule**.
4. Choose the **field** you want to evaluate from the dropdown (for example, amount, currency, or card brand).
5. Select an **operator** (for example, equals, greater than, or contains). See [Condition Operators Reference](./operators) for the full list.
6. Enter the **value** to compare against.


## Combining rules with AND / OR logic

When a branch has multiple rules, you control how they combine:

- **AND** -- All rules in the group must be true for the branch to match. Use AND when you need every condition to be met simultaneously.
- **OR** -- At least one rule in the group must be true for the branch to match. Use OR when any single condition is sufficient.

You can switch between AND and OR using the logic toggle between rules in the settings panel.


## Negating rules with NOT

You can negate any individual rule by enabling the **NOT** toggle. When NOT is enabled, the rule matches when the comparison is *not* true. For example, a rule "currency equals USD" with NOT enabled matches when the currency is anything *other than* USD.

## Connecting steps after branches

Each branch on a condition step has its own connection point on the canvas. After adding a condition:

1. Connect a subsequent step to each branch's connection point.
2. Make sure every branch, including Default, has at least one step connected.
3. End every terminal path with a **Notify** step to ensure your integration always receives a response.


## Common use cases

### Route by payment amount

Create branches for different transaction value ranges:

- **High value** -- Amount greater than 1000: route through additional fraud checks before authorization.
- **Standard** -- Amount less than or equal to 1000: proceed directly to authorization.
- **Default** -- Handle any edge cases with a fallback path.

### Route by currency

Direct transactions to different providers based on currency:

- **EUR transactions** -- Currency equals EUR: route to a European payment provider.
- **USD transactions** -- Currency equals USD: route to a US payment provider.
- **Default** -- Route to a global fallback provider.

### Route by card brand

Apply different processing logic by card brand:

- **Visa / Mastercard** -- Card brand is one of Visa, Mastercard: use the primary provider.
- **Amex** -- Card brand equals Amex: use a provider with Amex support.
- **Default** -- Reject unsupported card brands with a Notify step.

### Route by country

Handle regional requirements:

- **EU countries** -- Country is one of a list of EU country codes: include 3DS verification.
- **US** -- Country equals US: skip 3DS and proceed directly.
- **Default** -- Apply a conservative path with additional verification.

## Best practices

- **Always connect every branch.** An unconnected branch causes the workflow to end silently at that point, with no notification sent to your integration.
- **End every terminal path with Notify.** Whether the branch leads to a success or failure scenario, add a Notify step so your systems are always informed.
- **Keep branch names descriptive.** Use names like "High-value EUR" rather than "Branch 1" so the workflow is easy to read at a glance.
- **Order branches by specificity.** Place the most specific rules first and use Default as a catch-all. Since branches are evaluated top to bottom, more specific rules should be checked before broader ones.
- **Start simple.** Begin with two branches plus Default, and add more as needed. You can always refine your conditions later.