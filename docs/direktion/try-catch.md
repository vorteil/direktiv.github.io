---
layout: default
title: Error handling
nav_order: 10
parent: Direktion (scripting)
---

# Error handling

'direktion' provides a much simpler (and possibly more familiar) approach to error handling. Using a 'try-catch' approach, it is possible to handle any number of different errors from any number of states.

## Example

```
workflow helloworld

var request = isolate(
	image: "vorteil/request:v5",
)

try {
    request(input:'{
        "method": "GET",
        "host": "https://jsonplaceholder.typicode.com/posts"
    }')
    transform('{ response: .return.status }')

    request(input:'{
        "method": "GET",
        "host": "https://jsonplaceholder.typicode.com/posts"
    }')
    transform('{ response: .return.status }')
} catch 'com.example.*' {
    log('("caught com.example.* error")')
} catch 'com.request.*' {
    log('("caught com.request.* error")')
} catch 'direktiv.cancels.timeout' {
    log('("timed out")')
}
```

## Explained

Converting this script in to a Direktiv workflow file (YAML) reveals what is actually going on here. We can see that for both states (`request(...)`) that were defined within the `try` statement, an individual state was created with error catching matching the `catch` statements.

```yaml
id: helloworld
functions:
- id: request
  image: vorteil/request:v5
states:
- id: request-0
  type: action
  catch:
  - error: com.example.*
    transition: log-0
  - error: com.request.*
    transition: log-1
  - error: direktiv.cancels.timeout
    transition: log-2
  action:
    function: request
    input: '{ "method": "GET", "host": "https://jsonplaceholder.typicode.com/posts"
      }'
  async: false
  transform: '{ response: .return.status }'
  transition: request-1
- id: request-1
  type: action
  catch:
  - error: com.example.*
    transition: log-0
  - error: com.request.*
    transition: log-1
  - error: direktiv.cancels.timeout
    transition: log-2
  action:
    function: request
    input: '{ "method": "GET", "host": "https://jsonplaceholder.typicode.com/posts"
      }'
  async: false
  transform: '{ response: .return.status }'
- id: log-0
  type: noop
  log: ("caught com.example.* error")
- id: log-1
  type: noop
  log: ("caught com.request.* error")
- id: log-2
  type: noop
  log: ("timed out")
```

