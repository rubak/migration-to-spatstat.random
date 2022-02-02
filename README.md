# Migration to spatstat.random

This repository only contains this README about migration to `spatstat.random` for package developers using `spatstat`/`spatstat.core`/`spatstat.linnet` in their own package (in the rest of the text we only refer to `spatstat.core` but it applies as well to `spatstat.linnet`).

After the big split up of `spatstat` into subpackages in early 2021 we (@baddstats really) have now managed to split out another 10% of the remaining code of `spatstat.core` into the new package `spatstat.random`.
If your package uses random generators from `spatstat.core` it will almost certainly need a minor update to be prepared for the upcoming version of `spatstat.core` which doesn't have these.
The new `spatstat.random` is already on CRAN, so you can easily install it and test it out.
We expect to submit the new version of `spatstat.core` in late February, so you need to have things sorted out by then.
We would really like you to submit a new version of your package before we submit the new version of `spatstat.core` since our submission will break functionality in your package and cause unnecessary problems for everyone (you, us, CRAN).

## Packages known to be impacted

We have identified roughly 30 packages that needs to be updated (apart from our own packages such as `spatstat.linnet`):  
`dbmss, ecespa, ENMTools, envi, ETAS, globalKinhom, GmAMisc, highriskzone, idar, lacunaritycovariance, MetaLandSim, mobsim, NLMR, onpoint, rlfsm, selectspm, sf, shar, smacpod, spacejamr, spagmix, sparr, sparrpowR, SpatEntropy, spatsurv, stlnpp, stpp, swdpwr, ttbary`.  
Around half of these we have found to be actively developed on GitHub and we have made pull requests with suggested changes.
If you use GitHub and have not received a pull request please open an issue if you would like us to make one.

## How to check the impact on your package

Install the development version of the spatstat familiy of packages with the command below and run `R CMD check` on your package to see how it impacts your package.

```r
install.packages(c("spatstat.data", "spatstat.utils", "spatstat.sparse", 
                   "spatstat.geom", "spatstat.random", "spatstat.core", 
                   "spatstat.linnet", "spatstat"), repos = "https://spatstat.r-universe.dev")
```

## Suggested changes to adapt

How to adpat your package really depends on which functions from `spatstat.core` you use and how you use them. Typical cases are listed below with some examples.

### Using `spatstat.core` only for random generators

Then you now longer need `spatstat.core` but only `spatstat.random`. Typically it will suffice to search and replace `spatstat.core` with `spatstat.random` in all files of your packages (easy with <ctrl>+<shift>+f if you use RStudio).
For example:
  
  - [spacejamr](https://github.com/dscolby/spacejamr/commit/b4c6fdecfc09213d35429697053701bb89e74a02)
  - [sf](https://github.com/r-spatial/sf/pull/1892/commits/f27e64577720f47d7ca13199dcbc5a15aa62bf72)
  
### Using `spatstat.core` exclusively through `spatsat.core::`

If you always call `spatstat.core` with syntax `spatstat.core::` you typically don't have `spatstat.core` in your `NAMESPACE` file, and you only need to add `spatstat.random` to `Imports:` (or `Suggests:`) in `DESCRIPTION` and replace functions calls such as `spatstat.core::rpoint` with `spatstat.random::rpoint` throughout you package.
For example:
  
  - [onpoint](https://github.com/r-spatialecology/onpoint/pull/13/commits/63a6d18ee250d887af166274efc88b9f5f5b7db7)

### Using `spatstat.core` by loading namespace
  
If you have `spatstat.core` in your `NAMESPACE` file you can call functions such as `rpoint` without the prefix `spatstat.core::`. In `NAMESPACE` you can have two different forms of entries:
  
  - `importFrom("spatstat.core", ...)`. Then you typically just need to move any random generator functions from `...` to a new entry of the form `importFrom("spatstat.random", ...)` and add `spatstat.random` to the `Imports:` section of `DESCRIPTION`.
  
  - `import("spatstat.core")`. When the new version of `spatstat.core` is submitted to CRAN it will suffice to add an additional line `import("spatstat.random")`. However, until then the random generators are in both packages and importing all functions from both raises a warning. The interim solution is to modify the existing line to `import("spatstat.core", excepct = ...)` and add `importFrom("spatstat.random", ...)` with the functions used from `spatstat.random` in place of `...`. For example:
      + [sparr](https://github.com/tilmandavies/sparr/pull/21/commits/00db6db19a43e8474906b8d1479cdfebf743b3b3)

### Changes to documentation links
  
If you link to `spatstat.core` documentation of random generators you also have to update entries of the form `\code{\link[spatstat.core]{rpoint}}` to `\code{\link[spatstat.random]{rpoint}}`. 
