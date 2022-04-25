# raft
CS3700 Project 6: Distributed Key-Value Database

Sean Hoerl and Jessica van de Ven

## High-Level Approach

When an election starts, the server votes for itself and requests votes, and tracks the start time of the election for aging beyond the randomly-generated timeout threshold. 

When a leader goes down and dies or the election has aged past the timeout, an election is started. The winner is the first server to be voted for that doesn't have a log that is surpassed by the majority of replicas. 

The log contains all of the put requests. When a put request is made, it is redirected to the leader and updates the state machine of the replicas. The request is only responded to by the destination in order to prevent duplicate communications from the servers to the requesting client. If the leader is down, the request will fail and the client will re-send the request. 

Our Replica class has a looping run function that will read incoming requests from the sockets and apply any put requests to its state machine. Then it will redirect the request to the leader if necessary or it will start an election. Then the request will be handled by its respective handler function, and redirect the message to the leader or will respond to the client with an ok message. 

Each request handler function has specific logic according to each request (ex. get request will return the value for the key requested).

------

Logs only flow from leader to other servers. Randomized timers on leader election. Replicas to be able to continue to function even if other servers are down, survives leader crashes. 

Only put requests are appended into the log, not get requests. If a get request is made and a leader dies before filling it: ????



Start each loop through the messages with applying the puts received if needed, then handling the messages otherwise. 


Get_log appends all gets to the log, 





## Challenges
After setting up an initial election sequence that handles timeouts, the complexity increased when it needed to be expanded to handling partitions and deaths of leaders, especially in sequence. 



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