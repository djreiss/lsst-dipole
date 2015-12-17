# lsst-dipole
Notebooks and analyses of dipole fitting for LSST

# dipole fitting notes

degeneracy between bright, close dipoles and fainter distant dipoles
would be helpful to have source centroids/fluxes from template and observation as input to dipole fitting (initial parameters?) — see (3) below for caveats.
how about just measuring the dipole from the original images?
Note that the sources may not be detected (or their fluxes inaccurate) in the the original calexps. — but in many (most?) cases they will be.
can use center-of-mass measurements of pos/neg peaks. The distance will be overestimated but the orientation will be correct? Then PSF fit is just a 1-d fit (distance). THIS NEEDS TO BE VERIFIED!
tests —
results as a function of:
S/N
separation
source flux
PSF shape/size
compare results with those from “pure python” fits (astropy / scipy / lmfit)


NOTES:

LSST pixel scale 0.2” / pixel
expected seeing is  ~ 0.69”
   —> ~ 3.5 pixel FWHM seeing
