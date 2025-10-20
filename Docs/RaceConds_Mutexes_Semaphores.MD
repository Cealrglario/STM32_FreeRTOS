# A guide on Race Conditions, Mutexes, and Semaphores and how they're connected

## Race Conditions

Race conditions occur in an embedded context when interrupts modify global data/variables shared between, for example, a main loop and the interrupt while the main loop is in the middle of processing said shared data. This causes an issue where the data that main is handling is now potentially a mix of old, pre-interrupt data, and new, post-interrupt data, leading to the result of the main loop calculation being "garbage".

#### Solutions in a Bare-Metal Context
One easy solution to prevent a race condition from occurring is to disable interrupts before globally shared data is "processed", and to keep interrupts disabled until the processing has completed, ensuring that data doesn't become contextually garbage.
	- This can be done in a bare-metal context by using the __disable_irq() or __enable_irq() calls, depending on the architecture of the board you are using. For example, in our PIC24F16KA101 microcontroller, we wouldn't have explicitly defined functions to do this. We would need to set register values in order to "pause" interrupts.

#### In an RTOS Context
Race conditions become extremely problematic within RTOS'. RTOS schedulers use preemption to rapidly switch between tasks, likely mid-execution. The issue with this is that tasks can be interrupted not only by ISR's, but also by other tasks, potentially also sharing global data. Disabling interrupts is not an option here because interrupts are needed by the RTOS scheduler for efficiency sake, and disabling them will lead to abysmal performance.

So we need some sort of way to protect shared data in specific contexes/instances without outright disabling interrupts. Enter mutexes and semaphores, RTOS tools for managing concurrency and shared resources.

## Mutexes (Mutual Exclusion)
Mutexes ensure that shared resources are only accessible by one task at a tine, preventing data modification mid-task. They act like a "key" to the shared resource that only one task can have at a time. 

#### How they work
	1. You create a mutex explicitly, like tempData_mutex.
	2. A task takes the aforementioned mutex to access a shared resource.
		○ Later, this task must also return the mutex. A task that takes a mutex must also return it.
	3. While the aforementioned task has the mutex, the mutex appears "invisible" to the rest of the tasks who also want to access that shared resource, meaning they don't have access to it.
		○ If a task cannot proceed due to a "missing" mutex, the RTOS scheduler will set its state to "blocked" until the mutex is available again, ensuring CPU resources are not wasted.
	4. Once the mutex is available again, the next task with the highest priority (at least, in FreeRTOS) is allowed to access the mutex, and its state is set to "running".

#### Primary usage
Mutexes are for preventing a single shared resource from being accessed by multiple tasks at the same time. This is excellent for preventing race conditions on global flags and any peripherals like I2C or UART.

## Semaphores (Signalling  / Counting)
A semaphore, unlike a mutex, is a more general tool used for signalling/counting pertaining to a shared resource pool rather than a single shared resource.

Semaphores do not act like a "key" in the way that mutexes do (where the task that takes a mutex must also return it, and that mutexes only handle a single shared resource). Instead, semaphores act as a signal or counter that can indicate the status of a pool of shared resources, or more simply, whether or not a task can "go" while it waits for an event to occur. 

#### How they work
	1. Explicitly create a semaphore, like xButton_Semaphore.
		○ Note that the "x" at the beginning of the name, specifically in FreeRTOS naming convention, indicates the type of the data. The "x" in FreeRTOS indicates BaseType_t which uses the most efficient integer size for the processor configured in FreeRTOS. "v" would indicate void, "p" for pointer, etc.
	2. Configure an interrupt that, if used as a flag (in a binary semaphore), gives the semaphore.
	3. Any task that is waiting to "take" a semaphore is waken up and given it (in order of priority level).
		○ Note that this implies that if a task tries to take a semaphore when it is not available (like when a binary semaphore is "0"), the scheduler will block the task, making sure it does not take up any CPU resources, until the semaphore can be given to it.
	4. The task can now execute its code, as though the semaphore being given to it were a "go" flag allowing its execution.
	5. The task will "give" the semaphore back when it is done executing its code, and the next-priority task will be waken up and given the semaphore by the scheduler.
