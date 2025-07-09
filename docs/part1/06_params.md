# 1.6 Parameters

!!! info "Learning objectives"

    1. Understand how parameters enable flexible and configurable pipelines
    2. Apply command-line flags to define parameters during execution
    3. Modify pipeline behaviour dynamically using `params` within process and workflow scopes
    4. Evaluate when to use parameters for outputs to support reproducibility and avoid overwrites

Parameters are special values that can be set from command line arguments and therefore allow you to write flexible and dynamic pipelines. Parameters in Nextflow are comparable to "variables" in other programming languages (e.g. R, Python).

Here you're going to update the script with parameters to make it more flexible.

## 1.6.1 Why are parameters useful?

Parameters are useful because they can be set with a convenient default value in a script but can then be overwritten at runtime using a flag. Simply, parameters allow us to configure some aspect of a pipeline without editing the script itself.

Nextflow has multiple levels of configuration and, as different levels may have conflicting settings, they are ranked in order of priority and some configuration can be overridden.

Parameters can be created within the top level of your Nextflow script (i.e. outside of the `workflow` and `process` definitions) by prefixing a parameter name with the `params` scope (e.g. `params.greeting`). They are globally accessible by both processes and workflows anywhere in your workflow. They can be modified when you run your pipeline by adding a double hyphen (`--`) to the start of the parameter name (e.g. `--greeting`) and adding it to an execution command:

```bash
nextflow run hello-world.nf --greeting 'Hello World!'
```

## 1.6.2 Updating your workflow with the `--greeting` parameter

Instead of hard coding 'Hello World!' as an input, a parameter, with a default value, can be created:

```groovy
params.greeting = 'Hello World!'
```

Parameters must begin with `params.`, and the name the follows directly can be of your choosing.
In this case, the name is `greeting`.

The parameter can then be used in a channel factory (just like the hard coded string):

```groovy
greeting_ch = Channel.of(params.greeting)
```

The parameter can then be flexibly changed using a `--greeting` flag in the run command:

```bash
nextflow run hello-world.nf --greeting 'Bonjour le monde!'
```

!!! warning

    When setting parameters from the command line, it is vital that you wrap multi-word strings (or anything else containing spaces or special characters) within single or double quotes as shown above. If you don't, only the first word will be captured in the parameter. Anything after a space will be considered to be a new argument to Nextflow and behave unexpectedly.

!!!question "Exercise"

    Update the `hello-world.nf` script to use a `greeting` parameter as an input. Define the default for the `greeting` parameter at the top of the script and give it the default value `'Hello World!'`.

    ???Solution

        ```groovy title="hello-world.nf" hl_lines="1 2 23"
        // Set default greeting
        params.greeting = 'Hello World!'

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
            greeting_ch = Channel.of(params.greeting)

            // Emit a greeting
            SAYHELLO(greeting_ch)

        }
        ```

The `hello-world.nf` pipeline can now be executed with the `--greeting` flag and a custom greeting:

```bash
nextflow run hello-world.nf --greeting 'Bonjour le monde!'
```

## 1.6.3 Setting a dynamic publishing directory with `--outdir`

Hardcoding a single output directory in your pipeline is often inconvenient -
each new run can overwrite the previous results. By using a parameter for the `publishDir`
process directive, it makes our pipeline more configurable, and results more reproducible:

```groovy
publishDir params.outdir
```

To ensure this works even if a parameter is not provided, we can set default values
for `publishDir`:

```groovy
params.outdir = 'results'
```

However, you may consider having no default values for parameters and letting the pipeline fail to prevent the accidental overwriting of results.

This can be especially useful when you want to:

- Keep outputs from each run separate
- Name output folders based on the input data or command being run
- You are iteratively testing a pipeline and want a clear record of each run

!!!question "Exercises"

    1. Update the `hello-world.nf` script to use an `outdir` parameter as the publishing directory. Define the default for the `outdir` parameter at the top of the script and give it the default value `'results'`.

    ???Solution

        ```groovy title="hello-world.nf" hl_lines="4 5 9"
        // Set default greeting
        params.greeting = 'Hello World!'

        // Set default output directory
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

        workflow {

            // Create a channel for inputs
            greeting_ch = Channel.of(params.greeting)

            // Emit a greeting
            SAYHELLO(greeting_ch)

        }
        ```

    <ol start="2">
        <li>Re-run the pipeline with `nextflow run hello-world.nf`. Include different values to your new `--greeting` and `--outdir` parameters and inspect the outputs. What happens if you do not include any flags? i.e. `nextflow run hello-world.nf`</li>
    </ol>

    ???Solution

        Running without any flags will use the default parameters specified in `hello-world.nf`:
        ```bash
        nextflow run hello-world.nf
        ```

        You can pass any string into either parameter. In this case, it will redirect 'Good day, Earth!' into a file `output.txt`, and publish that file into a folder called `gday_earth`:
        ```bash
        nextflow run hello-world.nf --greeting 'Good day, Earth!' --outdir 'gday_earth'
        ```

        To illustrate how we can clearly name and separate runs using parameters:
        ```bash
        nextflow run hello-world.nf --greeting 'Bonjour le monde' --outdir 'bonjour'
        ```


        Don't forget to wrap strings with spaces or special characters within single or double quotes!

!!! abstract "Summary"

    In this lesson, we updated our pipelines to support flexible execution using parameters:

    1. Declared global parameter values for the greeing message and output directory
    2. Used parameterised inputs with `Channel.of(params.greeting)`
    3. Applied the `--greeting` and `--outdir` flags to override defaults at runtime
    4. Replaced hardcoded values with configurable options
    5. Recognised the benefits of configuring output names for reproducibility

