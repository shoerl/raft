# raft
CS3700 Project 6: Distributed Key-Value Database

Sean Hoerl and Jessica van de Ven

## High-Level Approach

Our high level approach is based on the specifications outlined in the RAFT paper.

When an election starts, the server votes for itself and requests votes, and tracks the start time of the election for aging beyond the randomly-generated timeout threshold. We use randomized timeout intervals for the election timeout. 

When a leader goes down and dies or the election has aged past the timeout, an election is started. The winner is the server who recieved the majority of the votes, thus enforcing that there is only one leader at any given time, and that the new leader has a log that is as up to date or more up to date than the majority of replicas.

The log contains all of the put requests, which are each coupled with the term that they were recieved in. Logs only flow from leader to other servers. Replicas to be able to continue to function even if other servers are down, survives leader crashes. 

A request is only responded to by the destination in order to prevent duplicate communications from the servers to the requesting client. If the leader is down, the request will be redirected until a leader is back up and able to accept the request.

Our Replica class has a looping run function that will read incoming requests from the sockets. Before it responds to that request, it will apply any log entries which are committed and still need to be applied. If we are the leader, it also will send heartbeat messages if needed (if heartbeat timeout has elapsed). If we are not the leader, we will check if the election timeout has elapsed, and start the election if so. Then we will handle any requests which we have recieved from our sockets, and respond to them if needed. The request will be handled by its respective handler function. Each request handler function has specific logic according to each request (ex. get request will return the value for the key requested).

When a follower recieves a put request, it redirects the client who made that request to the leader. When a leader recieves a put request, it appends the request to its log, and then sends an AppendEntries RPC to the the rest of the replicas. Once it hears that a majority of the replicas have replicated the log entry, it commits it and then applies it, and then responds to the client who made the put request.

When a follower recieves a get request, it redirects the client who made that request to the leader. When a leader recieves a get request, it appends the request to its buffer, and then sends a heartbeat to confirm that it is still the leader. Once we hears back from a majority of the replicas, we respond to the get message with the value in our state machine (ok message) if we are still the leader. If we found out that we are no longer the leader, we will redirect the client who sent the get request to the new leader.


## Challenges
After setting up an initial election sequence that handles timeouts, the complexity increased when it needed to be expanded to handling partitions and deaths of leaders, especially in sequence. Figuring out where to place things was also a challenge at times, for example it was difficult to figure out exactly where to place the functionality for updating the commitIndex for the leader. There were also a lot of edge cases surrounding term numbers (what to do when you recieve a request/response with a larger/smaller term number than our own). Dealing with that logic for term numbers took a lot of thinking through. Figuring out when to reset the election timer was also challenging. Figuring out the logic for appending entries was also difficult (when we have to delete every log entry after and including the one with conflicting term). Figuring out the timing of things was also a bit weird, but having descriptive log messages helped us work through that.

Working through the logic of sending RPC's and then handling the responses was also hard because you couldn't just send RPC's to all of the other replicas and then deal with those exact responses in the same function, you would have to wait for those to come in on the socket and then delegate them to the corresponding function to handle them and then try to match it up with what you were trying to achieve. This was also due to the nature of having many things going on simultaneously at once.

Debugging was also particularly hard, as there were a lot of messages being sent back and forth and there is also a LOT of information stored for any given replica, so figuring out how to pick out just the information we needed to debug a particular scenario took a lot of effort. Debugging partitions was also particularly weird and took a lot of thinking to work through.

## Features

Uses a mix of fields to track which requests have been committed to the leader's log, and which requests have not. Most of the fields came from the RAFT paper, though some we came up with ourselves. There are a lot of fields within the class, which in some sense isn't ideal, but it allowed us to accurately track the state of a replica and everything that corresponded with it. I believe we also did a pretty good job breaking up functionality into seperate functions, allowing our code to be more modular and easy to read. We also did a good job using descriptive function names which makes it easier to figure out what is going on at any point or within any function. We also did a good job reusing functions wherever possible in an effort to remove code duplication. Breaking everything up into functions also allowed our main run method to be pretty concise and clean. Using a match/next index for each follower was also a good choice as it allowed us to inherently resend any log entries which did not get recieved by the replica (particularly useful for any test where messages got dropped a lot). We also stored the time of the last heartbeat sent to a given replica, which was a good choice as it allowed us to only send heartbeats if needed. This is because it then allowed us to treat an AppendEntries message sent to a specific rid as a heartbeat regardless of if it contained log entries or not. If we generalized the last heartbeat sent time, we would likely be sending heartbeats to some replicas more often then we actually needed to. We also redirect log messages that are about to be deleted from the old leader, as if they are about to be deleted then that means that the new leader does not have them. We thought this was a particularly clever enhancement.

## Testing
Developing this project iteratively using the steps outlined in the project description made it easy to test at first, as we could usually attribute any issues to the specific thing we just implemented, and then work through the logic of that what we just implemented. Having the RAFT paper to guide us was also particularly useful. The graphics and images within the RAFT paper were probably the most useful for us. For testing general logic (to ensure a function was actually doing what we expected it to do), it was helpful to disable printing of most log messages and start printing in just that function, with just all of the necessary things to figure out what was going on in that function specifically. We were able to pull out some functions and test them individually, but not for many as most functions within our project relied on a lot of state.

When the tests got more difficult and introduced many things at once (the advanced test), debugging also became more difficult. We were able to work through this by doing many things. Such as:

* Having a lot of descriptive log messages to see exactly what was going on at any point within a run
* Using the testing suite as a baseline, occasionally modifying parameters for testing.
* Customizing a random seed was useful for recreating a testing instance (32 = seed).
* Focusing on certain test types was also helpful to isolate the exact area of improvement needed when dealing with a bug/etc.
* Modified testing script to print out details of failed/missed requests so we could parse through the rest of the log and figure out exactly where things went wrong
