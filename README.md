# RWPublicJuliaRegistry
Public registry for some of Roy Wang's Julia packages. Created using [LocalRegistry.jl](https://github.com/GunnarFarneback/LocalRegistry.jl).

# To use a package in this registry
If you want to use one of the packages in this registry, say `MakiePlots`, do:
```
using Pkg
pkg"registry add General https://github.com/RoyCCWang/RWPublicJuliaRegistry"
pkg"add MakiePlots"
```
See the post by Gunnar Farnebäck in the forum post [how to install package dependencies for unregistered package](https://discourse.julialang.org/t/how-to-install-package-dependencies-for-unregistered-package/36088/16?u=royw).

Users who do not want to know how to set up and register/remove packages from a registry can ignore the rest of this document.

# My steps for creating this online registry
Here are the steps I took to set this registry up. This is a tutorial for package authors who wish to set up their own registry for their unregistered packages. These steps were done on Fedora Linux.

It is possible to create your local registry locally via [LocalRegistry.jl](https://github.com/GunnarFarneback/LocalRegistry.jl), but it is not covered here.

I use the term *registry repository* to mean the file folder (if you're storing your registry locally) or the online repository that hosts your registry. The registry repository is usually a totally separate repository than the repositories that host the packages that are referenced in the registry.

## Preparation: put package on development path

If you don't have a local copy of your package, do:
```
import Pkg; Pkg.develop(url = "url_to_your_package")
```
If you do have a local copy of your package but it isn't in the `./julia/dev` path, then navigate to the root directory of your package in your shell, launch Julia REPL, then do
```
import Pkg; Pkg.develop()
```

### Why?
[LocalRegistry.jl](https://github.com/GunnarFarneback/LocalRegistry.jl) requires the package you want to add to be on the list of *development paths* of your Julia install. On Fedora Linux, one default development path is at `~/.julia/dev/`, but it won't exist until you've actually obtained a package via `Pkg.develop()`. The relevant ideas for our discussion are:
- `Pkd.add()` makes available a package to your activated environment.
- `Pkg.develop()` makes available a package to all your enviornments. In other words, Julia will have access to the contents of packages (functions, modules, etc.) from Julia's list of development paths.

Although the `Pkg.develop()` behaviour sounds simple, it ignores the concept of virtual environments, and one can quickly run into dependency mismatch-related issues causing bugs to not be reproducible. Therefore, usually one only keep Julia projects that are actively being developed in the development path as a prototyping workflow, and remove it from the path once the project reaches its next release state. Please research the topics *dependency hell* and *Julia package manager* in the [official package manager documentation](https://pkgdocs.julialang.org/v1/managing-packages/#developing) to learn more about `Pkg.add()`, `Pkg.dev()`, and `Pkg.free()`.

Adding the directory of the package you want to the list of development paths allows [LocalRegistry.jl](https://github.com/GunnarFarneback/LocalRegistry.jl) to find and obtain the required information from packages on the development paths. 

We will remove the package directory from the list of development paths once we're done with the registration process.


## Step 1: Create a new registry
In the Julia REPL, I did the following to create the online repository that you're viewing now:
```
using LocalRegistry
create_registry("RWPublicJuliaRegistry", "https://github.com/RoyCCWang/RWPublicJuliaRegistry",
description = "Public unregistered Julia packages")
```

### Step 2: Add registry reference to the registry
To use [LocalRegistry.jl](https://github.com/GunnarFarneback/LocalRegistry.jl) to add a package reference to the registry (called a *package registration to the registry*), do the following:


For HTTPS GitHub access.
```
using Pkg
pkg"registry add https://github.com/RoyCCWang/PublicJuliaRegistry"
# this is only for HTTP.
```
For SSH access:
```
pkg"registry add git@github.com:RoyCCWang/RWPublicJuliaRegistry.git"
```

This step creates the following in the registry repository:
- a folder that contain the meta information about the package
- a corresponding entry in `Registry.toml`

## Usage: Check which registries are currently in use
`Pkg.Registry.status()` to check which registries are used. By default, Julia ships with the `General` registry.

### To add unregistered packages to a registry
To add to the registry, need to add the package (e.g. the `Utilities` packages mentioned below) via Pkg's `dev` instead of `Pkg.add` first. Remove the `dev` package once this procedure is done if you don't need Utilities in dev mode.
Example for adding three unregistered packages to the `PublicJuliaRegistry` registry.

```
using LocalRegistry
register("Utilities";
registry = "PublicJuliaRegistry",
repo = "https://gitlab.com/RoyCCWang/utilities")

register("ProbabilityUtilities";
registry = "PublicJuliaRegistry",
push = true,
repo = "https://gitlab.com/RoyCCWang/probabilityutilities")

register("RiemannianOptim";
registry = "PublicJuliaRegistry",
push = true,
repo = "https://gitlab.com/RoyCCWang/riemannianoptim")
```

### To update the version and compatibility meta info of a package that is already in a registry
Once you make changes to a package that is listed in a registry, you might update its release number or other meta info. To incorporate this information into the registry:
Make sure the latest version of the package of interest (e.g. Utilities in the example below) is installed in Julia locally via Pkg's `dev` instead of `add`.
The following would update the meta data of the `Utilities` package to whichever URL/location that is installed locally in dev mode.

```
using LocalRegistry
register("Utilities")
```


## Manual edits
You can check the `Registry.toml` file and see that it contains the information of the steps we discussed so far.