# RShell
# CS 100 Project: Spring 2019

## Partner 1: Jorge L. Garcia
## SID: 862089506
## Partner 2: Achyutha Komarlu
## SID: 862011438
#

# Introduction

RShell is an interactive shell where the user inputs their command(s) and our program runs a series of parsing functions on the input which will create a composite object tree from that line of command(s). Once the tree is created the commands will be forked and ran in child processes. RShell has been implemented using the composite design pattern.


# Diagram

<img width="906" alt="Screen Shot 2019-10-29 at 3 45 31 PM" src="https://user-images.githubusercontent.com/45085816/67815955-a34ec480-fa65-11e9-9a8d-e06c844cb7a9.png">


# Classes

**Line** :
* This will be the abstract class that all classes must implement. All implementations of ```Execute()``` will return a 1 upon successful execution and 0 otherwise. All implementations of ```Exit()``` will deal with system exit calls.

**Command** :
* This class will contain the command that was used along with a vector containing the arguments that were passed to that command. The execute function will be the only implementation of ```Execute()``` which will actually create a child process to execute the command. ```Execute()``` will return a 1 upon successful completion and 0 upon failure. ```Exit()``` will safely terminate all running processes.

**Connector** :
* This will be another abstract class which all of the control operators will implement. The control operators will have the capacity to hold a collection of commands and/or one or more control operators.
* Control operator classes which will implement the Connector abstract class are:
  * *And_Op*
  * *Or_Op*
  * *Semicol_Op*

**And_Op** :
* This class will contain a collection of Command and/or Connector objects and will represent this collection as private members ```line_1``` & ```line_2```. The execute function will run both ``` line_1->Execute()``` and ``` line_1->Execute()``` regardless of their corresponding return value. The class’ execute function will return a 1 if and only if both line_1 and line_2 executions succeed. The exit function will call all of the children’s exit functions to safely terminate any running processes.

**Or_Op** :
* This class will contain a collection of Command and/or Connector objects and will represent this collection as private members ```line_1``` & ```line_2```. The execute function will only run ```line_2->Execute()``` if ```line_1->Execute()``` fails (returns a 0). This class’ execute function will return a 1 if either ```line_1->Execute()``` or ```line_2->Execute()``` succeed. The exit function will call all of the children’s exit functions to safely terminate any running processes.

**Semicol_Op**
* This class will contain a single Command and will represent it as private member ```line```. The execute function will run ```line->Execute()``` and will return the ```line->Execute()``` return value (1 upon completion and 0 otherwise). The exit function will call its child's exit function to safely terminate any running processes.

# Prototypes/Research
```
#include <iostream>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <cstring>
#include <string>

int main(int argv, char **argc) {

       pid_t pid = fork();

       if (pid == 0) {
               printf("Child process id: %d\n", pid);
               printf("Child: Running execvp()\n");

               if (execvp(argc[1], (argc + 1)) == -1) {
               // Pointer arithmetic used to shift argc pointer to actual
               //   command that was passed rather than this main's executable
                      
                       printf("\n\nError in child process\n\n");
                       perror(argc[1]);
               }
               std::cout << "\nChild: Process finished.\n\n"; // Anything beyond execvp will not run
       }
       if (pid > 0) {
               printf("Parent process id: %d\n", pid);
               printf("Parent: Waiting for child process to finish.\n");

               pid_t pid_wait = waitpid(0, NULL, 0);

               printf("pid_wait = %d\n\n", pid_wait);
               if (pid_wait == -1) {
                       perror("Parent");
               } else {
                       std::cout <<"\nParent: Child process completed\n\n";
               }

       }
       return 0;
}

```
**Steps Taken**:

 * **Compile**: ``` g++ -std=c++11 few_prototype.cpp -o few_prototype ```

 * **Run 1**: ``` ./few_prototype ls -l && echo prototype```

 * **Output 1**:
 ```
Parent process id: 9188                                                        
Parent: Waiting for child process to finish.                                   
Child process id: 0                                                            
Child: Running execvp()                                                        
total 40                                                                       
-rwxr-xr-x 1 jgarc271 csmajs  9256 Apr 27 13:03 few_prototype                  
-rw-r--r-- 1 jgarc271 csmajs   947 Apr 26 17:34 few_prototype.cpp              
pid_wait = 9188                                                                
                                                                              
                                                                              
Parent: Child process completed                                                
                                                                              
prototype
```
 * **Run 2**: ```./few_prototype la```

 * **Output 2**:
```
Parent process id: 12066                                                       
Parent: Waiting for child process to finish.                                   
Child process id: 0                                                            
Child: Running execvp()                                                        
                                                                              
                                                                              
Error in child process                                                        
                                                                              
la: No such file or directory                                                  
                                                                              
Child: Process finished.                                                       
                                                                              
pid_wait = 12066                                                               
                                                                              
                                                                              
Parent: Child process completed                                               
```
* **Findings**:

   * The waitpid function returns before the execvp finishes outputting its results to the standard output as seen in *Output 1*.

     * **Use Case**: We can have the parent process perform other functionality while the child is done running but hasn’t outputted to the standard output.
   * The execvp function returns a -1 upon an unsuccessful command as seen in *Output 2*.
     * **Use Case**: We can implement any necessary logic within an if statement that checks for this value and handles any extra functionality that we may want to implement upon failure of command.
   * If the execvp function runs successfully, it terminates the child process therefore preventing the execution of any code that proceeds it.
     * **Use Case**: Like in our previous finding (b), we can implement we extra functionality. For example, after the execvp call, we may decide to prompt the user and ask if they would like to modify and rerun that specific command instead of having them input an entire command, or multiple commands in the case of the usage of connectors.

