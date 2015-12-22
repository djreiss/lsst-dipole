# lsst-dipole
Notebooks and analyses of dipole fitting for LSST

# dipole fitting notes

1. degeneracy between bright, close dipoles and fainter distant dipoles
2. would be helpful to have source centroids/fluxes from template and observation as input to dipole fitting (initial parameters?) — see (3) below for caveats.
3. how about just measuring the dipole from the original images?
   * Note that the sources may not be detected (or their fluxes inaccurate) in the the original calexps. — but in many (most?) cases they will be.
4. can use center-of-mass measurements of pos/neg peaks. The distance will be overestimated but the orientation will be correct? Then PSF fit is just a 1-d fit (distance). THIS NEEDS TO BE VERIFIED!
5. tests —
   * results as a function of:
     * S/N
     * separation
     * source flux
     * PSF shape/size
     * compare results with those from “pure python” fits (astropy / scipy / lmfit)
6. Current PSF fit implementation may be able to be sped up:
   * Multiple vectorized calls in DipoleAlgorithms.cc -- PsfDipoleFlux::chi2
   * chi2 uses PSF image vs. using analytic kernel function
   * optimization spends much time floating around minimum - possible earlier cut-off (also see point above, that may fix it)
   * improve starting parameters for optimization -- e.g. starting fluxes from original (template/obs) images -- see (3) above.
   * compute chi2 over subset of image closest to centroids rather than over entire footprint
   * more parameters are optimized than necessary (e.g. separate positive and negative flux values)
   * parameters are currently unconstrained (e.g. can constrain flux to be > 0)

NOTES:

1. LSST pixel scale 0.2” / pixel
2. expected seeing is  ~ 0.69”
3.   —> ~ 3.5 pixel FWHM seeing