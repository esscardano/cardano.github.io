---
layout: post
title: Bundle your micro-services with graphQL
date: 2018-09-07
author: r.lopes
categories: graphql rest microservices javascript
description: Aggregating your REST micro-services with a graphQL API
---
At Cardano, we have moved towards a micro-service architecture for the data flowing in our organization. REST Micro-services are in charge of fetching specific data from our myriad of systems for trades, financial instruments or even static reference data. Micro-services like these use other individual micro-services when calculations or logic are required for valuation, transaction costs, etc. This gives us a huge flexibility when building up new applications. We just extend, add or hook up existing micro-services.

With this architecture in place, a crucial challenge emerged: how do we aggregate and distribute data from these micro-services to our consumers? 
We identified a key set of requirements which are important for us:
  * how do we make the aggregation be a standard, i.e. a single view of the world whithin our organization?
  * how can that standard still be flexible enough to be consumed by different use cases, e.g. data lake ingestion, Python scripts done by analysts, PowerBI reports, different operational tools?
  * how do we keep it performing, i.e. avoid over fetching more than what each consumer requires?
  * how do we keep it low maintenance and stable?

The answer was graphQL - it's pretty cool! The first two points above were covered: this api design pattern gives us a schema, i.e. a standard interface for an API where consumers query and receive only what they need. Our journey was about figuring out if we could  answer the last two questions.
If you are new to GraphQL, please read more about it [here](https://graphql.org/learn/) before continuing, especially on schemas.

# Don't over fetch

To distribute a certain entity whithin our organization we want a flexible API that can handle multiple use cases. A GraphQL API allows you to query an entity by asking which fields you want from it and only get those fields back in a DTO. Imagine the following two different queries on the same event entity (GraphQL uses JSON for its queries)

```json
events(date: "2088-08-08" {
  eventType
	transactionIdentifier
  tradedNotional           
  tradeTime
	instrumentIdentifier
  instrument {
    settlementDate
    fixingDate
    instrumentTypeName
  }
 }

events(date: "2088-08-08" {
  eventType
	transactionIdentifier
  tradedNotional           
  tradeTime
  orderidentifier
  order {
    tradingDirection
    targetNotional 
    targetExecutionDate
    targetFixingDate
  }          
}
}
```
On both cases, we are interested in events that happened on a date and in a set of common properties. However, in one use case the consumer asks graphql for a set of properties related to the instrument and in the other case another consumer asks for a set of properties related to the order. The GraphQL API returns only the requested data in JSON. 
Before GraphQL, alternatives to serve different consumer interests were already possible: a REST endpoint with all the properties for both uses cases (overfetching orders and instruments always, no thanks!), or multiple REST endpoints and correspondings DTOS for each specific use case (meh, the graphQL design is better, right?). But GraphQL gives you the flexibility of a query language and it can be used to support multiple cases in one endoint, eliminate versioning and a bunch of smart reasons you are already thinking of at this point.

As explained, our idea was to resolve the data that the GraphQL api returns by hooking it up to spreaded pre-existing individual REST micro-services on events, instruments and orders. If we want to aggregate such data on a GraphQL api, how can we make sure that only the micro-services the user asked for are invoked? How do we avoid over fetching in GraphQL?

Most GraphQL libraries can support this type of behavior in their resolvers and this is called resolving per property. We developed our api in JavaScript using the graphql-tools library from [Apollo](https://github.com/apollographql/graphql-tools). With it, we can specify something called a resolver map which serves the types of the properties in our GraphQL schema:

```javascript
export default {
  Query: {
    events(obj, args) {
      return getTransactionEvents(args);
    },
  },
  Event: {
    instrument(event, args) {
      return getInstrument(transaction);
    },
    order(event, args) {
      return getOrder(transaction);
    }  
}
};
}
```

Query, Event, events, instrument, order are all types we specified in our GraphQL schema - which specifies what consumers can query. The resolver map above tells our code where to redirect the fetching of data in case the consumer requests that type. In this case, the get functions will fetch the relevant data from an underlying micro-service. Looking back to our previous example, if no order is requested then the corresponding entry in the resolver map (getOrder) is never executed.

Looking at the resolver, you can think about this as a database query with a couple of joins. There is always a main table and you join other tables to it. In our world, this means there is a leading micro-service which is always requested first and then you do a couple of joins to it. In our case, getTransactionEvents(args) leads and returns an array of events partially filled with the properties that the micro-service knows. Afterwards, the resolver map kicks in and getInstrument and getOrder are executed (if they were requested) and they should fill the remaining requested properties.

There is a catch: the type sub-resolvers (the joins) are invoked per parent object, i.e. once for each event from getTransactionEvents (notice the argument 'event' on each sub-resolver). The more results we have on getTransactionEvents, the more individual requests to the micro-services in getInstrument and getOrder can happen. Which is not really what we would call "keep it performing".

# Book a one way trip

This problem is solved with [DataLoader](https://github.com/facebook/dataloader), a library that is able to batch internal requests into a single outgoing request. DataLoader keeps track of individual load requests in your code, promising to solve each one. On the "last one", it batches them into a single outgoing request. After returning, DataLoader can re-distribute the results to the individual promised load requests. It sounds complicated but it's very simple and magic. There are a couple of moving pieces which are important for our story.

```javascript
app.use("/graphql",
  (req, res) => {
    const loaders = {
      instrumentsDataLoader: new DataLoader(fetchInstruments),
      ordersDataLoader: new DataLoader(fetchOrders)      
	  };
    GraphQLHTTP({
      schema: realSchema,
      context: { loaders }
    })(req, res);
  }
);
```

First off, we can tell our DataLoaders how to define the "last one" in a batch. In our case, that means that for every leading call to getTransactionEvents, we should batch all getInstrument and all getOrder. In other words, we should batch getInstrument and getOrder calls per single external request to the GraphQL API. That is what is happening above by constructing new DataLoaders on each http request to our GraphQL app.

Furthermore, the resolver functions getInstrument and getOrder need access to the DataLoader objects so they can use its batching promising abilities (we will see how later). To do this, we can use the GraphQL context object, a container object which stores relevant data per each GraphQL request. The context object is available automatically in the resolver map and can be passed down.

```javascript
export default {
  Query: {
    events(obj, args, context) {
      return getTransactionEvents(args);
    },
  },
  Event: {
    instrument(transaction, args, context) {
      return getInstrument(transaction, context.instrumentsDataLoader);
    },
    order(transaction, args) {
      return getOrder(transaction, context.ordersDataLoader);
    }  
}
};
```

This means each call to the resolver function can now "keep track of individual load requests, promising to solve each one."

```javascript
function getInstrument(event, dataLoader) {
  return new Promise(async (resolve, reject) => {
      const instrument = await dataLoader.load(event.instrumentIdentifier);
      const schemaInstrument = convert(instrument);
      resolve(schemaInstrument);
  });
}
```

The load function of the DataLoader must take an identifier which will: i) be added to an array of things to batch, ii) be eventually used to fetch the actual data (more on that later). This load function will return a Javascript Promise, so your code will end up being blocked somewhere waiting for the all the promises to be fulfilled by the single batched request.

There is one piece left - where are we fulfilling the promises and fetching the data? If you scroll up, you will see it in the DataLoaders constructors. These are data loader batching functions, the callbacks in charge of making a single batched request for all the promised identifiers. We have to make this function and adhere to certain rules.

```javascript
function ordersDataLoader(instrumentIds) {
  const instrumentsUrl = encodeURI(`${config.instrumentUrl}`);
  const data = await get(instrumentsUrl, instrumentIds);
  return data;
}
```

The rules are simple: 1) the batching function must receive an array of identifiers, 2) return a Promise which returns an array of objects, 3) all the requested objects should be returned and 4) in the same order as the requested identifiers. Our get function above does all that: builds a Promise that invokes our micro-service to fetch an array of instruments per an array of ids. And yes, this all hinders on the fact that such a micro-service exists.

