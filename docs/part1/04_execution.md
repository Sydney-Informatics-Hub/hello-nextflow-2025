# Running your first pipeline

!!! info "Learning objectives"
    1. Running a Nextflow pipeline
    2. Explore the outputs of a pipeline run
    2. Publish results files using with directives

In this step, we will run our `hello-world.nf` Nextflow pipeline and explore
the outputs of the run. We will look at the components that get printed
to the terminal when executing a workflow, 
how to interpret these, as well as the common log and output files of a run.
You will be introduced to your first process directive and best practices on
managing output files.

## Executing `hello-world.nf`

To run a Nextflow pipeline we use the **`nextflow run`** command, followed by the name of the script.

```bash
nextflow run <pipeline.nf>
```

Let's run the `.nf` script we just created - remember to save the file first!

!!!question "Exercise"

    Use the `nextflow run` command to execute `hello-world.nf`

    ???Solution

        ```bash
        nextflow run hello-world.nf
        ```

**Yay! You have just run your first pipeline!**

Your console should look something like this:

```console linenums="1"
N E X T F L O W  ~  version 24.10.2
Launching `hello-world.nf` [mighty_murdock] DSL2 - revision: 80e92a677c
executor >  local (1)
[4e/6ba912] process > SAYHELLO [100%] 1 of 1 ✔
```

**What does each line mean?**

1. The version of Nextflow that was executed
2. The script and version names
3. The executor used (in the above case: local)
4. The first process is executed once, which means there is one task. The line starts with a unique hexadecimal value, and ends with the task completion information

Currently it is not obvious where our `output.txt` file has been written to.

## Understanding the task directories

When a task is created, Nextflow stages the task input files, script, and other helper files into the task directory. The task writes any output files to this directory during its execution, and Nextflow uses these output files for downstream tasks and/or publishing.

These directories do not share a writable state, and any required files or information must be passed through channels (this will be important later).

!!!note

    You can execute `tree work` to view the work directory structure.

!!! warning

    The work directory might not have the same hash as the one shown above.

A series of files **log** files and any outputs are created by each task in the work directory:

- **`.command.begin`**: Metadata related to the beginning of the execution of the process task
- **`.command.err`**: Error messages (stderr) emitted by the process task
- **`.command.log`**: Complete log output emitted by the process task
- **`.command.out`**: Regular output (`stdout`) by the process task
- **`.command.sh`**: The command that was run by the process task call
- **`.exitcode`**: The exit code resulting from the command

These files are created by Nextflow to manage the execution of your pipeline. While these file are not required now, you may need to interrogate them to troubleshoot issues later.

!!!question "Exercise"

    Browse the `work` directory and view the `.command.sh` file

    ??? "Solution"

        _Note: The hash may be different to the example shown below._

        ```bash
        cat work/4e/6ba9138vhsbcbsc83bcka/.command.sh
        ```

## Publishing outputs

By default, all files created by processes exist only inside the `work` directory. To make our outputs more accessible and neatly organised, we define a **publishing strategy**, which determines which outputs should be copied to a final **publishing directory**.

The [`publishDir` directive](https://www.nextflow.io/docs/latest/process.html#publishdir) can be used to specify where and how output files should be saved. For example:

```groovy
publishDir 'results'
```

By adding the above to a process, all output files would be saved in a new folder called `results` in the current working directory. The `publishDir` directive is process specific.

!!!question "Exercise"

    1. Add `publishDir 'results'` in the `SAYHELLO` process block. 

    ???Solution

        ```groovy title="hello-world.nf" hl_lines="3"
        // Use echo to print 'Hello World!' and redirect to output.txt
        process SAYHELLO {
            publishDir 'results'

            output:
            path 'output.txt'

            script:
            """
            echo 'Hello World!' > output.txt
            """
        }
        ```

    <ol start="2">
        <li>Execute the pipeline again. View your new `results` folder in the working directory.</li>
    </ol>

!!! warning "Do not use `publishDir` as an input into processes"

    Recall that `output` definitions tells Nextflow when to run the next process and ensure that the process ran successfully. The `publishDir` directive does not allow for these checks, and is a way to make results more findable after  the pipeline has finished running. We will revisit this in more detail in the next step.

!!! abstract "Summary"

    In this step you have learned:

    1. How to use `nextflow run` to execute your first Nextflow pipeline
    2. How to view log files created by Nextflow
    3. How to publish results
