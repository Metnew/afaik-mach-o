# Mach-O

How OS X Executes Applications - http://0xfe.blogspot.com/2006/03/how-os-x-executes-applications.html - cool but old post

Q&A: How OS X Executes Applications http://0xfe.blogspot.com/2006/03/qa-how-os-x-executes-applications.html

##### https://cocoaintheshell.whine.fr/2009/07/universal-binary-mach-o-format/

> But how did they manage to do this ? Well, it’s very simple, an UB application is nothing more than an archive of 2 applications with a special header.


Question 5. What are Two-Level Namespaces?

It is a feature included since OS X 10.1, that prevents collisions with symbol names in dynamic libraries. It works by associating library names with symbol names at compile time.

Suppose you have an application that is linked against libfirst and libsecond. libfirst exports a function called dothis(). At a later time, a new version of libsecond comes out with its own dothis() function. Now, the application may execute whichever dothis() function it loads first, which may not be the one that was intended.

With two-level namespaces (enabled by default), the linker associates dothis() with libfirst at compile time. This prevents the chances of symbol collisions in future versions of linked libraries.

Running an Application

Now that we know what a Mach-O file looks like, let us see how OS X loads and runs an application.

When you run an application, the shell first calls the fork() system call. Fork creates a logical copy of the calling process (the shell) and schedules it for execution. This child process then calls the execve() system call providing the path of the program to be executed.

The kernel loads the specified file, and examines its header to verify that it is a valid Mach-O file. It then starts interpreting the load commands, replacing the child process's address space with segments from the file. 

At the same time, the kernel also executes the dynamic linker specified by the binary, which proceeds to load and link all the dependent libraries. After it binds just enough symbols that are necessary for running the file, it calls the entry-point function.

The entry-point function is usually a standard function statically linked in from /usr/lib/crt1.o at build time. This function initializes the kernel environment and calls the executable's main() function.

The application is now running.

The Dynamic Linker

The OS X dynamic linker, /usr/lib/dyld, is responsible for loading dependent shared libraries, importing the various symbols and functions, and binding them into the current process. 

When the process is first started, all the linker does is import the shared libraries into the address space of the process. Depending on how the program was built, the actual binding may be performed at different stages of its execution.
Immediately after loading, as in load-time binding.
When a symbol is referenced, as in just-in-time binding.
Before the process is even executed, an optimization technique known as pre-binding
If a binding type is not specified, the just-in-time binding is used.

An application can only continue to run when all the required symbols and segments from all the different object files can be resolved. In order to find libraries and frameworks, the standard dynamic linker, /usr/bin/dyld, searches a predefined set of directories. To override these directories, or to provide fallback paths, the DYLD_LIBRARY_PATH or DYLD_FALLBACK_LIBRARY_PATH environment variables can be set a colon-separated list of directories.

Within the __TEXT segment, there are four major sections:
__text : The compiled machine code for the executable.
__const : General constants data.
__cstring : Literal string constants.
__picsymbol_stub : Position-independent code stub routines used by the dynamic linker.
This keeps the executable and non-executable code clearly separated within the segment.




### https://www.defcon.org/images/defcon-16/dc16-presentations/defcon-16-hotchkies.pdf


Mach-O Text Segment
.text ( TEXT, text) Code, same as everywhere else
.const ( TEXT, const) Initialized constants
.static const ( TEXT, static const) Not defined*
.cstring ( TEXT, cstring) Null terminated byte strings
.literal4 ( TEXT, literal4) 4 byte literals
.literal8 ( TEXT, literal8) 8 byte literals
.constructor ( TEXT, constructor) C++ constructors*
.destructor ( TEXT, destructor) C++ destructors*
.fvmlib init0 ( TEXT, fvmlib init0) fixed virtual memory shared library initialization*
.fvmlib init1 ( TEXT, fvmlib init1) fixed virtual memory shared library initialization*
.symbol stub ( TEXT, symbol stub) Indirect symbol stubs
.picsymbol stub ( TEXT, picsymbol stub) Position-independent indirect symbol stubs.
.mod init func ( TEXT, mod init func) C++ constructor pointers*



