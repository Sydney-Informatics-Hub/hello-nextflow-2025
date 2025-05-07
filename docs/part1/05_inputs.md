# Inputs and Channels

!!! info "Learning objectives"

    1. Describe Nextflow channel types
    2. Utlizie Nextflow process input blocks

So far, you've been emitting a greeting ('Hello World!') that has been hardcoded into the script block. In a more realistic situation, you might want to pass a variable input to your script, much like you pass files to command line tools for analysis.

Here you're going to to add some flexibility by introducing **channels** to your workflow and an **input definition** to your `SAYHELLO` process.

## Channels

In Nextflow, processes primarily communicate through **channels**. Channels are essentially the 'pipes' of our pipeline, providing a way for data to flow between our processes and defining the overall strucutre of the workflow.

Channels come in two flavours:

### Value channels

Value channels, as their name suggests, simply store a value. Importantly:

- Value channels can be bound (i.e. assigned) with one and only one value.
- They be consumed any number of times.

When a value channel is passed as an input to a process, its value will be used for every run of that process. Typically, you will encounter value channels when using channel operators (discussed later) like `first` and `collect`, which take **queue channels** (discussed below) and output single values.

### Queue channels

Queue channels are the more common type of channel that hold a series of values in a first-in, first-out queue structure. This has the advantage of supporting the parallel nature of Nextflow; when a new value becomes available, a process that consumes that channel can be run. However, they can sometimes be tricky to work with because this first-in, first-out nature makes them **non-deterministic**; that is, you won't know ahead of time the order of values within the queue. For example, for a channel of file paths created by a process, the order of the queue will depend on the order in which the jobs finished.

The important take-aways about queue channels are:

- They are first-in, first-out queues of values.
- Each value can be consumed only once by a given process or operator.
- Their order is non-deterministic.

## Creating channels

Channels are created in one of two ways. The first is as outputs of processes. Each entry in the `output` block of a process creates a separate channel that can be accessed with `<process_name>.out` - or, in the case of named outputs, with `<process_name>.out.output_name`.

The other way to create channels is with special functions called **channel factories**. There are numerous types of channel factories which can be utilized for creating different channel types and data types. The most common channel factories you will use are `Channel.of()`, `Channel.fromPath()`, and `Channel.fromFilePairs()`. The latter two are faily self explanatory, creating channels of file paths and pairs of file paths, respectively. The `Channel.of()` factory is a much more generic method used to create a channel of whatever values are passed to it. For example, the following creates a channel called `ch_greeting` that contains two values - "Hello World!", and "Goodbye!":

```groovy
ch_greeting = channel.of('Hello World!', 'Goodbye!')
```

A process consuming this channel would run twice - once for each value.

## Adding channels to our pipeline

You're going to start by creating a channel with the `Channel.of()` channel factory that will contain your greeting.

!!!note

    You can build different kinds of channels depending on the shape of the input data.

Channels need to be created within the `workflow` definition.

!!!question "Exercise"

    Create a channel named `greeting_ch` with the 'Hello World!' greeting.

    ???Solution

        ```groovy title="hello-world.nf" hl_lines="2 3 4"
        workflow {

            // Create a channel for inputs
            greeting_ch = Channel.of('Hello world!')

            // Emit a greeting
            SAYHELLO()
        }
        ```

## Adding channels as process inputs

Before `greeting_ch` can be passed to the `SAYHELLO` process as an input, you must first add an **input block** in the process definition.

The inputs in the input block, much like the output block, must have a qualifier and a name:

```
<input qualifier> <input name>
```

Input names can be treated like a variable, and while the name is arbitrary, it should be recognizable.

No quote marks are needed for variable inputs. For example:

```
val greeting
```

Similar to the output qualifiers discussed in the previous chapter, there are several different input qualifiers, with some of the more common ones being:

- `val`: A value, such as a string or number.
- `path`: A file path.

!!!question "Exercise"

    Add an `input` block to the `SAYHELLO` process with an input value. Update the comment at the same time.

    ???Solution

        ```groovy title="hello-world.nf" hl_lines="1 4 5 6"
        // Use echo to print a string and redirect to output.txt
        process SAYHELLO {
            publishDir 'results'

            input:
            val greeting

            output:
            path 'output.txt'

            script:
            """
            echo 'Hello World!' > output.txt
            """
        }
        ```

The `SAYHELLO` process is now expecting an input value.

The `greeting_ch` channel can now be supplied to `SAYHELLO()` process within the workflow block:

```groovy
SAYHELLO(greeting_ch)
```

Without this, Nextflow will throw an error.

!!!question "Exercise"

    Add the `greeting_ch` as an input for the `SAYHELLO` process.

    ???Solution

        ```groovy title="hello-world.nf" hl_lines="7"
        workflow {

            // Create a channel for inputs
            greeting_ch = Channel.of('Hello world!')

            // Emit a greeting
            SAYHELLO(greeting_ch)
        }
        ```

### Using Nextflow variables within scripts

The final piece is to update the `script` block to use the `input` value.

Each input can be accessed as a variable via the name in its definition. Within the script block, this is done by prepending a `$` character to the input name:

```groovy
script:
"""
echo '$greeting' > output.txt
"""
```

The `'` quotes around `$greeting` are required by the `echo` command to treat the greeting as a single string.

!!!question "Exercise"

    Update `hello-world.nf` to use the greeting input.

    ???Solution

        ```groovy title="hello-world.nf" hl_lines="13"
        // Use echo to print 'Hello World!' and redirect to output.txt
        process SAYHELLO {
            publishDir 'results'

            input:
            val greeting

            output:
            path 'output.txt'

            script:
            """
            echo '$greeting' > output.txt
            """
        }

        workflow {

            // Create a channel for inputs
            greeting_ch = Channel.of('Hello world!')

            // Emit a greeting
            SAYHELLO(greeting_ch)
        }
        ```

**Yes! Your pipeline now uses an input channel!**

## A note about multiple inputs

The input block can be used to define multiple inputs to the process. Importantly, the number of inputs passed to the process call within the workflow must match the number of inputs defined in the process. For example:

```groovy title="example.nf"
process MYFUNCTION {
    debug true

    input:
    val input_1
    val input_2

    output:
    stdout

    script:
    """
    echo $input_1 $input_2
    """
}

workflow {
    MYFUNCTION('Hello', 'World!')
}
```

Another important aspect of multiple inputs is that when working with **queue channels**, they can result in **non-deterministic** results. This is because a process will execute as soon as a new value is ready for all input channels. For that reason, multiple inputs will typically be used with either value channel inputs (since they are single values that will be reused over and over again) or by using the `each` qualifier, which allows you to run the process once for every value in a collection or queue. For example:

```
process cat_message {
    input:
    val greeting
    each noun

    output:
    path "message.txt"

    script:
    """
    echo '$greeting' '$noun' > message.txt
    """
}

workflow {
    ch1 = Channel.of(
        'Hello',
        'Bonjour'
    )
    ch2 = Channel.of(
        'world',
        'everyone'
    )

    cat_message(ch1, ch2)
}
```

This will output the following four lines (possibly in a different order):

```
Bonjour everyone
Hello world
Hello everyone
Bonjour world
```

In contrast, if we didn't use the `each` qualifier with the `noun` input, we would only get two output lines, such as:

```
Hello everyone
Bonjour world
```

Because the queues are non-deterministic, the exact combination we would get is uncertain.

!!! abstract "Summary"

    In this step you have learned:

    1. How to use Channel factories
    2. How to how to add process inputs
