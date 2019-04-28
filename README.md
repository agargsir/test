# test
1.Introduction - Our goal is to create a command shell with C++ called rshell which will contain a command prompt that reads commands on one singular line. One example of the syntax is,cmd= executable [ argumentList ] [ connector cmd ]
connector = || , &&, and ;
The command consists echoor ls,followed by an argument separated by spaces. The user will have the following to use to run multiple command lines:
            || àexecutes the next command if the first one fails.
            && àexecutes the next command if the first one succeeds
            ;à always executes the next command.
n One thing to note is that the user should have the ability to use all three of these commands as long as the syntax is correct.
