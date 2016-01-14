

```python
%pylab inline

import numpy as np

import matplotlib
matplotlib.style.use('ggplot')
import matplotlib.pyplot as plt

from scipy.stats import norm
matplotlib.rcParams['figure.figsize'] = (12.0, 6.0)
```

    Populating the interactive namespace from numpy and matplotlib


### Degeneracy between faint, widely-separated dipole and bright, close dipole:


```python
if True:
    x = np.arange(-5,5, 0.001)
    plt.plot(x, 1. * (norm.pdf(x, 0.5, 1) - norm.pdf(x, -0.5, 1)))   ## faint, distant diple
    plt.plot(x, 5. * (norm.pdf(x, 0.1, 1) - norm.pdf(x, -0.1, 1)))   ## bright, close dipole
    plt.legend(['amp=1; sep=1', 'amp=5; sep=0.2'])
    plt.show()
```


![png](simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_2_0.png)


### In the below, set amp1=1.3, sep1=0.8, amp2=5.0, amp2=0.2 ...


```python
from ipywidgets import interact

def do_plot(amp1=1., sep1=1., amp2=5., sep2=0.2, show_components=True):
    x = np.arange(-5, 5, 0.1)
    plt.plot(x, amp1 * (norm.pdf(x, sep1/2., 1) - norm.pdf(x, -sep1/2., 1)), color='red')   ## faint, distant diple
    plt.plot(x, amp2 * (norm.pdf(x, sep2/2., 1) - norm.pdf(x, -sep2/2., 1)), color='blue')   ## bright, close dipole
    if show_components:
        plt.plot(x, amp1 * -norm.pdf(x, -sep1/2., 1), linestyle='dashed', color='red')
        plt.plot(x, amp1 *  norm.pdf(x,  sep1/2., 1), linestyle='dashed', color='red')
        plt.plot(x, amp2 * -norm.pdf(x, -sep2/2., 1), linestyle='dashed', color='blue')
        plt.plot(x, amp2 *  norm.pdf(x,  sep2/2., 1), linestyle='dashed', color='blue')
        
    plt.plot([-sep1, sep1], [0,0], 'ro')
    plt.plot([-sep2, sep2], [0,0], 'bo')
    plt.legend(['amp=%.2f; sep=%.2f' % (amp1, sep1), 'amp=%.2f; sep=%.2f' % (amp2, sep2)])
    plt.show()
    
##do_plot()

interact(do_plot, amp1=(0., 10.), sep1=(0., 2.), amp2=(0., 10.), sep2=(0., 2.), show_components=True)
```


![png](simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_4_0.png)


### Try fitting it...


```python
import lmfit

def gen_dipole2(x, amp1=1., sep1=1.):
    ypos = amp1 * norm.pdf(x,  sep1/2., 1)
    yneg = amp1 * norm.pdf(x, -sep1/2., 1)                  
    y = ypos - yneg
    return np.append(y, [ypos, yneg])

def gen_dipole(x, amp1=1., sep1=1.):
    #y = amp1 * (norm.pdf(x, -sep1/2., 1) - norm.pdf(x, sep1/2., 1))
    y = gen_dipole2(x, amp1, sep1)[:len(x)]
    return y

def fitDipole(ydata, xdata, amp1Start=3., sep1Start=2., fitFunc=gen_dipole, verbose=False):
    gmod = lmfit.Model(fitFunc, verbose=verbose)
    result = gmod.fit(ydata, x=xdata, amp1=amp1Start, sep1=sep1Start, verbose=verbose)

    if verbose: 
        print(result.fit_report(show_correl=False))
        ci = lmfit.conf_interval(result, result)
        lmfit.printfuncs.report_ci(ci)

    return result
```


