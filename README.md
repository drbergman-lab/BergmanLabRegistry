# PCVCTRegistry

This will likely one day migrate to [here](https://github.com/drbergman-lab/BergmanLabRegistry).

## Steps to register a new package
### Create a new package using `PkgTemplates`
Use the `PkgTemplates` package to create a new package:
```julia-repl
pkg> add PkgTemplates
```
Interactively create a new package:
```julia-repl
julia> using PkgTemplates
julia> t = Template(interactive=true)("MMStudio") # no .jl extension
```

The following options are recommended to change:
- `julia`: set to the version of Julia you are developing on, e.g. `1.11`
- `plugins`: add `Codecov` and `Documenter` to the default list
- change the `version` to `0.0.1`
- `ignore` (for `.gitignore`): add `.DS_Store`, `.vscode`, and any other files you want to ignore
- GitHubActions `extra_versions`: use `"lts","1","pre"`
- `devbranch`: set to `development`

### Add the PCVCTRegistry
Make sure Julia knows about the PCVCTRegistry:
```julia-repl
pkg> registry add https://github.com/drbergman/PCVCTRegistry.git
```

### Set up the GitHub actions
Add `development` to the `branches` for tests to run on:
```yaml
name: CI
on:
  push:
    branches:
      - main
      - development
```

Add the PCVCTRegistry for **both** tests and docs
```yaml
- uses: julia-actions/cache@v2

- name: Add PCVCTRegistry
  run: julia -e 'import Pkg; Pkg.Registry.add("General"); Pkg.Registry.add(Pkg.RegistrySpec(url="https://github.com/drbergman/PCVCTRegistry.git"))'
```

And set the TagBot to use the PCVCTRegistry:
```yaml
jobs:
  TagBot:
    if: github.event_name == 'workflow_dispatch' || github.actor == 'JuliaTagBot'
    runs-on: ubuntu-latest
    steps:
      - uses: JuliaRegistries/TagBot@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          ssh: ${{ secrets.DOCUMENTER_KEY }}
          registry: drbergman/PCVCTRegistry # here!
```

### Prepare first version and register
After getting your first version ready, you can register it with the PCVCTRegistry:
```julia-repl
julia> using LocalRegistry
julia> register() # if you have other registries, you may need to specify which registry
```

### Create the repository on GitHub
Create a new repository on GitHub with the same name as your package, e.g. `MyPkg.jl`.

### Push the package to GitHub
Make sure you have a remote set up for your GitHub repository:
```sh
git remote add origin https://github.com/<your-username>/MyPkg.jl.git
```

Then push your package to GitHub:
```sh
git push -u origin main
```

### Set up the Documenter Key
Use DocumenterTools.jl to set up the key for your package:
```julia-repl
julia> using DocumenterTools
julia> DocumenterTools.genkeys(; user="your-github-username", repo="MyPkg.jl")
```
Then follow the instructions in the console.

### Allow GitHub Actions to create and approve pull requests
Go to `Settings > Actions > General` and check this box at the bottom.
Save the changes.