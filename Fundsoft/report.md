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

## Requirements related to the issue
The requirements of the issue weren't entirely laid out in the issue itself. The [issue](https://github.com/pdm-project/pdm/issues/2628) mainly contained the functionality that the new feature should have. But by looking at old pull requests and the existing test cases, we were able to identify a few key requirements:

|ID | Name | Description|
----|------|-------------|
**1**| Updating sub-dependency| Running `pdm update --include-transitive <sub_dependency>`for a sub-dependency in the lock file should update it and update the lock file.|
**2**| Unknown sub-dependency should throw error| Running `pdm update --include-transitive <sub_dependency>` for a sub-dependency not in the lock file should throw an error. The update command is not supposed to add dependencies. | 
**3** | Passing direct dependencies should update as usual| Running `pdm update --include-transitive <dependency>` for a direct dependency in the pyproject.toml file should update as usual. This should be equal to just runningn`pdm update <dependency>`.|
**4** | Mixing dependencies and sub-dependencies is possible| Passing a mix of direct and sub-dependencies should update both the dependencies and the sub-dependencies. It should therefore be possible to run the command `pdm update --include-transitive <dependency> <subdependency>`. Note that `<subdependency>` doesn't have to be a dependency to `<subdependency>` here.
**5** | Updating a sub-dependency doesn't add it as dependency| Updating a sub-dependency should not add it as a direct dependency in the pyproject.toml file.

Additionally, we should add completion for any arguments added to the CLI interface. This is so a user can type `pdm update --i` and then press TAB so it autocompletes to `pdm update --include-transitive`.

### Project plan for testing
1. **ID 1**: This is the very core of the issue and can be tested by first adding a dependency with a specific sub-dependency, asserting that it has a specific version, updating the sub-dependency and then asserting that the sub-dependency has an updated version. The one difficulty with this is adding a dependency with a sub-dependency that is not the latest version so that it can then be updated.
2. **ID 2**: PDM uses mocking tools which means that it is easy to simulate `pdm` commands. Here we can write a test that tries to update a sub-dependency that does not exist in the lock file. Then we can assert that this throws an error.
3. **ID 3**: This can easily be tested by updating a regular dependency but including the `--include-transitive` flag. Then we can assert that the dependency has a new version.
4. **ID 4**: This just involves a combination of the tests for ID 3 and ID 1. Write a test that updates both a dependency and a sub-dependency and assert that they get new versions.
5. **ID 5**: Here we can update the sub-dependency and then look at the pyproject.toml file to ensure that the sub-dependency has not been added as a direct dependency.