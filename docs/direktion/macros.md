---
layout: default
title: Macros
nav_order: 20
parent: Direktion (scripting)
---

# Macros
The 'direktion' scripting language includes a number of features designed to improve readability and bootstrap workflow development. This article will demonstrate the use of 'macros' to reduce the complexity and size of a direktion script.

In this example, we need to perform three actions that each run  the same isolate. In each case, the majority of the input fields remain the same. Instead of duplicating/reassigning values to these fields each time the isolate is invoked, macros allow us to only modify the field that we know must change between each invocation.

## Example

```
workflow  initdb

var psql = isolate(
	image: "vorteil/psql:v1",
)

macro  dbInput(query) = printf(`
    {
        "host": .database.host,
        "port": .database.port,
        "user": .database.user,
        "password": .database.password,
        "database": .database.default,
        "disable-ssl": .database."disable-ssl",
        "query": %v
    }
`, query)

transform('.db1 = "secrets" | .db2 = "api" | .db3 = "keycloak"')

initDB1:
try {
    psql(
        input: dbInput('("CREATE DATABASE " + .db1)'),
    )
} catch  'com.psql.error' {}

initDB2:
try {
    psql(
        input: dbInput('("CREATE DATABASE " + .db2)'),
    )
} catch  'com.psql.error' {}
 
initDB3:
try {
    psql(
        input: dbInput('("CREATE DATABASE " + .db3)'),
    )
} catch  'com.psql.error' {}
```

*Note: This could also be achieved with a for-each loop.*

## Explanation

By defining the `dbInput` macro, only the `query` field needs to be provided to generate the input that is provided to each invocation of the `psql` isolate. 
