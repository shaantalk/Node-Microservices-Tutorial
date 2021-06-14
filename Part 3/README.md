# Node_Microservice_Tutorial_3

In the previous project we implement the Asynchronous solution. We made our own express based Event Bus and Query Service.

Now, in this project we will add one more service for comment moderation. We will flag comments that has bad words in it.

Let's say 'orange' is a bad word.

## Feature details :
1. If the comment has bad words in it it will be rejected
2. Comment can be in 3 stage i.e. awating moderation, moderated and rejected

## Solution 1 : Implement the filter in UI
Pro : very easy to implement
Con : Let's say now the filter word is changed from orange to banana then we have to re-deploy the whole React app

## Solution 2 : Add the feature inside comments service
Pro : very easy to implement
Con : If this filtering takes longer to flag a comment then Comments service will be slow. We asume that this feature will take a long time to flag a comment.

## Solution 3: Make a seperate Moderation serivce which communicates event creation to query service
- Comment service will create a CommentCreated event
- Event bus will emit the CommentCreated event to all services
- Only Moderation service cares about this event i.e. approves or rejects a comment
- if Moderation Service approves the comment then Moderation service will emit CommentModerated event by adding status flag to the comment it received from CommentCreated event
- Event bus will emit the CommentModerated event to all services
- Only Query service cares about this event i.e. persists the comment
- Con : There will be a delay between comment creation and when comment appears in the UI because the Moderation service will take time. Awaiting moderation info won't apprear in the UI.

## Solution 4: Make a seperate Moderation serivce which updates status at both Comments and Query service
- Comment service will create a CommentCreated event with a default status of pendng moderation
- Event bus will emit the CommentCreated event to all services
- Moderation service and Query Service cares about this event. Moderation service will start working on moderation while the query service will persist the comment
- In the UI we can show that the comment is pending moderation
- When moderation is done the Moderation service will emit CommentModerated event which will go over to the query service
- Query service will then change the status to approved
- Con : Does it make sense to let the query service update the comment. in future may be there are other things done to comments like upvotes and downvotes. it will be hard for the query service to do all this updates

## Solution 5: Make a seperate Moderation serivce whose event will be handled by Comments service
- Comment Servce will handle everything related what a comment is
- Comment Service will deal with CommentModerated ( or later upvoted, downvoted etc ) and emit a simple event called ComentUpdated
- Query service will take in CommentUpdated event and update the comment, also it will take it CommentCreated event as well