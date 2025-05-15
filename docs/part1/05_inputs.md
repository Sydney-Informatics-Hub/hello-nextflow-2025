# Inputs and Channels

!!! info "Learning objectives"

    1. Describe Nextflow channel types
    2. Utlizie Nextflow process input blocks
    3. Use channels to run multiple inputs through a process
    4. Use operators to manipulate data within a channel

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

## Running processes on multiple inputs

Now that we have a channel set up and our process has been reworked to use it, we can very easily start feeding more inputs into the channel and watch `SAYHELLO` run on each one.

The `Channel.of()` factory can take any number of values, separated by commas. Each one will become a separate element in the queue channel. Any process consuming that channel will run once for every element; each run will be separate and in parallel to the rest.

!!!question "Exercise"

    Add additional greetings to `greeting_ch`.

    ???Solution

        ```groovy title="hello-world.nf" hl_lines="7"
        workflow {

            // Create a channel for inputs
            greeting_ch = Channel.of('Hello world!', 'Bonjour le monde!', 'Holà mundo')

            // Emit a greeting
            SAYHELLO(greeting_ch)
        }
        ```

If you now run the workflow again, you should see that `SAYHELLO` runs three times:

```
Launching `main.nf` [curious_hugle] DSL2 - revision: 243f7816c2

executor >  local (3)
[27/ed09aa] SAYHELLO (1) [100%] 3 of 3 ✔
```

Notice that by default Nextflow only prints out one line per process rather than one line *per run* of each process. Sometimes it may be useful to you to have it print out each run on a separate line. To do this, you can add the `-ansi-log false` flag to the command line:

```bash
nextflow run hello-world.nf -ansi-log false
```

The output now looks like:

```
Launching `main.nf` [deadly_wilson] DSL2 - revision: 243f7816c2
[f2/84d334] Submitted process > SAYHELLO (2)
[f4/9f72e1] Submitted process > SAYHELLO (1)
[dc/52fa3d] Submitted process > SAYHELLO (3)
```

## Manipulating data within channels using operators