And that's it. It really is magic. DataLoader works by hooking up these things together. We got this far pretty easily - we can aggregate micro-services on a graphql api layer without any type of over fetching by combining resolvers per type and DataLoader.

# Clean up the schema

At the end of this journey, we were kind of happy. We are not over-fetching and the GraphQL aggregator api is pretty low maintenance. However, there is a side-effect to all of this which can be itchy for some.

Let's explain it: I need to fetch an instrument per parent event. To fetch an instrument I need an identifier to be able to load it from a micro-service (batched by DataLoader). If in my instrument resolver I only have the parent to look into, the parent needs to include that identifier. This just forced me to add an instrument identifier to my public facing schema, available to all consumers. And the same thing for orders. Look into our code above: we are forced to do this to be able to use the resolver map accordingly.

The itch we got at this point was that our implementation choices were leaking into our public interface - the schema. Most of the times, it will not be a big deal if your underlying micro-service is already public facing anyway. But what if they are not and/or you do not intend to expose them to consumers? Our answer was the context.

```javascript
app.use("/graphql",
  (req, res) => {
    const loaders = {
      instrumentsDataLoader: new DataLoader(fetchInstruments),
      ordersDataLoader: new DataLoader(fetchOrders)      
    };
    const eventsPrivateDictionary = {};
    GraphQLHTTP({
      schema: realSchema,
      context: { loaders, eventsDictionary }
    })(req, res);
  }
);
```

