# Rust Events
Wanted to create a modular event streamer that could be ued by mutliple services. The functionality should be very similiar to the subgraph with a couple of improvements. Being
- choosing your own RPC, this allows for syncing between mutliple different services, if the front end knows about an event then so does this service
- no need to enforce graphql, register an event, contract and function handler and handle however you want

# Components
This service is design to be as composable as possible and is broken into three parts.

## Streamer
This handles listening to particular contract events and streams them to their appropriate reactor. This is fixed and is initialised with a yaml file containing the contracts, their events, block starts, and reactors.

## Reactors
Reactors are any method of "reacting" to streamed events.
You can register multiple reactors for individual contract events. Reactors are linked to the streamer with a function handler id. When the event is picked up by the streamer it will call the reactors function. The reactors can do any computation related to the event, including reading or setting storage. 

## Handlers
Handlers are broken down into two parts. This is probably the pointy end of the stick to make it modular its quite complex. 

Handlers can be any end goal for data. This could include postgres, sql, graphql, websocket etc. The idea is that handlers should be decided by the developer to fit into whichever architecture. Keeping these seperate with the goal that community writes handlers for specific storage they want, eg. the graphql handler will take some input from the reactor and put it into the graphql database. How this data is put in will look slightly different for each database, but the input "row" from the reactor will be the same.

### Setters
Setters relates to taking the data from the reactors and putting them into a DB. These can be re-used from use case to use case since taking data and putting it into storage should be the same for each specific storage system.

### Getters
Getters relate to accessing the data. In the graphql example this will be simple since graphql queries are part of the request. For other storage systems these will have to be decided by the developers to define how and which data they are accessing. 

Initially I want to create two handler systems
- graphql storage. A proof of concept service. Initialises a DB, handles input from the reactor and stores them into the db. Handles queries to the DB
- Websocket storage. A websocket isnt necessarily persistent storage but I think the idea that it shouldnt matter where the data goes after the reactor is cool so wanted to also create a websocket which could be connected to externally. In this case instead of handling input data and putting it into a database its emitting the data in some event
