## 常见的GraphQL路径

```
/graphql
/graphql-console
/graphql-devtools
/graphql-explorer
/graphql-playground
/graphql-playground-html
/graphql.php
/graphql/console
/graphql/graphql
/graphql/graphql-playground
/graphql/schema.json
/graphql/schema.xml
/graphql/schema.yaml
/graphql/v1
/HyperGraphQL
/je/graphql
/laravel-graphql-playground
/lol/graphql
/portal-graphql
/v1/api/graphql
/v1/graphql
/v1/graphql-explorer
/v1/graphql.php
/v1/graphql/console
/v1/graphql/schema.json
/v1/graphql/schema.xml
/v1/graphql/schema.yaml
/v2/api/graphql
/v2/graphql
/v2/graphql-explorer
/v2/graphql.php
/graph
/graphql/console/
/graphiql
/graphiql.php
```
## 内省安全

https://graphql.cn/learn/introspection/

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/GraphQL

### 常见的内省攻击方法

- 模式泄露：通过发送特定的内省查询（通常是__schema查询）来获取GraphQL模式的详细信息。攻击者可以利用这些信息来了解API的结构、类型和字段，甚至可能发现隐藏或敏感的功能。

- 类型混淆：攻击者可以通过查询和分析GraphQL模式来发现可能存在的类型混淆漏洞。类型混淆是指在GraphQL模式中存在多个具有相同名称但不同定义的类型，可能导致意外的数据访问或安全问题。

- 字段枚举：通过查询GraphQL模式，攻击者可以枚举目标API中的所有字段和关联关系。这些字段和关联关系的信息可以帮助攻击者了解数据模型、关系和功能，并进行后续攻击。

- 查询分析：攻击者可以通过发送大量的查询来分析GraphQL API的性能和复杂性。这可以帮助攻击者发现潜在的性能问题、资源消耗过高的查询以及可能的漏洞。

- 敏感信息泄露：通过分析GraphQL模式和执行查询，攻击者可以尝试获取敏感信息，如用户凭据、API密钥、数据库结构等。如果API没有正确保护这些信息，可能会导致信息泄露漏洞。

### 绕过内省

特殊字符绕过
尝试使用空格、换行符和逗号等字符，因为它们会被GraphQL忽略
弱正则匹配绕过，修改请求方式绕过，修改请求类型绕过


1. 插件分析不了接口，利用内省查询

POST查询

```
{
  __schema {
    types {
      name
    }
  }
}
```

GET查询

```
query=query{__typename}
```

把查询语句写入插件，插件可以转换

获取接口结构

```
fragment FullType on __Type {
  kind
  name
  description
  fields(includeDeprecated: true) {
    name
    description
    args {
      ...InputValue
    }
    type {
      ...TypeRef
    }
    isDeprecated
    deprecationReason
  }
  inputFields {
    ...InputValue
  }
  interfaces {
    ...TypeRef
  }
  enumValues(includeDeprecated: true) {
    name
    description
    isDeprecated
    deprecationReason
  }
  possibleTypes {
    ...TypeRef
  }
}
fragment InputValue on __InputValue {
  name
  description
  type {
    ...TypeRef
  }
  defaultValue
}
fragment TypeRef on __Type {
  kind
  name
  ofType {
    kind
    name
    ofType {
      kind
      name
      ofType {
        kind
        name
        ofType {
          kind
          name
          ofType {
            kind
            name
            ofType {
              kind
              name
              ofType {
                kind
                name
              }
            }
          }
        }
      }
    }
  }
}

query IntrospectionQuery {
  __schema {
    queryType {
      name
    }
    mutationType {
      name
    }
    types {
      ...FullType
    }
    directives {
      name
      description
      locations
      args {
        ...InputValue
      }
    }
  }
}
```

保存json数据到插件上解析

