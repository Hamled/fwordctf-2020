# Notes

## Situation
Due to the use of `tcsetattr` and other termios functions in this binary, I was not able to figure out how to pipe input or use pwntools.

There might be a way to achieve that, but I'm not quite adept enough with terminal / tty stuff to know what would be needed.

Instead, it seems like I need to type the input by hand (pasting in printable ASCII characters is also acceptable).


## Solution
I was able to solve this by overwriting the return pointer from `main`, having it return to the `rasengan` function instead of `__libc_start_main`.

Because I needed to type characters in, and the address of `rasengan` was not entirely printable ASCII characters, I needed to use some control codes, entered via the standard [escape sequences](https://en.wikipedia.org/wiki/C0_and_C1_control_codes).

In particular, I needed to use the address `0x400918`, which is actually `rasengan+1`, because the control character with value `0x17` is "End of Transmission Block" and it appeared to have the effect of discarding all previous input from the terminal buffer before sending to the program's standard input file.

So in the end, the exploit can be achieved by first logging into the target machine with SSH (credentials are in [./README.md](the README)), running the program directly and entering the following input via keyboard in your terminal:

* 168 printable ASCII characters (can be pasted in)
* `0x18` (control code for CANCEL) by typing ctrl+x
* `0x09` (control code for TAB) by typing ctrl+i (could probably also hit tab key, but I didn't try that)
* `0x40` normal `@` character
* `0x00` (control code for NUL) five times, by typing ctrl+@
* Carriage return by pressing enter

