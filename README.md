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
")] = ' ';
```
1. **Infinite Loop**: Runs the shell continuously until the user exits.
2. **Prompt**: Displays `SimpleShell>` for user input.
3. **`fgets`**: Reads a line of input from the user.
4. **Remove Newline**: Replaces the newline character (`
`) at the end of the input with a null terminator (` `).

---

### **Exit Condition**
```c
        if (strcmp(command, "exit") == 0) {
            break;
        }
```
- If the user types `exit`, the shell terminates.

---

### **Handling Special Characters**
```c
        char *output_redirect = strchr(command, '>');
        char *input_redirect = strchr(command, '<');
        char *pipe_operator = strchr(command, '|');
```
- **`strchr`**: Checks if the command contains:
  - **`>`**: Output redirection.
  - **`<`**: Input redirection.
  - **`|`**: Piping.

---

#### **Output Redirection (`>`)**
```c
        if (output_redirect != NULL) {
            *output_redirect = '\0';
            char *output_file = strtok(output_redirect + 1, " ");
            freopen(output_file, "w", stdout);
            execute_command(command);
            fclose(stdout);
        }
```
1. Splits the command into two parts: command and output file.
2. **`freopen`**: Redirects `stdout` to the specified file.
3. Executes the command, saving the output to the file.

---

#### **Input Redirection (`<`)**
```c
        else if (input_redirect != NULL) {
            *input_redirect = '\0';
            char *input_file = strtok(input_redirect + 1, " ");
            freopen(input_file, "r", stdin);
            execute_command(command);
            fclose(stdin);
        }
```
1. Splits the command into two parts: command and input file.
2. **`freopen`**: Redirects `stdin` to read from the specified file.
3. Executes the command using the file as input.

---

#### **Piping (`|`)**
```c
        else if (pipe_operator != NULL) {
            *pipe_operator = '\0';
            char *first_command = command;
            char *second_command = pipe_operator + 1;

            int pipefd[2];
            pipe(pipefd);
```
1. Splits the command into two parts: before and after the pipe (`|`).
2. **`pipe(pipefd)`**: Creates a pipe with two file descriptors:
   - `pipefd[0]`: Read end.
   - `pipefd[1]`: Write end.

---

##### **First Command**
```c
            pid_t pid = fork();
            if (pid == 0) {
                close(pipefd[0]);
                dup2(pipefd[1], STDOUT_FILENO);
                close(pipefd[1]);
                execute_command(first_command);
                exit(EXIT_SUCCESS);
            }
```
1. Creates a child process to execute the first command.
2. Redirects `stdout` to the pipe’s write end (`pipefd[1]`).

---

##### **Second Command**
```c
            else if (pid > 0) {
                wait(NULL);
                close(pipefd[1]);
                dup2(pipefd[0], STDIN_FILENO);
                close(pipefd[0]);
                execute_command(second_command);
            }
```
1. Parent process waits for the child to finish.
2. Redirects `stdin` to the pipe’s read end (`pipefd[0]`).
3. Executes the second command.

---

#### **Default Command Execution**
```c
        else {
            execute_command(command);
        }
```
- Executes the command directly if no special characters are present.

---

### **Program Exit**
```c
    return 0;
}
```
- Ends the program when the shell is terminated.

---

Let me know if you need clarification on any specific part!
