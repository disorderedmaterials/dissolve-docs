# Data Visualisation

- Status: **Proposed**
- Deciders: 
- Date: 24-01-2022

## Context and Problem Statement

Dissolve requires extensive 2D and 3D data visualisation encompassing, but not limited to, standard line graphs, histograms, 3D voxel surfaces, and representation of atomic and molecular coordinates and structure. Having this capability built-in to the code helps deliver seamless user experience as well as promoting efficiency.

## Decision Drivers

The current data visualisation is achieved by custom code leveraging `QOpenGLWidget` and offers:

1. Full 2D plotting (within a 3D scene) with standard and logarithmic axes, automatic tick determination, view interaction / zooming etc.
2. Full 3D plotting allowing axes to be shown e.g. for `Configurations` or calculated 3D analysis data.
3. Reasonable performance (use of vertex arrays etc.)

Drawbacks and limitations of this approach are:

1. Significant custom code beyond core functionality is required to handle OpenGL-related data (custom object instancing etc.) and provide basic primitives for (3D) rendering
2. Dependence on outdated `FTGL` library in order to provide text rendering
3. Relatively poor quality of on-screen results

Drivers for change are therefore:

- Reduction of custom code by moving to a suitable alternative platform
- Removal of the FTGL library dependence from the code in favour of more modern alternative (if not built-in to alternative platform)

## Considered Options

1. `QChart`
2. `Qt3D` (reimplement)
3. Use `QCustomPlot`
4. Matplotlib

## Decision Outcome

Chosen option: XXX

### Positive Consequences

- XXX

### Negative Consequences

- XXX

## Pros and Cons of the Options

### `QChart`

- Good, because requires no additional dependencies (module in Qt6)
- Good, because generates high quality graphs
- Bad, because some features do not behave as expected (for example log axes, rubberband zooming)
- Bad, because some features are not implemented and cannot be easily added (error bars)
- Bad, because no 3D support meaning existing custom code still required

### `Qt3D` (reimplement)

- Good, because requires no additional dependencies (module in Qt6)
- Good, because offers potential more performant solution to current 3D visualisation
- Bad, because significant work required to move across

### `QCustomPlot`

- Good, because generates high quality graphs
- Good, because has many plot types and options
- Good, because code is mature (> 10 years in development)
- Bad, because creates additional external dependence
- Bad, because no 3D support meaning existing custom code still required

### `Matplotlib`

- Good, because has many plot types and options
- Good, because has 3D graph support
- Bad, because limited other 3D support meaning existing custom code still required
- Bad, because creates additional external dependence
- Bad, because library based in Python and called via C++ wrapper
