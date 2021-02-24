---
title: GraphQL in Swift
layout: post
image: /public/graphql.png
category: Architecture
---

I spent last weeks sharing more about type-safety and building robust APIs in Swift. I want to continue the type-safety topic by talking about GraphQL. GraphQL is a query language for your API. This week we will talk about the benefits of GraphQL, and we will learn how to utilize it in Swift.

#### Basics
Let me introduce GraphQL first. GraphQL is a query language for your API. Usually, a backend developer or web service should provide you a schema file and a single GraphQL endpoint. Schema file contains all the types and queries. Let's take a look at the example of the schema file.

=====================================================

The schema file should contain Query and Mutation types. These types define all the queries and mutations that the current GraphQL endpoint supports. The schema file also describes the list of all the types that you can use in your queries.

=====================================================

GraphQL is a strongly typed language. Every field defined inside a GraphQL custom type must declare its type. By default, every field can contain nil as a value. The fields having exclamation marks can't be nil.

I use Star Wars API to show you examples in this post. Let's continue by making some queries. You can easily play with GraphQL API using the GraphiQL app.

=====================================================

As you can see, we use the data types from the schema file to build our query. One thing that I love about GraphQL is the response format. Request format is directly mapped to the response format. You can add more fields to your request, and the response will have them also.

=====================================================

#### ApolloGraphQL
ApolloGraphQL is a great framework that allows you easily make GraphQL queries and mutations. ApolloGraphQL iOS framework takes care of caching and code generation. ApolloGraphQL generates Swift types for queries and mutations that you define in your project. It saves your time by generating all the boilerplate for you automatically.

There are a few steps that you need to do to setup ApolloGraphQL in your project.

You should embed ApolloGraphQL into your project using SPM or another package manager.
Add [run script]() to your build phases above the compile sources section. This script downloads the schema and generates Swift types for your queries. You can easily change the GraphQL endpoint in this script to connect to your GraphQL backend.

We have prepared the project to use ApolloGraphQL. Now we can add the first query to our project. We should create a file in the project with the .graphql extension and put these lines into the file.

=====================================================

Let's build the project now. ApolloGraphQL generates an API.swift file that you should add to the project. There are all the needed types to make GraphQL queries in a very type-safe way. Every request type defines its response type. ApolloGraphQL generated AllFilmsQuery and Data types that describe the request and response. Now we can use generated code to make GraphQL requests.

=====================================================

#### Conclusion
There are a lot of benefits of GraphQL over REST API. But remember, everything comes with its own set of pros and cons. GraphQL is a great way to build an efficient and type-sade backend for your app. I will try to cover more advanced features of GraphQL in the next posts. I hope you enjoy the post. Follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!