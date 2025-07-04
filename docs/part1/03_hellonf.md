# Writing your first pipeline

!!! info "Learning objectives"

    1. Write your first Nextflow pipeline
    2. Understand the main components of a Nextflow script
    3. Understand the components of a Nextflow process

Workflow languages are better than Bash scripts because they handle errors and run tasks in parallel more easily, which is important for complex jobs. They also have clearer structure, making it easier to maintain and work on with others.

Here, you're going learn more about the Nextflow language and take your first steps making **your first pipeline** with Nextflow.

## Understanding the `process` and `workflow` scopes

Nextflow pipelines are written inside `.nf` files. They consist of a combination of two main components: **processes** and the **workflow** itself. Each process describes a single step of the pipeline, including its inputs and expected outputs, as well as the code to run it. The workflow then defines the logic that puts all of the processes together.

```groovy
process < name > {
  [ directives ]

  input:
    < process inputs >

  output:
    < process outputs >

  script:
  """
  <script to be executed>
  """
}

workflow {
    < processes to be executed >
}
```

A process definition starts with the keyword `process`, followed by a process name, and finally the process body delimited by curly braces. The process body must contain a `script` block which represents the command or, more generally, a script that is executed by it.

A process may contain any of the following definition blocks. The ones we will be focusing on this workshop are presented in bold: **`directives`**, **`input`**, **`output`**, `stub`, `when` clauses, and of course, **`script`**.

A workflow is a composition of processes and dataflow logic.

The workflow definition starts with the keyword `workflow`, followed by an optional name, and finally the workflow body delimited by curly braces.

!!! info "Tip"

    The `process` and `workflow` definitions are analogous to functions in R or
    Python languages. You first have to define a function (the `process`) that
    contains the instructions of what to do. In order to do the action, the
    function needs to be called (in the `workflow`).

Let's review the structure of `hello-world.nf`, a toy example you will be developing and executing in Part 1.

```groovy title="hello-world.nf" linenums="1"
process SAYHELLO {

    output:
    path 'output.txt'

    script:
    """
    echo 'Hello World!' > output.txt
    """
}

workflow {
    SAYHELLO()
}
```

The first piece of code (lines 1-11) describes a **process** called `SAYHELLO` with two definition blocks:

- **output**: defines that the process will output a file called `output.txt`. It also contains the `path` qualifier. We will review this in the next section.
- **script**: the `echo 'Hello World!'` command redirects to a file called `output.txt`

The second block of code (12-14) lines describes the **workflow** itself, which consists of one call to the `SAYHELLO` process.

We will start building the `hello-world.nf` script, piece-by-piece, so you can get a feel
for the requirements for a writing a Nextflow workflow.


!!! question "Exercises"

    1. Create a new file `hello-world.nf`.

    !!! info "Tip"
        
        We can use the `code` command in the terminal to view files in a VSCode tab. If it doesn't exist, it will create that file. The following command will create a new file called `hello-world.nf`:
        ```bash
        code hello-world.nf
        ```

    2. In the new file, define an empty `process` and call it `SAYHELLO`.
    
    ??? "Solution"
        ```groovy title="hello-world.nf"
        process SAYHELLO {
        
        }
        ```

    <ol start="3">
        <li>Add the `script` definition that writes 'Hello World!' to a file called `output.txt`.</li>
    </ol>

    ??? "Solution"
        ```groovy title="hello-world.nf" hl_lines="3-6"
        process SAYHELLO {
        
            script:
            """
            echo 'Hello World!' > output.txt
            """
        }
        ```

To complete this process, we need to add an `output` definition.

## Capturing process outputs 

In the previous section, you have defined the `script` - what the `SAYHELLO` process should do.
We also need to tell Nextflow to expect this output file - otherwise, it will ignore it!

We declare outputs using the `output` definition block. Typically this will require
both an **output qualifier** and an **output name**:

```groovy
output:
<output qualifier> <output name>
```

Common output qualifiers include `val` and `path`:

- `val`: Emit the variable with the specified name
    - For example, `val 'Hello World!'`
- `path`: Emit a file produced by the process with the specified name
    - For example, `path 'output.txt'`

