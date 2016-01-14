[Additional random dipole characterization thoughts](README.md).

There is a degeneracy in dipole fitting between closely-separated dipoles from bright sources and widely-separated dipoles from faint sources. This is further explored using 1-d simulated dipoles in [this notebook](notebooks/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise.ipynb).

[Here](notebooks/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_4_0.png) is an example:

<!--
![Figure 1](notebooks/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_4_0.png)
-->
<img src="notebooks/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_4_0.png" width=300>

There are many such examples, and this is most easiest shown by plotting error contours from a [least-squares fit to simulated 1-d dipole data](notebooks/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_7_2.png):

<!--
![Figure2](notebooks/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_7_2.png)
-->
<img src="notebooks/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_7_2.png" width=300>

Here are the error contours, where the blue dot indicates the input parameters (used to generate the data), the yellow dot shows the starting parameters for the minimization and the green dot indicates the least-squares parameters:

![Figure2](notebooks/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_8_1.png)

This degeneracy is a big problem if we are going to fit dipole parameters using the subtracted data alone. Three possible solutions are:

1. Use starting parameters and parameter bounds based on measurements from the pre-subtracted images (obs. and template) to constrain the dipole fit.
2. Include the pre-subtracted image data in the fit to constrain the minimization.
3. A combination of (1.) and (2.).

As an example, I performed a fit to the same data as shown above, but included the "pre-subtracted" data as two additional planes. In this example, I chose to down-weight the pre-subtracted data points to 1/10th of the subtracted data points for the least-squares fit. The resulting contours are [here](notebooks/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_16_1.png):

![Figure 3](notebooks/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_16_1.png)

Unsurprisingly, including the original data serves to significantly constrain the fit and reduce the degeneracy.

I believe that this is a possible way forward in the dipole characterization task for `ip_diffim`. The primary drawback is if the source falls on a bright background or a background with a steep gradient - which is what we do the `imDiff` for anyway. This will also require passing the two pre-subtraction planes (and their variance planes) to the dipole characterization task.

*Recommendation:* Test the dipole fitting including using the additional (pre-subtraction) data planes, including simulating bright and steep-gradient backgrounds.

-

This same degeneracy is seen in simulated 2-d dipoles, as shown in [this notebook](notebooks/dipole%20fit%20error%20contours.ipynb). First, a brief overview. Here is [a simulated 2-d dipole](notebooks/dipole%20fit%20error%20contours_files/dipole%20fit%20error%20%20contours_10_3.png) and the footprints for positive and negative detected sources in the image:

![Figure 4](notebooks/dipole%20fit%20error%20contours_files/dipole%20fit%20error%20contours_10_3.png)

and here is the [least-squares model fit and residuals](notebooks/dipole%20fit%20error%20contours_files/dipole%20fit%20error%20contours_10_4.png):

![Figure 5](notebooks/dipole%20fit%20error%20contours_files/dipole%20fit%20error%20contours_10_4.png)

A contour plot of confidence interval contours shows a similar degeneracy as that described above, here between dipole flux and x-coordinate of the positive dipole lobe:

![Figure 6](notebooks/dipole%20fit%20error%20contours_files/dipole%20fit%20error%20contours_32_1.png)

This is also seen in the covariance between x- and y-coordinate of the positive lobe centroid, which points generally toward the dipole centroid:

![Figure 7](notebooks/dipole%20fit%20error%20contours_files/dipole%20fit%20error%20contours_31_1.png)

These contours look surprisingly similar for fits to closely-separated and widely-separated dipoles of (otherwise) similar parameterization (see the [notebook](notebooks/dipole%20fit%20error%20contours.ipynb) for more).