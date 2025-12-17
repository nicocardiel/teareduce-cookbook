The following lines of code show how to run the cleaning of the first example image used in this tutorial. The results will be compared with the previously computed cleaned images, which serves as a test of the code.



```python
from astropy.io import fits
import numpy as np
from scipy import ndimage

import teareduce as tea
from teareduce import cleanest
```

Download the requested files


```python
for fname in [
    'docs/cleanest/examplecr1.fits',
    'docs/cleanest/examplecr2.fits',
    'docs/cleanest/examplecr1_cleaned_with_surface.fits',
]:
    tea.get_cookbook_file(fname)
```

    File examplecr1.fits already exists locally. Skipping download.
    File examplecr2.fits already exists locally. Skipping download.
    File examplecr1_cleaned_with_surface.fits already exists locally. Skipping download.


### Single execution of L.A.Cosmic

Read the input file


```python
with fits.open('examplecr1.fits') as hdul:
    data = hdul[0].data
```

**Detect CRs in image using L.A.Cosmic**

Here we are using the `cleanest.lacosmicpad()` function, which is a wrapper around the `cosmicray_lacosmic()` function from the **ccdproc** package. Our function pads the input image array before applying L.A.Cosmic. If `inbkg` or `invar` arrays are provided, they are also padded accordingly. After processing, the padding is removed to return an array of the original size.


```python
pad_width = 10
```


```python
cleandata_lacosmic, mask_crfound = cleanest.lacosmicpad(
    pad_width=pad_width,
    ccd=data,
    gain=1.0,
    readnoise=6.5,
    sigclip=5.0,
    sigfrac=0.3,
    objlim=5.0,
    niter=4,
    fsmode='median',
    verbose=True,
)
```

    Starting 4 L.A.Cosmic iterations
    Iteration 1:
    6419 cosmic pixels this iteration
    Iteration 2:
    353 cosmic pixels this iteration
    Iteration 3:
    29 cosmic pixels this iteration
    Iteration 4:
    17 cosmic pixels this iteration


Check image shapes


```python
data.shape, cleandata_lacosmic.shape, mask_crfound.shape
```




    ((1596, 2016), (1596, 2016), (1596, 2016))



**Clean the detected CRs**

Using the cosmic-ray pixel mask, we can proceed to interpolate the cosmic rays using the desired method. 

The cosmic-ray pixels are initially dilated by the specified number of pixels (0 means no dilation). The resulting flagged pixels are grouped into cosmic-ray features. Each cosmic-ray feature is then interpolated using the specified interpolation method.


Note that the interpolation methods `lacosmic` and `auxfile`,available in the interactive use of **tea-cleanest** are not implemented in this function because both cases are simply an inmediate replacement of the cosmic ray pixels in the data array by the corresponding pixels in another array using the mask array. Therefore, these two methods do not require any interpolation algorithm.


```python
data_cleaned, masked = cleanest.interpolate(data, mask_crfound, interp_method='s', npoints=2, degree=1, debug=True)
```


<pre style="white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace">Number of cosmic ray pixels to be cleaned: <span style="color: #008080; text-decoration-color: #008080; font-weight: bold">6590</span>
</pre>




<pre style="white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace">Number of cosmic rays <span style="font-weight: bold">(</span>grouped pixels<span style="font-weight: bold">)</span><span style="color: #808000; text-decoration-color: #808000">...</span>:  <span style="color: #008080; text-decoration-color: #008080; font-weight: bold">749</span>
</pre>



    100%|█████████████████████████████████████████| 749/749 [00:19<00:00, 38.64it/s]



<pre style="white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace">Number of cosmic rays cleaned<span style="color: #808000; text-decoration-color: #808000">............</span>:  <span style="color: #008080; text-decoration-color: #008080; font-weight: bold">749</span>
</pre>



### Double execution of L.A.Cosmic

