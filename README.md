# NLHDH_PhungPhucHau_NguyenDinhHai
4.3 Code Reading Questions

4.3.1 Thread questions

1.What happens to a thread when it calls thread_exit? What about when it sleeps?
Answer: Briefly when a Thread calls thread_exit(), then thread_exit() will set interrupts off to avoid race condition with context switch and then will save current threads VM space and set next threads as null, manipulate(decrement) the reference counts and add the thread to zombies' list, we don't actually destroy all its resources, it will be done by thread_destroy(). When thread calls thread_sleep it sets the address of the current thread as sleepaddr(sleep_address)and then gives a call to machinde independent context switch routine( Which in turn gives call to machine-dependent switch which calls mips_switch ) and sets sleep address of current thread as null 

2.What function—​or functions—​handle(s) a context switch?
There are functions at different level and interleaved that handle the context switch. From thread.c we give call to static void mi_switch(threadstate_t nextstate) which is machine independent call to context switch that in turn gives a call to void md_switch(struct pcb *old, struct pcb *nu) which is machine dependent call to handle context switch that in turn calls mips_switch(struct pcb *old, struct pcb *nu), this is the real function at system level that actually does the switch. At each level data structures such as data references and current thread and stack, interrupt are manipulated 

3.What does it mean for a thread to be in each of the possible thread states?
There are 4 thread states that are defined in thread.c as enum data structures namely run,ready,sleep,zombie. Following is the structural defination: typedef enum{ S_RUN, S_READY S_SLEEP S_ZOMB} threadstate_t; 

4.What does it mean to turn interrupts off? How is this accomplished? Why is it important to turn off interrupts in the thread subsystem code?
Turning interrupts off means that we are disabling interrupt handling till we don't reset it to handle interrupts. We turn it off to ensure atomicity while we execute the following kernel code so that it is uninterrupted execution moreover, it also allows us to avoid race condition if there is a shared variable. Thereby making the execution safe. It is accomplished using splhigh() function that sets interrupt off. It is important to turn interrupts off especially in thread code so that race condition is avoided when updating the shared variable such as refrence counts, number of threads with for example context switch which will try to manipulate the same data at the same time. Also to ensure atomicity of the execution as well as in to avoid thread sleeping while handling interrupts. 

5.What happens when a thread wakes up another thread? How does a sleeping thread get to run again?
When a thread wakes up another thread than it passes the address of the thread that it wants to wake up. thread_wakeup() is invoked which will wake up that thread and changes its stated to runnable. The calling thread than has to call yeild the cpu to thread but stay runnable using call void thread_yeild(void) 

4.3.2 Scheduling questions

1.What function (or functions) choose the next thread to run?
We call scheduler() to determine the next thread to run which in turn calls void * q_remhead(struct queue *q), which returns nothing but next in runqueue and sets all the structures.

2.How is the next thread to run chosen?
Scheduler() calls q_remhead that will access runqueue which is passed as its parameter and will return the next thread in queue waiting for allocation by assigning the return thread as ret =q->data[q->nextthread]; and sets next thread as q->nextthread = (q->nextthread+)% q->size;

3.What role does the hardware timer play in scheduling?
In OS161 we imply round robin scheduling. Whenever, assigned quanta time is finished for that process in executing, hardclock timer interrupts are generated indicating the quanta expiartion(interrupts every Hz value) and context switch happens

4.What hardware independent function is called on a timer interrupt?
On timer interrupt, static void mi_switch(threadstate_t nextstate) is called which is hardware independent function. 

4.3.3 Synchronization questions

1.Describe how wchan_sleep and wchan_wakeone are used to implement semaphores.
While using thread_sleep() in tt3.c we have simply disable interrupts and in thread_wakeup() we have used semaphores P and V so that mutual exclusion is achieved as shared variable wakersem, is tested and set using P and V one and only one thread can execute the instructions of setting the “done” variable at one time. Once a process updates “done: it can release the lock the wakersem and starts its execution. While updating the “done” no other can set it and thus only one thread can execute it at one time. Hence providing the desired functionality.

2.Why does the lock API in OS/161 provide lock_do_i_hold, but not lock_get_holder?
We use lock_do_i_hold() to first check if other thread is holding the lock instead of straight giving lock_get_holder() because we dont want the lock to be hold when other thread has got access to it else it will defeat the purpose of mutual exclusion which is one of the reason that locks are implemeted. Below is the precise condition mentioned in os161 for locks: “When the lock is created, no thread should be holding it. Likewise,when the lock is destroyed, no thread should be holding it.”
