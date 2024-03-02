# Refactoring the function `do_update` from update.py module

## Idea

The major drawback of our patch to solve issue #2628 from the pdm GitHub is the increased cyclomatic complexity and overall length of the do_update() function. Thus this refactor-plan aims to provide one possible way of refactoring it into smaller functions in order to improve readability and facilitate future improvements along with reducing the technical debt acquired from our patch. Our way of achieving this is by dividing the function into several smaller functions. 

Our refactor plan mainly consists of the `Extract function` technique along with with extracting conditional expressions using the `Consolidate Conditinoal Expression` technique.


### The `do_update` function after our patch

```python
def do_update(
        project: Project,
        *,
        selection: GroupSelection,
        strategy: str = "reuse",
        save: str = "compatible",
        unconstrained: bool = False,
        top: bool = False,
        dry_run: bool = False,
        packages: Collection[str] = (),
        sync: bool = True,
        no_editable: bool = False,
        no_self: bool = False,
        prerelease: bool | None = None,
        fail_fast: bool = False,
        hooks: HookManager | None = None,
        allow_transitives : bool = False
    ) -> None:
        """Update specified packages or all packages"""
        from itertools import chain

        from pdm.cli.actions import do_lock, do_sync
        from pdm.cli.utils import check_project_file, populate_requirement_names, save_version_specifiers
        from pdm.models.requirements import strip_extras
        from pdm.models.specifiers import get_specifier
        from pdm.utils import normalize_name

        hooks = hooks or HookManager(project)
        check_project_file(project)
        if len(packages) > 0 and (top or len(selection.groups) > 1 or not selection.default):
            raise PdmUsageError(
                "packages argument can't be used together with multiple -G or " "--no-default or --top."
            )
        all_dependencies = project.all_dependencies
        updated_deps: dict[str, dict[str, Requirement]] = defaultdict(dict)
        locked_groups = project.lockfile.groups
        if not packages:
            if prerelease is not None:
                raise PdmUsageError("--prerelease/--stable must be used with packages given")
            selection.validate()
            for group in selection:
                updated_deps[group] = all_dependencies[group]
        else:
            group = selection.one()
            if locked_groups and group not in locked_groups:
                raise ProjectError(f"Requested group not in lockfile: {group}")
            dependencies = all_dependencies[group]
            for name in packages:
                normalized_name = normalize_name(name)
                matched_req = next(
                    (v for k,v in dependencies.items() if normalize_name(strip_extras(k)[0]) == normalized_name),
                    None,
                )
                if not matched_req and allow_transitives:
                    candidates = project.locked_repository.all_candidates
                    matched_req = next(
                        (v.req for k,v in candidates.items() if normalize_name(strip_extras(k)[0]) == normalized_name),
                        None,
                    )
                if not matched_req:
                    raise ProjectError(
                        f"[req]{name}[/] does not exist in [primary]{group}[/] "
                        f"{'dev-' if selection.dev else ''}dependencies."
                    )
                matched_req.prerelease = prerelease
                updated_deps[group][normalized_name] = matched_req
            project.core.ui.echo(
                "Updating packages: {}.".format(
                    ", ".join(f"[req]{v}[/]" for v in chain.from_iterable(updated_deps.values()))
                )
            )
        if unconstrained:
            for deps in updated_deps.values():
                for dep in deps.values():
                    dep.specifier = get_specifier("")
        reqs = [
            r
            for g, deps in all_dependencies.items()
            for r in deps.values()
            if locked_groups is None or g in locked_groups
        ]
        # Since dry run is always true in the locking,
        # we need to emit the hook manually with the real dry_run value
        hooks.try_emit("pre_lock", requirements=reqs, dry_run=dry_run)
        with hooks.skipping("pre_lock", "post_lock"):
            resolved = do_lock(
                project,
                strategy,
                chain.from_iterable(updated_deps.values()),
                reqs,
                dry_run=True,
                hooks=hooks,
                groups=locked_groups,
            )
        hooks.try_emit("post_lock", resolution=resolved, dry_run=dry_run)
        for deps in updated_deps.values():
            populate_requirement_names(deps)
        if unconstrained:
            # Need to update version constraints
            save_version_specifiers(updated_deps, resolved, save)
        if not dry_run:
            if unconstrained:
                for group, deps in updated_deps.items():
                    direct_deps = {dep: req for dep,req in deps.items() if dep in all_dependencies[group]}
                    project.add_dependencies(direct_deps, group, selection.dev or False)
            project.write_lockfile(project.lockfile._data, False)
        if sync or dry_run:
            do_sync(
                project,
                selection=selection,
                clean=False,
                dry_run=dry_run,
                requirements=[r for deps in updated_deps.values() for r in deps.values()],
                tracked_names=list(chain.from_iterable(updated_deps.values())) if top else None,
                no_editable=no_editable,
                no_self=no_self or "default" not in selection,
                fail_fast=fail_fast,
                hooks=hooks,
            )
```

