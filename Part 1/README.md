# Node_Microservice_Tutorial_1

This version of the project is just breaking our monolith project into two serivces.

For every one of the posts we have to make another followup request for the comments. We need to condense it to just one. One single request for for all posts and all the associated comments for the posts.

If we try to do this in a monolith, this will be straight forward to do.

We call the post api with a param like this : GET /posts?comments=true
And the server will return the posts with embedded comments

But in Microservices architecture we have two solutions here :

1. Synchronous communication : We make a get request to out Post service and to ensure that all the comments are embedded we write code in the GET /post api of the Posts service to reach out to the Comments service and bring the comments associated with a list of post ids. Comments service will reply with the set of comments and Posts service will assemble them together with the relevant post and send the entire bundle back to the browser.

The cons of this solution are :
- Pro : Easier to understand and implement
- Con : Introduces dependecy between services
- Con : If any inter-service request fails then the overall resuest fails
- Con : The entire request is only as fast as the slowest request
- Con : Can easily introduce webs of requests

2. Asynhronous Communication : We introduce a event broker in the system. The purpose of a event broker is to receive requests/events/notifications and sends them on to interested parties. We will add a Query Service to our application. The goal of this Query Service is to listen to any time a post/comment is created. On creation of post/comment those service will emit an event, the Query Service will then listen for those events and take the post/comment and assemble them into an efficient data structure.

The pros and cons are :
- Pro : Query service has no dependencies on the other services
- Pro : Query service will be extremely fast
- Con : Data Duplication. Post Sevice will still store all posts and Comments service will still store all comments. On top of that Query Service will store everything.
- Con : Harder to understand and implement