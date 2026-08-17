# Phase 1 Complete, and Into the Final Stretch

A lot has happened since my last update, and this time there is a particularly nice milestone to share: **Phase 1 of my GSoC project is now fully completed and merged!**

## Phase 1: Done!

The main goal of Phase 1 was to standardize the handling of diffuse irradiance components throughout `pvlib`.

All of the sky diffuse irradiance models now support returning separate diffuse components, with two exceptions: `king`, which is being deprecated, and `klucher`, whose formulation does not allow for a meaningful separation of the components.

The higher-level wrapper functions in `pvlib.irradiance` — `get_sky_diffuse`, `poa_components`, and `get_total_irradiance` — now support `return_components` as well. This means that users can access the diffuse components without having to use the individual transposition models directly.

This completes the first major milestone of the project: diffuse component handling is now available consistently from the individual transposition models through the higher-level irradiance workflow. This provides the foundation needed for the next phase, where these components will be used to improve optical loss modelling in `ModelChain`.

With that, it's time to move on to the more substantial part of the project.

## Phase 2: Component-Specific IAM

The main goal of Phase 2 is to extend `ModelChain` so that it can make use of these diffuse irradiance components and apply component-specific Incident Angle Modifier (IAM) calculations.

Currently, `ModelChain` calculates an IAM for the direct irradiance contribution, while diffuse irradiance is accounted for through the `fd` (diffuse fraction) term. In practice, `fd` is generally equal to 1, meaning that no optical loss is applied to the diffuse irradiance.

This is the limitation I want to address in Phase 2: by separating the diffuse irradiance into its different components, we can calculate appropriate IAM values for each component and incorporate their optical losses into the overall irradiance calculation.

Before getting to `ModelChain` itself, however, I ran into a small consistency issue in the existing diffuse IAM functions.

### Standardizing diffuse IAM outputs

The different diffuse IAM models weren't quite speaking the same language in terms of their outputs. `marion_diffuse` returns a dictionary containing IAM values for the sky diffuse, horizon, and ground components. Meanwhile, `martin_ruiz_diffuse` and `schlick_diffuse` return tuples containing IAM values for the sky diffuse and ground components.

Since these models don't all provide the same set of components, a dictionary is a much cleaner way of representing the results: each function can return the components it supports without relying on the position of values within a tuple.

I therefore started the main work in Phase 2 by opening a PR to standardize these outputs around dictionaries.

### Adding diffuse IAM to `Array` and `PVSystem`

I've also opened a PR adding support for diffuse IAM calculations to the `Array` and `PVSystem` classes.

The main change is the addition of a new `get_iam_diffuse` method, which acts as a counterpart to the existing `get_iam` method used for direct irradiance.

This is an important piece of groundwork because of the way these classes are connected: `ModelChain` relies on `PVSystem`, which in turn relies on `Array`. Once these lower-level classes can handle the new diffuse IAM calculations, the final step is to bring everything together within `ModelChain` itself.

## One More Step

With Phase 1 now complete and the foundations for Phase 2 in place, there is really only one major piece of the project left: updating `ModelChain` itself.

My GSoC is also approaching its final stretch, so I expect this to be my second-to-last blog post. Once the `ModelChain` work is complete, I'll write one final update covering the details of that last step, followed by a broader look back at the project and what I've learned throughout this incredible journey.

For now, though, there's still some code to write. On to `ModelChain`!
