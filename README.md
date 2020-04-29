# CompatHelper

[![Build Status](https://travis-ci.com/bcbi/CompatHelper.jl.svg?branch=master)](https://travis-ci.com/bcbi/CompatHelper.jl)
[![Codecov](https://codecov.io/gh/bcbi/CompatHelper.jl/branch/master/graph/badge.svg)](https://codecov.io/gh/bcbi/CompatHelper.jl)

CompatHelper is a Julia package that helps you keep your `[compat]` entries up-to-date.

Whenever one of your package's dependencies releases a new version, CompatHelper opens a pull request on your repository that modifies your `[compat]` entry to reflect the newly released version.

## Installation

The easiest way to use CompatHelper is to install it as a GitHub Action.

To install CompatHelper as a GitHub Action on your repository:

1. Go to the GitHub page for your repository.
2. Click on the "Actions" tab. (If you don't see the "Actions" tab, follow the instructions [here](#actions-setup).) The Action tab is across the top as shown in this screenshot:
![action](readme_images/action_tab.png)
3. If you have never set up any GitHub Actions on your repository, you will be brought to a page that says "Get started with GitHub Actions". In the top right-hand corner, click on the button that says "Skip this: Set up a workflow yourself". Then go to step 5.
4. If you have previously set up a GitHub Action on your repository, you will be brought to a page that says "All workflows" and has a list of all of the GitHub Actions workflows on your repository. Click on the "New workflow" button. Then, in the top right-hand corner, click on the button that says "Skip this: Set up a workflow yourself". Then go to step 5.
5. An editor will open with some content pre-populated by GitHub. Delete all of the pre-populated content.
6. Copy the following text and paste it into the empty editor:
```yaml
name: CompatHelper

on:
  schedule:
    - cron: '00 00 * * *'

jobs:
  CompatHelper:
    runs-on: ubuntu-latest
    steps:
      - name: Pkg.add("CompatHelper")
        run: julia -e 'using Pkg; Pkg.add("CompatHelper")'
      - name: CompatHelper.main()
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: julia -e 'using CompatHelper; CompatHelper.main()'

```
7. Name the file `CompatHelper.yml`. (The full path to the file should be `.github/workflows/CompatHelper.yml`.)
8. In the top right-hand corner, click on the green "Start commit" button, and then click on the green "Commit new file" button.

CompatHelper is now installed as a GitHub Action on your repository.

## Overriding the default branch

By default, CompatHelper will open pull requests against your repository's default branch. If you would like to override this behavior, set the `master_branch` keyword argument. For example:
```julia
CompatHelper.main(; master_branch = "my-custom-branch")
```

## Custom registries

To use a list of custom registries instead of the General registry, use the `registries` keyword argument. For example:
```julia
my_registries = [Pkg.RegistrySpec(name = "General",
                                  uuid = "23338594-aafe-5451-b93e-139f81909106",
                                  url = "https://github.com/JuliaRegistries/General.git"),
                 Pkg.RegistrySpec(name = "BioJuliaRegistry",
                                  uuid = "ccbd2cc2-2954-11e9-1ccf-f3e7900901ca",
                                  url = "https://github.com/BioJulia/BioJuliaRegistry.git")]

CompatHelper.main(; registries = my_registries)
```

## Using subdirectories

By default, CompatHelper expects your git repository to contain a single package, and that the `Project.toml` for that package exists in the top-level directory. You can indicate that you want CompatHelper to process one or many packages that exist in subdirectories of the git repository by passing the `subdirs` keyword to the main function. For example:
```julia
CompatHelper.main(; subdirs = ["", "Subdir1", "very/deeply/nested/Subdir2"])
```
Note that the convention for specifying a top-level directory in the `subdirs` keyword is `[""]`

## Actions setup
* Open the specific repository, navigate to the Settings tab, click Actions option, check if the Actions is enabled for this repository.


## Custom pre-commit hooks

CompatHelper supports running a custom function (called a "precommit hook") just before commiting changes. To provide a precommit hook, simple pass a zero-argument function as the first argument to `CompatHelper.main`.

### Default precommit hook

If you do not specify a precommit hook, CompatHelper will run the default precommit hook (`CompatHelper.update_manifests`), which updates all `Manifest.toml` files in your repository.

### Examples

#### Disable all precommit hooks

If you want to disable all precommit hooks, simply pass a dummy function that does nothing:

```yaml
run: julia -e '
  using CompatHelper;
  CompatHelper.main( () -> () );'
```

#### Print a logging message

You can add functionality by passing your own zero-argument function to `CompatHelper.main`, like so:

```yaml
run: julia -e '
  using CompatHelper;
  CompatHelper.main() do;
    CompatHelper.update_manifests();
    println("I did it!");
  end;'
```


This snippet uses `;` to specify the ends of lines, because according to YAML, the entire block of Julia code is a single line.
Also to note is that you cannot use `'` inside of your Julia command, since it's used to quote the Julia code.

A full example is available [here](https://github.com/tkf/Kaleido.jl/blob/42f8125f42413ef21160575d870819bba33296d5/.github/workflows/CompatHelper.yml).

#### Only update the `Manifest.toml` in the root of the repository

The following snippet tells CompatHelper to update the `Manifest.toml` file in the root of the repository but not any of the other `Manifest.toml` files. So, for example, `/Manifest.toml` will be updated, but `/docs/Manifest.toml`, `/examples/Manifest.toml`, and `/test/Manifest.toml` will not be updated.

```yaml
run: julia -e 'using CompatHelper; CompatHelper.main( (; registries) -> CompatHelper._update_manifests(pwd(); registries = registries) )'
```

## Acknowledgements

- This work was supported in part by National Institutes of Health grants U54GM115677, R01LM011963, and R25MH116440. The content is solely the responsibility of the authors and does not necessarily represent the official views of the National Institutes of Health.
