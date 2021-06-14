# Node_Microservice_Tutorial_2

In the previous project we divided the monolith in two services i.e. Posts and Comments

Here we implement the Asynchronous solution. We will make our own express based Event Bus. But people can use RabbitMQ, Kafka etc.

Here is a quick refresher of the solution.

## Asynhronous Communication

We introduce a event broker in the system. The purpose of a event broker is to receive requests/events/notifications and sends them on to interested parties. We will add a Query Service to our application. The goal of this Query Service is to listen to any time a post/comment is created. On creation of post/comment those service will emit an event, the Query Service will then listen for those events and take the post/comment and assemble them into an efficient data structure.

### The pros and cons are :
- Pro : Query service has no dependencies on the other services
- Pro : Query service will be extremely fast
- Con : Data Duplication. Post Sevice will still store all posts and Comments service will still store all comments. On top of that Query Service will store everything.
- Con : Harder to understand and implement

### Steps in Asynchronous Communication solution :
- Whenever someone creates a post from the Posts Service we will emit an event (PostCreated), the Posts Service will through the event in the Event Bus and then the Event Bus will route the event to the Query Service.
- Then it will be upto the Query Service to process this event and produce some data from it.
- When someone creates a comment tied to that post, the Comments service will emit an event (CommentCreated), that will go to our event bus. Event bus will take the event and route it to the Query Service.
- Now it is upto the Query Service to process this event. This time, Query Service will find the related post and add it to some kind of comments array.
- At this point, someone can ask the Query Service to send all the different posts and its related comments

### How our JS implementation of simple Event Bus works
- We are going to add a new route to our Posts, Comments Service called POST /events
- We will make the Query Service similarly as well
- Whenever a service needs to emit an event, it will make a POST request to the Event-Bus Service's POST /event api...essentially we are making a post request from an service to Event-Bus service.
- In that post event we will attach some data that represents that event
- The Event-Bus will take the event and make a series of POST requests with it to Posts, Comments or Query service

### What will the Query Service do ?
- Query service will have two routes i.e. GET /posts and POST /events
- GET /posts will return all posts and its associated comments
- POST /events will receive events from our event bus and assembles the data in those events in a data structure

### Benifits

Even if we kill the Post Service and Comments Service, still out page with all the posts will load. So, we can now say that some functionality has failed but the total app is not down.