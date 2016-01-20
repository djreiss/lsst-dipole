## Dipole measurement and classification

---

* [Additional random dipole characterization thoughts](README.md) in no particular order.

---

- [Summary of current implementation (`ip_diffim` `dipoleMeasurement`)](#summary-of-current-implementation-ip_diffim)
- [Evaluation of dipole fitting accuracy](#evaluation-of-dipole-fitting-accuracy)
- [Putative issues with the `dipoleMeasurement` PSF fitting algorithm](#putative-issues-with-the-ip_diffim-psf-fitting-algorithm)
- [Generic dipole fitting complications](#generic-dipole-fitting-complications)
- [Possible solutions and tests](#possible-solutions-and-tests)

---

### Summary of current implementation (`dipoleMeasurement`)

The current dipole measurement task is intialized by `SourceDetection` being performed in both positive and negative modes to identify significant pos. and neg. sources. These pos. and neg. source catalogs are merged to identify candidate dipoles with overlapping footprints. The measurement task then performs two separate measurements on these dipole candidates:

1. A "naive" dipole measurement which computes a 3x3 weighted moment around the nominal centroids of each peak in the Source Footprint. It estimates the pos./neg. fluxes as pos./neg. sums of the pixel values within the merged footprint.
2. Measurements resulting from a joint-Psf model fit to the negative and positive lobe simultaneously. This fit simultaneously solves for the negative and positive lobe centroids and fluxes using non-linear
least squares minimization.

The two measurements are performed independently and do not (AFAICT) inform each other; in other words the centroids and fluxes from the naive dipole measurement are not used to initialize the starting parameters in the least-squares minimization.

---
### Evaluation of dipole fitting accuracy

We implemented a faux dipole generation routine (separate from lsst code, i.e., using `numpy`) with realistic non-background-limited (Poisson) noise. We then implemented a separate 2-D dipole fitting function in "pure" python (we used the `lmfit` package, which is basically a wrapper around `scipy.optimize.leastsq()`). The dipole (and the function which is minimized) is generated using a 2-D double Gaussian. Interesting findings include:

1. The "pure python" optimization is nearly twice as fast as the `dipoleMeasurement` implementation. Some of the reasons for this may lie in putative `dipoleMeasurement` issues described above.
2. Using similar constraints (i.e., none), the "pure python" optimization results in model fits with similar characteristics to those of the `dipoleMeasurement` code. For example, a plot of recovered x-coordinate of dipoles of fixed flux with gradually increasing separations is shown [here](notebooks/7b.%20test%20new%20(fixed!)%20and%20ip_diffim%20dipole%20fitting%20on%20same%20sources-Copy3%20(more%20realistic%20noise)_files/7b.%20test%20new%20(fixed!)%20and%20ip_diffim%20dipole%20fitting%20on%20same%20sources-Copy3%20(more%20realistic%20noise)_20_2.png):

<a name="figure1"></a>
![Figure 1](notebooks/7b.%20test%20new%20(fixed!)%20and%20ip_diffim%20dipole%20fitting%20on%20same%20sources-Copy3%20(more%20realistic%20noise)_files/7b.%20test%20new%20(fixed!)%20and%20ip_diffim%20dipole%20fitting%20on%20same%20sources-Copy3%20(more%20realistic%20noise)_20_2.png)

Note in all figures, including this one, "New" refers to the "pure python" dipole fitting routine, and "Old" refers to the fitting in the existing `dipoleMeasurement` code.

A primary result of comparisons of both dipole fitting routines showed that if unconstrained, they would have difficulty finding accurate fluxes (and separations) at separations smaller than ~1 FWHM. This is best shown [here](notebooks/7b.%20test%20new%20(fixed!)%20and%20ip_diffim%20dipole%20fitting%20on%20same%20sources-Copy3%20(more%20realistic%20noise)_files/7b.%20test%20new%20(fixed!)%20and%20ip_diffim%20dipole%20fitting%20on%20same%20sources-Copy3%20(more%20realistic%20noise)_23_1.png), which shows the fitted dipole fluxes as a function of dipole separation for a number of realizations per separation (and input flux of 3000).

<a name="figure2"></a>
![Figure 2](notebooks/7b.%20test%20new%20(fixed!)%20and%20ip_diffim%20dipole%20fitting%20on%20same%20sources-Copy3%20(more%20realistic%20noise)_files/7b.%20test%20new%20(fixed!)%20and%20ip_diffim%20dipole%20fitting%20on%20same%20sources-Copy3%20(more%20realistic%20noise)_23_1.png)

Here, 'pos'itive and 'neg'ative lobe parameters are shown side-by-side in the same (positive) flux axis.

Below we investigate this issue and find that it arises from the extreme covariance between the dipole separation and flux parameters, which exacerbates the optimizers at low signal-to-noise.

---

### Putative issues with the `dipoleMeasurement` PSF fitting algorithm

The PSF fitting is slow. It seems to take ~60MS for some fits (especially for closely-separated dipoles).

Why is it slow? Thoughts on possible reasons (they will need to be evaluated further if deemed important):

1. `PsfDipoleFlux::chi2()` computes the PSF *image* (pos. and neg.) to compute the model, rather than using something like `afwMath.DoubleGaussianFunction2D()`. Or if that is not possible (may need to use a pixelated input PSF) then potentially speed up the computation of the dipole model image (right now it uses multiple vectorized `afw::Image` function calls).
2. It spends a lot of time floating around near the minimum and perhaps can be cut off more quickly (note this may be exacerbated by (1.)).
3. Perhaps the starting parameters (derived from the naive coordinates) could be made more accurate. At least it looks like the starting flux values are being initialized from the peak pixel value in the footprint, rather than (an estimate of) the source flux.
4. `chi2` is computed over the entire footprint bounding box (confirm this?) rather than the inner 2,3,4, or 5 sigma of the PSF.
5. Some calculations are computed each time during minimization (in `chi2` function) that can be moved outside (not sure if these calc's are really expensive though).
6. There are no constraints on the parameters (e.g. `fluxPos` > 0; `fluxNeg` < 0; possibly `fluxPos` = `fluxNeg`; centroid locations, etc.). Fixing this is also likely to increase fitting accuracy (see below).

Note: It seems that the dipole fit is a lot faster for dipoles of greater separation than for those that are closer (it seems the optimization [via `minuit`] takes longer to converge).

---

### Generic dipole fitting complications

There is a degeneracy in dipole fitting between closely-separated dipoles from bright sources and widely-separated dipoles from faint sources. This is further explored using 1-d simulated dipoles in [this notebook](notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise.ipynb).

[Here](notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_4_0.png) is an example:

<!--
![Figure 1](notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_4_0.png)
-->
<a name="figure3"></a>
<img src="notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_4_0.png" width="60%">

There are many such examples, and this strong covariance between amplitude (or flux) and dipole separation is most easiest shown by plotting error contours from a [least-squares fit to simulated 1-d dipole data](notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_7_2.png):

<!--
![Figure2](notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_7_2.png)
-->
<a name="figure4"></a>
<img src="notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_7_2.png" width="60%">

Here are the error contours, where the blue dot indicates the input parameters (used to generate the data), the yellow dot shows the starting parameters for the minimization and the green dot indicates the least-squares parameters:

<a name="figure5"></a>
<img src="notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_8_2.png" width="60%">

#### Possible solutions and tests

This degeneracy is a big problem if we are going to fit dipole parameters using the subtracted data alone. Three possible solutions are:

1. Use starting parameters and parameter bounds based on measurements from the pre-subtracted images (obs. and template) to constrain the dipole fit.
2. Include the pre-subtracted image data in the fit to constrain the minimization.
3. A combination of (1.) and (2.).

It is noted that these solutions will not help in cases of dipoles detected on top of bright backgrounds (or backgrounds with large gradients), such as cases of a faint dipole superimposed on a bright-ish background galaxy. But these cases will be rare, and I believe we can adjust the weighting of the pre-subtracted image data (i.e., in [2] above) to compensate (see below).

As an example, I performed a fit to the same data as shown above, but included the "pre-subtracted" data as two additional planes. In this example, I chose to down-weight the pre-subtracted data points to 1/20th (5%) of the subtracted data points for the least-squares fit. The resulting contours are [here](notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_16_1.png):

<a name="figure6"></a>
<img src="notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_16_1.png" width="60%">

Unsurprisingly, including the original data serves to significantly constrain the fit and reduce the degeneracy.

I believe that this is a possible way forward in the dipole characterization task in `dipoleMeasurement`. The primary drawback is if the source falls on a bright background or a background with a steep gradient - which is what we do the `imDiff` for anyway. This will also require passing the two pre-subtraction planes (and their variance planes) to the dipole characterization task.

*Recommendation:* Test the dipole fitting including using the additional (pre-subtraction) data planes, including simulating bright and steep-gradient backgrounds. Investigate the tolerance of very low weighting (5 to 10%) on the pre-subtraction planes in order to ensure that we are "mostly" fitting on the imDiff plane.

-

This same degeneracy is seen in simulated 2-d dipoles, as shown in [this notebook](notebooks/7c.%20dipole%20fit%20error%20contours.ipynb). First, a brief overview. Here is [a simulated 2-d dipole](notebooks/7c.%20dipole%20fit%20error%20contours_files/7c.%20dipole%20fit%20error%20%20contours_10_3.png) and the footprints for positive and negative detected sources in the image:

<a name="figure7"></a>
![Figure 7](notebooks/7c.%20dipole%20fit%20error%20contours_files/7c.%20dipole%20fit%20error%20contours_10_3.png)

and here is the [least-squares model fit and residuals](notebooks/7c.%20dipole%20fit%20error%20contours_files/7c.%20dipole%20fit%20error%20contours_10_4.png):

<a name="figure8"></a>
![Figure 8](notebooks/7c.%20dipole%20fit%20error%20contours_files/7c.%20dipole%20fit%20error%20contours_10_4.png)

A contour plot of confidence interval contours shows a similar degeneracy as that described above, here between dipole flux and x-coordinate of the positive dipole lobe (below, left). This is also seen in the covariance between x- and y-coordinate of the positive lobe centroid, which points generally toward the dipole centroid (below, right):

<a name="figure9"></a>
<img src="notebooks/7c.%20dipole%20fit%20error%20contours_files/7c.%20dipole%20fit%20error%20contours_32_1.png" width="50%"><img src="notebooks/7c.%20dipole%20fit%20error%20contours_files/7c.%20dipole%20fit%20error%20contours_31_1.png" width="50%">

These contours look surprisingly similar for fits to closely-separated and widely-separated dipoles of (otherwise) similar parameterization (see the [notebook](notebooks/7c.%20dipole%20fit%20error%20contours.ipynb) for more).

After updating the dipole fit code to include the pre-subtraction images (again with 5% weighting), as shown in [this notebook](notebooks/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit.ipynb), the fits once again improves. 

<!--Here is the original result:

<a name="figure11"></a>
<img src="notebooks/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_files/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_39_2.png" width="50%"><img src="notebooks/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_files/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_40_2.png" width="50%">
-->

The new (constrained) result, fitting to the same simulated dipole data (note the difference in axis limits):

<a name="figure12"></a>
<img src="notebooks/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_files/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_42_2.png" width="50%"><img src="notebooks/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_files/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_43_2.png" width="50%">

Adding the constraining data to the fit unsurprisingly improves the flux fits for a variety of dipole separations (the figure [below](notebooks/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_files/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_21_1.png) may be compared with the similar one shown [above](#figure2), generated without any constraint).

<a name="figure13"></a>
![Figure 13](notebooks/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_files/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_21_1.png)

---

*Additional recommendations and tests*:

1. complete refactoring of existing `dipoleMeasurement` code.
2. investigate the robustness of this updated fitting method, including variable backgrounds (with large gradients) that are removed in the image difference but bright and noisy in the template/science images.
2. investigate adding these constraints to the existing `dipoleMeasurement` code, including parameter windowing. *This will require refactoring of `diffIm` code to pass pre-subtraction images to `dipoleMeasurement`*.
