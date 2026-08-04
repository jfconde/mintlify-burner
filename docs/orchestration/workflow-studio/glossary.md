---
title: Glossary
---

# Glossary

This page defines the key terms used throughout the Workflow Studio documentation.

---

### Workflow

A complete payment process built visually in Workflow Studio. A workflow consists of a trigger, one or more steps connected by connection lines, and typically ends with Notify steps on every terminal path. Each workflow belongs to a merchant configuration and can be activated or deactivated independently.

---

### Trigger

The entry point of a workflow. Every workflow has exactly one trigger, which determines what event or API call starts the workflow. Triggers are always the first step on the canvas. Examples include **Start Authorize**, **Start Capture**, and **Handle Dispute Update**.

---

### Action

An operational step that performs a specific task within a workflow. Actions include payment operations (such as Authorize, Capture, Cancel, and Refund), fraud checks, notifications, and more. Each action produces one or more outcomes that determine the next step in the workflow.

---

### Condition

A decision step that evaluates rules to route the workflow down different paths. Each condition has one or more custom branches plus a Default branch. Rules use operators to compare field values, and branches are evaluated from top to bottom. The first branch whose rules match is followed. See [Conditions and Branching](workflow-studio-conditions) for details.

---

### Step

Any individual element on the workflow canvas -- including triggers, actions, and conditions. Steps are the building blocks of a workflow. You connect steps together with connection lines to define the execution order.

---

### Connection Line

A visual line on the canvas that connects one step to the next. Connection lines define the order in which steps execute. They run from an outcome of one step to the input of the next step, and workflow execution always follows connection lines in one direction from the trigger toward the terminal steps.

---

### Outcome

The result produced by an action step that determines what happens next in the workflow. Each outcome creates a separate connection point on the step. Common outcomes include:

- **Completed** -- The action finished successfully.
- **Paused** -- The action is waiting for an external event before it can continue.
- **Requested** -- The action has been submitted and is pending processing.
- **Updated** -- The action received an update (available on some actions such as Authorize, Capture, Cancel, and Refund).

---

### Settings Panel

The configuration panel that opens when you select a step on the canvas. The settings panel lets you configure step-specific options such as condition rules, provider selection, and other parameters. The contents of the settings panel vary depending on the type of step selected.


---

### Provider Selection

A configuration option available on certain action steps that lets you choose which payment provider handles that operation. Actions that support provider selection include Authorize, Fraud Check, Fraud Update, 3DS, and Provision Network Token. You configure the provider in the step's settings panel.

---

### Execution

A single run of a workflow from start to finish. When a trigger fires, it creates an execution that moves through the workflow's steps following the connection lines. Each execution has a unique ID, a status, and a complete record of which steps were executed and what data was processed. You can view executions in the [monitoring interface](workflow-studio-monitoring).

---

### Status Code

A code assigned to an execution that indicates its current state. Status codes are color-coded by severity: **green** for success, **yellow** for in progress, and **red** for failure. Examples include `authorizeSuccessful`, `captureRequested`, and `refundFailed`. See the [Status Codes Reference](workflow-studio-status-codes) for the complete list.

---

### Branch

A path within a condition step. Each branch has a set of rules that determine when it should be followed. You can create multiple custom branches to handle different scenarios. Branches are evaluated from top to bottom, and the first matching branch is followed.

---

### Default Branch

A special branch on every condition step that cannot be removed. The Default branch is followed when none of the custom branch rules match. It acts as a fallback to guarantee the workflow always has a path forward, even when no custom conditions apply.