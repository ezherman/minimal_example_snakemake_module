# Minimal example snakemake repository with a module
Repository to exemplify the issue when importing Python modules within Snakemake modules. This repository was created using Snakemake v8.11.3 and additionally tested with Snakemake v7.28.3. 

# Overview of the workflow
This repository has the following directory structure:

```
.
├── README.md
└── workflow
    ├── minimal_module
    │   └── workflow
    │       ├── scripts
    │       │   ├── __init__.py
    │       │   └── input_functions_module.py
    │       └── Snakefile
    ├── scripts
    │   ├── __init__.py
    │   └── input_functions_main.py
    └── Snakefile

5 directories, 7 files
```

There is a main workflow and a module workflow. The latter is stored within `minimal_module`.

The main workflow creates an output file `results/output.txt`, which requires the input `results/intermediate.txt`. 
The intermediate file is the output of `minimal_module`.

The main workflow uses an input function defined inside `input_functions_main.py`. 
The module uses an input function defined inside `input_functions_module.py`. 

# Overview of the bug
When executing the main workflow with `snakemake -c1`, the following error is returned:

```
ModuleNotFoundError in file /home/esox/github/minimal_example_snakemake_module/workflow/minimal_module/workflow/Snakefile, line 1:
No module named 'scripts.input_functions_module'
  File "/home/esox/github/minimal_example_snakemake_module/workflow/Snakefile", line 6, in <module>
  File "/home/esox/github/minimal_example_snakemake_module/workflow/minimal_module/workflow/Snakefile", line 1, in <module>
```

In other words, the input function of the module cannot be imported.

When executing the module from within `workflow/minimal_module` using `snakemake -c1 results/intermediate.txt`, the 
input function is found:

```
snakemake -c1 results/intermediate.txt -n
Building DAG of jobs...
Job stats:
job                    count
-------------------  -------
create_input               1
create_intermediate        1
total                      2

Execute 1 jobs...

[Wed Jun 12 10:36:57 2024]
rule create_input:
    output: results/input.txt
    jobid: 1
    reason: Missing output files: results/input.txt
    resources: tmpdir=<TBD>

Execute 1 jobs...

[Wed Jun 12 10:36:57 2024]
rule create_intermediate:
    input: results/input.txt
    output: results/intermediate.txt
    jobid: 0
    reason: Missing output files: results/intermediate.txt; Input files updated by another job: results/input.txt
    resources: tmpdir=<TBD>

Job stats:
job                    count
-------------------  -------
create_input               1
create_intermediate        1
total                      2

Reasons:
    (check individual jobs above for details)
    input files updated by another job:
        create_intermediate
    output files have to be generated:
        create_input, create_intermediate

This was a dry-run (flag -n). The order of jobs does not reflect the order of execution.
```
