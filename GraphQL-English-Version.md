# GraphQL - Flexible query with Schema defines first


## Introduction

GraphQL is a powerful query language for APIs, enabling clients to request exactly the data they need. One popular development strategy is the **schema-first approach**, where the GraphQL schema is defined before any code is written. This promotes clarity, consistency, and collaboration across teams.

In this guide, we'll explore how to create flexible queries using the schema-first method in GraphQL, along with best practices and tools that can enhance your workflow.

---

## What Is GraphQL?

GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data. GraphQL provides a complete and understandable description of the data in your API, gives clients the power to ask for exactly what they need and nothing more, makes it easier to evolve APIs over time, and enables powerful developer tools.
http://graphql.org

In other words, GraphQL is a declarative data fetching specification and query language. It is meant to provide a common interface between the client and the server for data fetching and manipulations.

To further understand what GraphQL is, let’s take a look at some of its features:

- **Declarative**: With GraphQL, what you queried is what you get, nothing more and nothing less..
- **Hierarchical**: GraphQL naturally follows relationships between objects. With a single request, we can get an object and its related objects. For instance, with a single request, we can get an `Author` along with the `Posts` he has created, and in turn get the `Comments` on each of the posts..
- **Strongly-typed**:With GraphQL type system, we can describe possible data that can be queried on the server and rest assured that the response we’ll get from a server is in line with what was specified in the query.
- **Not Language specific**: GraphQL is not tied to a specific programming language as there are several implementations of the GraphQL specification in different languages.
- **Compatible with any backend**: GraphQL is not limited by a specific data storage; you can use your existing data and code or even connect to third-party APIs.
- **Introspective**: With this, a GraphQL server can be queried for details about the schema.

In addition to these features, GraphQL is easy to use as it has a JSON like syntax and also provides lots of performance benefits.

Let’s dive deeper. We’ll see the syntax and the operations that can be performed with GraphQL. GraphQL operations can be either read or write operations.

---

## Query

Queries are used to perform read operations, that is, fetching data from the server. Queries define the actions we can perform on the schema (more on schema shortly). Below is a simple GraphQL query and its corresponding response:

```graphql
// Basic GraphQL Query
{
  author {
    name
    posts {
      title
    }
  }
}
```

```json
// Basic GraphQL Query Response
{
  "data": {
    "author": {
      "name": "Chimezie Enyinnaya",
      "posts": [
        {
          "title": "How to build a collaborative note app using Laravel"
        },
        {
          "title": "Event-Driven Laravel Applications"
        }
      ]
    }
  }
}
```

This is a simple query that gets the name of an author and the posts created by the author. Notice that the query and the response have the same structure. Also, the response contains an array of posts created by the author and we were able to achieve this with a single request.

Let’s go over and explain the components of a GraphQL query.

In the query above, we omitted the `query` keyword. If an operation doesn’t include a type, by default GraphQl will treat such operation as a query. A query can have a name. Though the name is optional, it makes it easy to identify what a particular query does. In production applications, it is recommended to use the `query` keyword and name your queries so as to make your code easy to understand.

```graphql
query GetAuthor {
  author {
    # The name of the author
    name
  }
}
```

We have given our query a name `GetAuthor` which is quite descriptive. Queries can also contain comments. Each line of comment must start with the `#` sign.

### Fields

Fields are basically parts of an object we want to retrieve from a server. In the query above, `name` is a field on the `author` object.

### Arguments

Just like functions in programming languages can accept arguments, a query can accept arguments. Arguments can be either optional or required. So we can rewrite our query to accept the ID of the author as an argument:

```graphql
{
  author(id: 5) {
    name
  }
}
```

**Note:** GraphQL requires strings to be wrapped in double quotes.

### Variables

In addition to arguments, a query can also have variables which make a query more dynamic instead of hard coding the arguments. Variables are prefixed with the `$` sign followed by their type.

```graphql
query GetAuthor($authorID: Int!) {
  author(id: $authorID) {
    name
  }
}
```

Variables can also have default values:

```graphql
query GetAuthor($authorID: Int! = 5) {
  author(id: $authorID) {
    name
  }
}
```

Like arguments, variables can be either optional or required. In our example, `$authorID` is required because of the `!` symbol (more on this under schema) included in the variable definition.

### Aliases

With the above query, let’s assume we want to get authors with IDs 5 and 7 respectively. We might be tempted to do something like this:

```graphql
{
  author(id: 5) {
    name
  }
  author(id: 7) {
    name
  }
}
```

This will throw an error as there would be conflict between the two `name` fields. To resolve this, we’ll use aliases. With aliases, we can give the fields customised names and request data from the same field with different arguments.

```graphql
{
  chimezie: author(id: 5) {
    name
  }
  neo: author(id: 7) {
    name
  }
}
```

And the response will be something similar to:

```json
# Response
{
  "data": {
    "chimezie": {
      "name": "Chimezie Enyinnaya"
    },
    "neo": {
      "name": "Neo Ighodaro"
    }
  }
}
```

### Fragments

Fragments are reusable set of fields that can be included in queries as needed. Assuming we need to fetch a `twitterHandle` field on the `author` object, we can easily do that with:

