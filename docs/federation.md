# Federation

Support for [GraphQL Federation](https://www.apollographql.com/docs/federation) is enabled by default. If you add one of the federation annotations, the corresponding directives will be declared to your schema and the additional Federation queries will be added automatically. You can also disable Federation completely by setting the `smallrye.graphql.federation.enabled` config key to `false`.

You can add the Federation directives by using the equivalent Java annotation, e.g. to extend a `Product` entity with a `price` field, you can write a class:

```java
package org.example.price;

import org.eclipse.microprofile.graphql.Id;

import io.smallrye.graphql.api.federation.Extends;
import io.smallrye.graphql.api.federation.Key;

@Extends @Key(fields = "id")
public class Product {
    @Id
    private String id;

    @Description("The price in cent")
    private Integer price;

    // getters and setters omitted
}
```

And a normal query method that takes the `key` fields as a parameter and returns the requested type:

```java
package org.example.price;

import org.eclipse.microprofile.graphql.GraphQLApi;
import org.eclipse.microprofile.graphql.Id;
import org.eclipse.microprofile.graphql.Query;

@GraphQLApi
public class Prices {
    @Query
    public Product product(@Id String id) {
        return ...
    }
}
```

The GraphQL Schema then contains:

```graphql
type Product @extends @key(fields : "id") {
    id: ID
    price: Int
}

union _Entity = Product

type Query {
    _entities(representations: [_Any!]!): [_Entity]!
    _service: _Service!
    product(id: ID): Product
}
```

If you can resolve, e.g., the product with different types of ids, you can add multiple `@Key` annotations.
## Federation Batch Resolver

For better performance, there is the option to use batch resolvers with federation. This is not activated on default. It can be activated by setting `smallrye.graphql.federation.batchResolving.enabled` config key to `true`.

It is not needed to provide batch resolvers for all entities, if there is no batch resolver, a non-batch resolver is used.

```java
package org.example.price;

import org.eclipse.microprofile.graphql.GraphQLApi;
import org.eclipse.microprofile.graphql.Id;
import org.eclipse.microprofile.graphql.Query;

@GraphQLApi
public class Prices {
    @Query
    public List<Product> product(@Id List<String> id) {
        return ...
    }
}
```

It is crucial that the sequence of argument list matches with the order of result list. Currently, the name of the Argument `id` must match with the property name in the type.

