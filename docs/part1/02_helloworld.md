# 1.2 Hello World!

!!! info "Learning objectives"

    1. Recall bash commands to manipulate strings (`echo`, `ls`, `cat`)
    2. Understand how output redirection works (`>`)

A **Hello, World!** is a minimalist example that is meant to demonstrate the basic syntax and structure of a programming language or software framework. The example typically consists of printing the phrase 'Hello World!' to the output, such as the console or terminal, or writing it to a file.

Let's demonstrate this with simple commands that you can run directly in the terminal.

## 1.2.1 Printing a string

The **`echo`** command in Linux is a built-in command that allows users to display lines of text or strings that are passed as arguments. It is commonly used in shell scripts and batch files to output status text to the screen or a file.

The most straightforward usage of the `echo` command is to display text or a string on the terminal. To do this, you simply provide the desired text or string as an argument to the `echo` command:

```bash
echo <string>
```

!!!question "Exercise"

    Use the `echo` command to print the string `'Hello World!'` to the terminal.

    ??? Solution

        ```bash
        echo 'Hello World!'
        ```

        ``` title="Output"
        Hello World!
        ```

## 1.2.2 Redirecting outputs

The output of the `echo` can be redirected to a file instead of displaying it on the terminal. You can achieve this by using the **`>`** operator for output redirection.

!!!question "Exercise"

    Use the `>` operator to redirect the output of echo to a file named `output.txt`.

    ??? "Solution"

        ```bash
        echo 'Hello World!' > output.txt
        ```

        ``` title="Output"

        ```

        Instead of printing the output to the terminal, this will write the output of the echo command to the file name `output.txt`.

## 1.2.3 Listing files

The Linux shell command **`ls`** lists directory contents of files and directories. It provides valuable information about files, directories, and their attributes.

`ls` will display the contents of the current directory:

```bash
ls
```

!!!question "Exercise"

    List the files in the working directory to verify `output.txt` was created.

    ??? Solution

        ```bash
        ls
        ```

        A file named `output.txt` should now be listed in your current directory.

        ``` title="Output"
        output.txt
        ```

## 1.2.4 Viewing file contents

The **`cat`** command in Linux is a versatile companion for various file-related operations, allowing users to view, concatenate, create, copy, merge, and manipulate file contents.

The most basic use of `cat` is to display the contents of a file on the terminal. This can be achieved by simply providing the filename as an argument:

```bash
cat <file name>
```

!!!question "Exercise"

    Use the `cat` command to print the contents of `output.txt`.

    ??? "Solution"

        ```bash
        cat output.txt
        ```

        You should see `Hello World!` printed to your terminal.

        ``` title="Output"
        Hello World!
        ```

In preparation for the following lessons, delete `output.txt`.

```bash
rm output.txt
```

!!! abstract "Summary"

    This lesson recalls basic bash commands using a classic "Hello World!" example, including:

    1. Printing to the terminal with `echo`
    2. Redirecting output to a file with `>`
    3. Viewing files and file contents with `ls` and `cat`
