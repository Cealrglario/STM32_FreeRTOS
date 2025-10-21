# Message Queues and their Uses

Message queues (or simply queues) are thread-safe FIFO data structures that are used to transfer data between tasks and/or ISR's. They act almost like a semaphore (in the way that they can be used as signals) but have the advantage that they can also share data like temperature data or peripheral data.

## Usages/Advantages

#### Modularity
Message queues promote code modularity as tasks sending data don't need to know any information about the tasks that will be receiving that data. They can simply send and receive independently.

Data sent into a message queue is copied, ensuring that the task receiving the data cannot modify the original data and that the original data does not need to worry about the task receiving the data before it continues executing further.

- For example, if sending temperature data into a queue, the task storing and sending that data can immediately sample new temperature data without worrying about the previously sent data being overwritten, as a copy has already been made and sent.

#### Synchronization/Efficiency
A task that is awaiting a queue message is put into a blocked state until that message has been received by the task (in other words, the message has been sent by a sender). This saves processing power for tasks that need it and are presently running.

Additionally, if a task tries to send to a full queue, it will be blocked until that queue has space again.

#### Effectively, queues are very similar to binary semaphores as they can not only act as a signal for a task to begin executing, but they can also send data along with that signal so that a task can begin processing data as it is received.