```graphql
{
  chimezie: author(id: 5) {
    name
    twitterHandle
  }
  neo: author(id: 7) {
    name
    twitterHandle
  }
}
```

But what about if we want to pull more fields? This can quickly become repetitive and redundant. That’s where fragments comes into play. Below is how we would solve the above situation using fragments:

```graphql
{
  chimezie: author(id: 5) {
    ...authorDetails
  }
  neo: author(id: 7) {
    ...authorDetails
  }
}

fragment authorDetails on Author {
  name
  twitterHandle
}
```

Now we will only need to add our fields in one place.

### Directives

Directives provide a way to dynamically change the structure and shape of our queries using variables. As of this post, the GraphQL specification includes exactly two directives:

<ul>
  <li>`@include` will include a field or fragment only when the `if` argument is `true`</li>
  <li>`@skip` will skip a field or fragment when the `if` argument is `true`</li>
</ul>

The two directives both accept a Boolean (true or false) as arguments.

```graphql
query GetAuthor($authorID: Int!, $notOnTwitter: Boolean!, $hasPosts: Post) {
  author(id: $authorID) {
    name
    twitterHandle @skip(if: $notOnTwitter)
    posts @include(if: $hasPosts) {
      title
    }
  }
}
```

The server will not return a response with the author’s Twitter handle if `$notOnTwitter` is `true`. Also, the server will return a response with the author’s post only if the author has written some posts, that is, `$hasPosts` is `true`.

---

## Mutation

In a typical API usage, there are scenarios where we would want to modify the data on the server. That’s where mutations come into play. Mutations are used to perform write operations. By using mutations, we can make a request to a server to amend or update specific data, and we would get a response that contains the updates made. It has a similar syntax to the query operation with a slight difference.

```graphql
mutation UpdateAuthorDetails($authorID: Int!, $twitterHandle: String!) {
  updateAuthor(id: $authorID, twitterHandle: $twitterHandle) {
    twitterHandle
  }
}
```

We send data as a payload in a mutation. For our example mutation, we could send the following data as payload:

```json
// Update data
{
 "authorID": 5,
 "twitterHandle": "ammezie"
}
```

And we’ll get the following response after the update has been made on the server:
```json
// Response after update
{
  "data": {
    "id": 5,
    "twitterHandle": "ammezie"
  }
}
```

Notice the response contains the newly updated data.

Just like queries, mutations can also accepts multiple fields. An important distinction between mutations and queries is that mutations are executed serially in order to ensure data integrity, whereas queries are executed in parallel.

---

## Schemas

Schemas describe how data are shaped and what data on the server can be queried. Schemas provide object types used in your data. GraphQL schemas are strongly typed, hence all the object defined in a schema must have types. Types allow the GraphQL server to determine whether a query is valid or not at runtime. Schemas can be of two types: Query and Mutation.

Schemas are constructed using what is called GraphQL schema language, which is quite similar to the query language we saw in the previous sections. Below is a sample GraphQL schema:

```graphql
type Author {
  name: String!
  posts: [Post]
}
```

The above schema defines an `Author` object type with two fields (name and posts). This means that we can only use `name` and `posts` fields on any GraphQL query operation on `Author`. The fields on an object types can be optional or required. The `name` is required because of the `!` symbol after it type name.

### Arguments

Fields in a schema can accept arguments. These arguments can either be optional or required. Required arguments are denoted with the `!` symbol:

```graphql
type Post {
  allowComments(comments: Boolean!)
}
```

### Scalar Types

Out of the box, GraphQL comes with the following scalar types:

<ul>
  <li>**Int**: a signed 32‐bit integer</li>
  <li>**Float**: a signed double-precision floating-point value</li>
  <li>**String**: A UTF‐8 character sequence</li>
  <li>**Boolean**: **true** or **false**</li>
  <li>**ID**: represents a unique identifier</li>
</ul>

Fields defined as one of the scalar types cannot have fields of their own. We could also specify custom `scalar` types using the `scalar` keyword. For example, we could define a `Date` type:

```graphql
scalar Date
```

### Enumeration types

Also called `Enums`, are a special kind of scalar that is restricted to a particular set of allowed values. With Enums, we can:

<ul>
  <li>Validate that any arguments of this type are one of the allowed values</li>
  <li>Communicate through the type system that a field will always be one of a finite set of values</li>
</ul>

Enums are defined with the enum keyword:

```graphql
enum Media {
  Image
  Video
}
```

### Input types

Input types are valuable in the case of mutations, where we might want to pass in a whole object to be created. In the GraphQL schema language, input types look exactly the same as regular object types, but with the keyword `input` instead of `type`. The input type is defined as below:

```graphql
# Input Type
input CommentInput {
  body: String!
}
```

---

## Conclusion

Using a schema-first approach in GraphQL encourages thoughtful API design and makes it easier to build flexible, efficient queries. By clearly defining your schema and resolvers, you enable frontend and backend teams to collaborate effectively and independently. Combined with modern tools and best practices, you can build scalable, maintainable GraphQL APIs.