```python
xdata = np.arange(-5, 5, 0.25)
ydataOrig = gen_dipole2(xdata, amp1=1.3, sep1=0.8)
ydataOrigNoisy = ydataOrig + np.random.normal(scale=np.abs(ydataOrig)/10.+0.02, size=ydataOrig.shape) ## save it for fitting below...
ydataOrigNoisy[:len(xdata)] = ydataOrigNoisy[len(xdata):len(xdata)*2] - ydataOrigNoisy[len(xdata)*2:]
ydata = ydataOrigNoisy[:len(xdata)] 

result = fitDipole(ydata, xdata, amp1Start=1.5, sep1Start=1.5, fitFunc=gen_dipole, verbose=True)
plt.plot(xdata, ydata, 'bo')
plt.plot(xdata, ydataOrig[:len(xdata)], linewidth=3, linestyle='dotted')
plt.plot(xdata, result.best_fit, linewidth=3, linestyle='dashed')
plt.show()
```

    /Users/dreiss/PYTHONUSERBASE/lib/python2.7/site-packages/lmfit/confidence.py:298: UserWarning: Warning, rel_change=-0.0347796880334 < 0.01  at iteration 3 and prob(sep1=-0.365056556708) = 0.727710713167 < max(sigmas).
      warn(errmsg)
    /Users/dreiss/PYTHONUSERBASE/lib/python2.7/site-packages/lmfit/confidence.py:298: UserWarning: Warning, rel_change=9.84331419712e-06 < 0.01  at iteration 91 and prob(amp1=26.1496928277) = 0.764048893498 < max(sigmas).
      warn(errmsg)


     - Adding parameter "sep1"
     - Adding parameter "amp1"
    [[Model]]
        Model(gen_dipole)
    [[Fit Statistics]]
        # function evals   = 15
        # data points      = 40
        # variables        = 2
        chi-square         = 0.114
        reduced chi-square = 0.003
    [[Variables]]
        sep1:   1.32614774 +/- 0.563734 (42.51%) (init= 1.5)
        amp1:   0.80277817 +/- 0.278537 (34.70%) (init= 1.5)
    
             99.70%    95.00%    67.40%    _BEST_    67.40%    95.00%    99.70%
     sep1:      -inf      -inf  -0.77211   1.32615  +0.46802  +0.85500  +1.22928
     amp1:  -0.34293  -0.26639  -0.17190   0.80278  +0.95614      +inf      +inf



![png](simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_7_2.png)



```python
cx, cy, grid = lmfit.conf_interval2d(result, result, 'amp1', 'sep1', 100, 100, 
                                     limits=((0.,np.max([6.5,result.params['amp1'].value])), 
                                             (0., np.max([1.6,result.params['sep1'].value]))))
plt.contourf(cx, cy, grid, np.linspace(0,1,11)) #, cmap=plt.cm.bone)
plt.xlabel('amp1')
plt.colorbar()
plt.ylabel('sep1')
plt.plot(1.3, 0.8, 'bo') ## input params
plt.plot(1.5, 1.5, 'yo') ## starting params
plt.plot(result.params['amp1'].value, result.params['sep1'].value, 'go')  ## fitted params
```




    [<matplotlib.lines.Line2D at 0x106913b90>]




![png](simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_8_1.png)


The F-test is used to compare our null model, which is the best fit we have found, with an alternate model, where one of the parameters is fixed to a specific value. The value is changed until the difference between $\chi^2_0$ and $\chi^2_{f}$ can’t be explained by the loss of a degree of freedom within a certain confidence.

$F(P_{fix},N-P) = \left(\frac{\chi^2_f}{\chi^2_{0}}-1\right)\frac{N-P}{P_{fix}}$

$N$ is the number of data-points, $P$ the number of parameter of the null model. $P_{fix}$ is the number of fixed parameters (or to be more clear, the difference of number of parameters between our null model and the alternate model).

### Try my idea of using image and template separately to constrain fit...


```python
## Use same input data as above for direct comparison
ydata = ydataOrigNoisy
result = fitDipole(ydata, xdata, amp1Start=1.5, sep1Start=1.5, fitFunc=gen_dipole2, verbose=True)
plt.plot(xdata, ydata[:len(xdata)], 'bo')
plt.plot(xdata, ydataOrig[:len(xdata)], linewidth=3, linestyle='dotted')
plt.plot(xdata, result.best_fit[:len(xdata)], linewidth=3, linestyle='dashed')
plt.show()
```

     - Adding parameter "sep1"
     - Adding parameter "amp1"
    [[Model]]
        Model(gen_dipole2)
    [[Fit Statistics]]
        # function evals   = 15
        # data points      = 120
        # variables        = 2
        chi-square         = 0.210
        reduced chi-square = 0.002
    [[Variables]]
        sep1:   0.74795817 +/- 0.038500 (5.15%) (init= 1.5)
        amp1:   1.31988909 +/- 0.027383 (2.07%) (init= 1.5)
    
             99.70%    95.00%    67.40%    _BEST_    67.40%    95.00%    99.70%
     sep1:  -0.11463  -0.07541  -0.03784   0.74796  +0.03841  +0.07773  +0.12007
     amp1:  -0.08292  -0.05423  -0.02701   1.31989  +0.02703  +0.05431  +0.08309



