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

You can argue critically about the benefits, drawbacks, and limitations of your work carried out, in the context of current software engineering practice, such as the SEMAT kernel (covering alphas other than Team/Way of Working).

Optional (point 6): How would you put your work in context with best software engineering practice?

The work that we carried out solves an issue requested by several people from the pdm GitHub. This corresponds to the "Oppurtunity" alpha from ESSENCE which descibes the circumstances appropriate to create or develop the software system/patch. This infers that our starting point and intentions for creating this patch were of good character where we aim to solve a real issue which can benefit the entire community. A limitation regarding our patch is that we've only taken into account the opinions and desires of individuals active on pdms GitHub. As we did not, and weren't able to, conduct a wider survey we might have missed valuable input which could've made our patch even more helpful to a wider majority. This corresponds to the "Stakeholders" alpha from ESSENCE which states that as much stakeholder involvement as possible throughout a software endeavor is necessary to ensure that acceptable software is produced. The stakeholders in this case are primarily the users of the project as well as the different contributors. In our case we were greatly limited by the timeline of the assignment and possibility to gain more insight regarding the customer desires making more customer input non-feasable. Our patch was also written in an existing function which was quite long and had a high cyclomatic complexity of 35 before we began implementing our patch. Thus our patch contributed to an even longer function with an even greater cyclomatic complexity of 41 which is one major drawback of our project. Thus our patch results in a function even more difficult to understand which increases the workload for future improvements and increases our technical debt. This is not optimal. Refactoring was not a part of our issue but nonetheless we created a refactoring plan aimed to solve this drawback which can be found below. We actively worked through the different requirements together thus sharing this understanding among the different stakeholders and our team members which expedited the development process.


Refactoring plan for do_update(): 

More about ESSENCE alphas:
* Oppurtunity
    * The issue that we solved was requested by several people already utilizing the pdm open source library. Thus there existed a "customer" need for the patch that we created. The articulated needs of the users were the foundation for our patch development and justified the reason for implementing it. We avoided implementing something that was not requested which can sometimes happen with large software companies/games where the company implements features that no one requested and actively ignore customer input which usually results in a worse product or customer experience. 
* Stakeholders
    * Since pdm is an open source project there is a large overlap between the stakeholders of the project and the project customers. Many of the people that use the library also contribute to its continuous evolution and improvement. Thus improving the library ultimately improves ones own work which creates a great incentive to contribute as well as ensure that patches are of high quality and throughly tested. 
* Requirements
    * Again since pdm is an open source project with many different contributors it can sometimes be difficult to understand the specific requirements for a given issue. Thankfully our issue was well explained with a concensus on how it should work and be implemented. We did however need to analyse the code ourselves in order to find specific requirements related to the actual implementation and testing of the patch.
* Software system
    * The software system alpha considers that the primary value of the software system is given by the execution of the software. Our patch is well implemented and well tested which adds to the overall performance of the open source library.

Optional (point 7): Is there something special you want to mention here?

Optional (point 8): Where do you put the project that you have chosen in an ecosystem of open-source and closed-source software? Is your project (as it is now) something that has replaced or can replace similar proprietary software? Why (not)? (Open source vs closed source software)
