---
interact_link: content/jump.ipynb
kernel_name: python3
has_widgets: false
title: 'jump'
prev_page:
  url: /refpix
  title: 'refpix'
next_page:
  url: 
  title: ''
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


## `jump` step

This step detects jumps in the pixel ramps by looking for outliers. On output, the GROUPDQ array is updated with the DQ flag JUMP_DET to indicate the location of each jump that was found. 

Official documentation for `jump` can be found here:

<https://jwst-pipeline.readthedocs.io/en/latest/jwst/jump/index.html>




### Input data

An example of running the `jump` step is now shown using a simple simulated observation of a galaxy with the MIRI Imager (F1130W filter) produced with [MIRISim v2.1](http://miri.ster.kuleuven.be/bin/view/Public/MIRISimPublicRelease2dot1), with precending pipeline steps applied, i.e. `refpix` output.



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
from jwst import datamodels

# set the CRDS_CONTEXT
os.environ["CRDS_CONTEXT"] = "jwst_0535.pmap"

```
</div>

</div>



Import `jump` and print the docstring and spec to show some information



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
# import the step
from jwst.jump import jump_step

# print the description and options
print(jump_step.JumpStep.__doc__)
print(jump_step.JumpStep.spec)


```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```

    JumpStep: Performs CR/jump detection on each ramp integration within an
    exposure. The 2-point difference method is applied.
    

        rejection_threshold = float(default=4.0,min=0) # CR rejection threshold
        maximum_cores = option('quarter', 'half', 'all', default=None) # max number of processes to create
    
```
</div>
</div>
</div>



Set the name of the input file and run the step. This will produce an output file ending with `_jumpstep.fits`

*Parameters used:*

`output_use_model` : boolean, optional, default=False  
&nbsp;&nbsp;&nbsp;&nbsp; propagate the input filename to the output
    
`save_results`: boolean, optional, default=False  
&nbsp;&nbsp;&nbsp;&nbsp; save the results to file

Note that the `jump` will return the output datamodel so we set this to the `dm` variable.




<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
# user specified
my_input_file = 'det_image_seq1_MIRIMAGE_F1130Wexp1_refpixstep.fits'

# run the step
dm = jump_step.JumpStep.call(my_input_file, output_use_model=True, save_results=True)


```
</div>

</div>



We can plot the location of detected jumps



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
# set the sample pixel (selected knowing there is a jump on the ramp)
pixel = [821, 385]

# define group numbers for integration ramps
group = range(1,dm.data[0,:,pixel[0],pixel[1]].shape[0]+1,1)

# plot--------------------------------------
fig, axs = plt.subplots(2, 1, figsize=(10, 8), sharex=True)

# first integration for input/output ramps
axs[0].plot(group, dm.data[0,:,pixel[1],pixel[0]], c='b', marker='o', linestyle='-', linewidth=2, label='ramp')
axs[0].set_title('jump detection',fontsize=15)
axs[0].set_ylabel('DN',fontsize=15)
axs[0].set_xlim(-1,max(group)+1)
axs[0].set_ylim(min(dm.data[0,:,pixel[1],pixel[0]])-200,max(dm.data[0,:,pixel[1],pixel[0]])+200)
axs[0].legend(prop={'size':12}, loc=2)

# DQ flag values
axs[1].plot(group, dm.groupdq[0,:,pixel[1],pixel[0]], c='b', marker='o', linestyle='-', 
            linewidth=2, label='GROUPDQ')
axs[1].plot([-10,100],[4,4], linestyle='--', linewidth=3, c='g', label='JUMP_DET (GROUPDQ=4)')
axs[1].set_ylabel('GROUPDQ value',fontsize=15)
axs[1].set_xlim(-1,max(group)+1)
axs[1].set_ylim(-0.5,6.5)
axs[1].legend(prop={'size':12}, loc=2)

# draw lines to show the groups which have been flagged as jumps
for n, val in enumerate(group):
    if (dm.groupdq[0,n,pixel[1],pixel[0]] >= 4): 
        axs[0].plot([n+1,n+1],[min(dm.data[0,:,pixel[1],pixel[0]])-200,max(dm.data[0,:,pixel[1],pixel[0]])+200], 
                    linestyle='--', linewidth=0.3, c='k')
        axs[1].plot([n+1,n+1],[-1,6], linestyle='--', linewidth=0.3, c='k')

plt.tight_layout(h_pad=0)
plt.show()



```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](images/jump_9_0.png)

</div>
</div>
</div>



### Command line

To achieve the same result from the command line there are a couple of options. 

**Option 1:**
Run the `JumpStep` class using the `strun` command:

```bash
strun jwst.jump.JumpStep det_image_seq1_MIRIMAGE_F1130Wexp1_refpixstep.fits
```

**Option 2:**
If they don't already exist, collect the pipeline configuration files in your working directory using `collect_pipeline_configs` and then run the `JumpStep` using the `strun` command with the associated `jump.cfg` file. 

```bash
collect_pipeline_cfgs cfgs/

strun cfgs/jump.cfg det_image_seq1_MIRIMAGE_F1130Wexp1_refpixstep.fits
```

This will produce the same output file ending with `_jumpstep.fits` 




A full list of the command line options are given by running the following:

```bash
strun jwst.jump.JumpStep -h
```

or 

```bash
strun cfgs/jump.cfg -h
```




### Override reference file

The `jump` step uses the readnoise and gain reference files. To override these in Python:

```python
# set the readnoise and gain reference file names
my_ref_rn = 'my_readnoise.fits'
my_ref_gn = 'my_gain.fits'

dm = jump.JumpStep.call(my_input_file, output_use_model=True, save_results=True,
                        override_readnoise=my_ref_rn, override_gain=my_ref_gn)
```



and using the command line:

```bash
strun jwst.jump.JumpStep det_image_seq1_MIRIMAGE_F1130Wexp1_refpixstep.fits  --override_readnoise my_readnoise.fits --override_gain my_gain.fits
```

or

```bash
strun cfgs/jump.cfg det_image_seq1_MIRIMAGE_F1130Wexp1_refpixstep.fits  --override_readnoise my_readnoise.fits --override_gain my_gain.fits
```



### Manually set the `rejection_threshold` parameter

By default, the rejection threshold is set at 4 sigma. To manually set this in Python:

```python
dm = jump.JumpStep.call(my_input_file, output_use_model=True, save_results=True,
                        rejection_threshold=3.0)
```



and using the command line:

```bash
strun jwst.jump.JumpStep det_image_seq1_MIRIMAGE_F1130Wexp1_refpixstep.fits  --rejection_threshold 3.0
```

or

```bash
strun cfgs/jump.cfg det_image_seq1_MIRIMAGE_F1130Wexp1_refpixstep.fits --rejection_threshold 3.0
```

