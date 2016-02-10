# lsst-dipole
Notebooks and analyses of dipole fitting for LSST

1. PRIMARY QUESTION: What is the goal of the dipole measurement as
part of the LSST pipelines?
   * Clearly identification and flagging of false positives.
   * But what about using the machine learning techniques that have
     been developed over the years for transient classification?
   * Beyond simple classification, measurement of dipoles (orientation
     and amplitude) is helpful for quantifying DCR and astrometric
     misalignments. Is this a goal?
     * If so, then we need to enumerate how these measurements will
       feed back into the image differencing.

### some random dipole fitting notes in no relevant order:

1. degeneracy between bright, close dipoles and fainter distant
   dipoles (see this [report](REPORT.md)])
2. would be helpful to have source centroids/fluxes from template and
   observation as input to dipole fitting (initial parameters? or as
   additional input data for fitting?) — see (3) below for caveats.
3. how about just measuring the dipole from the original images?
   * Note that the sources may not be detected (or their fluxes
     inaccurate) in the the original calexps. — but in many (most?)
     cases they will be.
   * At least incoporate centroids/fluxes from original pre-subtracted
     images as prior/constraint on dipole fit.
   * Otherwise, incorporate pre-subtracted data into the fit itself
4. can use center-of-mass measurements of pos/neg peaks. The distance
   will be overestimated but the orientation will be correct? Then PSF
   fit is just a 1-d fit (distance). *THIS IS A CLAIM AND NEEDS TO BE
   VERIFIED!*
6. TBD: single flux value for both lobes or separate values for each lobe? Single value is more stable but doesn't allow for varying sources.
5. tests —
   * results as a function of:
     * S/N
     * separation
     * source flux (same as S/N)
     * PSF shape/size
     * compare results with those from “pure python” fits (astropy/scipy/lmfit)
6. It is possible to do the fitting in "pure" python (I used `lmfit` which is a wrapper around `scipy.optimize`); my implementation is about 50% faster than the `C++` optimization in `ip_diffim`.
7. May be possible to speed up the current `ip_diffim` dipole fit implementation:
   * Multiple vectorized calls in `DipoleAlgorithms.cc` -- `PsfDipoleFlux::chi2`
   * `chi2` uses PSF image vs. using analytic kernel function
   * some calculations are computed each time during minimization (in
     `chi2` function) that can be moved outside (not sure if these
     calc's are really expensive though).
   * optimization spends much time floating around minimum - possible
     earlier cut-off (also see point above, that may fix it)
   * (DONE) improve starting parameters for optimization -- e.g. starting
     fluxes from original (template/science) images -- see (3) above. It's
     currently not clear how starting fluxes are set.
   * compute `chi2` over smaller subset of image closest to centroids rather
     than over entire footprint
   * more parameters are optimized than necessary? (e.g. separate
     positive and negative flux values)
   * parameters are currently unconstrained during optimization
     (e.g. can constrain flux to be > 0)

NOTES:

1. LSST pixel scale 0.2” / pixel
2. expected seeing is  ~ 0.69” in *r*
3.   —> ~ 3.5 pixel FWHM seeing