Although the prior execution of L.A.Cosmic has allowed the removal of cosmic rays in the initial image, visual inspection reveals that the tails of some cosmic rays (i.e., pixels whose signal is less strongly affected, but still affected) have not been interpolated. The way **tea-cleanest** addresses this situation is by running L.A.Cosmic a second time, using a lower threshold to detect cosmic rays. To prevent the appearance of many false positives, the code enforces that the newly detected cosmic-ray pixels must be in contact with pixels already identified in the first execution of L.A.Cosmic.



```python
cleandata_lacosmic2, mask_crfound2 = cleanest.lacosmicpad(
    pad_width=pad_width,
    ccd=data,
    gain=1.0,
    readnoise=6.5,
    sigclip=3.0,      # modified
    sigfrac=0.3,
    objlim=5.0,
    niter=4,
    fsmode='median',
    verbose=True,
)
```

    Starting 4 L.A.Cosmic iterations
    Iteration 1:
    9375 cosmic pixels this iteration
    Iteration 2:
    810 cosmic pixels this iteration
    Iteration 3:
    107 cosmic pixels this iteration
    Iteration 4:
    38 cosmic pixels this iteration



```python
np.sum(mask_crfound), np.sum(mask_crfound2)
```




    (np.int64(6590), np.int64(9954))



Merge peak and tail masks


```python
mask_crfound = cleanest.merge_peak_tail_masks(
    mask_peaks=mask_crfound, 
    mask_tails=mask_crfound2,
    verbose=True
)
```


<pre style="white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace">Number of cosmic ray pixels <span style="font-weight: bold">(</span>peaks<span style="font-weight: bold">)</span><span style="color: #808000; text-decoration-color: #808000">...............</span>: <span style="color: #008080; text-decoration-color: #008080; font-weight: bold">6590</span>
</pre>




<pre style="white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace">Number of cosmic ray pixels <span style="font-weight: bold">(</span>tails<span style="font-weight: bold">)</span><span style="color: #808000; text-decoration-color: #808000">...............</span>: <span style="color: #008080; text-decoration-color: #008080; font-weight: bold">9954</span>
</pre>




<pre style="white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace">Number of cosmic ray pixels <span style="font-weight: bold">(</span>merged peaks &amp; tails<span style="font-weight: bold">)</span>: <span style="color: #008080; text-decoration-color: #008080; font-weight: bold">7648</span>
</pre>



We then proceed to interpolate the cosmic-ray pixels using the new mask.



```python
data_cleaned, masked = tea.cleanest.interpolate(data, mask_crfound, interp_method='s', npoints=2, degree=1, debug=True)
```


<pre style="white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace">Number of cosmic ray pixels to be cleaned: <span style="color: #008080; text-decoration-color: #008080; font-weight: bold">7648</span>
</pre>




<pre style="white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace">Number of cosmic rays <span style="font-weight: bold">(</span>grouped pixels<span style="font-weight: bold">)</span><span style="color: #808000; text-decoration-color: #808000">...</span>:  <span style="color: #008080; text-decoration-color: #008080; font-weight: bold">748</span>
</pre>



    100%|█████████████████████████████████████████| 748/748 [00:19<00:00, 38.30it/s]



<pre style="white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace">Number of cosmic rays cleaned<span style="color: #808000; text-decoration-color: #808000">............</span>:  <span style="color: #008080; text-decoration-color: #008080; font-weight: bold">748</span>
</pre>



### Compare with reference image


```python
reference_filename = 'examplecr1_cleaned_with_surface.fits'
tea.get_cookbook_file(f'docs/cleanest/{reference_filename}')
```

    File examplecr1_cleaned_with_surface.fits already exists locally. Skipping download.



```python
with fits.open(reference_filename) as hdul:
    reference_data = hdul[0].data
    reference_mask = hdul["CRMASK"].data
```

Check data array


```python
np.array_equal(data_cleaned, reference_data)
```




    True



Check CR mask array


```python
np.array_equal(masked, reference_mask)
```




    True


