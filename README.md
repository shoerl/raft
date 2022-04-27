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
After setting up an initial election sequence that handles timeouts, the complexity increased when it needed to be expanded to handling partitions and deaths of leaders, especially in sequence. Figuring out where to place things was also a challenge at times, for example it was difficult to figure out exactly where to place the functionality for updating the commitIndex for the leader. There were also a lot of edge cases surrounding term numbers (what to do when you recieve a request/response with a larger/smaller term number than our own). Dealing with that logic for term numbers took a lot of thinking through. Figuring out when to reset the election timer was also challenging.

Debugging was also particularly hard, as there were a lot of messages being sent back and forth and there is also a LOT of information stored for any given replica, so figuring out how to pick out just the information we needed to debug a particular scenario took a lot of effort.


## Features

Uses a mix of fields to track which requests have been committed to the leader's log, and which requests have not. 

* self.commit_index - the 
* self.last_applied - TODO?
* self.next_index = - for leader to track the next index of log to send to each replica
* self.match_index - for leader to track the sync commit index on each replica's received requests




## Testing
Using the testing suite as a baseline, sometimes we modified parameters for testing. Customizing a random seed was useful for recreating a testing instance (32 = seed)
Focusing on certain test types was also helpful to isolate the exact area of improvement needed when some bug 

Modified testing script to print out details of failing requests as logs were difficult to parse through and read 