## REFACTORING PLAN:
1. Move out the logic for matching names: 
   from line `133` to line `148` there is a fair bit of complexity
   when matching all the package names with the dependencies for 
   the given group. This could be moved out to a seperate function.
   Moreover, the if-else statement which encapsulates this call can 
   be reduced by replacing the if-body with a function generalising
   these statements. A function named `update_groups_only` or similar
   as this branch deals with unspecified updates (no explicitly named
   package).

2. The calculation for `reqs` can be moved out into a function 
   with a clear name such as `get_locked_reqs`
3. The double for at line `155` can be moved out into a function 
   called `reset_specifiers`. i.e. this:
   ```python 
    if unconstrained:
        for deps in updated_deps.values():
            for dep in deps.values():
                dep.specifier = get_specifier("")

   ``` 
   can be refactored to this: 
   ```python 
    if unconstrained: 
        reset_specifiers(updated_deps)
   ```

   ## Refactoring implementation

```diff
def do_update(
        project: Project,
        *,
        selection: GroupSelection,
        strategy: str = "reuse",
        save: str = "compatible",
        unconstrained: bool = False,
        top: bool = False,
        dry_run: bool = False,
        packages: Collection[str] = (),
        sync: bool = True,
        no_editable: bool = False,
        no_self: bool = False,
        prerelease: bool | None = None,
        fail_fast: bool = False,
        hooks: HookManager | None = None,
        allow_transitives : bool = False
    ) -> None:
        """Update specified packages or all packages"""
        from itertools import chain

        from pdm.cli.actions import do_lock, do_sync
        from pdm.cli.utils import check_project_file, populate_requirement_names, save_version_specifiers
        from pdm.models.requirements import strip_extras
        from pdm.models.specifiers import get_specifier
        from pdm.utils import normalize_name

        hooks = hooks or HookManager(project)
        check_project_file(project)
        if len(packages) > 0 and (top or len(selection.groups) > 1 or not selection.default):
            raise PdmUsageError(
                "packages argument can't be used together with multiple -G or " "--no-default or --top."
            )
        all_dependencies = project.all_dependencies
        updated_deps: dict[str, dict[str, Requirement]] = defaultdict(dict)
        locked_groups = project.lockfile.groups
        if not packages:
            if prerelease is not None:
                raise PdmUsageError("--prerelease/--stable must be used with packages given")
            selection.validate()
            for group in selection:
                updated_deps[group] = all_dependencies[group]
        else:
-            group = selection.one()
-            if locked_groups and group not in locked_groups:
-                raise ProjectError(f"Requested group not in lockfile: {group}")
-            dependencies = all_dependencies[group]
-            for name in packages:
-                normalized_name = normalize_name(name)
-                matched_req = next(
-                    (v for k,v in dependencies.items() if normalize_name(strip_extras(k)[0]) == normalized_name),
-                    None,
-                )
-                if not matched_req and allow_transitives:
-                    candidates = project.locked_repository.all_candidates
-                    matched_req = next(
-                        (v.req for k,v in candidates.items() if normalize_name(strip_extras(k)[0]) == normalized_name),
-                        None,
-                    )
-                if not matched_req:
-                    raise ProjectError(
-                        f"[req]{name}[/] does not exist in [primary]{group}[/] "
-                        f"{'dev-' if selection.dev else ''}dependencies."
-                    )
-                matched_req.prerelease = prerelease
-                updated_deps[group][normalized_name] = matched_req
+                mathced_req, updated_deps = update_groups_only(selection, locked_groups, all_dependencies, packages, normalize_name, dependencies, updated_deps, allow_transitives, project)
            project.core.ui.echo(
                "Updating packages: {}.".format(
                    ", ".join(f"[req]{v}[/]" for v in chain.from_iterable(updated_deps.values()))
                )
            )
        if unconstrained:
-            for deps in updated_deps.values():
-                for dep in deps.values():
-                    dep.specifier = get_specifier("")
+           self.reset_specifiers(updated_deps)  

-        reqs = [
-            r
-            for g, deps in all_dependencies.items()
-            for r in deps.values()
-            if locked_groups is None or g in locked_groups
-        ]
+        reqs = self.get_locked_reqs(deps, all_dependencies, locked_groups)
        # Since dry run is always true in the locking,
        # we need to emit the hook manually with the real dry_run value
        hooks.try_emit("pre_lock", requirements=reqs, dry_run=dry_run)
        with hooks.skipping("pre_lock", "post_lock"):
            resolved = do_lock(
                project,
                strategy,
                chain.from_iterable(updated_deps.values()),
                reqs,
                dry_run=True,
                hooks=hooks,
                groups=locked_groups,
            )
        hooks.try_emit("post_lock", resolution=resolved, dry_run=dry_run)
        for deps in updated_deps.values():
            populate_requirement_names(deps)
        if unconstrained:
            # Need to update version constraints
            save_version_specifiers(updated_deps, resolved, save)
        if not dry_run:
            if unconstrained:
                for group, deps in updated_deps.items():
                    direct_deps = {dep: req for dep,req in deps.items() if dep in all_dependencies[group]}
                    project.add_dependencies(direct_deps, group, selection.dev or False)
            project.write_lockfile(project.lockfile._data, False)
        if sync or dry_run:
            do_sync(
                project,
                selection=selection,
                clean=False,
                dry_run=dry_run,
                requirements=[r for deps in updated_deps.values() for r in deps.values()],
                tracked_names=list(chain.from_iterable(updated_deps.values())) if top else None,
                no_editable=no_editable,
                no_self=no_self or "default" not in selection,
                fail_fast=fail_fast,
                hooks=hooks,
            )

```

