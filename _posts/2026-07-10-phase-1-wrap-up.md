# Wrapping Up Phase 1

Most of the work on Phase 1 of the project is now complete, with only a few final loose ends remaining before I fully transition to Phase 2. Over the past few weeks, the focus has been on standardizing support for diffuse irradiance components across `pvlib`'s transposition models and ensuring that this information can be propagated through the higher-level irradiance APIs.

## Recently merged work
A couple of smaller pull requests have already been merged:
- Updating existing transposition models (`perez` and `perez_driesse`) to use standard dictionaries instead of `OrderedDict`s when `return_components=True`.
- Removing the horizon component from `haydavies`, which was always zero. All sky diffuse functions will return only the components supported by their respective model.

## The Klucher Question

The most interesting discussion during this phase ended up revolving around the Klucher transposition model.

Unlike the other models, Klucher's formulation is built around isotropic, horizon brightening, and circumsolar effects, but these contributions are not naturally separable because the correction terms are multiplied together:

$I_d = \mathrm{DHI} \frac{1 + \cos\beta}{2} \left(1 + F' \sin^3\left(\frac{\beta}{2}\right)\right)\left(1 + F' \cos^2\theta \sin^3\theta_z\right)$

If we define:

- Isotropic component: $I = \frac{1 + \cos\beta}{2}$
- Horizon brightening term: $h = F' \sin^3\left(\frac{\beta}{2}\right)$
- Circumsolar term: $c = F' \cos^2\theta \sin^3\theta_z$

then the expression becomes:

$I (1+h) (1+c) = I (1+c+h+hc)$

The challenge is the $hc$ term. Separating the irradiance into individual components would require deciding how to allocate this interaction term between the horizon and circumsolar contributions.

One possible approach would be to distribute it proportionally between the two components. However, doing so would introduce an assumption that is not present in the original formulation. Although this interaction term generally represents only a small fraction of the total irradiance (see plot below), it would still constitute a meaningful departure from the source material.

![image](https://private-user-images.githubusercontent.com/97249533/613810682-3f8e1f0a-0279-408d-b024-4ffe815c4678.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3ODM2OTYyNzIsIm5iZiI6MTc4MzY5NTk3MiwicGF0aCI6Ii85NzI0OTUzMy82MTM4MTA2ODItM2Y4ZTFmMGEtMDI3OS00MDhkLWIwMjQtNGZmZTgxNWM0Njc4LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjA3MTAlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwNzEwVDE1MDYxMlomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTMxZGMwMTk2Mzk1NjVhZDA0MWEzOWJjOTkxYTgzZTE3OTYyMTljYjIzOWE5MWVmNTE2YmQwM2ZlMDM2MTQ4NTgmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JnJlc3BvbnNlLWNvbnRlbnQtdHlwZT1pbWFnZSUyRnBuZyJ9.xJQMPk3nMB8amIjjIDTo0anBdIqYhTNpk-2Dq7jE_Zk)

After [discussion](https://github.com/pvlib/pvlib-python/issues/2794), the decision was made not to pursue this approach.

An additional observation was that Klucher's decomposition differs substantially from the behavior of the other transposition models in terms of the proportions between different components. The isotropic component alone is equal to the full isotropic model irradiance, with the other effects added on top. This suggests that even a simple isotropic + anisotropic decomposition may be less straightforward than initially expected.

![image](https://private-user-images.githubusercontent.com/97249533/615114280-48c6d611-0817-4c2e-8e11-6153206e49f4.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3ODM2OTYyNzIsIm5iZiI6MTc4MzY5NTk3MiwicGF0aCI6Ii85NzI0OTUzMy82MTUxMTQyODAtNDhjNmQ2MTEtMDgxNy00YzJlLThlMTEtNjE1MzIwNmU0OWY0LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjA3MTAlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwNzEwVDE1MDYxMlomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTI0NWJjM2EwODZlNDJkZjBkYmFjODcyMWNlODQ1NTYwNGUyZjIwYjU1ZjM5M2IyZTgyZTVlMzNmZTc0NGNlOTgmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JnJlc3BvbnNlLWNvbnRlbnQtdHlwZT1pbWFnZSUyRnBuZyJ9.xAx8aGJWp2bkdJVCPcIdXkfDRwycW9M8rpRoUQ5OLWE)

As a result, `klucher` will not support component decomposition through `return_components`.

## Should Total Diffuse Irradiance Be Returned?

Another discussion that (re)emerged during this phase concerns the behavior of `return_components` across all functions.

Currently, when `return_components=True`, the returned dictionary includes both the individual diffuse components and the total diffuse irradiance. An open question is whether the total diffuse value is actually necessary, since it can be reconstructed by summing the components.

This remains [an active topic of discussion](https://github.com/pvlib/pvlib-python/issues/2801). For now, the existing behavior has been preserved for all newly updated functions in order to maintain consistency with the models that already supported `return_components`. Since this is largely independent from the broader project, it is something that can be revisited later if necessary without affecting other implemented features.

## Extending the Irradiance Wrappers

The final task in Phase 1 is extending support for `return_components` to the higher-level wrapper functions in `pvlib.irradiance`:
- `get_sky_diffuse`
- `poa_components`
- `get_total_irradiance`

Adding this support will make component information available without requiring users to call the transposition models directly. It also lays the groundwork for Phase 2, where these diffuse components will need to be propagated through the `ModelChain` workflow.

There is currently an open PR implementing these changes, although it will remain pending until the earlier PRs are fully resolved.

## Looking Ahead

With most of the diffuse components work now complete, I'm currently getting started on Phase 2 of the project: integrating diffuse irradiance components into `ModelChain` and enabling component-specific optical loss calculations.

More on that soon!
