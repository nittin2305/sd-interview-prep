# REST vs GraphQL

> **References:** [GraphQL Spec](https://graphql.github.io/graphql-spec/) | [REST vs GraphQL](https://www.howtographql.com/basics/1-graphql-is-the-better-rest/) | [Netflix GraphQL](https://netflixtechblog.com/our-learnings-from-adopting-graphql-f099de39ae5f)

---

## REST Overview

Representational State Transfer — resource-based, HTTP verbs, stateless.

```
GET    /users/{id}           → get user
POST   /users                → create user
PUT    /users/{id}           → replace user
PATCH  /users/{id}           → partial update
DELETE /users/{id}           → delete user
GET    /users/{id}/orders    → get user's orders
```

---

## GraphQL Overview

Query language for APIs — client specifies exactly what data it needs.

```graphql
query GetUserWithOrders {
  user(id: "123") {
    name
    email
    orders(last: 5) {
      orderId
      status
      items {
        productName
        price
      }
    }
  }
}
```

Single endpoint: `POST /graphql`

---

## Comparison Table

| Dimension | REST | GraphQL |
|-----------|------|---------|
| Endpoints | Multiple (1 per resource) | Single /graphql |
| Data fetching | Fixed response shape | Client-defined |
| Over-fetching | Yes (returns all fields) | No (request only what's needed) |
| Under-fetching | Yes (N+1 API calls) | No (single query) |
| Versioning | URL versions (v1, v2) | Schema evolution with deprecation |
| Caching | HTTP cache (GET) | Manual (POST requests not cached) |
| Type system | No (OpenAPI optional) | Built-in (strongly typed) |
| Learning curve | Low | Medium |
| Tooling | Mature | Growing |
| Real-time | Need WebSocket/SSE | Subscriptions built-in |

---

## Java: REST Controller

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserRestController {

    private final UserService userService;
    private final OrderService orderService;

    @GetMapping("/{userId}")
    public ResponseEntity<UserDto> getUser(@PathVariable String userId) {
        return ResponseEntity.ok(userService.getUser(userId));
    }

    @GetMapping("/{userId}/orders")
    public ResponseEntity<List<OrderDto>> getUserOrders(@PathVariable String userId) {
        return ResponseEntity.ok(orderService.getByUserId(userId));
    }
}
```

## Java: GraphQL with Spring for GraphQL

```java
// Schema: src/main/resources/graphql/schema.graphqls
// type Query {
//   user(id: ID!): User
// }
// type User {
//   id: ID!
//   name: String!
//   email: String!
//   orders: [Order]
// }

@Controller
public class UserGraphQLController {

    private final UserService userService;
    private final OrderService orderService;

    @QueryMapping
    public User user(@Argument String id) {
        return userService.getUser(id);
    }

    // DataLoader to solve N+1 problem
    @SchemaMapping(typeName = "User", field = "orders")
    public CompletableFuture<List<Order>> orders(User user, DataLoader<String, List<Order>> loader) {
        return loader.load(user.getId()); // Batched loading
    }
}

// DataLoader batches N individual user IDs into one DB call
@Bean
public DataLoaderRegistry dataLoaderRegistry(OrderService orderService) {
    DataLoader<String, List<Order>> ordersLoader = DataLoader.newMappedDataLoader(
        userIds -> {
            Map<String, List<Order>> ordersByUser = orderService.getByUserIds(userIds);
            return CompletableFuture.completedFuture(ordersByUser);
        }
    );
    DataLoaderRegistry registry = new DataLoaderRegistry();
    registry.register("ordersLoader", ordersLoader);
    return registry;
}
```

---

## The N+1 Problem in GraphQL

```graphql
# This query fetches 10 users, then for each user fetches their orders
# Without DataLoader: 1 + 10 = 11 DB queries
query {
  users(limit: 10) {
    name
    orders { # N+1 problem!
      orderId
    }
  }
}
```

Solution: DataLoader batches all `orders` requests from the same execution into a single `IN (userId1, userId2, ...)` query.

---

## AWS Mapping

| Feature | AWS Service |
|---------|------------|
| REST API hosting | API Gateway (REST) |
| GraphQL hosting | AWS AppSync |
| Auto-generated CRUD | AppSync + DynamoDB resolvers |
| Real-time (subscriptions) | AppSync (WebSocket managed) |
| CDN for REST responses | CloudFront |
| Schema registry | AWS Glue Schema Registry (for event schemas) |

---

## When to Use Each

| Use REST when | Use GraphQL when |
|--------------|-----------------|
| Simple CRUD APIs | Mobile apps needing custom data shapes |
| Public APIs (easy to consume) | Complex queries joining many resources |
| Caching is critical (HTTP GET) | Rapid frontend iteration |
| File upload/download | Multiple clients with different data needs |
| Microservice to microservice | BFF (Backend for Frontend) layer |

---

## Interview Q&A

**Q1: How does GraphQL solve over-fetching and under-fetching?**
> Over-fetching: REST /users/{id} returns all user fields even if you only need the name. GraphQL lets you specify exactly `{ user(id: "1") { name } }`. Under-fetching: REST requires separate calls for user + orders + address. GraphQL fetches all in one query. This reduces bandwidth (critical for mobile) and reduces round trips.

**Q2: What is the N+1 problem in GraphQL?**
> For a list of N users, naively resolving the `orders` field means N separate DB calls (one per user). DataLoader solves this by collecting all user IDs requested in a single execution tick and batching them into one `SELECT * FROM orders WHERE userId IN (...)` query. This is transparent to the resolver code.

**Q3: Why would you use REST instead of GraphQL for a public API?**
> (1) HTTP caching: GET endpoints can be cached by CDN, browsers, proxies. GraphQL uses POST — harder to cache. (2) Familiarity: REST is universally understood; GraphQL requires learning the spec. (3) Simplicity: for simple CRUD, REST is less complex. (4) Partial responses: REST can return 206 Partial Content. (5) File uploads are simpler with REST multipart.
