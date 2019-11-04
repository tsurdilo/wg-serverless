# Serverless Workflow Specification

## Abstract

Serverless applications are becoming increasingly complex. Nowdays they have to coordinate, manage, and define
the execution order (steps) for countless functions triggered by as many events.

When we are dealing with large number of functions, managing their execution is not a simpler task.
For example we have to coordinate functions and event triggers, orchestrate function execution (sequential,  parallel,
in branches depending on different event triggers), etc.

Workflows have become key components of serverless applications as they excel at orchestration and coordination
of their functional flow.

The goal of the Serverless Workflow sub-group is to come up with a standard way for users to specify their serverless application workflow, as well as help facilitate
portability of serverless applications across different vendor platforms.

Serverless Workflow is a vendor-neutral and portable specification which meets these goals.

## Status of this document

This document is a working draft.

## Table of Contents

- [Introduction](#Introduction)
- [Use Cases](#Use-Cases)
- [Specification Details](#Specification-Details)
  - [Workflow Model](#Workflow-Model)
  - [Workflow Definition](#Workflow-Definition)
- [Examples](#Examples)

## Introduction

Serverless Workflow can be used to:

- **Orchestrate serverless application logic**: serverless applications are typicall event-driven and can be

very hard to manage. Serverless Workflow groups the application events and functions into a coherent unit and
simplifies orchestration of the app logic.

- **Define and coordinate application control flow**: allow the users to define the execution/operation
control flow and how/which functions are to be invoked on arrival of events.

- **Define and manage application data flow**: allows the users to define how data is passed and filtered from incoming events to states,
rom states to functions, from one function to another function, and from one state to another state.

### Functional Scope

Serverless Workflow allows users to:

1. Define and orchestrate steps/states involved in a serverless application.
2. Define which functions are executed in each step.
3. Define which event or combination of events trigger function execution.
4. Define function execution behavior (sequential, parallel, etc).
5. Specify information filtering throughout the execution of the serverless workflow.
6. Define error conditions with retries.
7. If a function is triggered by two or more events, define what label/key should be used to correlate those events to the same serverless workflow instance.

Following example illustrates a Serverless Workflow that involves events
and functions. It specifies the interaction between events, states and functions to be invoked.

<p align="center">
<img src="media/sample-serverless-workflow1.png" with="400px" height="260px" alt="Serverless Workflow Diagram"/>
</p>

## Use Cases

You can find different Serverless Workflow usescases [here](spec-usecases.md)

## Specification Details

In sections below we describe all each section of the Serverless Workflow in details. We first show properties in table format,
and you can also click on the "Click to view JSON Schema" to see the detailed definision defines with [JSON Schema](https://json-schema.org/).

You can find the entire schema document [here](schema/serverless-workflow-schema-01.json). Please note just like this document, this is also
work in progress.

### Workflow Model

Serverless Workflow can be viewed as a collection of states and the transitions and branching between these states.
Each state could have associated events and/or functions. Serverless Workflow may be invoked from a CLI command or triggered dynamically upon arrival of events from event sources.
An event from an event source may also be associated with a specific state within a Serverless Workflow.
States within a Serverless Workflow can wait on the arrival of an event or events from one or more event sources before performing their associated action and progressing to the next state.
Additional workflow functionality includes:

- Results from a cloud function can be used to initiate retry operations or determine which function to execute next or which state to transition to.

- Provide a way to filter and transform the JSON event payload as it progresses through the Serverless Workflow.

- Provide a way for the application developer to specify a unique field in the event that can be used to correlate events from the event sources to the same serverless workflow instance

A Serverless Workflow can be naturally modeled as a state machine.
Specification of a workflow is called a workflow template.
Instantiation of the workflow template is called a workflow instance.

Serverless Workflow definition/specification provides following list of states:

* **[Event State](#Event-State)**: Used to wait for events from event sources and then to invoke one or more functions to run in sequence or in parallel.

* **[Operation State](#Operation-State)**: Allows one or more functions to run in sequence or in parallel without waiting for any event.

* **[Switch State](#Switch-State)**: Permits transitions to multiple other states (eg. Different function results in the previous state trigger branching/transition to different next states).

* **[Delay State](#Delay-State)**: Causes the workflow execution to delay for a specified duration or until a specified time/date.

* **[Parallel State](#Parallel-State)**: Allows a number of states to execute in parallel.

* **[End State](#End-State)**: Terminates the workflow with Fail/Success.

### Workflow Definition

Here we define details of the Serverless Workflow definitions:

| Parameter | Description | Type | Required |
| --- | --- |  --- | --- |
| name | Workflow name | string |yes |
| exec-status |Workflow execution status | string |no |
| [trigger-defs](#Trigger-Definition) |Array of workflow triggers | array | no |
| [states](#State-Definition) | Array of workflow states | array | yes |

<details><summary><strong>Click to view JSON Schema</strong></summary>
<p>

```json
{
    "$id": "https://wg-serverless.org/workflow.schema",
    "$schema": "http://json-schema.org/draft-07/schema#",
    "description": "Vendor-neutral and portable specification that standardizes the definition of serverless application flows",
    "type": "object",
    "properties": {
        "name": {
            "type": "string",
            "description": "Workflow name"
        },
        "exec-status": {
            "type" : "string",
            "enum": ["SYS.Timeout", "SYS.Fail", "SYS.Success", "SYS.InvalidParameter"],
            "description": "Execution status"
        },
        "trigger-defs": {
            "type": "array",
            "description": "Trigger Definitions",
            "items": {
                "type": "object",
                "$ref": "#definitions/triggerevent"
            }
        },
        "states": {
            "type": "array",
            "description": "State Definitions",
            "items": {
                "type": "object",
                "anyOf": [
                    { "$ref": "#definitions/delaystate" },
                    { "$ref": "#definitions/endstate" },
                    { "$ref": "#definitions/eventstate" },
                    { "$ref": "#definitions/operationstate" },
                    { "$ref": "#definitions/parallelstate" },
                    { "$ref": "#definitions/switchstate" }
                ]
            }
        }
    },
    "required": ["name", "states"]
}
```

</p>
</details>

### Trigger Definition

Triggers define incoming events which can be associated with invocation of one or more states.
If there are multiple events involved in an application workflow, a correlation-token, which is used to correlate an event with other
events for same workflow instance, must be specified in that event trigger.

| Parameter | Description | Type | Required |
| --- | --- | --- | --- |
| name | Unique trigger name | string |yes |
| source |CloudEvent source | string | yes |
| type |CloudEvent type | string | yes |
| correlation-token | path used for event correlation | string | no |

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
    "type": "object",
    "properties": {
        "name": {
            "type": "string",
            "description": "Trigger unique name"
        },
        "source": {
            "type": "string",
            "description": "CloudEvent source"
        },
        "type": {
            "type": "string",
            "description": "CloudEvent type"
        },
        "correlation-token": {
            "type": "string",
            "description": "Path used for event correlation."
        }
    },
    "required": ["name", "source", "type"]
}
```

</details>

### State Definition

States define building blocks of the Serverless Workflow. The specification defines six different types of states:

- **[Event State](#Event-state)**: Used to wait for events from event sources and
    then to invoke one or more functions to run in sequence or in parallel.

- **[Operation State](#Operation-State)**: Allows one or more functions to run in sequence
    or in parallel without waiting for any event.

- **[Switch State](#Switch-State)**: Permits transitions to multiple other states (eg.
    Different function results in the previous state trigger
    branching/transition to different next states).

- **[Delay State](#Delay-State)**: Causes the workflow execution to delay for a
    specified duration or until a specified time/date.

- **[Parallel State](#Parallel-State)**: Allows a number of states to execute in
    parallel.

- **[End State](#End-State)**: Terminates the workflow with Fail/Success.

We will start defining each individual state:

### <img src="media/state-icon-small.png" with="30px" height="26px"/>Event State

| Parameter | Description | Type | Required |
| --- | --- | --- | --- |
| name | state name (unique) | string | yes |
| type |start type | string | yes |
| start |Is this state a start state | boolean | no |
| [events](#eventstate-eventdef) |Array of event | array | yes |

<details><summary><strong>Click to view JSON Schema</strong></summary>
<p>

```json
{
    "type": "object",
    "description": "This state is used to wait for events from event sources and then to invoke one or more functions to run in sequence or in parallel.",
    "properties": {
        "name": {
            "type": "string",
            "description": "Unique name of the state"
        },
        "type": {
            "type" : "string",
            "enum": ["EVENT"],
            "description": "State type"
        },
        "start": {
            "type": "boolean",
            "default": false,
            "description": "Is the start event"
        },
        "events": {
            "type": "array",
            "description": "Event State Definitions",
            "items": {
                "type": "object",
                "$ref": "#definitions/event"
            }
        }
    },
    "required": ["name", "type", "events"]
}
```

</p>
</details>

Event state can hold one or more events definitions, so let's define those:

#### <a name="eventstate-eventdef"></a> Event State: Event Definitions

| Parameter | Description | Type | Required |
| --- | --- | --- | --- |
| event-expression |Expression used to associate trigger-defs with event state  | string | yes |
| timeout |Time period to wait for the events in the event-expression | string | no |
| action-mode |Specifies if functions are executed in sequence of parallel | string | no |
| [actions](#Action-Definition) |Array of actions | array | yes |
| next-state|Next state to transition to after all the actions for the matching event have been successfully executed | string | yes |

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
    "type": "object",
    "description": "Event associated with a State",
    "properties": {
        "event-expression": {
            "type": "string",
            "description": "Boolean expression which consists of one or more Event operands and the Boolean operators"
        },
        "timeout": {
            "type": "string",
            "description": "Specifies the time period waiting for the events in the event-expression"
        },
        "action-mode": {
            "type" : "string",
            "enum": ["SEQUENTIAL", "PARALLEL"],
            "description": "Specifies whether functions are executed in sequence or in parallel"
        },
        "actions": {
            "type": "array",
            "description": "Action Definitions",
            "items": {
                "type": "object",
                "$ref": "#definitions/action"
            }
        },
        "next-state": {
            "type": "string",
            "description": "Name of the next state to transition to after all the actions for the matching event have been successfully executed"
        }
    },
    "required": ["event-expression", "actions", "next-state"]
}
```

</details>

The event expression attribute is used to associate this event state with one or more trigger events.

Note that each event definition has a "next-state" property, which is used to idetify the state which
should get executed after this event completes (value should be the unique name of a state).

Each event state's event definition includes one or more actions. Let's define these actions now:

#### Action Definition

| Parameter | Description | Type | Required |
| --- | --- | --- | --- |
| [function](#Function-Definition) |Function to be invoked | object | yes |
| timeout |Max amount of time in seconds to wait for the completion of the function's execution | integer | no |
| [retry](#Retry-Definition) |Defines if funtion execution needs a retry | object | no |

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
    "type": "object",
    "description": "Action Definition",
    "properties": {
        "function": {
            "$ref": "#definitions/function",
            "description": "Function to be invoked"
        },
        "timeout": {
            "type": "integer",
            "default":"0",
            "minimum": 0,
            "description": "Specifies the maximum amount of time in seconds to wait for the completion of the function's execution. This must be a positive integer. The function timer is started when the request is sent to the invoked function"
        },
        "retry": {
            "type": "object",
            "$ref": "#definitions/retry",
            "description": "Specifies how the result from a function is to be handled"
        }
    },
    "required": ["function"]
}
```

</details>

An action defines a collection of functions that are to be invoked when this action is triggered.
It also defines a timeout wait period if one is needed, as well as a retry definition, so lets look at those now:

#### Function Definition

| Parameter | Description | Type | Required |
| --- | --- | --- | --- |
| name |Function name | string | yes |
| type |Function type. Implementors may define custom types. | string | yes |
| parameters |Function parameters | object | no |

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "description": "Function name"
    },
    "type": {
      "type": "string",
      "description": "Type of function to implement. Implementors may define custom types here."
    },
    "parameters": {
      "type": "object",
      "description": "Function parameters"
    }
  },
  "required": ["name", "type"]
}
```

</details>

The function name is a string that can evaluate to a call and execution of a serverless function. Implementors
can define how this string maps to their actual function call(s). Functions can have a type
as well as define parameters (key/value pairs).

#### Retry Definition

| Parameter | Description | Type | Required |
| --- | --- | --- | --- |
| match |Result matching value | string | yes |
| retry-interval |Interval value for retry | integer | no |
| max-retry |Max retry value | integer | no |
| next-state |Name of the next state to transition to when exceeding max-retry limit | string | yes |

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
    "type": "object",
    "description": "Retry Definition",
    "properties": {
        "match": {
            "type": "string",
            "description": "Specifies the matching value for the result"
        },
        "retry-interval": {
            "type": "integer",
            "default":"0",
            "minimum": 0,
            "description": "Specifies retry interval"
        },
        "max-retry": {
            "type": "integer",
            "default":"0",
            "minimum": 0,
            "description": "Specifies the max retry"
        },
        "next-state": {
            "type": "string",
            "description": "Name of the next state to transition to when exceeding max-retry limit"
        }
    },
    "required": ["match", "next-state"]
}
```

</details>

### <img src="media/state-icon-small.png" with="30px" height="26px"/>Operation State

| Parameter | Description | Type | Required |
| --- | --- | --- | --- |
| name |State name | string | yes |
| type |State type | string | yes |
| start |Is this event a start | boolean | no |
| action-mode |Should actions be executed sequentially or in parallel | string | yes |
| [actions](#Action-Definition) |Array of actions | array | yes |
| next-state |State to transition to after all the actions have been successfully executed | string | yes |

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
    "type": "object",
    "description": "This state allows one or more functions to run in sequence or in parallel without waiting for any event.",
    "properties": {
        "name": {
            "type": "string",
            "description": "Unique name of the state"
        },
        "type": {
            "type" : "string",
            "enum": ["OPERATION"],
            "description": "State type"
        },
        "start": {
            "type": "boolean",
            "default": false,
            "description": "Is the start event"
        },
        "action-mode": {
            "type" : "string",
            "enum": ["SEQUENTIAL", "PARALLEL"],
            "description": "Specifies whether actions are executed in sequence or in parallel."
        },
        "actions": {
            "type": "array",
            "description": "Actions Definitions",
            "items": {
                "type": "object",
                "$ref": "#definitions/action"
            }
        },
        "next-state": {
            "type": "string",
            "description": "Name of the next state to transition to after all the actions have been successfully executed"
        }
    },
    "required": ["name", "action-mode", "actions", "type", "next-state"]
}
```

</details>

Unlike Event states, Operation states do not wait for an incoming trigger event. When they
are invoked, their set of actions are executed in SEQUENTIAL, or PARALALLEL modes. Once these
actions execute, a transition to "next state" happens.

### <img src="media/state-icon-small.png" with="30px" height="26px"/>Switch State

| Parameter | Description | Type | Required |
| --- | --- | --- | --- |
| name |State name | string | yes |
| type |State type | string | yes |
| start |Is this event a start | boolean | no |
| [choices](#switch-state-choices) |Ordered set of matching rules to determine which state to trigger next | array | yes |
| default |Name of the next state if there is no match for any choices value | string | yes |

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
    "type": "object",
    "description": "Permits transitions to other states based on criteria matching.",
    "properties": {
        "name": {
            "type": "string",
            "description": "Unique name of the state"
        },
        "type": {
            "type" : "string",
            "enum": ["SWITCH"],
            "description": "State type"
        },
        "start": {
            "type": "boolean",
            "default": false,
            "description": "Is the start event"
        },
        "choices": {
          "type": "array",
          "description": "Defines an ordered set of Match Rules against the input data to this state",
          "items": {
            "ref": "#/definitions/choice"
          }
        },
        "default": {
            "type": "string",
            "description": "Specifies the name of the next state if there is no match for any choices value"
        }
    },
    "required": ["name", "type", "choices", "default"]
}
```

</details>

Switch states can be viewed as gateways. They define matching choices which then define which state should be
triggered next upon successful match.
The current definitions of switch state defines multiple "choices". And, Not, and Or choices
define a single boolean operator which is to be applied to its elements.
Multiple or nesting of boolean operators is currently not allowed.
We will lift this limitation to allow a more flexible approach in the near future.

#### <a name="switch-state-choices"></a>Switch State: Choices

Switch states can be viewed as gateways. They define matching choices which then define which state should be
triggered next upon successful match.

A choice is composed by a `condition` and a `next-state`. If the result of the `condition`
 is successful, the workflow will transition to the `next-state`.

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
    "type": "object",
    "description": "Each choice of a SWITCH state",
    "allOf": [
        {
            "ref": "#/definitions/condition"
        },
        {
            "properties": {
                "next-state": {
                    "type": "string",
                    "description": "Specifies the name of the next state to transition to if there is a value match"
                }
            }
        }
    ]
}
```

</details>

#### <a name="switch-state-conditions"></a>Switch State: Conditions

A condition allows to build complex logic for a `choice`

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
    "type": "object",
    "anyOf": [
        {
            "title": "Single condition",
            "$ref": "#/definitions/singlecondition"
        },
        {
            "title": "And condition",
            "$ref": "#/definitions/andcondition"
        },
        {
            "title": "Not condition",
            "$ref": "#/definitions/notcondition"
        },
        {
            "title": "Or condition",
            "$ref": "#/definitions/orcondition"
        }
    ]
}
```

</details>

There are found types of conditions defined:

- [Single condition](#switch-state-single-condition)
- [And condition](#switch-state-and-condition)
- [Not condition](#switch-state-not-condition)
- [Or condition](#switch-state-or-condition)

##### <a name="switch-state-single-condition"></a>Switch State conditions: Single condition

| Parameter | Description | Type | Required |
| --- | --- | --- | --- |
| path |Path that selects the data input value to be matched | string | yes |
| value |Matching value | string | yes |
| operator |Data Input comparator | string | yes |

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
    "type": "object",
    "description": "Single condition",
    "properties": {
        "path": {
            "type": "string",
            "description": "JSON Path that selects the value of the input data to be matched"
        },
        "value": {
            "type": "string",
            "description": "Matching value"
        },
        "operator": {
            "type": "string",
            "enum": ["Exists", "Equals", "LessThan", "LessThanEquals", "GreaterThan", "GreaterThanEquals"],
            "description": "Specifies how the input data is compared with the values"
        }
    },
    "required": ["path", "value", "operator"]
}
```

</details>

##### <a name="switch-state-and-condition"></a>Switch State conditions: And condition

| Parameter | Description | Type | Required |
| --- | --- | --- | --- |
| and |List of conditions | array | yes |

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
    "type": "object",
    "description": "And condition",
    "properties": {
        "and": {
            "type": "array",
            "description": "List of conditions",
            "items": {
                "type": "object",
                "$ref": "#/definitions/condition"
            }
        }
    },
    "required": ["and"]
}
```

</details>

##### <a name="switch-state-not-condition"></a>Switch State conditions: Not condition

| Parameter | Description | Type | Required |
| --- | --- | --- | --- |
| not |condition | object | yes |

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
    "type": "object",
    "description": "Not condition",
    "properties": {
        "not": {
            "type": "object",
            "$ref": "#/definitions/condition",
            "description": "Not condition"
        }
    },
    "required": ["not"]
}
```

</details>

##### <a name="switch-state-or-condition"></a>Switch State conditions: Or condition

| Parameter | Description |  Type | Required |
| --- | --- | --- | --- |
| or |List of conditions | array | yes |

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
    "type": "object",
    "description": "Or condition",
    "properties": {
        "or": {
            "type": "array",
            "description": "List of conditions",
            "items": {
                "type": "object",
                "$ref": "#/definitions/condition"
            }
        }
    },
    "required": ["or"]
}
```

</details>

### <img src="media/state-icon-small.png" with="30px" height="26px"/>Delay State

| Parameter | Description | Type | Required |
| --- | --- | --- | --- |
| name |State name | string | yes |
| type |State type | string | yes |
| start |If this is a start event | boolean | no |
| time-delay |Amount of time (seconds) to delay when in this state | integer | yes |
| next-state |Name of the next state to transition to after the delay | string | yes |

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
    "type": "object",
    "description": "Causes the workflow execution to delay for a specified duration",
    "properties": {
        "name": {
            "type": "string",
            "description": "Unique name of the state"
        },
        "type": {
            "type" : "string",
            "enum": ["DELAY"],
            "description": "State type"
        },
        "start": {
            "type": "boolean",
            "default": false,
            "description": "Is the start event"
        },
        "time-delay": {
            "type": "integer",
            "default":"0",
            "minimum": 0,
            "description": "Amount of time in seconds to delay in this state"
        },
        "next-state": {
            "type": "string",
            "description": "Name of the next state to transition to after the delay"
        }
    },
    "required": ["name", "type", "time-delay", "next-state"]
}
```

</details>

Delay state simple waits for a certain amount of time before transitioning to a next state.

### <img src="media/state-icon-small.png" with="30px" height="26px"/>Parallel State

| Parameter | Description | Type | Required |
| --- | --- | --- | --- |
| name |State name | string | yes |
| type |State type | string | yes |
| start |If this is a start event | boolean | no |
| [branches](#parallel-state-branch) |List of branches for this parallel state| array | yes |
| next-state |Name of the next state to transition to after all branches have completed execution | string | yes |

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
    "type": "object",
    "description": "Consists of a number of states that are executed in parallel",
    "properties": {
        "name": {
            "type": "string",
            "description": "Unique name of the state"
        },
        "type": {
            "type" : "string",
            "enum": ["PARALLEL"],
            "description": "State type"
        },
        "start": {
            "type": "boolean",
            "default": false,
            "description": "Is the start event"
        },
        "branches": {
            "type": "array",
            "description": "Branch Definitions",
            "items": {
                "type": "object",
                "$ref": "#definitions/branch"
            }
        },
        "next-state": {
            "type": "string",
            "description": "Specifies the name of the next state to transition to after all branches have completed execution"
        }
    },
    "required": ["name", "type", "branches", "next-state"]
}
```

</details>

Parallel state defines a collection of branches which are to be executed in parallel.
Each branch consists of a collection of states. It can be regarded as a sub-workflow
which must have a start (state with "start" property set to true) and an end state which
represents its completion.

Let's define a branch now:

#### <a name="parallel-state-branch"></a>Parallel State: Branch

| Parameter | Description | Type | Required |
| --- | --- | --- | --- |
| name |State name | string | yes |
| [states](#State-Definition) |List of states to be executed in this branch | array | yes |
| wait-for-completion |If workflow execution must wait for this branch to finish before continuing | boolean | yes |

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
    "type": "object",
    "description": "Branch Definition",
    "properties": {
        "name": {
            "type": "string",
            "description": "Branch name"
        },
        "states": {
            "type": "array",
            "description": "State Definitions",
            "items": {
                        "type": "object",
                        "anyOf": [
                            { "$ref": "#definitions/delaystate" },
                            { "$ref": "#definitions/eventstate" },
                            { "$ref": "#definitions/operationstate" },
                            { "$ref": "#definitions/parallelstate" },
                            { "$ref": "#definitions/switchstate" },
                            { "$ref": "#definitions/endstate" }
                        ]
                    }
        },
        "wait-for-completion": {
            "type": "boolean",
            "default": false,
            "description": "Workflow execution must wait for this branch to finish before continuing"
        }
    },
    "required": ["name", "states", "wait-for-completion"]
}
```

</details>

Each branch receives a copy of the Parallel state's input data.
Transitions for states within a branch can only be to other states in that branch.
In addition, states outside a Parallel state cannot transition to a state within a branch of a Parallel state.
The Parallel state generates an output array in which each element is the output for a branch.
The elements of the output array need not be of the same type.

The "wait-for-completion" property allows the parallel state to manage branch executions. If this flag is set to
true, the branches parallel parent state must wait for this branch to finish before continuing execution.

### <img src="media/state-icon-small.png" with="30px" height="26px"/>End State

| Parameter | Description | Type | Required |
| --- | --- | --- | --- |
| name |State name | string | yes |
| type |State type | string | yes |
| status |Workflow termination status | string | no |

<details><summary><strong>Click to view JSON Schema</strong></summary>

```json
{
    "type": "object",
    "description": "End State",
    "properties": {
        "name": {
            "type": "string",
            "description": "Unique name of the state"
        },
        "type": {
            "type" : "string",
            "enum": ["END"],
            "description": "State type"
        },
        "status": {
            "type" : "string",
            "enum": ["SUCCESS", "FAILURE"],
            "description": "Specifies the workflow termination status"
        }
    },
    "required": ["name", "type"]
}
```

</details>

End state defines the ending transition of the workflow. It provides a stats of successful or failed
workflow execution.

### Information Passing

The diagram below shows data flow through a Serverless Workflow that includes an
Event state that invokes two serverless functions. Output data from one state is
passed as input data to the next state. Filters are used to filter and transform
the data on ingress to and egress from each state. Input data from a previous
state may be delivered to serverless function when it is invoked from an
Operation state in the Workflow.

Data contained in a response from a serverless function is sent as output data
to the next state. If a state (Operation state or Event state) includes a list
of sequential actions, data contained in the response from one serverless
function is filtered and then sent in the request to the next function.

In an Event state, CloudEvent metadata received in a request from an event
source may be transformed and combined with data received from a previous state
before it is delivered to a serverless function. Likewise CloudEvent metadata
received in a response from a serverless function may be transformed and
combined with data received from a previous state before it is delivered in a
response sent to the event source.

<p align="center">
<img src="media/information-passing1.png" with="400px" height="260px" alt="Async Event Diagram"/>
</p>

There may be cases where an event source such as an API gateway expects to
receive a response from the workflow. In this case CloudEvent metadata received
in a response from a serverless function may be transformed and combined with
data received from a previous state before it is delivered in a response sent to
the event source as shown below.

<p align="center">
<img src="media/information-passing2.png" with="400px" height="260px" alt="Sync Event Diagram"/>
</p>

### Filter Mechanism

Serverless Workflow maintains an implicit JSON object which is accessed from each
filter via JSONPath expression '$.'

There are three kinds of filters

- Event Filter
  - Invoked when data is passed from an event to the current state
- State Filter
  - Invoked when data is passed from the previous state to the current state
  - Invoked when data is passed from the current state to the next state
- Action Filter
  - Invoked when data is passed from the current state to the first action
  - Invoked when data is passed from an action to action
  - Invoked when data is passed from the last action to the current state

Each Filter has three kinds of path filters

- InputPath
  - Select input data of either Event, State or Action as JSONPath
  - Default value is '\$'
- ResultPath
  - Specify result JSON node of Action Output as JSONPath
  - Default value is '\$'
- OutputPath
  - Specify output data of State or Action as JSONPath
  - Default value is '\$'

<p align="center">
<img src="media/filter-sequential.png" with="480px" height="270px" alt="Sequential FilterDiagram"/>
</p>

<p align="center">
<img src="media/filter-parallel.png" with="480px" height="270px" alt="Parallel FilterDiagram"/>
</p>

## Examples

You can find different Serverless Workflow examples [here](spec-examples.md)