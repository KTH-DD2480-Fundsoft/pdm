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

Title: *Option to update specific sub-dependency in lock file*

URL: [here](https://github.com/pdm-project/pdm/issues/2628)

Summary in one or two sentences:

Feature requests; possibility to update specific transitive dependencies, not added as a direct depencency.
With the use of a single command.

Scope (functionality and code affected):
* Command line arguments / UI: A new argument to the CLI interface was added
  this change is quite isolated and just meant adding a flag that is parsed into
  a boolean variable passed to `do_update`.
* Update logic: The way dependencies are collected for updating was changed and subsequently altered other, unrelated parts of the function. For example, since the collection of to-be-updated packages included transitive dependencies, this list had to be filtered again when setting specificers and writing back to the pyproject file.


## Requirements for the new feature or requirements affected by functionality being refactored

Optional (point 3): trace tests to requirements.

* test_update_transitive()
    * This test adds the dependency `requests` and later updates the sub-dependency `chardet`. Asserts that the sub-dependency is still not in the pyproject.toml file and that the sub-dependency has been updated to the newest version from the virtual repository. Also asserts that the dependency `requests` has not been updated as this was not requested. 
    * Fulfills requirements **ID 1** and **ID 5**

* test_update_transitive_nonexistant_dependencies()
    * This test adds the dependency `requests` and later attempts to update the non-existant sub-dependency `numpy`. This results in a "ProjectError" which is checked with two assertions. Thus throwing an error that the sub-dependency doesn't exist in the lock file.
    * Fulfills requirements **ID 2**

* test_update_transitive_non_transitive_dependencies()
    * This test adds two dependencies `requests` and `pytz` and later updates both of these along with the sub-dependency `chardet`. It asserts that the sub-dependency is still not in the pyproject.toml file and asserts that the sub-dependency and the two dependencies are both updated to newer versions from the virtual repository even when utilizing the flag "--allow_transitive". 
    * Fulfills requirements **ID 3**, **ID 4** and **ID 5**

The three tests fulfill requirements **ID 1**, **ID 2**, **ID 3**, **ID 4** and **ID 5**. Thus we can see that our three different tests in turn test all our stated requirements resulting in good test coverage for our patch.

## Code changes

### Patch

```diff
diff --git a/src/pdm/cli/commands/update.py b/src/pdm/cli/commands/update.py
index 39561b6f..7cbdd353 100644
--- a/src/pdm/cli/commands/update.py
+++ b/src/pdm/cli/commands/update.py
@@ -66,6 +66,13 @@ class Command(BaseCommand):
             action="store_false",
             help="Only update lock file but do not sync packages",
         )
+        parser.add_argument(
+            "--allow-transitive",
+            dest="allow_transitives",
+            default=False,
+            action="store_true",
+            help="Allow updating of transitive dependencies",
+        )
         parser.add_argument("packages", nargs="*", help="If packages are given, only update them")
         parser.set_defaults(dev=None)

@@ -85,6 +92,7 @@ class Command(BaseCommand):
             prerelease=options.prerelease,
             fail_fast=options.fail_fast,
             hooks=HookManager(project, options.skip),
+            allow_transitives=options.allow_transitives
         )

     @staticmethod
@@ -104,6 +112,7 @@ class Command(BaseCommand):
         prerelease: bool | None = None,
         fail_fast: bool = False,
         hooks: HookManager | None = None,
+        allow_transitives : bool = False
     ) -> None:
         """Update specified packages or all packages"""
         from itertools import chain
@@ -135,17 +144,24 @@ class Command(BaseCommand):
                 raise ProjectError(f"Requested group not in lockfile: {group}")
             dependencies = all_dependencies[group]
             for name in packages:
-                matched_name = next(
-                    (k for k in dependencies if normalize_name(strip_extras(k)[0]) == normalize_name(name)),
+                normalized_name = normalize_name(name)
+                matched_req = next(
+                    (v for k,v in dependencies.items() if normalize_name(strip_extras(k)[0]) == normalized_name),
                     None,
                 )
-                if not matched_name:
+                if not matched_req and allow_transitives:
+                    candidates = project.locked_repository.all_candidates
+                    matched_req = next(
+                        (v.req for k,v in candidates.items() if normalize_name(strip_extras(k)[0]) == normalized_name),
+                        None,
+                    )
+                if not matched_req:
                     raise ProjectError(
                         f"[req]{name}[/] does not exist in [primary]{group}[/] "
                         f"{'dev-' if selection.dev else ''}dependencies."
                     )
-                dependencies[matched_name].prerelease = prerelease
-                updated_deps[group][matched_name] = dependencies[matched_name]
+                matched_req.prerelease = prerelease
+                updated_deps[group][normalized_name] = matched_req
             project.core.ui.echo(
                 "Updating packages: {}.".format(
                     ", ".join(f"[req]{v}[/]" for v in chain.from_iterable(updated_deps.values()))
@@ -183,7 +199,8 @@ class Command(BaseCommand):
         if not dry_run:
             if unconstrained:
                 for group, deps in updated_deps.items():
-                    project.add_dependencies(deps, group, selection.dev or False)
+                    direct_deps = {dep: req for dep,req in deps.items() if dep in all_dependencies[group]}
+                    project.add_dependencies(direct_deps, group, selection.dev or False)
             project.write_lockfile(project.lockfile._data, False)
         if sync or dry_run:
             do_sync(
```
tests:
```diff
diff --git a/tests/cli/test_update.py b/tests/cli/test_update.py
index e59a4c46..a7e149fc 100644
--- a/tests/cli/test_update.py
+++ b/tests/cli/test_update.py
@@ -161,6 +161,60 @@ def test_update_specified_packages_eager_mode(project, repository, pdm):
     assert locked_candidates["pytz"].version == "2019.3"


+@pytest.mark.usefixtures("working_set")
+def test_update_transitive(project, repository, pdm):
+    pdm(["add", "requests", "--no-sync"], obj=project, strict=True)
+    repository.add_candidate("chardet", "3.0.5")
+    repository.add_candidate("requests", "2.20.0")
+    repository.add_dependencies(
+        "requests",
+        "2.20.0",
+        [
+            "certifi>=2017.4.17",
+            "chardet<3.1.0,>=3.0.2",
+            "idna<2.8,>=2.5",
+            "urllib3<1.24,>=1.21.1",
+        ],
+    )
+    pdm(["update", "--allow-transitive", "chardet"], obj=project, strict=True)
+    locked_candidates = project.locked_repository.all_candidates
+    assert not any("chardet" in dependency for dependency in project.pyproject.metadata["dependencies"])
+    assert locked_candidates["chardet"].version == "3.0.5"
+    assert locked_candidates["requests"].version == "2.19.1"
+
+
+@pytest.mark.usefixtures("working_set")
+def test_update_transitive_nonexistant_dependencies(project, repository, pdm):
+    pdm(["add", "requests", "--no-sync"], obj=project, strict=True)
+    result = pdm(["update", "--allow-transitive", "numpy"])
+    assert "ProjectError" in result.stderr
+    assert "numpy does not exist in" in result.stderr
+
+
+@pytest.mark.usefixtures("working_set")
+def test_update_transitive_non_transitive_dependencies(project, repository, pdm):
+    pdm(["add", "requests", "pytz", "--no-sync"], obj=project, strict=True)
+    repository.add_candidate("pytz", "2019.6")
+    repository.add_candidate("chardet", "3.0.5")
+    repository.add_candidate("requests", "2.20.0")
+    repository.add_dependencies(
+        "requests",
+        "2.20.0",
+        [
+            "certifi>=2017.4.17",
+            "chardet<3.1.0,>=3.0.2",
+            "idna<2.8,>=2.5",
+            "urllib3<1.24,>=1.21.1",
+        ],
+    )
+    pdm(["update", "--allow-transitive", "requests", "chardet", "pytz"], obj=project, strict=True)
+    locked_candidates = project.locked_repository.all_candidates
+    assert not any("chardet" in dependency for dependency in project.pyproject.metadata["dependencies"])
+    assert locked_candidates["requests"].version == "2.20.0"
+    assert locked_candidates["chardet"].version == "3.0.5"
+    assert locked_candidates["pytz"].version == "2019.6"
+
+
```


Optional (point 4): the patch is clean.
```bash
git diff --check
echo $?
0
```

Optional (point 5): considered for acceptance (passes all automated checks).

## Test results

Overall results with link to a copy or excerpt of the logs (before/after
refactoring).

## UML class diagram and its description

### Key changes/classes affected

Optional (point 1): Architectural overview.

Optional (point 2): relation to design pattern(s).

Some of the changes differ a bit from the previous patterns, as the way transitive dependencies were updated before, via the `eager` strategy, seperated out the *stepping* from dependency to *transitive* dependency, and so on, to the *Provider* class and used in the *Provider*-*Resolver* producer-consumer pattern. But, since we are specifying *specific* packages, this is instead performed directly in the driver function -- `do_update`.

Since there is no clear API for reading the lockfile, we used the `(Locked)Repository` class to access all locked dependencies, and match the provided packages with those in the lockfile. This class is, as we understand it, used when *resolving* dependencies, and there is therefore some overlap in functionality. We opted for this, as the alternative, as we saw it, would involve writing a new API for accessing the lockfile, which would mean we either would have to re-parse the packages in the lock or aggregate classes and leverage method calls meant for resolution.


## Overall experience

What are your main take-aways from this project? What did you learn?

How did you grow as a team, using the Essence standard to evaluate yourself?

Optional (point 6): How would you put your work in context with best software engineering practice?

Optional (point 7): Is there something special you want to mention here?

Optional (point 8): Where do you put the project that you have chosen in an ecosystem of open-source and closed-source software? Is your project (as it is now) something that has replaced or can replace similar proprietary software? Why (not)? (Open source vs closed source software)
