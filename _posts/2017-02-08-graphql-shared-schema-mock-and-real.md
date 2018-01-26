---
layout: post
title: GraphQL API Driven Design
date: 2017-02-08
author: a.johnston
categories: cardano graphql mocking api
description: Using API driven design with GraphQL to create mock data for a schema that is shared with the real data for a self documenting API.
---

This article is not a [GraphQL](http://graphql.org/) tutorial but rather focuses on the specific case of mocking data with GraphQL.  It discusses some investigations I did into using GraphQL to rapidly create a mock service which helps the consumers and producers to rapidly agree up the api schema then use that exact schema to create provide the real data as well as a portal where users can try it out. If you know all about this stuff then jump to [Setting up the Mocking](#setting-up-the-mocking) below, if not then read on to see how this is an efficient way to start projects.  

<img src="/assets/images/post-images/graphiql.png" alt="GraphiQL" style="width: 60%;
    display: block;
    margin: 0 auto" />

_GraphiQL allows you to easiy create and try your queries then copy the query to use as a standard json get request or post in your application._

# Why API Driven Design and Why Graph QL
Just as in the case of interface driven design in the world of Java or C#, API driven design means before any code is written the schema of the API has been agreed upon.  Providing mock data is quick and easy then allows the consumers to actually use the api and verify it is what they want.  This also means that both UI development can start before the real service is ready.

In this sample both the initial mock is the start of the final service, it is the actual project.  Because [GraphQL](http://graphql.org/) uses a schema we create the schema and provide mock data then deploy where the UI developers can access it.  Then the api developers start to fill out the real data using exactly the same schema.  No chance of getting out of sync with the mock.  If one changes, the other changes.  Whilst I really like [Apiary](https://apiary.io/) and even [Apigee](https://apigee.com/) they are separate from your project so need syncing and rely on developers to remember.  They do some checks and may even code gen but with GraphQL and mocking you are always in the real project.

I strongly recommend reading up on [GraphQL](http://graphql.org/).  If you have ever written a rest api before then you will read about GraphQL and just smile.  Think of it like SQL for rest.  It is still completely rest under the covers and rest clients can call it still but it adds a whole lot more.  It is a superset of rest.  Plus it's self documenting and you get this great api portal built in. This app is part of your api, not separate.  If you want the documentation, send a text/html content type, if you want to query the api send an application/json content type. 

If you are new to GraphQL I suggest you read up a little before continuing.

## Star Wars
Try the GraphQL stars wars api.  [Click this query](http://graphql.org/swapi-graphql/?query=%7B%0A%20%20allFilms(first%3A%201)%20%7B%0A%20%20%20%20films%20%7B%0A%20%20%20%20%20%20title%0A%20%20%20%20%20%20speciesConnection%20%7B%0A%20%20%20%20%20%20%20%20edges%20%7B%0A%20%20%20%20%20%20%20%20%20%20node%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20name%0A%20%20%20%20%20%20%20%20%20%20%20%20classification%0A%20%20%20%20%20%20%20%20%20%20%20%20averageHeight%0A%20%20%20%20%20%20%20%20%20%20%20%20homeworld%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20name%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20population%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D%0A&operationName=undefined) to open the GraphQL editor with a query ready to run.  Click the "Prettify" button to pretty format it, then hit the play button.  The query defines which fields you want from the API as well as filtering.  You can also get multiple results in one server request.
 

# Setting up the Mocking
Source code is available on [GitHub](https://github.com/chookie/graphql-mock).  

The starting point for GraphQL is a standard node/express service then you add the graph middleware as an endpoint.  So wherever you normally setup your routes use this:

{% highlight javascript linenos %}
  app.use('/graphql', GraphQLHTTP({
      schema: realSchema,
      graphiql: true
  }));
{% endhighlight %}

The core of GraphQL is the idea of a schema.  You can create a javascript file with the schema and the data loaders, known as resolvers.  You can, however, also separate the schema and the load so you can easily provide both mock data and real data on the same schema file. We use the new ES6 template string and the [GraphQL Tools](https://github.com/apollographql/graphql-tools) library from Apollo, an npm package and an opinionated structure for how to build a GraphQL schema and resolvers in JavaScript, following the GraphQL-first development workflow.  I used [their documents](http://dev.apollodata.com/tools/graphql-tools/index.html) to create this sample. 

First the string schema, complete with comments which GraphiQL will also display.
{% highlight javascript linenos %}
export default`
    # This is the description for Link type
    type Link {
        id: String! # ! means required
        # This is the description for title
        title: String
        url: String
    }

    # This is the description for Query type
    type Query {
        # This is the description for links
        links(title: String, url: String): [Link]
    }

    # This is the description for schema type
    schema {
        # This is the description for query
        query: Query
    }
`
{% endhighlight %}

Since we are following the API Driven approach, we next create a mock resolver to get the data for this schema.
{% highlight javascript linenos %}
import casual from 'casual';
export default {
  Link: () => ({
    title: () => casual.title,
    url: () => casual.url
  })
}
{% endhighlight %}

Then you combine the two back in your main express code.
{% highlight javascript linenos %}
// Make a GraphQL schema with no resolvers
const mockSchema = makeExecutableSchema({ typeDefs: schemaString });
// Add mocks, modifies schema in place
addMockFunctionsToSchema({
  schema: mockSchema,
  mocks: mockResolver,
  preserveResolvers: false
});
app.use('/mock', GraphQLHTTP({
    schema: mockSchema,
    graphiql: true
}));
{% endhighlight %}
The graphiql: true is what show he GraphiQL UI.

That is it, seriously.  A couple of anonymous functions and using the excellent [casual-browserify](https://www.npmjs.com/package/casual-browserify) which makes it so simple to return many types of sample data from phone numbers, to addresses, url's, descriptions, names etc.  

So now put your service on Heroku or AWS etc and within around an hour we have a mock of our service that we can deploy and the UI developers can start using and the clients can test.

# The Real Thing
Now the api developers get to work on providing the real data.  In my sample I connected to a Mongo DB to get the data.  This goes in a new resolver then back in the express code we can either swap the endpoint for real data, or as I have done here provide two different endpoints, /mock for mock data and /graphql for real data.  Both use the same schema so a change in one requires a change in the other or we get a compile time error and both have the GraphiQL interface with the exact same schema and comments.  No more developes forgetting to update a wiki somewhere.
{% highlight javascript linenos %}
MongoClient.connect(projectConfig.mongo, (err, database) => {
  if (err) {
    throw err;
  }
  db = database;
  const realSchema = makeExecutableSchema({
    typeDefs: schemaString,
    resolvers: resolverMap(db)
  });

  app.use('/graphql', GraphQLHTTP({
      schema: realSchema,
      graphiql: true
  }));
  ....
{% endhighlight %}

There is a lot more to GraphQL as I said but even just this is a big win.  Rapid development, centralised schema, easy mocking, easy querying and automatic documentation. And it is all done in javasrcript, in the real project.
