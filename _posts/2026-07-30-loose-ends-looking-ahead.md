# Tying Up Loose Ends and Looking Ahead

Over the past couple of weeks, I've been wrapping up some remaining work from the first phase of the project while laying the groundwork for the second. I also got into some unexpected and interesting side quests.

## Phase 1 Progress

The first phase is now very close to completion.

Since my last update, the implementation of `return_components` for `irradiance.reindl` has been merged, which in turn allowed me to ensure the PR adding support for diffuse components to the wrapper functions (`get_sky_diffuse`, `poa_components`, and `get_total_irradiance`) is now in good shape. Those changes are now ready for review, leaving `isotropic` as the last model-specific PR still waiting to be merged.

One unexpected contribution also came out of the review process. While discussing the different transposition models, it became clear that the documentation wasn't always using the same terminology for the description of diffuse irradiance components. Although the implementations themselves were consistent, the docstrings were not. To address this, I opened a small cleanup PR to standardize the descriptions used throughout the relevant functions. It doesn't change any functionality, but it should make the API and documentation a little more coherent for users.

## Beginning Phase 2

With the diffuse component work nearly wrapped up, I've started focusing on the second half of the project: extending ModelChain to support component-specific IAM calculations.

Unlike the first phase, this isn't something that can be tackled with a series of mostly isolated pull requests. The changes span three closely connected classes: `Array`, `PVSystem`, and `ModelChain`. So before writing code, I wanted to make sure there was agreement on the overall design.

To that end, I opened an issue outlining the proposed implementation and, after feedback, decided to split the work into two stages: the first focusing on extending Array and PVSystem with the new IAM functionality, while the second will integrate those changes into ModelChain.

I've also opened a follow-up design discussion to iron out some of the API details. One example is how diffuse IAM calculations should fit into the existing `Array` interface. The current proposal is to keep the existing `get_iam` method for direct irradiance (which may or may not keep its name as is) and introduce a new `get_iam_diffuse` method capable of computing IAM values for the different diffuse components.

So far, a lot of Phase 2 has been about design rather than implementation. It's a different pace from Phase 1, but having these discussions early should make the actual coding much smoother.

### Side Quests

While waiting for the remaining Phase 1 work to make its way through review, I've had the chance to tackle a couple of smaller improvements that fit nicely within the broader scope of the project.

#### Adding the Schlick IAM Model to ModelChain

It turns out that `schlick` was the only direct IAM model already implemented in `pvlib` that wasn't available through `ModelChain`, so this is a relatively small change that helps make the available IAM models more complete. With this change, `ModelChain` can now use the full roster of available direct IAM models.

#### Making `marion_diffuse` More Efficient for Trackers

The other side quest has been more performance-oriented.

`marion_diffuse` is a function which computes diffuse IAM values by integrating over a solid angle for a given surface tilt. This works well for fixed-tilt systems, but becomes expensive for trackers, where the surface tilt is a timeseries of values rather than a single value.

The existing implementation effectively performs a new integration for every tilt angle (even repeating tilt angles). For long simulations, this can become both memory- and time-intensive.

To address this, I opened a PR implementing a new function, `marion_diffuse_tracking`, specifically for tracker systems. Instead of integrating for every individual tilt angle, it:

- precomputes the diffuse IAM values over a range of tilt angles (e.g. 0–90° or 0–180°, with a configurable resolution),
- performs the numerical integration only once for each sampled angle,
- and then interpolates those results to obtain IAM values for the actual tracker tilt time series.

The performance improvements are substantial. For one example using an 8760-point time series, the new implementation required only about 4% of the peak memory and 2% of the execution time of the original implementation.

One remaining point under discussion is whether these precomputed integrations should also be cached. Caching could avoid repeated calculations for systems with multiple arrays, but it is not currently used elsewhere in `pvlib`. Given that interpolation alone already provides most of the performance gains, it's still an open question whether caching is justified.

## What's Next?

With the remaining Phase 1 PRs approaching completion, I'm hoping to begin the core implementation of component-specific IAM support soon.

So far, a lot of the work has involved discussions around API design and implementation details. While this can sometimes feel slower than writing code, it's been a good reminder that designing an interface that fits naturally into an existing project is just as important as implementing the underlying functionality.

More updates soon!
