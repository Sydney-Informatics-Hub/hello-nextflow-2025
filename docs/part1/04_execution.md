# 1.4 Running your first pipeline

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

## 1.4.1 Executing `hello-world.nf`

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
[4e/6ba912] SAYHELLO [100%] 1 of 1 ✔
```

**What does each line mean?**

1. The version of Nextflow that was executed
2. The script and version names
3. The executor used (in the above case: local)
4. **The process is executed once, which means there is one task**. The line starts with a unique hexadecimal value, and ends with the task completion information

**Currently it is not obvious where our `output.txt` file has been written to.**

## 1.4.2 Understanding the work and task directories

When a task is created, Nextflow stages the task input files, script, and other helper files into the task directory. The task writes any output files to this directory during its execution, and Nextflow uses these output files for downstream tasks and/or publishing.

These directories do not share a writable state, and any required files or information must be passed through channels (this will be important later).

!!! warning

    The work directory might not have the same hash as the one shown above.

Let's inspect the work directory.

!!!question "Exercises"

    1. In the terminal, run `ls` to view the files in the directory.

        ??? Solution

            ```bash title="Terminal"
            ls
            ```
            ```console title="Output"
            hello-world.nf  output.txt  work 
            ```

            Running our `hello-world.nf` pipeline created a new directory called `work`.
            Note that `output.txt` was not from the pipeline we just ran, but
            from the exercises from lesson 1.2.

    2. Inspect the `work` directory by running `tree -a work` in the terminal.

        ??? Solution

            `tree` shows you the file and directory structure of `work`. The `-a` flag includes
            hidden files (files that start with a `.`).

            ```bash title="Terminal"
            tree -a work
            ```
            ```console title="Output"
            work/
            └── 4e
                └── 6ba9138vhsbcbsc83bcka
                    ├── .command.begin
                    ├── .command.err
                    ├── .command.log
                    ├── .command.out
                    ├── .command.run
                    ├── .command.sh
                    ├── .exitcode
                    └── output.txt
            ```

A series of files **log** files and any outputs are created by each task in the work directory:

- **`.command.begin`**: Metadata related to the beginning of the execution of the process task
- **`.command.err`**: Error messages (stderr) emitted by the process task
- **`.command.log`**: Complete log output emitted by the process task
- **`.command.out`**: Regular output (`stdout`) by the process task
- **`.command.sh`**: The command that was run by the process task call
- **`.exitcode`**: The exit code resulting from the command

These files are created by Nextflow to manage the execution of your pipeline. While these file are not required now, you may need to interrogate them to troubleshoot issues later.

Note that our `output.txt` file created by the `SAYHELLO` process is also in the same task directory.

!!!question "Exercise"

    View the `.command.sh` file

    ??? "Solution"

        _Note: The hash may be different to the example shown below._

        ```bash
        cat work/4e/6ba9138vhsbcbsc83bcka/.command.sh
        ```
        ```console title="Output"
        #!/bin/bash -ue
        echo 'Hello World!' > output.txt
        ```

        The `.command.sh` is the bash script that Nextflow creates and runs for the `SAYHELLO` process
        defined in `hello-world.nf`. In this example it shows the same `script` block as the process.
        Inspecting `.command.sh` is very useful for troubleshooting once you
        introduce parameters and dynamic naming, when it is not as clear how the `script` block will
        look like.

## 1.4.3 Caching tasks and resuming workflows

One of the core features of Nextflow is the ability to store task executions
(caching). These cached tasks and files can be reused by Nextflow to minimise
duplicating work, and let's you resume pipelines. 

Instead of having to run the entire pipeline from the beginning, you can tell
Nextflow to run only the processes that errored. This is extremely useful for
iteratively developing a pipeline.

!!! Note

    Each time a task runs, Nextflow creates a unique task directory inside the `work/`
    directory. 
    The generated hash ensures that each task can be uniquely identified. This is
    important for checkpointing, especially when you can be running thousands of
    tasks in a single pipeline. The hash is computed from different metadata
    such as your compute environment and some details of the process. More
    information can be found in the Nextflow docs on
    [task hash](https://www.nextflow.io/docs/latest/cache-and-resume.html#task-hash).

In the next exercise, we will run our `hello-world.nf` with the `-resume` flag
and review how caching allows resumability.

!!!question Exercise 

    Run the command `nextflow run hello-world.nf -resume`.

    ??? Solution

        ```console
        N E X T F L O W  ~  version 24.10.2
        Launching `hello-world.nf` [mighty_murdock] DSL2 - revision: 80e92a677c
        executor >  local (1)
        [4e/6ba912] SAYHELLO [100%] 1 of 1, cached: 1 ✔
        ```

The output you receive is the same as the first time the pipeline was ran, with the addition
of `cached: 1`. The workflow was execuuted from the beginning, however, before running the
task, Nextflow used the unique task ID to check if the task directory already exists and
was completed succesfully or not.

Since we already ran the `SAYHELLO` task, it completed without error, and the task directory
with the matching unique ID exists, these previous results are used as the process results.

## 1.4.4 Publishing outputs

By default, all files created by processes exist only inside the `work` directory. When we have
pipelines with multiple processes that generate many output files, it is not feasible to
view each task directory for each of our output files.

To make our outputs more accessible and neatly organised, we define a **publishing strategy**, which determines which outputs should be copied to a final **publishing directory**.

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
