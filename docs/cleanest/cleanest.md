# Interactive Cosmic Ray Removal in Single or Double Exposures

The installation of the **teareduce** package also includes an auxiliary
program called **tea-cleanest**, which enables the interactive cleaning of
cosmic rays. This code is inspired by the
[cleanest](https://cleanest.readthedocs.io/en/latest/) code
{cite}`2020ASPC..522..723C`, although the approach to detecting cosmic rays
differs. In particular, **tea-cleanest** allows to use the following
algorithms to identify pixels suspected of being affected by cosmic-ray hits:

- [L.A.Cosmic](http://www.astro.yale.edu/dokkum/lacosmic/): see
{cite}`2001PASP..113.1420V`
- [PyCosmic](https://github.com/brandherd/PyCosmic.git): see
{cite}`2012A&A...545A.137H`
- [deepCR](https://deepcr.readthedocs.io/en/latest/): see
{cite}`2020ApJ...889...24Z`
- [Cosmic-CoNN](https://cosmic-conn.readthedocs.io/en/latest/index.html): see
{cite}`2023ApJ...942...73X`

```{warning}
If you use **tea-cleanest**, please remember to cite this web and the
references corresponding to the algorithms employed.
```

The program also allows alternatively loading a mask that indicates the
location of the cosmic-ray pixels. In this case, the mask will be read from an
external FITS file containing an array of integers, where any pixel with a
value other than 0 is considered a cosmic-ray pixel.

The identified cosmic rays can be interpolated either automatically or under
user supervision. To replace the signal in the cosmic-ray pixels, information
from the neighboring pixels is used, and any of the following methods may be
applied:

- one-dimensional interpolation: along the X-axis or the Y-axis
- a constant average value: the mean or median
- 2D interpolation using a plane (degree 1)
- the values determined by *L.A.Cosmic*, *PyCosmic* or *deepCR*
- the values computed using the *maskfill* method described by {cite}`2024PASP..136c4503V` 
- the values present in an auxiliary image


```{note}
Although **tea-cleanest** was initially developed to clean individual
exposures, it now also allows the use of a second (auxiliary) exposure, whose
information can be used to replace pixels affected by cosmic rays in the first
image. 

In the case of equivalent double exposures, it is possible to remove the cosmic
rays from each exposure employing the information in the other exposure as an
auxiliary image. By swapping the roles of main and auxiliary image, we can
remove the cosmic rays from the second exposure as well. This method should
work properly as long as the number of cosmic rays is not so high that the same
pixel is likely to be affected by a cosmic ray in both exposures.

When three or more equivalent exposures are available, **tea-cleanest** can be
used by first cleaning a pair of those images and then using the result as the
auxiliary image for cleaning the remaining exposures. In any case, in this
scenario it may be preferable to use the median of the available exposures
instead of **tea-cleanest**.  It is also possible to use slightly more
sophisticated algorithms, such as the one implemented in
[numina-crmasks](https://guaix-ucm.github.io/numina-tools/crmasks/crmasks.html).
```

````{warning}
If you are planning to use **tea-cleanest**, you need to install the package
**teareduce** with extra dependencies. In this case employ:

```console
(venv_tea) $ pip install 'teareduce[cleanest]'
```

In addition, to use the *PyCosmic* method, you must manually install its
corresponding package; otherwise, **tea-cleanest** will run, but this method
will not be available. Since the package is not published on PyPI, it can be
installed directly from GitHub. To do so, use a recent fork of the repository
by running:

```console
$ pip install git+https://github.com/nicocardiel/PyCosmic.git@test
```


````

It is possible to check whether **tea-cleanest** is correctly installed by
running

```{include} files/help_tea-cleanest.md
```

## Simple execution (single image)

We can quickly run **tea-cleanest** by providing the name of the FITS file to
be cleaned (if no file name is given, the program will allow us to choose a
file through the operating system’s file-open interface).

Download first the sample file
[examplecr1.fits](https://raw.githubusercontent.com/nicocardiel/teareduce-cookbook/main/notebooks/cleanest/examplecr1.fits).

```console
(venv_tea) $ tea-cleanest examplecr1.fits
```

```{Note}
Note: when a character appears in brackets in the text of any button in the
**tea-cleanest** graphical interface, that key on the keyboard becomes a
shortcut for pressing the corresponding button. This is especially useful in
the `Review Cosmic Rays` window (see below), where the user can quickly clean
the different cosmic rays without having to move the mouse to click the various
buttons.
```

The upper part of the graphical interface displays four rows of buttons, which
we can use to control the program’s execution.

```{image} images/single_1.png
```

The most convenient way to work with **tea-cleanest** is to first perform a
search for pixels suspected of having been affected by cosmic rays. This task
can be carried out by running any of the available detection algorithms:
*L.A.Cosmic*, *PyCosmic*, *deepCR* or *Cosmic-CoNN*. To do so, simply press the
corresponding `Run <method>` button in the upper upper button row.

### `Run L.A.Cosmic`

After pressing the `Run L.A.Cosmic` button, a new window is then displayed,
allowing us to define the parameters to be used.

```{image} images/single_2_lacosmic.png
```

That window is divided into three parameter blocks:

- **L.A.Cosmic Parameters**: this section displays the parameters used by the
  `cosmicray_lacosmic()` function. Note that each parameters appear in two
  columns: run1 and run2. The idea here is to run this function twice. In run1,
  a set of parameters is used to identify pixels that most clearly stand out as
  likely to have been affected by cosmic rays. In run2, some parameters can be
  modified to try to find pixels whose signal has been less strongly affected
  by cosmic rays. Although this second pass may generate a large number of
  false positives, the **tea-cleanest** code only adds to the cosmic-ray-pixel
  mask the run2 pixels that lie in the neighborhood of the run1 cosmic-ray
  pixels. Users are advised to consult the documentation of the
  [`cosmicray_lacosmic()` function](https://ccdproc.readthedocs.io/en/latest/api/ccdproc.cosmicray_lacosmic.html) provided by the **ccdproc** package.
  

- **Additional Parameters**: here we can define a dilation factor that
  surrounds each cosmic-ray pixel with a ring of neighboring pixels, which can
  help define a more generous region when constructing the mask of
  cosmic-ray–affected pixels. The second parameter, *border padding*, allows
  the cleaned image to be extended beyond its physical edge, making it easier
  to detect cosmic-ray pixels that are located right at the borders of the
  images. If this padding is zero, the `cosmicray_lacosmic()` function is
  unable to detect cosmic-ray pixels at the edges of the image.

- **Region to be examined**: here you can define an arbitrary rectangular
  region in which to search for cosmic-ray pixels. By default, the values
  correspond to examining the entire image.

After clicking the `OK` button, **tea-cleanest** runs L.A.Cosmic twice and
displays the cosmic-ray pixels in the main window, overlaying them with a red
marker.

```{image} images/single_3_lacosmic.png
```

```{note}
The detected pixels are grouped into cosmic rays, each consisting of
one or several connected pixels.
```

At this point, the user can choose to automatically clean all detected cosmic
rays by selecting the `Replace detected CRs` button, or perform an individual
analysis of each cosmic ray. For the latter option, you can sequentially go
through all detected cosmic rays by clicking the `Review detected CRs` button,
or you can click on the image, and **tea-cleanest** will show the location of
the cosmic ray closest to the cursor position (if there is no cosmic ray near
the cursor, the program will show what is happening around the brightest
point).

#### Automatic cleaning of the detected CRs

If we choose to click the `Replace detected CRs` button, the program will
display a new window *Cleaning Parameters* that allows us to define the type of
interpolation to use for replacing the signal of the cosmic-ray pixels.

```{image} images/single_4a_lacosmic.png
```

The new window is divided into four sections:

- **Select Cleaning Method**: here we can choose to interpolate along the
  X-axis (`x_interp.`), the Y-axis (`y_interp.`), fit a plane
  (`surface_interp.`), or replace the affected pixels using the `mean`,
  `median`, the values computed by L.A.Cosmic (`lacosmic`), or the result
  computed by the `maskfill` method described by {cite}`2024PASP..136c4503V`.

- **Fitting Parameters**: in this section, we can define the number of
  points to use in the neighborhood of the cosmic ray (to the right, left, or
  around it, depending on the chosen method) and the degree of the polynomial
  to use (for interpolations along X or Y).

- **Maskfill Parameters**: here are the four parameters relevant to employing
  the maskfill algorithm. It is recommended to consult the information on the
  [maskfill website](https://github.com/dokkum/maskfill/tree/main), as well as
  the documentation of the
  [`maskfill()` function](https://github.com/dokkum/maskfill/blob/main/maskfill/maskfill.py).


- **Region to be Examined**: the rectangular region in which the interpolation
  of the cosmic-ray pixels will be applied.

After choosing, for example, `[s]urface interp.`, the program shows the
progress of the cosmic-ray interpolation through a window that displays the
relevant execution times.

```{image} images/single_5a_lacosmic.png
```

At the end of the interpolation process, a window appears showing a summary of
the results.

```{image} images/single_6a_lacosmic.png
```

After closing the previous window, the cosmic-ray–cleaned image is displayed.

```{image} images/single_7a_lacosmic.png
```

```{note}
After performing the cosmic-ray cleaning, the user should not forget to save
the result by clicking the `Save cleaned FITS` button in the main window.

**Important**: the image saved in this way will contain the same extensions as
the original image, with the cleaned image stored in the same extension that
was specified when reading the original image. In addition, **tea-cleanest**
adds an extra FITS extension called `CRMASKS`, which contains a mask of the
interpolated pixels: the only possible values in this mask are 0 (unmodified
pixels) and 1 (pixels interpolated during the cosmic-ray correction).

Alternatively, by clicking the `Save CR mask` button, it is also possible to
save a FITS file containing only the binary mask that indicates the pixels
detected as cosmic rays (the FITS file contains only the primary extension with
that array).
```

#### Manual cleaning of the detected CRs

If instead of the previous method the user chooses the `Review detected CRs`
button in the main window, the program displays a new window *Review Cosmic
Rays* where we can sequentially go through the detected cosmic rays and, based
on the preselected affected pixels in each case, make the appropriate
modifications before applying the desired interpolation method.

```{image} images/single_4b_lacosmic.png
```

In this window, the selected cosmic ray is displayed, with each suspected pixel
marked with a red X. The user can then mark or unmark suspected pixels using
the mouse. Interpolation can be performed using any of the following methods:

- `[x] interp.`: one-dimensional interpolation along the X-axis
- `[y] interp.`: one-dimensional interpolation along the Y-axis
- `[s]urface interp.`: 2D interpolation using a plane (degree 1)
- `me[d]ian`: replacement with a constant: the median of the pixels surrounding
  the cosmic ray
- `[m]ean`: same using the mean value
- `[L].A.Cosmic`, `PyCosmic` or `deepCR`: the values determined by
  *L.A.Cosmic*, *PyCosmic* or *deepCR* (only the option corresponding to the
  method used for detecting the cosmic rays will be available.)
- `mas[k]fill`: the values computed using the *maskfill* method described by
  {cite}`2024PASP..136c4503V` 
- `[a]ux. data`: the values present in an auxiliary image (available only if an
  auxiliary image has been loaded)


Before moving on to the next cosmic ray,
the user can undo the performed interpolation using the `[r]estore CR
data` button. Once we move to a new cosmic ray, it is no longer possible to go
back to previous ones.

The top-left button allows modifying the number of neighboring points used in
the interpolation, as well as the polynomial degree (for `[x] interp.` and `[y]
interp.` cases).  Analogously, the button `Maskfill params.` allows to modify
the four parameters employed by the maskfill algorithm.

If in the example shown we choose to replace the affected pixels with a plane
fit (by clicking the `[s]urface interp.` button), the result is as shown below.

```{image} images/single_5b_lacosmic.png
```

In this last image, the pixels used for the plane fit are shown with
superimposed filled magenta circles, while the interpolated pixels are indicated
with filled orange circles.

Clicking the `[c]ontinue` button allows you to proceed to the next detected
cosmic ray. The user can exit the cleaning sequence at any time by clicking the
`[e]xit review` button.

```{warning}
Although the user can directly start selecting pixels to interpolate by first
activating the cursor (clicking the `[c]ursor OFF` button, which should change
to `[c]ursor ON`) and then clicking on a suspected pixel in the main window
with the mouse, doing this without first running L.A.Cosmic has the drawback
that **tea-cleanest** will only show a single pixel as a preselection for
interpolation in the `Review Cosmic Rays` window. For this reason, it is more
convenient to run L.A.Cosmic beforehand to generate a mask of suspected pixels,
so that **tea-cleanest** preselects all neighboring pixels likely affected by
the same cosmic ray.

Alternatively, the user can use the `Load CR mask` button in the main window to
load a cosmic-ray mask from a FITS image (0: pixels not affected by cosmic
rays; 1: pixels affected by cosmic rays; the image extension must have BITPIX
equal to 8 or 16).  A dilation factor can be applied to this mask. Once it is
loaded, the affected pixels are regrouped into cosmic rays, and their removal
can proceed using either of the two methods described above: `Replace detected
CRs` or `Review detected CRs`.
```

### `Run PyCosmic`

Similar to the previous case, after clicking the `Run PyCosmic` button in the
main window, a new window opens showing the parameters to be configured for
running this algorithm.

```{image} images/single_2_pycosmic.png
```
Once the cosmic-ray pixels have been detected, the procedure to follow for
their interpolation is the same as that described for the L.A.Cosmic algorithm.

### `Run deepCR`

After clicking `Run deepCR`, the program will display two dialog windows asking
for the probability threshold to consider a pixel as a cosmic ray (default
0.5) and an optional dilation factor (default zero).

Once the cosmic-ray pixels have been detected, the procedure to follow for
their interpolation is the same as that described for the L.A.Cosmic algorithm.

### `Run Cosmic-CoNN`

After clicking `Run Cosmic-CoNN`, the program will display two dialog windows
asking for the probability threshold to consider a pixel as a cosmic ray
(default 0.5) and an optional dilation factor (default zero).

Once the cosmic-ray pixels have been detected, the procedure to follow for
their interpolation is the same as that described for the L.A.Cosmic algorithm.

## Advanced execution (two images)

In the case of having two equivalent exposures, we can use one as the main
image to be cleaned of cosmic rays and the second as an auxiliary image (whose
information can be used to replace the cosmic-ray pixels in the first image).
After cleaning the first image, the procedure can be repeated by swapping the
roles of the two exposures, so that in the end we obtain a cosmic-ray–cleaned
version of both original exposures.

Download the second sample file
[examplecr2.fits](https://raw.githubusercontent.com/nicocardiel/teareduce-cookbook/main/notebooks/cleanest/examplecr2.fits)
(you can also download now the first sample file
[examplecr1.fits](https://raw.githubusercontent.com/nicocardiel/teareduce-cookbook/main/notebooks/cleanest/examplecr1.fits)
if you did not do it previously).


```console
(venv_tea) $ tea-cleanest examplecr1.fits --auxfile examplecr2.fits
```

The auxiliary image can be loaded directly when starting **tea-cleanest** (as
shown in the previous command line), or it can be read from any FITS file
during program execution by using the `Load auxdata` button in the main window.

Running L.A.Cosmic is done in the same way as previously explained. However,
when using the `Replace detected CRs` button, an additional option now appears
among the available interpolation methods: `auxdata` (the selection of this
method is not possible when running the code in single-image mode). Choosing
this new interpolation method causes **tea-cleanest** to replace all selected
cosmic-ray pixels using the signal from the corresponding pixels in the
auxiliary image.

```{image} images/double_4a_lacosmic.png
```

On the other hand, if the user chooses the `Review detected CRs` option, the
`[a]ux. data` button will be active in the *Review Cosmic Rays* window, giving
access to interpolation by replacing the pixels with the signal from the
auxiliary image. Additionally, this window simultaneously displays the image to
be corrected (left panel) and the auxiliary image (right panel).

```{image} images/double_4b_lacosmic.png
```

The result of using `[a]ux. data` as the interpolation method in the previous
example is shown in the following figure.

```{image} images/double_5b_lacosmic.png
```

Note that in this case, information from neighboring pixels is not used for the
pixels previously selected for interpolation (filled orange circles are shown,
but no filled magenta circles).


## Description of the main window button actions

This section briefly describes the functionality associated with each button
that appears in the main window of **tea-cleanest**.

**First row of buttons**

- `Run L.A.Cosmic`: runs the generation of a mask of pixels affected by cosmic
  rays using [L.A.Cosmic](http://www.astro.yale.edu/dokkum/lacosmic/): see
  {cite}`2001PASP..113.1420V`

- `Run PyCosmic`: runs the generation of a mask of pixels affeced by cosmic
  using [PyCosmic](https://github.com/brandherd/PyCosmic.git): see
  {cite}`2012A&A...545A.137H` rays using 

- `Run deepCR`: runs the generation of a mask of pixels affeced by cosmic using
  [deepCR](https://deepcr.readthedocs.io/en/latest/): see
  {cite}`2020ApJ...889...24Z`

- `Run Cosmic-CoNN`: runs the generation of a mask of pixels affeced by cosmic
  using [Cosmic-CoNN](https://cosmic-conn.readthedocs.io/en/latest/index.html):
  see {cite}`2023ApJ...942...73X`

- `Stop program`: stops program execution.

**Second row of buttons**

- `Load auxdata`: loads an auxiliary image whose information can be used
  to replace pixels affected by cosmic rays in the main image.

- `Load CR mask`: loads a mask of pixels affected by cosmic rays
  from a FITS file. You can specify the extension where the mask is located.

- `Replace detected CRs`: enables automatic removal of all pixels affected
  by cosmic rays using any of the available interpolation methods.

- `Review detected CRs`: provides access to the individual visualization of each
  detected cosmic ray. This option allows interactive interpolation of the
  different cosmic rays.

**Thrid row of buttons**

- `[c]ursor: OFF`: toggles the use of the cursor to select pixels for
   interactive interpolation after clicking anywhere on the image shown in the
   main window.

- `[t]oggle data` (button active only when an auxiliary image has been loaded):
  quickly switches the image displayed in the main window, alternating between
  the original image and the auxiliary image.

- `[a]spect: equal`: displays the image shown in the main window using aspect
  `equal` or `auto` (the latter distorts the X/Y pixel ratio but maximizes use
  of the plotting window).

- `Save cleaned FITS`: saves the latest available version of the cleaned image.

- `Save CR mask`: saves the latest available version of the CR mask.

**Fourth row of buttons**

- `vmin: ???`: the `vmin` value used by matplotlib.

- `vmax: ???`: the `vmax` value used by matplotlib.

- `minmax[,]`: sets `vmin` and `vmax` to the minimum and maximum values,
  respectively, of the image shown in the main window.

- `zscale[/]`: adjusts `vmin` and `vmax` to use values close to the median of
  the image (IRAF-style).

- `CR overlay: ON`: toggles overlaying the detected cosmic rays on the image
  displayed in the main window.

- `Help`: shows help information for all buttons.

## Using the cleanest functionality programmatically

It is possible to use part of the functionality provided by **tea-cleanest**
programmatically from Python code, which is convenient if you need to automate
the workflow or work from a Jupyter notebook. See section [Interactive use of
cleanest functionality](../../notebooks/cleanest/interactive_cleanest).
