# First Steps

This is a quick update following my kickoff meeting with my project supervisors, where we discussed the initial direction of the project and the first implementation steps.

I will start by tackling phase 1 of the project: standardizing the outputs of diffuse transposition models.

Currently, only some models, such as [`perez`](https://pvlib-python.readthedocs.io/en/stable/reference/generated/pvlib.irradiance.perez.html), support a `return_components` parameter. When set to `True`, the function returns an `OrderedDict` or `DataFrame` containing the different diffuse irradiance components: isotropic, circumsolar and horizon, along with the total diffuse irradiance. This is also true for [`perez_driesse`](https://pvlib-python.readthedocs.io/en/stable/reference/generated/pvlib.irradiance.perez_driesse.html) and [`haydavies`](https://pvlib-python.readthedocs.io/en/stable/reference/generated/pvlib.irradiance.haydavies.html), although the horizon component for `haydavies` is always zero.

The remaining transposition models currently return only the total diffuse irradiance:

- `isotropic`
- `klucher`
- `reindl`
- `king`

The original idea was to fully standardize the outputs by having all functions support `return_components` and return the same four values as perez, perez_driesse, and haydavies. However, one thing to have in mind is that some transposition models do not inherently cover the three diffuse components (e.g., `isotropic` and `haydavies`). Our initial thought was to return zeros for missing components (as currently implemented for `haydavies`), but feedback from the pvlib GitHub community suggested avoiding this approach: since other transposition models that have yet to be implemented in pvlib-python may include different or additional diffuse components, it makes more sense to keep the interface flexible.

The approach we settled on is therefore:

- All transposition models will support `return_components`
- Each model will return whichever diffuse components it can physically represent

In practice, this means:

- Investigating which components can reasonably be extracted from `klucher` and `reindl`
- Updating these functions to expose those components
- Removing the empty horizon component from `haydavies`
- Having `isotropic` return an `OrderedDict` or `DataFrame` containing a single diffuse component

That last point may feel a bit silly, but it is probably the cleanest path toward a more consistent interface 🤓

As for `king`, [previous discussions](https://github.com/pvlib/pvlib-python/issues/2636) suggest that deprecating the model may be the best option. Its documentation is misleading, it mixes sky diffuse and ground-reflected irradiance, and it does not appear to be widely used. Adding a deprecation warning may therefore become part of this project.

Once these changes are in place, the next step will be updating the functions below so that they make use of the diffuse components:

- [`irradiance.get_sky_diffuse`](https://pvlib-python.readthedocs.io/en/stable/reference/generated/pvlib.irradiance.get_sky_diffuse.html)
- [`irradiance.poa_components`](https://pvlib-python.readthedocs.io/en/stable/reference/generated/pvlib.irradiance.poa_components.html)
- [`irradiance.get_total_irradiance`](https://pvlib-python.readthedocs.io/en/stable/reference/generated/pvlib.irradiance.get_total_irradiance.html)

Currently, since not all transposition models return diffuse components, the functions above only consider the total diffuse.

More updates soon!
