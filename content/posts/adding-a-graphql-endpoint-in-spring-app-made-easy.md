---
title: "Adding GraphQL to Your Spring App: Made Easy"
images: ["images/adding_a_graphql_endpoint_in_spring_app_made_easy.jpeg"]
featured_image: "images/adding_a_graphql_endpoint_in_spring_app_made_easy.jpeg"
date: 2018-08-13T07:42:43-04:00
---

I am sure that you have heard about [GraphQL](https://www.youtube.com/watch?v=Oh5oC98ztvI). It is the latest pattern that everyone is talking about. I want to walk you through adding your first GraphQL Endpoint.

Before we begin, I am assuming that you already have a Java Spring application and you have already read the [introduction to GraphQL](https://graphql.org/learn/). Just like your real application we are going to utilize code that is already written to add our new endpoint.

# Starting

In our sample app based on the tutorial here: [https://www.howtographql.com](https://www.howtographql.com). We build a Hacker News clone and already have a Link Repository. For simplicity sake LinkRepository only has one method allLinks and it stores the Links in memory.

```java
public class LinkRepository {
  List<Link> allLinks() {
    return Arrays.asList(
      new Link(1, "google.com", "A site to search"),
      new Link(2, "linkedin.com", "A site to remember birthdays")
    );
  }
}
```

To begin our journey let’s add a new dependency to [GraphQL Java](https://github.com/graphql-java/graphql-java) to your Pom file. This is a popular GraphQL implementation in Java that doesn’t make any assumptions about which frameworks are being used in our application.

```xml
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-java</artifactId>
    <version>9.0</version>
</dependency>
```

# GraphQL Schema

Now let’s define a basic GraphQL Schema. Here is the basic schema from the Hacker News clone.

```graphql
type Query{ allLinks: [Link] }

type Link {
    name: String
    description: String
}

schema {
    query: Query
}
```

# Instantiating GraphQL Java

In order to parse GraphQL queries we need to construct the library’s GraphQL object. Let’s add a Bean for it.

Notice that we also define a resolver for the top level Query type, though this framework uses DataFetchers to model resolvers. By default this library adds PropertyDataFetchers for each field on the Link class, allowing us to omit the type of Link. The implementation knows how to invoke the getter for each field.

```java
@Bean
public GraphQL graphQL(ResourceLoader resourceLoader) {
  SchemaParser schemaParser = new SchemaParser();
  TypeDefinitionRegistry typeDefinitionRegistry = schemaParser.parse(getSchema(resourceLoader));

  RuntimeWiring runtimeWiring = newRuntimeWiring()
    // wiring up our resolver
    .type("Query", builder -> builder.dataFetcher("allLinks", new QueryFetcher()))
    .build();

  GraphQLSchema graphQLSchema = new SchemaGenerator().makeExecutableSchema(typeDefinitionRegistry, runtimeWiring);

  return GraphQL.newGraphQL(graphQLSchema).build();
}

private static String getSchema(ResourceLoader resourceLoader) {
   // Loads a file via the ResourceLoader
}
```

The implementation of the QueryFetcher simply delegates to the `allLinks` method of the LinkRepository. *In a real application, it would be best to inject the repository via the constructor.*

```java
public class QueryFetcher implements DataFetcher<List<Link>> {
  private LinkRepository linkRepository = new LinkRepository();

  @Override
  public List<Link> get(DataFetchingEnvironment dataFetchingEnvironment) {
    // delegate to repository
    return linkRepository.allLinks();
  }
}
```

# Our Endpoint

The last thing we need to do is create a controller to house our endpoint. The controller takes the `query` attribute of a JSON payload and gives it to the GraphQL instance. The `execute` method validates the query against the schema and returns the appropriate errors. *For the purpose of this post, I have omitted proper error handling.*

```java
// LinkEndpoint.java
@RestController
public class LinkEndpoint  {
    @Autowired
    GraphQL graphQL;

    @PostMapping("/")
    public ResponseEntity<GraphQLResponse> home(@RequestBody() GraphQLQuery body) {
        ExecutionResult executionResult = graphQL.execute(body.getQuery());
        List<GraphQLError> errors = executionResult.getErrors();
        if (!errors.isEmpty())
            return new ResponseEntity<>(HttpStatus.BAD_REQUEST);

        return ResponseEntity.ok(new GraphQLResponse(executionResult.getData()));
    }
}
// GraphQLQuery.java
@Data
public class GraphQLQuery {
  String query;
}
```

Lastly, notice the GraphQLResponse contains a data attribute of type Object, this is because the ExecutionResult returns a variable type for data. So, I have just allowed us to take whatever the ExecutionResult returns.

```java
// GraphQLResponse.java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class GraphQLResponse {
  Object data; // Allows for any datatype to be returned
}
```

# Seeing It Work

Now for the moment of truth, when you run your Spring application. We can query our endpoint.

```bash
curl http://localhost:3000/ -X POST -H "Content-Type: text/plain" -d "{allLinks {name, description}}"
```

That CURL returns….
```json
{
    "data": {
        "allLinks": [
            {
                "name": "google.com",
                "description": "A site to search"
            },
            {
                "name": "linkedin.com",
                "description": "A site to remember birthdays"
            }
        ]
    }
}
```

🎉 There you have it. 🎉 Thanks for playing along at home. Until next time.
Links

Complete Code here: https://github.com/steven-solomon/graphql

{{< thank-you >}}

*Photo by Dan Meyers on Unsplash*