![png](simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_11_1.png)



```python
cx, cy, grid = lmfit.conf_interval2d(result, result, 'amp1', 'sep1', 100, 100, 
                                     limits=((0.,np.max([6.5,result.params['amp1'].value])), 
                                             (0., np.max([1.6,result.params['sep1'].value]))))
plt.contourf(cx, cy, grid, np.linspace(0,1,11)) #, cmap=plt.cm.bone)
plt.xlabel('amp1')
plt.colorbar()
plt.ylabel('sep1')
plt.plot(1.3, 0.8, 'bo') ## input params
plt.plot(1.5, 1.5, 'yo') ## starting params
plt.plot(result.params['amp1'].value, result.params['sep1'].value, 'go')  ## fitted params
```




    [<matplotlib.lines.Line2D at 0x10c6c6e90>]




![png](simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_12_1.png)


### This above weights the 3 planes equally. Let's weight the dipole plane 10x higher and see how it looks:


```python
def fitDipole2(ydata, xdata, amp1Start=3., sep1Start=2., fitFunc=gen_dipole, verbose=False):
    gmod = lmfit.Model(fitFunc, verbose=verbose)
    weights = np.ones_like(ydata)
    weights[:len(xdata)] = 10.
    result = gmod.fit(ydata, x=xdata, weights=weights, amp1=amp1Start, sep1=sep1Start, verbose=verbose)

    if verbose: 
        print(result.fit_report(show_correl=False))
        ci = lmfit.conf_interval(result, result)
        lmfit.printfuncs.report_ci(ci)

    return result
```


```python
ydata = ydataOrigNoisy
result = fitDipole2(ydata, xdata, amp1Start=1.5, sep1Start=1.5, fitFunc=gen_dipole2, verbose=True)
plt.plot(xdata, ydata[:len(xdata)], 'bo')
plt.plot(xdata, ydataOrig[:len(xdata)], linewidth=3, linestyle='dotted')
plt.plot(xdata, result.best_fit[:len(xdata)], linewidth=3, linestyle='dashed')
plt.show()
```

     - Adding parameter "sep1"
     - Adding parameter "amp1"
    [[Model]]
        Model(gen_dipole2)
    [[Fit Statistics]]
        # function evals   = 21
        # data points      = 120
        # variables        = 2
        chi-square         = 11.655
        reduced chi-square = 0.099
    [[Variables]]
        sep1:   0.79848115 +/- 0.137990 (17.28%) (init= 1.5)
        amp1:   1.24997719 +/- 0.193805 (15.50%) (init= 1.5)
    
             99.70%    95.00%    67.40%    _BEST_    67.40%    95.00%    99.70%
     sep1:  -0.29780  -0.22157  -0.12729   0.79848  +0.18167  +0.44436  +0.77981
     amp1:  -0.54844  -0.39804  -0.20691   1.24998  +0.21018  +0.42179  +0.64466



![png](simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_15_1.png)



```python
cx, cy, grid = lmfit.conf_interval2d(result, result, 'amp1', 'sep1', 100, 100, 
                                     limits=((0.,np.max([6.5,result.params['amp1'].value])), 
                                             (0., np.max([1.6,result.params['sep1'].value]))))
plt.contourf(cx, cy, grid, np.linspace(0,1,11)) #, cmap=plt.cm.bone)
plt.xlabel('amp1')
plt.colorbar()
plt.ylabel('sep1')
plt.plot(1.3, 0.8, 'bo') ## input params
plt.plot(1.5, 1.5, 'yo') ## starting params
plt.plot(result.params['amp1'].value, result.params['sep1'].value, 'go')  ## fitted params
```




    [<matplotlib.lines.Line2D at 0x10cc6ec10>]




![png](simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_files/simple%202-d%20dipole%20plotting%20-%20more%20realistic%20noise_16_1.png)



```python

```
