#### Objectives:
- Gain a detailed understanding of x86-64 stack organization.
- Gain a better understanding of what decisions are made at compile time vs. what modifications/decisions can occur when the program runs.
- Refine your understanding of strings and memory layout.
- Refine your skills examining x86-64 assembly in gdb.
- Understand how a couple types of buffer overflow vulnerabilities and exploits can affect a program.

This assignment involves applying two buffer overflow attacks to two separate binary executables.

You will gain experience with how poorly written programs can expose the system and user to
potential security flaws.  You will only perform rudimentary "stack smashing" to overwrite stack variables and a function return address. Manipulating the stack in this manner is a fundamental
approach that many hackers use to gain access (illegally) to systems, it a common weakness in
operating systems and network servers.  In this course you will not peform a full attack, 
we leave that for follow on systems courses.

Our purpose is to help you learn about the runtime operation of programs and to understand the nature of this form of security weakness so that you can avoid it when you write system code. We do not condone the use of these or any other form of attack to gain unauthorized access to any system resources. There are criminal statutes governing such activities.


#### Instructions
There are two programs each of which require you to perform a specific attack. 

Obtain your specific targets downloading it from a specific URL. Follow the steps below to obtain your binary bomb.
1. Pointing your browswer at the following URL ``https://courses.cs.duke.edu/compsci210d/fall21/student_targets/<NetID>``, where ``<NetID>`` is your Duke NetID. This directory requires Duke NetID Shibboleth authorization, and you may be prompted to authenticate if you aren't already.
2. Save the files called target1-``<NetID>`` and target2-``<NetID>`` to your local machine (right click "Save link as").
3. Drag the target1-``<NetID>`` and target2-``<NetID>`` to your container's P4_Attackproj folder
4. Open a terminal on your container and cd to your attack project directory.
5. Change permissions on each of the above files to allow execute by using the linux chmod command
```bash
chmod +x target1-<NetID>
chmod +x target2-<NetID>
```

Since the use of stack manipulation is a common method for attacking systems, there are some
mitigation techniques deployed that we need to disable for you to perform the attacks.  One in particular is address space layout randomization (ASLR) that changes which address your programs uses for data, text and stack on each individual execution of the program.  That makes it very difficult to attack since the memory locations of vulnerabilities are always changing.  ``gcc`` also includes some protective measures for stack buffer overflows. We disabled those to generate your target binaries, however, you need to disable the ASLR.  Do this by typing the following command at your linux prompt ``setarch `uname -m` -R /bin/bash``. This will disable ASLR for a new bash shell and any programs run from that shell.  If you are not running with ASLR disabled your program always use different addresses so even a correct soltion will result in a ``Segmentation fault``.  ***If you exit this shell or your container restarts, then you will need to run the above ``setarch`` command again before you can perform your attack.***

If you are running on a container locally, you need to start a new instance of your container in safe mode. 
```
docker run --name cs210-unsafe --security-opt seccomp=unconfined --mount type=bind,source=/Users/alvy/Containers/210,destination=/cs210 -it dev210 bash
```
This starts a new container called cs210-unsafe.  Attach to that container using VSCode or a ssh from a terminal.  Then disable ASLR by using the ``setarch `uname -m` -R /bin/bash`` command.

You are now ready to start your attacks.

### Target1
The first target program (target1) requires you to overwrite a specific array in the function ``my_bug()``. Within this function there is a call to ``gets()`` that reads a string into some memory location.  Your attack is to determine an input string that causes the ``gets()`` call to overwrite the existing string "This should not be here" with the string ``Use the force Luke``, it needs to be that exact string. 

A correct input will produce the output ``Success! You correctly hacked the program``.

You run your attack by typing ``./target1-<NetID>`` and you can directly type your input string on the keyboard.

The project repository contains a file called ***exploit1.txt*** where you add the correct inputs to your program when you determine the correct input string. You will submit this file to Gradescope, however grading will occur offline due the security implications of this project.  You can also use file redirect (e.g., ``./target1-<NetID>  < exploit1.txt``) to ensure your exploit runs correctly.

Push the file ***exploit1.txt*** to your remote repository when you have a working solution.


### Target2
The second target program (target2) requires you to overwrite a specific array in the function ``my_bug()``. Within this function there is a call to ``gets()`` that reads a string into buffer on the stack.  Your attack is to use a buffer overflow attack to overwrite the return address on the stack in ``my_bug()`` such that it returns to a specific location in the program.  that location is the address of the function ``call_me()``.  

You need to find the address of ``call_me()`` and create a string to input that causes that address to be written to the return address location on ``my_bug()``'s stack.  To get a correct value read into the program you need to convert an ASCII string to a raw binary set of values for input.  We provide a tool ``hex2raw`` to facilitate this conversion.

You encode your exploit string as a sequence of hex digit pairs separated by whitespace, 
where each hex digit pair represents a byte in the exploit string. The program ``hex2raw`` converts these strings into a sequence of raw bytes, which can then be fed as input to the target.  Use ``hex2raw`` in the following manner to generate your raw input file
```
cat exploit2.txt | ./hex2raw > exploit2.raw
```
Then run your attack by typing 
```
./target2-<NetID> < exploit2.raw
```
You may do this without file redirect using the following 
```
cat exploit2.txt | ./hex2raw | ./target2-<NetID>
```

### Important Points
- Your exploit string must not contain byte value 0x0a at any intermediate position, since this is the ASCII code for newline (‘\n’). When Gets encounters this byte, it will assume you intended to terminate the string.
- ``hex2raw`` expects two-digit hex values separated by one or more white spaces. So if you want to create a byte with a hex value of 0, you need to write it as 00. To create the word 0xdeadbeef you should pass “ef be ad de” to ``hex2raw`` (note the reversal required for little-endian byte ordering).

### TOOLS and TIPS
- ``gdb`` you will use the debugger in a variety of ways to step through programs, to examine register and memory state and to disassemble functions.  Disassembling within gdb provides the most detailed information. 
    - Setting breakpoints at correct locations is critical
    - Displaying memory and register values is critical
- Take notes! We strongly recommend that you keep notes of the what's stored at important addresses in memory and in registers at different points in the program's execution.
Losing ones place or forgetting what you've discovered on the way to a solution is not reason for extension, etc.
- ``objdump`` is a tool that prints information about various parts of binary executable programs.
    - ``objdump -t bomb > symtab.out`` prints out the symbol table information in a program which contains the names of all functions and variables, this instance redirects the output to a file named symtab.out.
    - ``objdump -d bomb > asm.out`` disassembles the entire program and redirects the output to a file named asm.out
- Make sure you are running with ASLR disabled as described above
- Make sure your input to hex2raw is in the correct format and correct byte order


#### Submission

- Submit your project (exploit1.txt, exploit2.txt, and exploit2.raw) on gradescope using the gitlab submission process, there is no autograding for this project.

- ***Team specific instructions*** 
    - Solve targets for a single NetID, ``target1-<NetID>`` and ``target2-<NetID>`` must be for the same NetID.
    - The student submitting must be for the attacks that you solved must be the same as the NetID.  That is if you solved targets for NetID ab1, then student with NetID ab1 must be the one submitting on Gradesope.
    - Teams must also edit the file called reflection.txt and include a detailed description of each team members contributions to the project.