Mach-O Data Segment
.data ( DATA, data) Initialized variables
.static data ( DATA, static data) Unused*
.non lazy symbol pointer ( DATA, nl symbol pointer) Non-lazy symbol pointers
.lazy symbol pointer ( DATA, la symbol pointer) Lazy symbol pointers
.dyld ( DATA, dyld) Placeholder for dynamic linker
.const ( DATA, const Initialized relocatable constant variables
.mod init func ( DATA, mod init func) C++ constructor pointers
.mod term func ( DATA, mod term func) Module termination functions.
.bss ( DATA, bss) Data for uninitialized static variables
.common ( DATA, common) Uninitialized imported symbol definitions


Objective-C Segment
.objc class ( OBJC, class)
.objc meta class ( OBJC, meta class)
.objc cat cls meth ( OBJC, cat cls meth)
.objc cat inst meth ( OBJC, cat inst meth)
.objc protocol ( OBJC, protocol)
.objc string object ( OBJC, string object)
.objc cls meth ( OBJC, cls meth)
.objc inst meth ( OBJC, inst meth)
.objc cls refs ( OBJC, cls refs)
.objc message refs ( OBJC, message refs)
.objc symbols ( OBJC, symbols)
.objc category ( OBJC, category)
.objc class vars ( OBJC, class vars)
.objc instance vars ( OBJC, instance vars)
.objc module info ( OBJC, module info)
.objc class names ( OBJC, class names)
.objc meth var names ( OBJC, meth var names)
.objc meth var types ( OBJC, meth var types)
.objc selector strs ( OBJC, selector strs)


What they say: ”All sections in the OBJC segment, including old
sections that are no longer used and future sections that may be
added, are exclusively reserved for the Objective C compiler’s use.”
What they mean: ”No docs 4 u LOL kthxbai!”


Mach-O
Dynamic Linking of Imported Functions in Mach-O - CodeProject
Knowing the principle of linking of imported functions in Mach-O libraries, we can achieve a rather interesting effect…www.codeproject.com


Elf vs Mach-O:
http://timetobleed.com/dynamic-linking-elf-vs-mach-o/ - some padding, WTF, IDK


http://timetobleed.com/dynamic-symbol-table-duel-elf-vs-mach-o-round-2/ - Symbol Table, WTF, IDK


DYLD explained - (maybe outdated)
http://www.newosxbook.com/articles/DYLD.html


Some useful tools: https://github.com/bx/machO-tools


#### http://www.blackhat.com/presentations/bh-dc-09/Iozzo/BlackHat-DC-09-Iozzo-Macho-on-the-fly.pdf

Presentation is dated 2009, but worth reading

Mach-O file
• Header structure: information on the target
architecture and options to interpret the file.
• Load commands: symbol table location,
registers state.
• Segments: define region of the virtual
memory, contain sections with code or data. 

Important segments
• __PAGEZERO, if a piece of code accesses
NULL it lands here. no protection flags.
• __TEXT, holds code and read-only data. RX
protection.
• __DATA, holds data. RW protection.
• __LINKEDIT, holds information for the
dynamic linker including symbol and string
tables. RW protection. 


Execution steps
Kernel
• Maps the dynamic linker
in the process address
space.
• Parses the header
structure and loads all
segments.
• Creates a new stack.
Dynamic linker
• Retrieves base address
of the binary.
• Resolves symbols.
• Resolves library
dependencies.
• Jumps to the binary entry
point. 


__PAGEZERO INFECTION
• Change __PAGEZERO protection flags
with a custom value.
• Store the crafted stack and the autoloader
code at the end of the binary.
• Point __PAGEZERO to the crafted
stack.
• Overwrite the first bytes of the file with
the auto-loader address

...Some notes about ASLR, but it's probably outdated


#### http://www.blackhat.com/presentations/bh-dc-10/Suiche_Matthieu/Blackhat-DC-2010-Advanced-Mac-OS-X-Physical-Memory-Analysis-wp.pdf - idk, some notes about headers

Launching an Application
When you launch an application from the Finder or the Dock, or when you run a program in a shell, the system ultimately calls two functions on your behalf, fork and execve. The fork function creates a process; the execve function loads and executes the program. There are several variant exec functions, such as execl, execv, and exect, each providing a slightly different way of passing arguments and environment variables to the program. In OS X, each of these other exec routines eventually calls the kernel routine execve.

When writing a Mac app, you should use the Launch Services framework to launch other applications. Launch Services understands application packages, and you can use it to open both applications and documents. The Finder and the Dock use Launch Services to maintain the database of mappings from document types to the applications that can open them. Cocoa applications can use the class NSWorkspace to launch applications and documents; NSWorkspace itself uses Launch Services. Launch Services ultimately calls fork and execve to do the actual work of creating and executing the new process. For more information on Launch Services, see Launch Services Programming Guide.

Forking and Executing the Process
To create a process using BSD system calls, your process must call the fork system call. The fork call creates a logical copy of your process, then returns the ID of the new process to your process. Both the original process and the new process continue executing from the call to fork; the only difference is that fork returns the ID of the new process to the original process and zero to the new process. (The fork function returns -1 to the original process and sets errno to a specific error value if the new process could not be created.)

To run a different executable, your process must call the execve system call with a pathname specifying the location of the alternate executable. The execve call replaces the program currently in memory with a different executable file.

A Mach-O executable file contains a header consisting of a set of load commands. For programs that use shared libraries or frameworks, one of these commands specifies the location of the linker to be used to load the program. If you use Xcode, this is always /usr/lib/dyld, the standard OS X dynamic linker.

When you call the execve routine, the kernel first loads the specified program file and examines the mach_header structure at the start of the file. The kernel verifies that the file appear to be a valid Mach-O file and interprets the load commands stored in the header. The kernel then loads the dynamic linker specified by the load commands into memory and executes the dynamic linker on the program file.

The dynamic linker loads all the shared libraries that the main program links against (the dependent libraries) and binds enough of the symbols to start the program. It then calls the entry point function. At build time, the static linker adds the standard entry point function to the main executable file from the object file /usr/lib/crt1.o. This function sets up the runtime environment state for the kernel and calls static initializers for C++ objects, initializes the Objective-C runtime, and then calls the program’s main function.

