# Report for assignment 4

## Project

**Name:** PDM

**URL:** https://github.com/pdm-project/pdm

A package manager for python projects.

## Onboarding experience

We continued with the project we worked on in assignment 3. Below is our onboarding experience from the project report for assignment 3.

The project has a good CONTRIBUTING.md, where clear instructions are given
on how to organize ones contribution. The project recomends using it's own 
product (PDM) to ensure formatting and linting and to document changes in 
a specific `news` directory. There is nothing wrong with the CONTRIBUTING 
file, but there is a lack of documentation of certain modules and the code 
in general. The instructions and documentation for running and using the 
product are very good and the product works as intended. 

**How easily can you build the project? Briefly describe if everything worked as documented or not:**

- **a) Did you have to install a lot of additional tools to build the software?**

    No we only needed to install PDM, and it handled the rest for us.

- **b) Were those tools well documented?**

    PDM itself is well documented on the user side.

- **c) Were other components installed automatically by the build script?**

    PDM automatically installs all the necessary dependencies.

- **d) Did the build conclude automatically without errors?**

    Yes, this is very well handled by PDM.

- **e) How well do examples and tests run on your system(s)?**

    It runs excellently. With single scripts `pdm test` and `pdm coverage`, the tests and test coverage is executed with `pytest`.


## Overview of the architecture and purpose of pdm

A package manager is a system for handling the dependencies (or "packages") of a project. It finds the correct package files in a remote repository, checks them for vulnerabilities, downloads them and puts them in the correct locations. It also does this for all the package's sub-dependencies and removes all the appropriate files when the user wants to remove packages. `pdm` is a package manager for python and its purpose is to provide extra flexibility and customizability compared to other existing python package managers like Poetry and Hatch. 

### Overview of PDM
**Build system**<br>
`pdm` uses a PEP 517 build backend, which is a build-system independent format, instead of relying on a tightly integrated built-in build backend like Poetry and Hatch does. This makes it possible for the user to choose their own build backend.

**Metadata**<br>
One of the features of pip is that it uses PEP 621 for project metadata and supports the latest PEP standard. The core metadata is described in a `pyproject.toml` file according to the latest standards which eliminates previous ambiguities. An example of this is that it is required to declare if metadata is static or dynamic.

**Resolving**<br>
When a new dependency is added, removed or updated `pdm` runs a series of check to make sure that the new set of dependencies are valid and there are no conflicts between dependencies. If there is, `pdm` tries to update and alter the versions within allowed ranges. 

**Lock file**<br>
`pdm` generates a lock file `pdm.lock` with the resolved result of all dependencies. locking in a package manager means locking down the versions of the dependencies that are used in the project. The function `do_lock` in `src/pdm/cli/actions.py` does that locking as well as updates the lock file. 

**CLI interface**<br>
The CLI interface consists of an extensive list of commands and options for initializing and managing the dependencies in the project. Below are a few examples of common CLI commands.
- `init` - initializes a pyproject.tom
- `add` - add packages to pyproject.toml and install them
- `build` - build artifacts for distribution 
- `remove` - removes files that match the argument pattern. 
- `update` - all packages or only a specified package. `--update-eager` is an option for `pdm` to try to update the packages and their dependencies recursively. Another example of an option is `--update-all` which updates all packages in the toml file and their subdependencies. 

For a more detailed overview of CLI commands and their options visit https://pdm-project.org/latest/reference/cli/

**Plugin System**<br>
This is a system that allows user to develop new `pdm` commands and command options and share them to the community. To develop a new command the user uses inheritance from a BaseCommand class and can add arguments with add_arguments and tether actions with handle. This is added to PyPI by the command publish. 

**Architecture**<br>
<div style="text-align:center;">
    <figure>
        <img src="img/architecture-diagram.png" alt="Diagram of the architecture" width="500"/>
        <figcaption>Diagram of the architecture of pdm.</figcaption>
    </figure>
</div>

**Conclusion**<br>
`pdm` is a python dependency manager that offers customizability and flexibility compared to alternative dependency managers. It is capable of solving dependency conflicts between all dependencies and stores the resolved result in a lock file. It uses modern standards for describing its core metadata unambiguously and supports users to share plugins to PyPI.


## Effort spent

For each team member, how much time was spent in

1. plenary discussions/meetings;

2. discussions within parts of the group;

3. reading documentation;

4. configuration and setup;

5. analyzing code/output;

6. writing documentation;

7. writing code;

8. running code?

For setting up tools and libraries (step 4), enumerate all dependencies
you took care of and where you spent your time, if that time exceeds
30 minutes.

## Overview of issue(s) and work done.

Title:

URL:

Summary in one or two sentences

Scope (functionality and code affected).

## Requirements for the new feature or requirements affected by functionality being refactored

Optional (point 3): trace tests to requirements.

## Code changes

### Patch

(copy your changes or the add git command to show them)

git diff ...

Optional (point 4): the patch is clean.

Optional (point 5): considered for acceptance (passes all automated checks).

## Test results

Overall results with link to a copy or excerpt of the logs (before/after
refactoring).

## UML class diagram and its description

### Key changes/classes affected

Optional (point 1): Architectural overview.

Optional (point 2): relation to design pattern(s).

## Overall experience

What are your main take-aways from this project? What did you learn?

How did you grow as a team, using the Essence standard to evaluate yourself?

Optional (point 6): How would you put your work in context with best software engineering practice?

Optional (point 7): Is there something special you want to mention here?

Optional (point 8): Where do you put the project that you have chosen in an ecosystem of open-source and closed-source software? Is your project (as it is now) something that has replaced or can replace similar proprietary software? Why (not)? (Open source vs closed source software)