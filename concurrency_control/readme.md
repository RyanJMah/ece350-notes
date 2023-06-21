# ECE 350 Concurrency Control Notes

## The Mutex

```C
mutex_lock();
mutex_unlock();
```

* Two states. Locked and unlocked.

* Thread that locked the mutex unlocks it later on.

* A function that tries to take a mutex while it is locked will
  block until the thread that has the mutex unlocks it.

## The Semaphore

```
sem_wait();
sem_post();
```

* A thread that waits on a on a semaphore will decrement it by 1.

 
* A thread that signals on a semaphore will increment it by 1.
 
* If a thread tries to wait on a semaphore, and the decrement operation makes the value
  negative, the calling thread is blocked until the value is incremented again.
   * If a semaphore's value is currently -3, and there are 3 threads waiting, a call to post
     will increment to -2 and unblock one of the waiting threads.
 
<br>

* The increment and decrement of the counter are unconditional.
 
* When a thread signals a semaphore, it does not know which waiting thread will be unblocked.
  This is a decision for the scheduler to make.
 
 
* **Binary semaphore** functions like a mutex (kinda). Value can be 0 or 1.
 
* **Counting semaphore's** (or **general semaphore**) value can be whatever.

## Readers-Writers Lock
* Read-after-read dependencies don't really exist. Allow some concurrency.

* Writer cannot enter critical section while any other thread (reader or writer) is there.

* While a writer is in the critical section, neither readers nor writers may enter
  the critical section.

* Multiple threads can read at once

Similar to how file systems are. Many threads can read, only one can write at a time.

## Condition Variable

```C
cond_wait(mutex);
cond_signal();
cond_broadcast();
```

* Like a semaphore, but can broadcast to all waiting threads to wake up at once.

* Always used with a mutex as well.
   * `wait()` called when mutex is locked, automatically releases while waiting for condition

   * Mutex is automatically locked again when the calling thread is unblocked.

## Implementation

### Disable Interrupts
Crude but gets the job done. May work in embedded system or a very simple OS.
Could miss interrupts though, so not very good.

Maybe thread that enters critical-section exits early and never re-enables them.

### Test and Set
Special machine instruction that is performed in 1 cycle, therefore not interruptible.

```C
// All this happens in 1 clock cycle
bool test_and_set(int* i)
{
   if ( *i == 0 )
   {
      *i = 1
      return true;
   }
   else
   {
      return false;
   }
}
```

It could be used like so.

```C
while ( !test_and_set(busy) )
{
   // do nothing, wait for my turn...
}

// critical section...

/*
 * can just set back to 0, since no other thread could
 * be in the critical section.
 */
busy = 0;
```

No matter how many threads are executing this code at once, only 1 sill be able to
set the value to 1 and break out of the while loop.

Only works for binary semaphore or mutex, not for counting semaphores.

### Compare and Swap
1 cycle instruction and is thus uninterruptible.

```C
int compare_and_swap(int* value, int old_value, int new_value)
{
   if ( (*value) == old_value )
   {
      *value = new_value;
      return old_value;
   }

   return *value;
}
```

How to use it to decrement a semaphore:

```C
int old = 1;

// sem_wait()...
while (1)
{
   int actual = compare_and_swap(sem, old, old - 1);

   if ( actual == old )
   {
      old = old - 1;
      break;
   }
   else
   {
      old = actual;
   }

}

// critical section...


// sem_post()...
while (1)
{
   int actual = compare_and_swap(sem, old, old + 1);

   if ( actual == old )
   {
      break;
   }
   else
   {
         old = actual;
   }
}

```

**Alternative:** if there is HW support, could just do an atomic increment or decrement.

### Blocking and Unblocking

If a thread is supposed to be blocked, just mark it as blocked.

Also need to know which particular semaphore or mutex it's blocked on though.

Maybe, each locking object has an associated queue of threads waiting for it.

Maybe, when a thread is blocked, there's somewhere in its PCB that says what its
waiting for.

### Condition Variables

If a thread signals on a condition variable, the threads that are waiting is
removed from the blocked queue of the CV to the blocked queue for the mutex.
