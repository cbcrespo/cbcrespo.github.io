# GSoC 2026 Project Kickoff

Hi everyone! 👋

I’m excited to share that I’ll be participating in **Google Summer of Code 2026** with pvlib-python, working on a project focused on improving diffuse irradiance handling and optical loss modeling within the library.

My mentors will be [Adam R. Jensen](https://github.com/AdamRJensen) and [Rodrigo Amaro e Silva](https://github.com/ramaroesilva).

## Project Summary

The project centers around two closely related improvements in pvlib:

- Standardizing the outputs of diffuse transposition models
- Extending `ModelChain` to support more physically consistent optical loss calculations

At the moment, different diffuse irradiance models in `pvlib.irradiance` return different kinds of outputs. Some models can expose individual diffuse components — such as circumsolar, isotropic, and horizon diffuse irradiance — while others only provide total diffuse irradiance.

One of the goals of this project is to make these outputs more consistent across models through a shared interface.

Building on that, I’ll also be working on improvements to `ModelChain` so that different irradiance components can use different IAM (Incident Angle Modifier) models. This should make optical loss modeling more flexible and physically realistic, while keeping the current behavior fully backward compatible.

## Motivation

Diffuse irradiance is often treated as a single quantity, but in reality it comes from different regions of the sky and interacts differently with PV modules depending on incidence angle.

By exposing and handling these components more consistently, pvlib users will be able to:
- Access richer irradiance information from transposition models
- Use more advanced optical loss workflows
- Apply all existing IAM models more naturally within the `ModelChain`

## Looking Forward

I’m excited to contribute to pvlib-python and work with the community over the summer. I’ve used pvlib in both teaching and research, so it’s very rewarding to now contribute back to the project.

I will be posting bi-weekly updates throughout the summer to share progress, implementation details, and lessons learned.

Feel free to look at my full work plan and join the discussion over on [GitHub](https://github.com/pvlib/pvlib-python/issues/2750).

Looking forward to collaborating with everyone!
