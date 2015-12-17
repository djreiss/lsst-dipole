# lsst-dipole
Notebooks and analyses of dipole fitting for LSST

# dipole fitting notes

1. degeneracy between bright, close dipoles and fainter distant dipoles
2. would be helpful to have source centroids/fluxes from template and observation as input to dipole fitting (initial parameters?) — see (3) below for caveats.
3. how about just measuring the dipole from the original images?
   a. Note that the sources may not be detected (or their fluxes inaccurate) in the the original calexps. — but in many (most?) cases they will be.
4. can use center-of-mass measurements of pos/neg peaks. The distance will be overestimated but the orientation will be correct? Then PSF fit is just a 1-d fit (distance). THIS NEEDS TO BE VERIFIED!
5. tests —
   a. results as a function of:
       i. S/N
       ii. separation
       iii. source flux
       iv. PSF shape/size
       v. compare results with those from “pure python” fits (astropy / scipy / lmfit)


NOTES:

1. LSST pixel scale 0.2” / pixel
2. expected seeing is  ~ 0.69”
3.   —> ~ 3.5 pixel FWHM seeing
