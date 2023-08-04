# ECE 350 - IO Devices and Drivers

IO is a lot slower than the CPU. Thus, how we handle IO is very important.

## Communication with Devices

Three main ways for SW to interact with HW:

1. Polling
2. Interrupts
3. DMA

## Application IO Interface

Device-to-device communication protocols vary a lot. Different IO devices also vary a lot. Thus, there is a need for a uniform interface for the OS to communicate with hardware - drivers.

This is especially true when writing a general-purpose OS like Windows or Linux, which is expected to work on a lots of different HW.

In order to create a uniform interface, can categorize HW devices into distinct groups that share important qualities:

* **Data Transfer Mode**
  * e.g., one byte at a time vs a block of bytes
* **Access Method**
  * e.g., sequential access vs random access
* **Transfer Schedule**
  * e.g., synchronous vs asynchronous
* **Dedication**
  * e.g., multiple threads allowed to access device, vs only one thread
* **Device Speed**
  * Obvious
* **Transfer Direction**
  * e.g., bi-directional vs uni-directional

<br>

Operating systems also implement an *escape* syscall to allow sending arbitrary commands to a device. In Linux, this is the `ioctl` syscall.

* Allows us to issue commands that OS designers didn't think of (and didn't create syscalls for)

### Character Devices

Something like a keyboard, works on a byte-by-byte basis.

Syscalls are `get()` and `put()`.

### Block Devices

Something like a hard disk drive - operates on blocks of data.

Syscalls are `read()`, `write()`, and `seek()`.

### Network Devices

Functionally different than all other devices (especially if wireless). There is a bunch of latency, potential for packet loss, etc.

Implemented via the socket API. A socket is treated like a file.

## Spooling and Reservations

A **spool** is a buffer for a device that can serve only 1 job at a time. (e.g., printer)

Printer needs to finish a whole print job before it starts the next. OS centralizes all communication to the printer through the spool.

## IO Protection

Recall that there are kernel mode and user mode instructions - some instruction are restricted so that only the OS can issue them (in kernel mode), otherwise it is an illegal instruction.

Want all user accesses to HW to be mediated through the OS - prevent invalid requests to HW. Tradeoff though, more security for less performance.

In certain scenarios, we want user programs to have direct access to HW (for performance reasons). An example would be allowing a video game direct access to the GPU

## Kernel HW Data Structures

Kernel must keep track of what HW devices are in use by which process, and the general state of the HW device.

Keeps the devices as an index to the **system-wide open-file table**.

Explains why if you print a file descriptor, you just see a number - it's the index to the open-file table.

![](./images/unix_kernel_io_structures.png)