# Development and Testing Roadmap

* **Development & Testing**:
 1. Create the component base class ```Line```.
 2. Create the leaf class ```Command```.
    * **Test**: Run unit tests on this class once completed to test that commands and arguments can be passed correctly and that child processes are created and terminated appropriately.
 3. Create the abstract composite class ```Connector```.
 4. Create all of the composite classes ```And_Op```, ```Or_Op```, and ```Semicol_Op```.
    * **Test**: Run unit tests on every composite class once completed to test that all of the logic runs correctly. We will also run integration tests to make sure that the classes properly integrate with ```Command``` classes and/or other ```Connector``` classes.
 5. Create the parsing functions that will parse the user's input and create the object trees.
    * **Test**: Run all of the necessary unit and integration tests to verify that trees are being properly formed and the interaction between these functions and the composite objects is as it should be.
 6. Create the main function which will the RShell and integrate all of the components of this project.
    * **Test**: Run an extensive set of integration tests to ensure that everything is working as it should be.
    
    
 
 
 
 
 
 
 # RShell
# CS 100 Project: Fall 2019

## Partner 1: Aditya Garg
## SID: 862014748
## Partner 2: Achyutha Komarlu
## SID: 862011438
#

# Introduction

RShell is a user input based interaction design, where a command is inputted and the program we have designed will run a series of parsed functions which in turn creates a composite object tree from the inputs given. Once tree is finalized the commands that were inputted will then be forked into a child class processor. Our RShell will be implemented utilizing the composite design pattern.


# Diagram

<img width="906" alt="Screen Shot 2019-10-29 at 3 45 31 PM" src="https://user-images.githubusercontent.com/45085816/67815955-a34ec480-fa65-11e9-9a8d-e06c844cb7a9.png">

# Classes that we will need to implement for our RShell design

Line Class (Main Abstract Class) : 
 All following classes will need to implement the Line Class. The implements of the Execute command if successful will return a 1 else a 0. If the Exit function is called all the implements will have to manage the system exit capability.

Command (Class which commands are processed) :
The command class will contain the commands passed through a vector with its arguments, and will be the only implement of the former Execute command which in turn makes it a child processor for that command. Like the Line class running the following Execute command will similarly return a 1 if successful else a 0. And also utilizes the Exit command which safely terminates the running programs.
 
Connector (Another Abstract Class with all the control operators) :
The Connector class will hold all the control operators which will be implemented. 

 There will be three Control Operator classes that will need to implement the Connector Abstract Class.
  - And_Op
  - Or_Op
  - Semicolon_Op

And_Op :
The And_Op Connector Class will contain some of the Command objects as well as Connector objects and will need representation by private variables i.e. (line_1 & line_2). When Execute is called both line_1 and line_2 will call  Execute and only if both return a 1 will they be deemed successful. And calling Exit  will make all the children classes’ exit to be called and terminate all running programs.
 
Or_Op:
The And_Op Connector Class will contain some of the Command objects as well as Connector objects and will need representation by private variables i.e. (line_1 & line_2). When Execute is called both line_1 and line_2 will call  Execute and only if both return a 1 will they be deemed successful. And calling Exit  will make all the children classes’ exit to be called and terminate all running programs.

Semicol_Op:
The class will utilize a single Command and will have just the private variable line. And when Execute is called the line member will return 1 if successful and 0 if not. And Exit will call the child to terminate all running programs.
 

# Prototypes/Research

#include <iostream>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <cstring>
#include <string>

int main(int argv, char **argc) { //arguments passed

    ParID_t ParID = fork(); 
 
       if (ParID == 0) {
               print("Child processor id: %d\n", ParID);
               print("Child: Running execvp()");

               if (execvp(argc[1], (argc + 1)) == -1) { //child arg
               // argc pointer shifted to actual by pointer arithmetic
   
                      
                       print("Error in child process");
                       Parerror(argc[1]);
               }
               std::cout << "Child: Processor finished."; 
       }
       if (ParID > 0) {
               print("Parent class processor ID: %d\n", ParID);
               print("Parent: Waiting for child processor to finish executing.\n");

               ParID_t ParID_wait = waitParID(0, NULL, 0); // checks parent id to null

               print("PairID_wait = %d\n", ParID_wait);
               if (ParID_wait == -1) {
                       Parerror("Parent");
               } else {
                       std::cout <<"Parent: Child process completed";
               }

       }
       return 0;
}

```
Needed commands in bash 

To Compile: `` g++ -std=c++11 few_prototype.cpp -o few_prototype ``

  To test Run 1: `` ./few_prototype ls -l && echo prototype``

 Output 1:
 ```
Parent Process ID: 9098                                                       
Parent: Waiting for child processor to finish executing.                                   
Child processor id: 0                                                            
Child: Running execvp()                                                        
total 40                                                                       
-rwxr-xr-x 1 akoma003 csmajs  9256 Oct 30 15:19 few_prototype                  
-rw-r--r-- 1 akoma003 csmajs   947 Oct 30 19:21 few_prototype.cpp              
ParID_wait = 9098                                                              
                                                                              
                                                                              
Parent: Child processor completed                                                
                                                                              
                      
# Development and Testing Roadmap

1. Base class Line 
2. Leaf Class Command
***Test Command*** - unit tests to check if commands and arguments pass properly and child processors terminate safely.
3. Composite Connector Class and its subclasses And_Op, Or_Op, and Semicol_Op.
***Test Connector*** - unit tests for each component and integration testing.
4. Parsing Functions for user input and obj trees
***Test*** - run tests to see code line up
5. Main func for Rshell to hold all parts aforementioned 
***Test Main*** - Integration tests


