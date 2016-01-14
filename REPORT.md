There is a degeneracy in dipole fitting between closely-separated dipoles from bright sources and widely-separated dipoles from faint sources. This is further explored using 1-d simulated dipoles in [this notebook](notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise.ipynb).

[Here](notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_4_0.png) is an example:

<!--
![Figure 1](notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_4_0.png)
-->
<img src="notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_4_0.png" width="70%">

There are many such examples, and this strong covariance between amplitude (or flux) and dipole separation is most easiest shown by plotting error contours from a [least-squares fit to simulated 1-d dipole data](notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_7_2.png):

<!--
![Figure2](notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_7_2.png)
-->
<img src="notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_7_2.png" width="70%">

Here are the error contours, where the blue dot indicates the input parameters (used to generate the data), the yellow dot shows the starting parameters for the minimization and the green dot indicates the least-squares parameters:

<img src="notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_8_2.png" width="60%">

This degeneracy is a big problem if we are going to fit dipole parameters using the subtracted data alone. Three possible solutions are:

1. Use starting parameters and parameter bounds based on measurements from the pre-subtracted images (obs. and template) to constrain the dipole fit.
2. Include the pre-subtracted image data in the fit to constrain the minimization.
3. A combination of (1.) and (2.).

It is noted that these solutions will not help in cases of dipoles detected on top of bright backgrounds (or backgrounds with large gradients), such as cases of a faint dipole superimposed on a bright-ish background galaxy. But these cases will be rare, and I believe we can adjust the weighting of the pre-subtracted image data (i.e., in [2] above) to compensate (see below).

As an example, I performed a fit to the same data as shown above, but included the "pre-subtracted" data as two additional planes. In this example, I chose to down-weight the pre-subtracted data points to 1/20th (5%) of the subtracted data points for the least-squares fit. The resulting contours are [here](notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_16_1.png):

<img src="notebooks/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/8a.%20simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_16_1.png" width="60%">

Unsurprisingly, including the original data serves to significantly constrain the fit and reduce the degeneracy.

I believe that this is a possible way forward in the dipole characterization task for `ip_diffim`. The primary drawback is if the source falls on a bright background or a background with a steep gradient - which is what we do the `imDiff` for anyway. This will also require passing the two pre-subtraction planes (and their variance planes) to the dipole characterization task.

*Recommendation:* Test the dipole fitting including using the additional (pre-subtraction) data planes, including simulating bright and steep-gradient backgrounds. Investigate the tolerance of very low weighting (5 to 10%) on the pre-subtraction planes in order to ensure that we are "mostly" fitting on the imDiff plane.

-

This same degeneracy is seen in simulated 2-d dipoles, as shown in [this notebook](notebooks/7c.%20dipole%20fit%20error%20contours.ipynb). First, a brief overview. Here is [a simulated 2-d dipole](notebooks/7c.%20dipole%20fit%20error%20contours_files/7c.%20dipole%20fit%20error%20%20contours_10_3.png) and the footprints for positive and negative detected sources in the image:

![Figure 4](notebooks/7c.%20dipole%20fit%20error%20contours_files/7c.%20dipole%20fit%20error%20contours_10_3.png)

and here is the [least-squares model fit and residuals](notebooks/7c.%20dipole%20fit%20error%20contours_files/7c.%20dipole%20fit%20error%20contours_10_4.png):

![Figure 5](notebooks/7c.%20dipole%20fit%20error%20contours_files/7c.%20dipole%20fit%20error%20contours_10_4.png)

A contour plot of confidence interval contours shows a similar degeneracy as that described above, here between dipole flux and x-coordinate of the positive dipole lobe:

<img src="notebooks/7c.%20dipole%20fit%20error%20contours_files/7c.%20dipole%20fit%20error%20contours_32_1.png" width="60%">

This is also seen in the covariance between x- and y-coordinate of the positive lobe centroid, which points generally toward the dipole centroid:

<img src="notebooks/7c.%20dipole%20fit%20error%20contours_files/7c.%20dipole%20fit%20error%20contours_31_1.png" width="60%">

These contours look surprisingly similar for fits to closely-separated and widely-separated dipoles of (otherwise) similar parameterization (see the [notebook](notebooks/7c.%20dipole%20fit%20error%20contours.ipynb) for more).

After updating the dipole fit code to include the pre-subtraction images (again with 5% weighting), as shown in [this notebook](notebooks/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit.ipynb), the fits once again improves. Here is the original result:

<img src="notebooks/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_files/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_39_2.png" width="50%"><img src="notebooks/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_files/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_40_2.png" width="50%">

And the new (constrained) result, fitting to the same simulated dipole data:

<img src="notebooks/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_files/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_42_2.png" width="50%"><img src="notebooks/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_files/8b.%20include%20down-weighted%20pre-subtraction%20image%20%22planes%22%20to%20constrain%202-d%20dipole%20fit_43_2.png" width="50%">

---
---

[Additional random dipole characterization thoughts](README.md).