Currently, our channel simply holds an arbitrary number of values and passes them to separate runs of `SAYHELLO`. How boring! To allow for more flexible and dynamic pipelines, Nextflow defines several **channel operators** - methods that operate on and manipulate the data within a channel. There are a number of operators that Nextflow supports, [which are documented here](https://www.nextflow.io/docs/latest/reference/operator.html). Their functions range from counting the number of items in a channel, to processing text files, to merging multiple channels together.

In this section, we will explore a few basic operators for manipulating list of values, as well as the versatile `map` operator that can run an arbitrary function on each element of a channel.

### Passing an array of values to our channel

Defining our `greeting_ch` channel using the `Channel.of` factory and hard-coded values is quite contrived and inflexible. In real-world situations you will more likely have an array of values to work with, defined at run time. In this step, we will move our list of greetings to an array called `greetings_array`, which will then be passed to our channel. Note that we will still be working with hard-coded values for simplicity, but by learning to maninpulate arrays of data within a channel, you will get a better feel for how real-word Nextflow pipelines are written.

!!!question "Exercise"

    Define an array called `greetings_array` containing our greetings and pass it to `Channel.of` when constructing the `greeting_ch` channel.

    ???Solution

        Before:

        ```groovy title="hello-world.nf" hl_lines="7"
        workflow {

            // Create a channel for inputs
            greeting_ch = Channel.of('Hello world!', 'Bonjour le monde!', 'Holà mundo')

            // Emit a greeting
            SAYHELLO(greeting_ch)
        }
        ```

        After:

        ```groovy title="hello-world.nf" hl_lines="7"
        workflow {

            // Create a channel for inputs
            greetings_array = [ 'Hello world!', 'Bonjour le monde!', 'Holà mundo' ]
            greeting_ch = Channel.of(greetings_array)

            // Emit a greeting
            SAYHELLO(greeting_ch)
        }
        ```

### `flatten`ing our input array

The change we just made actually breaks our workflow! When you pass an array to the channel factory `Channel.of`, the entire array is treated as *a single element*. Instead, we want to split up the array and emit each element as a separate value in the queue channel. Nextflow has an operator for just this very thing: `flatten()`.

```groovy
arr = [ 1, 2, 3 ]
ch = Channel.of(arr)
ch.view()
```

```
[1, 2, 3]
```

```groovy
ch.flatten().view()
```

```
1
2
3
```

Note that we also snuck in a second operator in the above examples: `view()`. This, like its name suggests, lets you view the contents of a channel by printing it out to the terminal. It can be very useful when prototyping new workflows to understand how your data is structured within your channels.

!!!question "Exercise"

    Flatten `greeting_ch` to emit each element of the array separately. 

    ???Solution

        Before:

        ```groovy title="hello-world.nf" hl_lines="7"
        workflow {

            // Create a channel for inputs
            greetings_array = [ 'Hello world!', 'Bonjour le monde!', 'Holà mundo' ]
            greeting_ch = Channel.of(greetings_array)

            // Emit a greeting
            SAYHELLO(greeting_ch)
        }
        ```

        After:

        ```groovy title="hello-world.nf" hl_lines="7"
        workflow {

            // Create a channel for inputs
            greetings_array = [ 'Hello world!', 'Bonjour le monde!', 'Holà mundo' ]
            greeting_ch = Channel.of(greetings_array).flatten()

            // Emit a greeting
            SAYHELLO(greeting_ch)
        }
        ```

Now our pipeline works again, with `SAYHELLO` running three times, once for each value in `greetings_array`.

### Transforming each value with the `map` operator

Sometimes, you will need to modify the values within a channel in a predictable way. For example, you might need to get part of a file name, or perhaps you need to take a numeric value and apply a mathematical operation to it. In these situations, you will likely want to use the `map()` operator. This takes a channel and applies a **closure** to every element in the channel.

An in-depth discussion of closures is outside of the scope of this workshop, but briefly, they are blocks of code, similar to functions, that can be passed to some Nextflow operators and control how they function. With the `map()` operator, the closure defines the exact operation that should be performed on each element of the channel. A closlure is defined within curly braces like so:

```groovy
{ x ->
    // Do something with x
}
```

At the start of the closure definition, we declare the name of a variable that will represent each element of our channel. Here we have simply called it `x`, but it could be named anything. Next, we write an arrow operator `->`; this signifies that the value on the left (`x`) will be processed by the code on the right. Finally, we write some code to do something with our variable. For example, we can square integer values:

```groovy
{ x -> x ** 2 }
```

Or, if we have strings as inputs, we could reverse the strings:

```groovy
{ x -> x.reverse() }
```

To use the `map()` operator with a channel, we simply do:

```groovy
ch.map { x -> ... }
```

!!!question "Exercise"

    Use the `map()` operator to reverse the greeting strings. 

    ???Solution

        ```groovy title="hello-world.nf" hl_lines="7"
        workflow {

            // Create a channel for inputs
            greetings_array = [ 'Hello world!', 'Bonjour le monde!', 'Holà mundo' ]
            greeting_ch = Channel.of(greetings_array)
                .flatten()
                .map { x -> x.reverse() }

            // Emit a greeting
            SAYHELLO(greeting_ch)
        }
        ```

## FAQ: Can I use the `publishDir` as an input to a process?

A common question about Nextflow is whether the `publishDir` can be used as an input to processes. This can sometimes seem like an attractive and useful pattern. For example, you may have several processes generating outputs that need to get collated or summarised in some way at the end of your workflow. If each process puts its final output in the `publishDir` directory, then the final summary process can simply look there for all its inputs.

Unfortunately, this is not good practice, and will very likely either lead to inconsistent results or not work at all. `publishDir` is not really part of the workflow itself; it's just a convenient place to put important outputs of the workflow. There are no guarantees about when output files will get copied into `publishDir`, and Nextflow won't know to wait for files to be generated inside before running processes that rely on them.

In the above scenario of a summary process gathering up the outputs of several previous jobs, you should instead be using some combination of the `mix()` and `collect()` operators; `mix()` will combine multiple channels into a single channel, while `collect()` will bring all of the elements in a channel into a single array that can be passed to a single instance of a process.

Ultiately, all processes should be working with channels as their inputs, and `publishDir` should only be used as a final output directory.

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
    3. How to use operators to manipulate the data within channels
