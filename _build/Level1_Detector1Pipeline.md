---
redirect_from:
  - "/level1-detector1pipeline"
interact_link: content/Level1_Detector1Pipeline.ipynb
kernel_name: python3
has_widgets: false
title: 'Ramps-to-slopes (CALDETECTOR1)'
prev_page:
  url: /Introduction
  title: 'Home'
next_page:
  url: /dq_init
  title: 'DQ_INIT'
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


## Ramps-to-slopes:  Detector1Pipeline   (MIRI CALDETECTOR1)

The `Detector1Pipeline` takes DMS Level 1B data and applies basic detector-level corrections to all exposure types. It is applied to one exposure at a time. It is sometimes referred to as “ramps-to-slopes” processing, because the input raw data are in the form of one or more ramps (integrations) containing accumulating counts from the non-destructive detector readouts and the output is a corrected countrate (slope) image.

Official documentation for `Detector1Pipeline` can be found here:

<https://jwst-pipeline.readthedocs.io/en/latest/jwst/pipeline/calwebb_detector1.html>

The `Detector1Pipeline` comprises a linear series of steps to first calibrate the pixel ramps, detect cosmic ray jumps, and fit a linear function to the calibrate ramps to produce a slope or DMS Level 2A image. The steps applied to MIRI data in order are:

|Step|Description|
|:---|:---|
|`dq_init`|populates the DQ mask for the input dataset|
|`saturation`|flags saturated pixel values|
|`firstframe`|flags the first group in every integration as bad|
|`lastframe`|flags the last group in every integration as bad|
|`linearity`|corrects for detector non-linearity|
|`rscd`|correct for the reset switch charge decay|
|`dark_current`|subtracts dark current data|
|`refpix`|corrects for pixel drifts using the reference pixels|
|`jump`|detects jumps in the ramps by looking for outliers in each integration|
|`ramp_fit`|determines count rate for each pixel by performing a linear fit to ramps|

For more information and examples of each of the steps click on the links in the side bar.



### Input data

An example of running the file through the Detector1Pipeline is now shown using a simple simulated observation of a galaxy with the MIRI Imager (F1130W filter) produced with [MIRISim v2.1](http://miri.ster.kuleuven.be/bin/view/Public/MIRISimPublicRelease2dot1).



### Python

Start by importing what will be used and set the `CRDS_CONTEXT`



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
# imports
import os, glob, shutil
import numpy as np
from matplotlib.colors import LogNorm
import matplotlib.pyplot as plt
from subprocess import call
from jwst import datamodels

# set the CRDS_CONTEXT
os.environ["CRDS_CONTEXT"] = "jwst_0535.pmap"

```
</div>

</div>



Import Detector1Pipeline and print the docstring to show some information



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
from jwst.pipeline import Detector1Pipeline
print(Detector1Pipeline.__doc__)

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```

    Detector1Pipeline: Apply all calibration steps to raw JWST
    ramps to produce a 2-D slope product. Included steps are:
    group_scale, dq_init, saturation, ipc, superbias, refpix, rscd,
    lastframe, linearity, dark_current, persistence, jump detection,
    ramp_fit, and gain_scale.
    
```
</div>
</div>
</div>



Set the name of the input file, where the output should be saved and run the pipeline. The output level 2A files will be saved in `my_output_dir` as `_rate.fits`. Note that we must explicitly skip the IPC step when running from Python. This is achieved by passing the `skip` parameter to the step in a Python `dict`.

*Parameters used:*

`output_use_model` : boolean, optional, default=False  
&nbsp;&nbsp;&nbsp;&nbsp; propagate the input filename to the output
    
`save_results`: boolean, optional, default=False  
&nbsp;&nbsp;&nbsp;&nbsp; save the results to file
    
`output_dir` : boolean, optional, default is the working directory   
&nbsp;&nbsp;&nbsp;&nbsp; the location to save the output
    
`steps` : dict, optional  
&nbsp;&nbsp;&nbsp;&nbsp; dict of user-defined parameters for individual steps  

Note that the `Detector1Pipeline` will return the rate datamodel so we set this to the `dm` variable.




<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
# user specified
my_input_file = 'det_image_seq1_MIRIMAGE_F1130Wexp1.fits'
my_output_dir = 'demo_output'

# the output directory should be created if it doesn't exist
if not os.path.exists(my_output_dir): 
    os.mkdir(my_output_dir)

# run the pipeline
dm = Detector1Pipeline.call(my_input_file, output_use_model=True, save_results=True, 
                            output_dir=my_output_dir, steps={'ipc': {'skip': True}})

```
</div>

</div>



We can plot the before (last frame of first integration) and after images



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
# open the input image as a jwst data model
with datamodels.open(my_input_file) as in_dm:

    fig, axs = plt.subplots(1, 2, figsize=(14, 7), sharey=True)

    axs[0].imshow(in_dm.data[0,-1,:,:], cmap='jet', interpolation='nearest', origin='lower', norm=LogNorm(vmin=1.1e4,vmax=6.5e4))
    axs[0].annotate('DMS Level 1B (ramp)', xy=(0.0, 1.02), xycoords='axes fraction', fontsize=12, fontweight='bold', color='k')
    axs[0].set_facecolor('black')
    axs[1].imshow(dm.data, cmap='jet', interpolation='nearest', origin='lower', norm=LogNorm(vmin=10, vmax=1000))
    axs[1].annotate('DMS Level 2A (slope)', xy=(0.0, 1.02), xycoords='axes fraction', fontsize=12, fontweight='bold', color='k')
    axs[1].set_facecolor('black')
    plt.tight_layout()
    plt.show()


```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](images/Level1_Detector1Pipeline_9_0.png)

</div>
</div>
</div>



### Command line

To achieve the same result from the command line there are a couple of options. 

**Option 1:**
Run the `Detector1Pipeline` class using the `strun` command:

```bash
mkdir demo_output

strun jwst.pipeline.Detector1Pipeline det_image_seq1_MIRIMAGE_F1130Wexp1.fits --output_dir demo_output
```

This will produce the same output file in the user-defined `--output_dir`


**Option 2:**
Collect the pipeline configuration files in your working directory using `collect_pipeline_configs` and then run the `Detector1Pipeline` using the `strun` command with the associated `calwebb_detector1.cfg` file. This option is a little more flexible as one can create edit the cfg files, use them again, etc.

```bash
mkdir demo_output

collect_pipeline_cfgs cfgs/

strun cfgs/calwebb_detector1.cfg det_image_seq1_MIRIMAGE_F1130Wexp1.fits --output_dir demo_output
```

This will produce the same output file in the user-defined `--output_dir`




## Further examples

Other notebooks with more complex examples can be found here:

*To be added*

