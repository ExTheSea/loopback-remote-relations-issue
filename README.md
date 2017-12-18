# Loopback remote connector relation sandbox
This is a sandbox project to showcase an issue with the loopback remote connector when using added relation.

## How to use
 1. Either run ``npm start`` in the root of the project or start all 3 loopback project individually with ``node .``
 2. Navigate to ``http://localhost:3000/explorer/#!/testmodel1/testmodel1_findById``
 3. Send a request with id = 1 

## Usecase
remote-source1 and remote-source2 are two Loopback APIs each containing their own model (testmodel1 and testmodel2) and in-memory datasource. Imagine these as part of a big microservice structure where creating a single CRUD API for all tables in a database would make the project way too big. As such the database was separated into multiple CRUD APIs by bundling them logically. 

remoting-parent is a sort of gateway/facade API that tries to facilitate the needed models used in a project and also add required logic. In this case it is adding a hasMany relation from the testmodel1 (from remote-source1) to the testmodel2 (from remote-source2). 

As a real life example: Imagine you created a CRUD API for all product related tables and a CRUD API for all customer related tables as they are way easier to manage this way even though the underlying tables are contained in the same database.
Now you create a customer-product API for a app project that is supposed to provide CRUD operations on both the customer and product tables. Furthermore it should als display the relation that a customer can buy multiple products. So the customer-product API contains both customer and product models but additionally contains the relation between them.

## Problem
To create this setup the remoting-parent contains the definition for testmodel1 and testmodel2 and connects those to the original source APIs using the loopback remote connector. This way you can interact with the models in the remoting-parent as if they were directly attached to a database. The added relation is then easily defined in the testmodel1. 

When you then try to access the relation using either 
``/api/testmodel1s/1/testmodel2s`` or ``/api/testmodel1s`` with the filter ``{"include": "testmodel2s"}`` though the remote connector just forwards the request to the source API which then throws an error as it doesn't know the relation. 

Weirdly enough when trying the rest connector instead (testable by changing the datasources in model-config to from "sourceX" to "rest-sourceX") calling ``/api/testmodel1s/1/testmodel2s`` at least tries to handle it correctly and calls the testmodel2 API with the correct filter. Unfortunately this filter is then not correctly passed to the API and as such is ignored causing all testmodel2 instances to be returned.
