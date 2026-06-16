# Two Weeks In: Progress on Diffuse Component Standardization

It's been about two weeks since I started working on my GSoC project, so I wanted to share a quick update on what I've been working on and where the project is heading next.

My time so far has been spent on the first phase of the project: standardizing the outputs of pvlib's diffuse transposition models.

## Completed work

### [Support for `return_components` in `irradiance.reindl`](https://github.com/pvlib/pvlib-python/pull/2775)

The main piece of work so far has been adding support for the `return_components` parameter to `irradiance.reindl`.

The implementation itself was relatively straightforward conceptually. I also added a new test covering the `return_components=True` behavior, largely based on the existing test for `haydavies`.

One small design decision that came up during review was the choice between returning an `OrderedDict` (like the existing behavior on `perez`, `perez_driesse` and `haydavies`) or a regular `dict`. Following feedback from the pvlib community, I switched to using regular dictionaries. Since modern Python dictionaries preserve insertion order, `OrderedDict` generally isn't necessary anymore, and using standard dictionaries keeps the implementation simpler and more consistent with current Python practices.

This PR has passed all checks and is currently waiting for review and merge.

### [Fixing a reference in `reindl`](https://github.com/pvlib/pvlib-python/pull/2776)

While working on `reindl`, one of my mentors noticed that one of the references cited in the implementation was incorrect and opened an [issue](https://github.com/pvlib/pvlib-python/issues/2765) for it.

This wasn't originally part of my GSoC plan, but it was a nice opportunity to contribute a small fix. Along the way, a reviewer also pointed out that the references in the function didn't follow pvlib's preferred citation style, so I updated them to IEEE format as well.

This PR has already been merged.

### [Deprecation warning for `king`](https://github.com/pvlib/pvlib-python/pull/2783)

As discussed in my previous blog post, there has been ongoing discussion within the community about deprecating `irradiance.king`.

The model mixes sky diffuse and ground-reflected irradiance, its documentation can be misleading, and it does not appear to be widely used. Based on those discussions, I opened a PR to add a deprecation warning as a first step toward eventually removing the function.

## Current work

I'm currently working on the remaining transposition models:

 - `haydavies` - this function already supports `return_components`, but when `True` it includes a horizon brightening component which is always zero. Since the model does not actually include horizon brightening, this component doesn't provide any useful information. The plan is to remove it so that the returned components more accurately reflect the underlying model.
 - `isotropic` - isotropic is probably the simplest case implementation-wise. The model only represents isotropic diffuse irradiance, so the main work consists of adjusting the output structure and adding appropriate tests.
 - `klucher` - klucher is proving to be the most challenging model so far. While other models can be cleanly decomposed into separate diffuse components, in the Klucher formulation the terms are multiplicative, making this separation much less straightforward. Separation into components will require some assumptions and simplifications.
 - `perez` and `perez_driesse` - I had originally expected to leave `perez` and `perez_driesse` untouched since they already support `return_components`. However, because of the decision to standardize on regular dictionaries instead of OrderedDicts, I'll need to make this change there as well to keep the interface consistent across all models.

## Learning along the way

One aspect of the project that has been more challenging than I expected is everything surrounding the code itself: testing, linting, checks, code review feedback, and the overall GitHub contribution workflow.

Before GSoC, I had no experience contributing to large open-source projects, so there has definitely been a learning curve.

That said, this has probably been one of the most valuable parts of the experience so far. Getting familiar with these development practices is helping me understand not just how pvlib works, but also how collaborative scientific software is maintained and developed.

## Looking ahead

The next goal is to finish the remaining transposition models and ensure they all support a consistent `return_components` interface.

Once that work is complete, I'll move on to updating higher-level functions such as `get_sky_diffuse`, `poa_components`, and `get_total_irradiance` so they can properly handle the new outputs.

More updates soon!
