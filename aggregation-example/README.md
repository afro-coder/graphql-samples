### This is an example of Aggregation

Using the default Petstore application you can create a custom schema

```
apiVersion: graphql.gloo.solo.io/v1beta1
kind: GraphQLApi
metadata:
  name: get-orders-and-pets-gql
  namespace: apps-configuration
spec:
  options:
    logSensitiveInfo: true
  executableSchema:
    executor:
      local:
        enableIntrospection: true
        resolutions:
          getPet:
            restResolver:
              request:
                headers:
                  :method: GET
                  :path: /v3/pet/{$parent.petId}
              upstreamRef:
                name: petstore-us
                namespace: apps-configuration
          getOrderById:
            restResolver:
              request:
                headers:
                  :method: GET
                  :path: /v3/store/order/{$args.orderId}
              upstreamRef:
                name: petstore-us
                namespace: apps-configuration
    schemaDefinition: |
      type Pet {
        id: Int!
        name: String!
      }

      type Order {
        id: Int
        petId: Int!
        pet: Pet @resolve(name: "getPet")
      }

      type Query {
        getOrderById(orderId: Int!): Order @resolve(name: "getOrderById")
      }
```
The output happens as such.
```
{
  "data": {
    "getOrderById": {
      "id": 3,
      "petId": 2,
      "pet": {
        "id": 2,
        "name": "Cat 2"
      }
    }
  }
}
```