See the [Nextflow documentation](https://www.nextflow.io/docs/latest/process.html#outputs)
for a full list of output qualifiers.

!!! warning

    If you set the wrong qualifier, the pipeline will likely throw errors.

The **output name** is a name given to the output variable. The output name and the file generated by the script must exactly match (or be
picked up by a glob pattern), or else Nextflow won't find it and will throw an error.

!!! note

    It is important to understand that the `output` block does not *determine* the output of the process. Instead, it simply *declares* what output should be expected. It is up to the logic inside the `script` block to ensure that the file is actually being created.

    This is particularly useful for a number of reasons:
    
    - It allows Nextflow to determine whether our process ran successfully or not. If an output is declared but missing at the end of a process, Nextflow will assume it failed.
    - It uses the outputs we declare to figure out how and when to run the next process.

!!!question "Exercise"

    In your `SAYHELLO` process, add an `output` block that captures `'output.txt'`.
    Since it is a file being emitted by the process, the `path` qualifier must be
    used.

    ??? "Solution"
        ```groovy title="hello-world.nf" hl_lines="3-5"
        process SAYHELLO {

            output:
            path 'output.txt'
        
            script:
            """
            echo 'Hello World!' > output.txt
            """
        }
        ```

This example is brittle because the output filename is hardcoded in two separate places
(the `script` and the `output` definition blocks). If you change one but not the other, the script will break.

**You have now defined your first process!**

## Commenting your code

It is worthwhile to **comment** your code so we, and others, can easily understand what the code is doing (you will thank yourself later).

In Nextflow, a single line comment can be added by prepending it with two forward slash (`//`):

```groovy
// This is my comment
```

Similarly, multi-line comments can be added using the following format:

```groovy
/*
 * Use echo to print 'Hello World!' to standard out
 */
```

As a developer you can to choose how and where to comment your code.

!!!question "Exercise"

    Add a comment to the pipeline to describe what the **process** block is doing

    ??? "Solution"

        The solution may look something like this:

        ```groovy title="hello-world.nf" hl_lines="1-3"
        /*
         * Use echo to print 'Hello World!' to a text file
         */
        process SAYHELLO {
        <truncated>
        ```

        Or this:

        ```groovy title="hello-world.nf" hl_lines="1"
        // Use echo to print 'Hello World!' to a text file
        process SAYHELLO {
        <truncated>
        ```

        As a developer, you get to choose!

## Calling the process in the `workflow` scope

We have now defined a functional process for `SAYHELLO`. To ensure that it runs,
you need to call the process in the `workflow` scope. Recall that the process
is analagous to a function that we need to instruct to run in `workflow`.

!!! question "Exercise"

    1. At the bottom of your `main.nf` script, after the `SAYHELLO` process,
    define an empty `workflow` scope:

    ??? "Solution"
        ```groovy title="hello-world.nf" hl_lines="11-14"
        // Use echo to print 'Hello World!' and redirect to output.txt
        process SAYHELLO {

            output:
            path 'output.txt'
        
            script:
            """
            echo 'Hello World!' > output.txt
            """
        }

        workflow {

        }
        ```

    <ol start="2">
        <li>Add the process call by adding <code>SAYHELLO()</code> inside `workflow {}`.</li>
    </ol>

    ??? "Solution"

        ```groovy title="hello-world.nf" hl_lines="14-15"
        // Use echo to print 'Hello World!' and redirect to output.txt
        process SAYHELLO {

            output:
            path 'output.txt'
        
            script:
            """
            echo 'Hello World!' > output.txt
            """
        }

        workflow {
            // Emit a greeting
            SAYHELLO()
        }
        ```

        Note, that we did not include any arguments to `SAYHELLO()`. This is because the process does
        not take any inputs. 

**Yay! You have just written your first pipeline!**

In the next step, we will run the pipeline and inspect the outputs of
our workflow.

!!! abstract "Summary"

    In this step you have learned:

    1. How to create a simple Nextflow pipeline
    2. The key components of a Nextflow script (`process` and `workflow`)
    3. How to define a process with `script` and `output` blocks
    4. How to comment your code
    5. How to call processes in the workflow scope
