# Interactive Cosmic Ray Removal in Single Exposures

The installation of the **teareduce** package also includes an auxiliary
program called `tea-cleanest`, which enables the interactive cleaning of cosmic
rays. This code is inspired by the
[cleanest](https://cleanest.readthedocs.io/en/latest/) code ([Cardiel
2020](https://articles.adsabs.harvard.edu/pdf/2020ASPC..522..723C)) although
the approach to detecting cosmic rays differs. In particular, `tea-cleanest`
uses the [L.A.  Cosmic algorithm](http://www.astro.yale.edu/dokkum/lacosmic/)
([van Dokkum 2001](https://iopscience.iop.org/article/10.1086/323894/pdf)) to
identify pixels suspected of being affected by cosmic-ray hits.

```{warning}
This code is a prototype under development, and its functionality is
still limited.
```

```bash
(tea) $ tea-cleanest example_cr.fits
```

By pressing the `Run L.A. Cosmic` button, the cosmic ray detection process
begins. The detected pixels are grouped into cosmic rays, each consisting of
one or several connected pixels. The program displays a zoom around each cosmic
ray in a different window titled `Review Cosmic Rays`, with the suspicious
pixels preselected. The user can then check or uncheck the initial selection of
pixels to be interpolated by clicking with the mouse over the different pixels.

For the interpolation, it is possible to choose a polynomial fit along the X
direction, the Y direction, or a 2D plane. The pixels used for the
interpolation are searched for around the selected pixels.

The `deg=1, n=2` button indicates that a polynomial of degree `deg=1`
will be used for interpolation along the X (or Y) direction, taking `n=2`
pixels to the right and left (or above and below for Y interpolation). These
numbers can be modified by clicking the same button. If a 2D plane fit is
selected, the degree is always 1, but the number of surrounding points used
around the cosmic ray in each direction is still determined by the value of
`n`.

```{image} images/example.png
```

In the example shown in the figure above, one can see the preselected affected
pixels (bottom-left image, with the suspicious pixels marked with red x's)
and the result of the interpolation using a 2D plane (bottom-right image, where
red x's indicate the interpolated pixels and magenta dots are displayed over
the pixels used for fitting the 2D plane).

```bash
(tea) $ tea-cleanest --help
usage: tea-cleanest [-h] [--extension EXTENSION] [--output_fits OUTPUT_FITS]
                    input_fits

Interactive cosmic ray cleaner for FITS images.

positional arguments:
  input_fits            Path to the FITS file to be cleaned.

options:
  -h, --help            show this help message and exit
  --extension EXTENSION
                        FITS extension to use (default: 0).
  --output_fits OUTPUT_FITS
                        Path to save the cleaned FITS file
```
