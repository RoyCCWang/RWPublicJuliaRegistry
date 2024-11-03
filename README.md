# RWPublicJuliaRegistry
Public registry for some of my Julia packages. Created using [LocalRegistry.jl](https://github.com/GunnarFarneback/LocalRegistry.jl).

# To connect this registry with your Julia installation
The following line connects this registry along with (re-adding just to be safe) the `General` (i.e. the default public) julia registry to your Julia installation. This only needs to be run once for every Julia installation.

HTTPS:
```
using Pkg
pkg"registry add General https://github.com/RoyCCWang/RWPublicJuliaRegistry"
```

SSH:
```
using Pkg
pkg"registry add git@github.com:RoyCCWang/RWPublicJuliaRegistry.git"
```

# To install a package from this registry
Make sure this registry is connected to your Julia installation. Then do the typical package installation as if you are installing a package from the main Julia registry. For example, to install `MakiePlots`, do
```
using Pkg
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
import Pkg
Pkg.develop(url = "url_to_your_package")
```

If you do have a local copy of your package but it isn't in the `./julia/dev` path, then navigate to the root directory of your package in your shell, launch Julia REPL, then do
```
import Pkg
Pkg.activate() # this forces the activation of the base environment, in case your current environment is the package environment you're trying to add.
Pkg.develop("./") # The package located at the current working directory will enter tracking-mode. Tested on Linux. Might need a different slash symbol or syntax for Windows and MacOS.
```


### Clean up step
You might want to remove tracking-mode once you've registered a package to the registry, or finished updating a registered packaged in a registry.

Remove tracking mode by uninstalling the package, or do `Pkg.free("PAKCAGE_NAME")`. The latter works because if `PACKAGE_NAME` is already in a registry that is connected with your current Julia installation. 

For example:
```
import Pkg
Pkg.free("MakiePlots")
```

### Why?
[LocalRegistry.jl](https://github.com/GunnarFarneback/LocalRegistry.jl) requires the package you want to add to be on the list of *development paths* of your Julia install. On Fedora Linux, one default development path is at `~/.julia/dev/`, but it won't exist until you've actually obtained a package via `Pkg.develop()`. The relevant ideas for our discussion are:
- a Julia package is identified by its `uuid`, not its hosted repository or local path. 
- `Pkd.add()` makes use of some registry to make sure dependencies can be reproduced for the packages. This is called *non-tracking* mode.
- `Pkg.develop()` puts a package in *tracking* mode, where the package directory is added to the list of development paths (think of it as tracking paths).

One can think of `Pkg.add()` as telling Julia to *stop tracking* (i.e., remove from the list of development paths) the package, and calling `Pkg.deveop()` tells Julia to start track the package at some location. Therefore, one can say `Pkg.add()` and `Pkg.deveop()` *overwrites* whatever tracking-related behavior for the package.

Tracking a lot of packages using  `Pkg.develop()` ignores the concept of virtual environments, and one can run into dependency mismatch-related issues causing bugs to not be reproducible. Therefore, usually one only keep Julia projects that are actively being developed in the development path as a prototyping workflow, and remove it from the path once the project reaches its next release state. Please research the topics *dependency hell* and *Julia package manager* in the [official package manager documentation](https://pkgdocs.julialang.org/v1/managing-packages/#developing) and [this forum post](https://discourse.julialang.org/t/add-vs-dev-at-a-local-path/58150/6?u=royw) to learn more about `Pkg.add()`, `Pkg.dev()`, and `Pkg.free()`.

Adding the directory of the package you want to the list of development paths allows [LocalRegistry.jl](https://github.com/GunnarFarneback/LocalRegistry.jl) to find and obtain the required information from packages on the development paths. 

We will remove the package directory from the list of development paths once we're done with the registration process.


## Step 1: Associate a custom registry with your Julia installation
Do step 1a if you want to add packages to a new registry. You'll create a new registry in this step. You can skip 1b.
Do step 1b if you want to add packages to an existing registry created by [LocalRegistry.jl](https://github.com/GunnarFarneback/LocalRegistry.jl). You can skip step 1a.

### Step 1a: Create a new custom registry

In the Julia REPL, I did the following to create the online repository that you're viewing now:
```
using LocalRegistry
create_registry("RWPublicJuliaRegistry", "https://github.com/RoyCCWang/RWPublicJuliaRegistry",
description = "Public unregistered Julia packages")
```
[LocalRegistry.jl](https://github.com/GunnarFarneback/LocalRegistry.jl) will automatically connect your Julia installation to this registry you just created.

Alternatively, you can just clone this online repository that you're viewing, delete the folders and their corresponding entries in `Registry.toml`, and upload it to a new location. In this case, you'll need to do step 1b.

### Step 1b: Connect with an existing custom registry
For HTTPS GitHub access, do:
```
using Pkg
pkg"registry add https://github.com/RoyCCWang/PublicJuliaRegistry"
# this is only for HTTP.
```


For SSH access:
```
pkg"registry add git@github.com:RoyCCWang/RWPublicJuliaRegistry.git"
```
If you get an error with SSH, you might need to do a `ssh-keyscan` to update known hosts for your operating system. For example, do the following from a bash shell terminal on Fedora Linux:
```
ssh-keyscan github.com >> ~/.ssh/known_hosts
```
However, you should consider the cyber security warning raised [here](https://github.com/JuliaLang/Pkg.jl/issues/2428#issuecomment-809669517) before running the `ssh-keyscan` command.


## Usage: Check which registries are connected (currently being used)
```
using Pkg
Pkg.Registry.status()
```
By default, Julia ships with the `General` registry. You should see the registry you added in step 1.

To remove a registry, do:
```
using Pkg
Pkg.Registry.rm("REGISTRY_NAME")
```
See the [official Pkg.Registry documentation](https://pkgdocs.julialang.org/v1/registries/) for more instructions.


## Step 2: add unregistered packages to a connected registry
This registration step creates the following in the registry repository:
- a folder that contain the meta information about the package
- a corresponding entry in `Registry.toml`


Make sure the package you want to register is on Julia's list of development paths. This should'ave been covered in the preparation step.
Use https for public repositories, and git/SSH for private repositories. Since this is a public registry, we use only https as our GIT command to pull.

Examples for using https:
```
using LocalRegistry

register("IntervalMonoFuncs";
registry = "RWPublicJuliaRegistry",
repo = "https://github.com/RoyCCWang/IntervalMonoFuncs.jl")

register("ConvexClustering";
registry = "RWPublicJuliaRegistry",
push = true,
repo = "https://github.com/RoyCCWang/ConvexClustering.jl")

register("MakiePlots";
registry = "RWPublicJuliaRegistry",
push = true,
repo = "https://github.com/RoyCCWang/MakiePlots.jl")
```

If you recently manually modifyed the registry repository, you might get an error when running the above. Try `Pkg.Registry.update()` to synchronize your local copy of the registry with the recently modified registry repository.

To remove tracking mode, do:
```
import Pkg
Pkg.free("IntervalMonoFuncs")
Pkg.free("ConvexClustering")
Pkg.free("MakiePlots")
```
Do `Pkg.rm()` instead of `Pkg.free()` if you want to remove the packages instead of convert to non-tracking mode.

## Step 3: To update an existing package reference
Suppose you made changes to a package that is referenced in a registry. It is the package-related meta information about the package that is important to a registry; if it has changed, then you nee to update the package's reference in registry. These meta information are:
- all version numbers so far
- dependencies
- compatibility ranges for Julia versions and dependencies
- package location
For example, these information for the MakiePlots package reference can be found in the `M/MakiePlots` folder in this repository, and `Registry.toml` has a corresponding reference to `M/MakiePlots` folder.

As an example: to update `MakiePlots`, make sure it is in tracking-mode (i.e. on the list of development paths), e.g. run:
```
import Pkg
Pkg.develop("MakiePlots")

```


To update, do:
```
using LocalRegistry
register("MakiePlots")
```

To remove tracking mode, do:
```
import Pkg
Pkg.free("MakiePlots")
```



## Troubleshooting: Manual edits
You can check the `Registry.toml` file and see that it contains the information of the steps we discussed so far. It is possible to just manually edit the meta information in the folders and `Registry.toml` then push/upload to the hosting repository manually, instead of using [LocalRegistry.jl](https://github.com/GunnarFarneback/LocalRegistry.jl).
