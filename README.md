JLFS
====

JLFS is a thin shared-folder filesystem used only on JSLinux, where Linux runs inside the browser on a Javascript x86 emulator. It is used to expose files on the host via the HTML5 File API.

It has two components:

1. The jlfs filesystem driver, run in the JSLinux instance. Currently, it provides read-only support, though it can be extended to be read-write in future.
2. The JLHost driver, a Javascript module which simulates a device on the "hypervisor side"

jlfs and JLHost communicate with each other using
1. 16 ports beginning with 0x180 by default
2. IRQ 5 (by default)
3. A request page and a result page. jlfs allocates these and tells JLHost about the physical address of these pages. Since memory is represented by a Javascript array, JLHost is able to read and write arguments and results by the simple expedient of reading and writing to the the right substring of the array.

The typical lifecycle of a request is as follows:
1. jlfs writes a JSON string containing request arguments to requestbuf, writes to the command port.
2. JLHost retrieves the arguments, carries out the request, writes the result to the result buffer as a JSON-like string, stores the result in a status register, and raises an interrupt.
3. The jlfs interrupt handler wakes up the corresponding thread, which processes the result buffer. Due to the limitations of kernel sscanf(), the result JSON syntax is restricted: order of elements is fixed, spaces in filenames are encoded as '/'es.

