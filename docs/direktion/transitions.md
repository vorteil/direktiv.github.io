---
layout: default
title: State Transitions
nav_order: 30
parent: Direktion (scripting)
---

# State Transitions

Unless explicitly defined, `direktion` scripts execute sequentially (line-by-line). The following `direktion` script example shows 3 states (`State1`, `State2`, and `State3`) that transition to each other without the transition being explicitly defined...

### Example

```
workflow example 

State1:
log('.')

State2:
log('.')

State3:
log('.')
```

... whereas the equivalent logic expressed in YAML requires that the `transition` field exists for each state that is not the end of the workflow:

```yaml
id: example
states:
- id: State1
  type: noop
  log: .
  transition: State2
- id: State2
  type: noop
  log: .
  transition: State3
- id: State3
  type: noop
  log: .
```

## Goto

Additionally, users may inclue `goto` commands in to order bypass the natural sequential progression of a `direktion` script. It is important to carefully consider the consequences of using a `goto` to avoid any undesired repetition of logic. The `exit` command can be used to end the workflow.

### Example

The following example will begin with `State1`, transition to `State3`, and then finally to `State2`.

```
workflow example

State1:
log('.')
goto State3

State2:
log('.')
exit

State3:
log('.')
goto State2
```

This same logic takes the following form in YAML:

```yaml
id: example
states:
- id: State1
  type: noop
  log: .
  transition: State3
- id: State2
  type: noop
  log: .
- id: State3
  type: noop
  log: .
  transition: State2
```
