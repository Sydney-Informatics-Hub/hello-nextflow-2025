# Adding new processes

!!! info "Learning objectives"

    1. Transform text strings 
    2. Chain Nextflow processes together
    3. Utilize process outputs as inputs

Up until now you've been modifying a single step. However, pipelines generally consist of multiple steps where outputs from one step are used as inputs for the next.

Here you're going to step things up again and add another process to the pipeline.

## Translating text

The `tr` command is a UNIX command-line utility for **translating** or deleting characters. It supports a range of transformations including uppercase to lowercase, squeezing repeating characters, deleting specific characters, and basic find and replace. It can be used with UNIX pipes to support more complex translation. `tr` stands for translate. The following example will translate all lower case letters (represented by the pattern `[a-z]`) to upper case (represented by `[A-Z]`):

```bash
tr '[a-z]' '[A-Z]'`
```

!!! note

    For the curious, the patterns we are using here - e.g. `[a-z]` - are called *regular expressions*. They are a way of describing patterns in text and can be immensely useful in manipulating text as they provide a way to search and replace text in more complex ways than simple exact matches. Be warned, they can get very complicated and confusing very quickly!

## Piping commands

The pipe command in Linux, represented by the vertical bar symbol `|`, is an essential tool for command-line enthusiasts and professionals alike. The primary purpose of the pipe command is to connect the output of one command directly into the input of another:

```bash
echo 'Hello World' | tr '[a-z]' '[A-Z]'
```

The contents of a file can be piped using the `cat` command:

```bash
cat output.txt | tr '[a-z]' '[A-Z]'
```

Like before, the output can be redirected to an output file:

```bash
cat output.txt | tr '[a-z]' '[A-Z]' > upper.txt
```

## Adding the `CONVERTTOUPPER` process

The output of the `SAYHELLO` process is a text file called `output.txt`.

In the next step of the pipeline, you will add a new process named `CONVERTTOUPPER` that will convert all of the lower case letters in this file to a uppercase letters and save them as a new file.

The `CONVERTTOUPPER` process will follow the same structure as the `SAYHELLO` process:

```groovy
process CONVERTTOUPPER {
    publishDir params.outdir

    input:
    <input qualifier> <input name>

    output:
    <output qualifier> <output name>

    script:
    """
    <script>
    """
}
```

Using what you have learned in the previous sections you will now write a new process using the `tr` command from above.

!!!question "Exercise"

    Add new process named `CONVERTTOUPPER` that will take an input text file, convert all of the lowercase letters in the text file to uppercase letters, and save a new text file that contains the translated letters.

    ???Tip "Hint: `input:`"

        ```
        path input_file
        ```

        _Hint 1: The input is a file and requires the `path` qualifier._

        _Hint 2: The input name is `input_file`, however, you may call it something different._

    ???Tip "Hint: `output:`"

        The output

        ```
        path 'upper_output.txt'
        ```

        _Hint 1: The output is a file and requires the `path` qualifier._

        _Hint 2: The output name is hard coded as 'upper.txt', however, you may call it something different._

    ???Tip "Hint: `script:`"

        The script might look something like this:

        ```groovy
        cat $input_file | tr '[a-z]' '[A-Z]' > upper.txt
        ```

        _Hint 1: `input_file` must be the same as what was specified as the input name in the input block._

        _Hint 2: The output text file is named `upper.txt`_

    ???Solution

        ```groovy title="hello-world.nf" hl_lines="23-36"
        // Set default greeting
        params.greeting = 'Hello World!'

        // Set a default output directory
        params.outdir = 'results'

        // Use echo to print 'Hello World!' and redirect to output.txt
        process SAYHELLO {
            publishDir params.outdir

            input:
            val greeting

            output:
            path 'output.txt'

            script:
            """
            echo '$greeting' > output.txt
            """
        }

        process CONVERTTOUPPER {
            publishDir params.outdir

            input:
                path input_file

            output:
                path 'upper.txt'

            script:
            """
            cat $input_file | tr '[a-z]' '[A-Z]' > upper.txt
            """
        }

        workflow {

            // Create a channel for inputs
            greeting_ch = Channel.of(params.greeting)

            // Emit a greeting
            SAYHELLO(greeting_ch)

        }
        ```

## Connecting the processes

As we learned in the [inputs module](05_inputs.md), Nextflow uses channels to connect processes. Each output defined in a process' `output` block defines a new channel that can be used as inputs for another process.

The output channel from a process can be accessed by adding `.out` to the end of a process name in the workflow definition:

```groovy
SAYHELLO.out
```

Outputs can then be used as an input for another process:

```groovy
CONVERTTOUPPER(SAYHELLO.out)
```

Alternatively, you can assign the output channel to a new variable name for convenience:

```groovy
hello_msg = SAYHELLO.out

CONVERTTOUPPER(hello_msg)
```

The process output behaves like any other channel and can be used as inputs for multiple downstream processes.

!!!warning

    Adding `.out` to the end of a process name only works for single outputs. If there are multiple outputs, you will need to use an integer index to select the appropriate output (e.g. `.out[0]` or `.out[1]` for the first and second inputs, respectively), or (more conveniently) use the `emit` option when defining the `output` block of the process, which allows you to select the output by name (e.g. `.out.some_output`). See the [additional options](https://www.nextflow.io/docs/latest/process.html#additional-options) section of the Nextflow documentation for more information.

!!!question "Exercise"

    Add the `CONVERTTOUPPER` process to your workflow definition. Use the output from `SAYHELLO` as its input.

    ???solution

        ```groovy title="hello-world.nf" hl_lines="46-47"
        // Set default greeting
        params.greeting = 'Hello World!'

        // Set a default output directory
        params.outdir = 'results'

        // Use echo to print 'Hello World!' and redirect to output.txt
        process SAYHELLO {
            publishDir params.outdir

            input:
            val greeting

            output:
            path 'output.txt'

            script:
            """
            echo '$greeting' > output.txt
            """
        }

        process CONVERTTOUPPER {
            publishDir params.outdir

            input:
                path input_file

            output:
                path 'upper.txt'

            script:
            """
            cat $input_file | tr '[a-z]' '[A-Z]' > upper.txt
            """
        }

        workflow {

            // Create a channel for inputs
            greeting_ch = Channel.of(params.greeting)

            // Emit a greeting
            SAYHELLO(greeting_ch)

            // Convert the greeting to uppercase
            CONVERTTOUPPER(SAYHELLO.out)

        }
        ```

Executing `hello-world.nf` will now show a second step:

```console
N E X T F L O W  ~  version 23.10.1
Launching `hello-world.nf` [mighty_murdock] DSL2 - revision: 80e92a677c
executor >  local (2)
[ef/b99a2f] SAYHELLO (1)       [100%] 1 of 1 ✔
[cd/c8cf1b] CONVERTTOUPPER (1) [100%] 1 of 1 ✔
```

!!! abstract "Summary"

    In this step you have learned:

    1. How to translate strings
    2. How add more processes to a script
    3. How to use outputs and inputs