```diff
+ def reset_specifiers(updated_deps):
+    for deps in updated_deps.values():
+       for dep in deps.values():
+          dep.specifier = get_specifier("")
```


```diff
+ def get_locked_reqs(deps, all_dependencies, locked_groups):
+    reqs = [r for g, deps in all_dependencies.items()
+            for r in deps.values()
+            if locked_groups is None or g in locked_groups
+        ]
+    return reqs
```

```diff
+ def update_groups_only(selection, locked_groups, all_dependencies, packages, normalize_name, dependencies, updated_deps, allow_transitives, project):
+    group = selection.one()
+    if locked_groups and group not in locked_groups:
+        raise ProjectError(f"Requested group not in lockfile: {group}")
+    dependencies = all_dependencies[group]
+    for name in packages:
+        normalized_name = normalize_name(name)
+        matched_req = next(
+            (v for k,v in dependencies.items() if normalize_name(strip_extras(k)[0]) == normalized_name),
+                None,
+        )
+        if not matched_req and allow_transitives:
+            candidates = project.locked_repository.all_candidates
+            matched_req = next(
+                (v.req for k,v in candidates.items() if normalize_name(strip_extras(k)[0]) == normalized_name),
+                None,
+            )
+        if not matched_req:
+            raise ProjectError(
+                f"[req]{name}[/] does not exist in [primary]{group}[/] "
+                f"{'dev-' if selection.dev else ''}dependencies."
+            )
+        matched_req.prerelease = prerelease
+        updated_deps[group][normalized_name] = matched_req
+   return matched_req, updated_deps
```

## Results
If we were to refactor the `do_update()` function in this matter we could separate it into 4 functions. This would reduce its cyclomatic complexity by 15 from 41 to 26 which is a `36.6%` decrease. The new functions would belong to the same class (`Command`) as do_update.