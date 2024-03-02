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

Overall work

Our contribution to the open source library pdm can be summarized as the following:

### Chosen issue and open source interaction
Chosen issue from pdm which was assigned to Rasmus (rasmus-d): https://github.com/pdm-project/pdm/issues/2628 

This issue intends to tackle a problem with pdm where a user cannot update a single sub-dependency. Before our patch is was only possible for a user to update a single dependency, all dependencies or all dependencies + all subdependencies but not a single sub-dependecy. Thus the issue relates to implementing a feature where a user can update a specific sub-dependency without changing/updating any other dependency or subdependency. 

It was very easy for Rasmus to simply comment and ask if we could be assigned to this issue on the projects github. The open source project manager responded to us the following day and gave us the issue. We had an overall very positive experience with the open source community. It is not guarenteed that a group of students would be given an issue from a well functioning open source project so we were very grateful for the opportunity.

### Distribution of work 
For this assignment we had two start-up meetings where we discussed the issue, how we roughly wanted to tackle it along with creating specific atomic issues that could be handled individually. After these were all created on GitHub some members of the group starting looking at solving the issue meanwhile others started working on tests for the issue. At the same time we discussed how we wanted to structure the report and create more atomic issues related to this. As we were already familiar with the pdm library we could effectively do all these things simultaneously and get to work quite quickly with little extra onboarding. It was also beneficial to do the tests and solving the issue at the same time in order to see how well they coincided with each other and allowed us to continuosly assess our patch.

### Work process
After creating a concrete and stable plan for how we could effectively solve this issue we worked both independently as well as utilizing pair programming over discord to solve the different issues. We benefited a lot from pair programming for this assignment as it is quite challenging to understand all the details and nuances of the open source project. Thus it was very helpful to discuss things and help each other understand different data structures and dependencies. When we had solved the issue with regards to creating a code patch and developing our own tests after a couple of days we finalized the report and prepared for the presentation. All tests that we implemented passed and worked well.


### Main take-aways
What are your main take-aways from this project? What did you learn?

* Importance of clear issue descriptions and conveying ones ideas when working on an open source project
    * As anyone can attempt to contribute to an open source project it is very important to be extra clear with the desired functionalities of ones issues as people will not be fully emersed in the details of the project
    * We were very thankful for the clear description of the issue along with fast response times from the open source manager. This helped us throughly understand the problem and distribute the work.
* Reinforce the importance of systematically organizing the group in order to tackle the problem efficiently
    * As this was a very open problem we felt that we really benefited from having start-up meetings aimed to clearly deconstruct the problem into atomic issues and spread the workload.
* We've learnt how to create a final patch that can be easily merged into an open source project and that is clean and almost surgecal in the sense that it does not negatively affect the exiting project.
* It is emmensely important to create and adhere to guidelines when contributing to an open source project. Otherwise you risk merging many different styles from potentially hundreds of small contributors which can result in a chaotic mess.

### Essence
How did you grow as a team, using the Essence standard to evaluate yourself?

We felt that we worked very well during this project. This along with our previous progress and experiences from earlier assignments make us we feel that we've reached the working well phase of our way-of-working. Our team naturally follows our workflow guide regarding issues, testing and implementations of testing and documentation. This along with the fact the tools and methods we use actually help us perform good work are clear signs of a working group. When it comes to our team we also feel that we're performing on this front as we've worked together for quite some time and have established a functioning workflow independent of outside help. We feel that this last assignment has been a great oppurtunity to improve our ability to take initiative and tackle new uncharted problems without a clear framework. It has allowed us to actually test our new found skills and experiences and observe that they pratically work in the real world.

### P+ 
Optional (point 6): How would you put your work in context with best software engineering practice?

Optional (point 7): Is there something special you want to mention here?

Optional (point 8): Where do you put the project that you have chosen in an ecosystem of open-source and closed-source software? Is your project (as it is now) something that has replaced or can replace similar proprietary software? Why (not)? (Open source vs closed source software)
