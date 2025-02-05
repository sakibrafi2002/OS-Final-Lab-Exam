
# Line-by-Line Explanation of the C Program

### **Headers**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
```
1. **`#include <stdio.h>`**: Provides functions for input and output (e.g., `printf`, `fgets`).
2. **`#include <stdlib.h>`**: Includes utility functions like `exit`.
3. **`#include <string.h>`**: Used for string manipulation functions like `strtok` and `strcmp`.
4. **`#include <unistd.h>`**: Provides access to POSIX API, including `fork`, `execvp`, and `pipe`.
5. **`#include <sys/types.h>`**: Defines data types like `pid_t` used for process IDs.
6. **`#include <sys/wait.h>`**: Provides functions to wait for process termination, like `wait`.

---

### **Constants and Function Declaration**
```c
#define MAX_COMMAND_LENGTH 100 // Maximum length of a command
```
- Defines the maximum length of a command that the user can input (100 characters).
  
```c
void execute_command(char *command);
```
- Declares a function to execute a shell command.

---

### **Function to Execute Commands**
```c
void execute_command(char *command) {
    char *args[MAX_COMMAND_LENGTH];
    int i = 0;

    // Split the command into arguments using spaces
    char *token = strtok(command, " ");
    while (token != NULL) {
        args[i++] = token;
        token = strtok(NULL, " ");
    }
    args[i] = NULL; // Mark the end of the arguments list
```
1. **`args[MAX_COMMAND_LENGTH]`**: Array to store command arguments (e.g., `ls -l` becomes `["ls", "-l", NULL]`).
2. **`strtok`**: Splits the `command` string into tokens (substrings separated by spaces).
3. **`args[i++] = token;`**: Stores each token into the `args` array.

---

```c
    pid_t pid = fork();
```
- **`fork()`**: Creates a new process. The parent process continues, and a child process starts.

```c
    if (pid == 0) {
        execvp(args[0], args);
        perror("Error");
        exit(EXIT_FAILURE);
    }
    else if (pid > 0) {
        wait(NULL);
    }
    else {
        perror("Error");
        exit(EXIT_FAILURE);
    }
```
1. **Child Process (`pid == 0`)**:
   - **`execvp`**: Executes the command stored in `args`. It replaces the child process with the new program.
   - **`perror`**: Prints an error if `execvp` fails.
   - **`exit(EXIT_FAILURE)`**: Exits the child process if an error occurs.

2. **Parent Process (`pid > 0`)**:
   - **`wait(NULL)`**: Waits for the child process to finish before continuing.

3. **Fork Error (`pid < 0`)**:
   - If `fork()` fails, prints an error and exits.

---

### **Main Program**
```c
int main() {
    while (1) {
        char command[MAX_COMMAND_LENGTH];
        printf("SimpleShell> ");
        fgets(command, sizeof(command), stdin);

        // Remove the newline character
        command[strcspn(command, "
")] = '