Per each GraphQL request, we initialize an eventsPrivateDictionary and store it in the context. That gives us a possibility of having a container of private properties accessible all the way to the resolver map.

```javascript
export default {
  Query: {
    events(obj, args, context) {
      return getTransactionEvents(args, context);
    },
  },
  Event: {
    instrument(transaction, args, context) {
      return getInstrument(transaction, context.instrumentsDataLoader, context.eventsPrivateDictionary);
    },
    order(transaction, args) {
      return getOrder(transaction, context.ordersDataLoader, context.eventsPrivateDictionary);
    }  
  }
};
```

So what's the idea? Well, we can pass the empty context to the root resolver function getTransactionEvents. This function will do two things: construct the events as we want to show them in the schema, and write the private properties we need for fetching instruments and orders in the context.

```javascript
function getTransactionEvents(args, context) {
  return new Promise(async (resolve, reject) => {
    const transactionEvents = await fetchTransactionEventsMicroService(args); 
    const eventsDictionary = _.groupBy(transactionEvents, x => x.transactionIdentifier);
    context.eventsPrivateDictionary = eventsDictionary;
    const schemaTransactionEvents = convert(transactionEvents);
    resolve(schemaTransactionEvents);
  });
}

function getInstrument(event, dataLoader, eventsPrivateDictionary) {
  return new Promise(async (resolve, reject) => {
    const eventsMicroService = eventsPrivateDictionary[event.transactionIdentifier][0];
    const instrument = await dataLoader.load(eventsMicroService.instrumentIdentifier);
    const schemaInstrument = convert(instrument);
    resolve(schemaInstrument);
  });
}
```

Ok, buckle up. In getTransactionEvents, we start by fetching the event data from the underlying micro-service. We then save that raw event data in the context and use the same raw data to create schema objects, removing the properties we want to keep private (happens in convert function). There is a key aspect to keep in mind: the context exists per GraphQL request and not per event and the sub-resolvers (like getInstrument) are per event. So, the context has to include all of the events of a request. And if the context is to be used later per event, that means it needs to be "searchable" later. We use [lodash](https://lodash.com/)'s groupBy to create a dictionary of raw event data, indexed by transactionIdentifier. This property is used as the index because it will be the key used to search the context later. It can be any property or set of properties, as long as they are part of the schema too. Why do they need to be in the schema? Because later, on a sub-resolver like getInstrument, what we have available to use in the search is the schema parent node, i.e. the event. This is not a big problem because the event (schema) will typically be a subset of the properties of the raw event and therefore able to match on some properties.

This implementation is one way of doing and there are of course many other ways. But the key design is: 1) store the raw data from the underlying micro-service in the context, 2) convert from the raw data to the schema format, 3) in the subsequent resolvers per type, use the parent schema object to search the context for smatching raw data, 4) you now have some private properties you can use.

Time to get critical and ask "but why?". The fact you can do this does not mean you should do it. We recommend this specific solution because it helps in a very specific problem: aggregating multiple data sources (in our case micro-services) in a graphQL layer, while keeping the structure the underlying data sources private. Don't overengineer if your micro-services are already public anyway.

And there you go! In the end, we are happy with this design. Our micro-services architecture is important to us since it helps us solve a maintenance and decoupling issue. Implementing, fixing and releasing smaller units of code in different technologies is a good fit for our unstructured financial domain and our teams. When we needed to join them into larger use cases for our consumers, we were able to create this standard, flexible and performing graphQL layer.
