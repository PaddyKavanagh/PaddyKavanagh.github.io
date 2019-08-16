---
redirect_from:
  - "/level2-spec2pipeline-mrs"
interact_link: content/Level2_Spec2Pipeline_MRS.ipynb
kernel_name: python3
has_widgets: false
title: 'Level 2 MRS (CALSPEC2)'
prev_page:
  url: /Level3_Image3Pipeline
  title: 'Level 3 Imager (CALIMAGE3)'
next_page:
  url: 
  title: ''
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


## Spec2Pipeline for MRS   (MIRI MRS CALSPEC2)

`Spec2Pipeline` applies additional instrumental corrections and calibrations to count rate products that result in a fully calibrated individual exposure.

Official documentation for `Spec2Pipeline` can be found here:

<https://jwst-pipeline.readthedocs.io/en/latest/jwst/pipeline/calwebb_spec2.html>

The `Spec2Pipeline` for MRS comprises a linear series of steps. The steps in order are:

|Step|Description|
|:---|:---|
|`assign_wcs`|attach WCS object to each science exposure|
|`background`|subtract dedicated background exposures|
|`flat_field`|correct the exposure using the flat-field reference file|
|`srctype`|if APT source type set to 'UNKNOWN', sets the 'SRCTYPE' keyword to 'EXTENDED' for MRS|
|`straylight`|remove straylight for Channels 1/2 by interpolating across the inter-slice regions|
|`fringe`|divide the data/error arrays by a fringe reference image|
|`photom`|apply flux (photometric) calibration|
|`cube_build`|produce 3D spectral cubes from the 2D detector image|
|`extract_1d`|extract 1D spectrum from 3D cube|


For more information and examples of each of the steps click on the links in the side bar. 

Note that `cube_build` and `extract_1d` are not calibration steps but are contained in `Spec2Pipeline` to provide users with cubes and 1D spectra from each exposure. Most users of these notebooks may not want to run these steps as they are only interested in the calibrated detector images and the `cube_build` step in particular will dramatically processing time. Therefore, these steps will be skipped.

In this example, no background exposure will be provided so the `background` step will not be run.




### Input data

An example of running the file through the `Spec2Pipeline` is now shown using a simple simulated fullband, 2-point dither observation of a point source (blackbody plus emission lines spectrum) with the MIRI MRS produced with [MIRISim v2.1](http://miri.ster.kuleuven.be/bin/view/Public/MIRISimPublicRelease2dot1) and brought to level 2A using the `Detector1Pipeline`. 



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
from jwst.associations.lib.member import Member
from jwst.associations.asn_from_list import asn_from_list
from jwst.associations.lib.rules_level2_base import DMSLevel2bBase

# set the CRDS_CONTEXT
os.environ["CRDS_CONTEXT"] = "jwst_0535.pmap"

```
</div>

</div>



`Spec2Pipeline` can be run by passing indivual exposures or datamodels but in this sample case we produce an association file so all can be run through `Spec2Pipeline` in a single call. It is assumed that the science files are in 'MRS/SHORT', 'MRS/MEDIUM', and 'MRS/LONG'. The content of the association file is printed for inspection.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
# load the science and background files to lists
my_science_files = glob.glob('MRS/*/*rate.fits')
my_science_files = sorted(my_science_files)

# setup an empty level 2 association structure
asn = asn_from_list(my_science_files, rule=DMSLevel2bBase)
asn.data['products'] = None

# set the association name
asn_name = 'my_point_source'

# set some metadata
asn['asn_pool'] = asn_name + '_pool'
asn['asn_type'] = 'spec2'

for n, sci in enumerate(my_science_files):
    asn.new_product('{}_exposure{}'.format(asn_name, str(n+1)))
    sci_member = Member({'expname': sci, 'exptype': 'science'})    
    new_members = asn.current_product['members']
    new_members.append(sci_member)
        
# print the association and save to file
name, ser = asn.dump()
print(ser)

asn_file = asn_name + '_lvl2_asn.json'
with open(asn_file, 'w') as f:
    f.write(ser)

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```
{
    "asn_type": "spec2",
    "asn_rule": "DMSLevel2bBase",
    "version_id": null,
    "code_version": "0.13.7",
    "degraded_status": "No known degraded exposures in association.",
    "program": "noprogram",
    "products": [
        {
            "name": "my_point_source_exposure1",
            "members": [
                {
                    "expname": "MRS/LONG/det_image_seq1_MIRIFULONG_34LONGexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure2",
            "members": [
                {
                    "expname": "MRS/LONG/det_image_seq1_MIRIFUSHORT_12LONGexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure3",
            "members": [
                {
                    "expname": "MRS/LONG/det_image_seq2_MIRIFULONG_34LONGexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure4",
            "members": [
                {
                    "expname": "MRS/LONG/det_image_seq2_MIRIFUSHORT_12LONGexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure5",
            "members": [
                {
                    "expname": "MRS/MEDIUM/det_image_seq1_MIRIFULONG_34MEDIUMexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure6",
            "members": [
                {
                    "expname": "MRS/MEDIUM/det_image_seq1_MIRIFUSHORT_12MEDIUMexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure7",
            "members": [
                {
                    "expname": "MRS/MEDIUM/det_image_seq2_MIRIFULONG_34MEDIUMexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure8",
            "members": [
                {
                    "expname": "MRS/MEDIUM/det_image_seq2_MIRIFUSHORT_12MEDIUMexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure9",
            "members": [
                {
                    "expname": "MRS/SHORT/det_image_seq1_MIRIFULONG_34SHORTexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure10",
            "members": [
                {
                    "expname": "MRS/SHORT/det_image_seq1_MIRIFUSHORT_12SHORTexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure11",
            "members": [
                {
                    "expname": "MRS/SHORT/det_image_seq2_MIRIFULONG_34SHORTexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure12",
            "members": [
                {
                    "expname": "MRS/SHORT/det_image_seq2_MIRIFUSHORT_12SHORTexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        }
    ],
    "asn_pool": "my_point_source_pool"
}
```
</div>
</div>
</div>



Import Spec2Pipeline and print the docstring to show some information



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
from jwst.pipeline import Spec2Pipeline
print(Spec2Pipeline.__doc__)

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```

    Spec2Pipeline: Processes JWST spectroscopic exposures from Level 2a to 2b.
    Accepts a single exposure or an association as input.

    Included steps are:
    assign_wcs, background subtraction, NIRSpec MSA imprint subtraction,
    NIRSpec MSA bad shutter flagging, 2-D subwindow extraction, flat field,
    source type decision, straylight, fringe, pathloss, barshadow,  photom,
    resample_spec, cube_build, and extract_1d.
    
```
</div>
</div>
</div>



The association file is passed as input. The output level 2B files will be saved in `my_output_dir` as `_cal.fits`. The cubes and 1D spectra will also be saved.

*Parameters used:*
    
`save_results`: boolean, optional, default=False  
&nbsp;&nbsp;&nbsp;&nbsp; save the results to file
    
`output_dir` : boolean, optional, default is the working directory   
&nbsp;&nbsp;&nbsp;&nbsp; the location to save the output

`steps` : dict, optional, default None  
&nbsp;&nbsp;&nbsp;&nbsp; pass parameters to steps

The `skip` parameters is passed to `cube_build` and `extract_1d` and we explictly save the `photom` output using the `save_results` parameter.
    
The output filenames will be determined by the `asn_name` in the association file which was set in the cell above.

Note that the following cell will result in an error at the very end of execution:

```
RuntimeError: One or more products failed to process. Failing calibration.
```

This results from skipping the `cube_build` and `extract_1d` steps and can be ignored.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
# user specified
my_output_dir = 'demo_output'

# the output directory should be created if it doesn't exist
if not os.path.exists(my_output_dir): 
    os.mkdir(my_output_dir)

# run the pipeline
Spec2Pipeline.call(asn_file, save_results=True, output_dir=my_output_dir, 
                   steps={'photom': {'save_results': True}, 
                          'cube_build': {'skip': True}, 'extract_1d': {'skip': True}})


```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_traceback_line}
```

    ---------------------------------------------------------------------------

    RuntimeError                              Traceback (most recent call last)

    <ipython-input-7-188d0373c8ab> in <module>()
          8 # run the pipeline
          9 Spec2Pipeline.call(asn_file, save_results=True, output_dir=my_output_dir, 
    ---> 10                    steps={'photom': {'save_results': True}, 'cube_build': {'skip': True}, 'extract_1d': {'skip': True}})
    

    /anaconda3/envs/jwst7.3/lib/python3.6/site-packages/jwst-0.13.7-py3.6.egg/jwst/stpipe/step.py in call(cls, *args, **kwargs)
        533         else:
        534             instance = cls(**kwargs)
    --> 535         return instance.run(*args)
        536 
        537     @property


    /anaconda3/envs/jwst7.3/lib/python3.6/site-packages/jwst-0.13.7-py3.6.egg/jwst/stpipe/step.py in run(self, *args)
        392                     self.prefetch(*args)
        393                 try:
    --> 394                     step_result = self.process(*args)
        395                 except TypeError as e:
        396                     if "process() takes exactly" in str(e):


    /anaconda3/envs/jwst7.3/lib/python3.6/site-packages/jwst-0.13.7-py3.6.egg/jwst/pipeline/calwebb_spec2.py in process(self, input)
        105         if has_exceptions and self.fail_on_exception:
        106             raise RuntimeError(
    --> 107                 'One or more products failed to process. Failing calibration.'
        108             )
        109 


    RuntimeError: One or more products failed to process. Failing calibration.


```
</div>
</div>
</div>



We can plot the before (rate) and after (cal) images for the first dither position of the SHORT band



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
# use the association to identify the linked files
cal_file = asn['products'][9]['name'] + '.fits'
rate_file = asn['products'][9]['members'][0]['expname']

# open the input level 2A image and calibrated 2B image a jwst data models
with datamodels.open(rate_file) as in_dm:
    with datamodels.open(rate_file) as out_dm:

        fig, axs = plt.subplots(1, 2, figsize=(14, 7), sharey=True)

        axs[0].imshow(in_dm.data, cmap='jet', interpolation='nearest', origin='lower', norm=LogNorm(vmin=1, vmax=100))
        axs[0].annotate('DMS Level 2A: {}'.format(os.path.basename(rate_file)), xy=(0.0, 1.02), xycoords='axes fraction', 
                        fontsize=10, fontweight='bold', color='k')
        axs[0].set_facecolor('black')
        axs[1].imshow(out_dm.data, cmap='jet', interpolation='nearest', origin='lower', norm=LogNorm(vmin=1, vmax=100))
        axs[1].annotate('DMS Level 2B: {}'.format(cal_file), xy=(0.0, 1.02), xycoords='axes fraction', 
                        fontsize=10, fontweight='bold', color='k')
        axs[1].set_facecolor('black')
        plt.tight_layout()
        plt.show()


```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](images/Level2_Spec2Pipeline_MRS_11_0.png)

</div>
</div>
</div>



### Command line

To achieve the same result from the command line there are a couple of options. However, one must still generate the association file. In this case, it is best to copy the template above into a text file and save it to `my_point_source_lvl2_asn.json`. The content is printed here for convenience.



```python
{
    "asn_type": "spec2",
    "asn_rule": "DMSLevel2bBase",
    "version_id": null,
    "code_version": "0.13.7",
    "degraded_status": "No known degraded exposures in association.",
    "program": "noprogram",
    "products": [
        {
            "name": "my_point_source_exposure1",
            "members": [
                {
                    "expname": "MRS/LONG/det_image_seq1_MIRIFULONG_34LONGexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure2",
            "members": [
                {
                    "expname": "MRS/LONG/det_image_seq1_MIRIFUSHORT_12LONGexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure3",
            "members": [
                {
                    "expname": "MRS/LONG/det_image_seq2_MIRIFULONG_34LONGexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure4",
            "members": [
                {
                    "expname": "MRS/LONG/det_image_seq2_MIRIFUSHORT_12LONGexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure5",
            "members": [
                {
                    "expname": "MRS/MEDIUM/det_image_seq1_MIRIFULONG_34MEDIUMexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure6",
            "members": [
                {
                    "expname": "MRS/MEDIUM/det_image_seq1_MIRIFUSHORT_12MEDIUMexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure7",
            "members": [
                {
                    "expname": "MRS/MEDIUM/det_image_seq2_MIRIFULONG_34MEDIUMexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure8",
            "members": [
                {
                    "expname": "MRS/MEDIUM/det_image_seq2_MIRIFUSHORT_12MEDIUMexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure9",
            "members": [
                {
                    "expname": "MRS/SHORT/det_image_seq1_MIRIFULONG_34SHORTexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure10",
            "members": [
                {
                    "expname": "MRS/SHORT/det_image_seq1_MIRIFUSHORT_12SHORTexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure11",
            "members": [
                {
                    "expname": "MRS/SHORT/det_image_seq2_MIRIFULONG_34SHORTexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        },
        {
            "name": "my_point_source_exposure12",
            "members": [
                {
                    "expname": "MRS/SHORT/det_image_seq2_MIRIFUSHORT_12SHORTexp1_rate.fits",
                    "exptype": "science"
                }
            ]
        }
    ],
    "asn_pool": "my_point_source_pool"
}
```



**Option 1:**
Run the `Spec2Pipeline` class using the `strun` command:

```bash
mkdir demo_output

strun jwst.pipeline.Spec2Pipeline my_point_source_lvl2_asn.json --output_dir demo_output --steps.photom.save_results True --steps.cube_build.skip True --steps.extract_1d.skip True
```

This will produce the same output file in the user-defined `--output_dir`


**Option 2:**
Collect the pipeline configuration files in your working directory (if they are not already there) using `collect_pipeline_configs` and then run the `Spec2Pipeline` using the `strun` command with the associated `calwebb_spec2.cfg` file. This option is a little more flexible as one can create edit the cfg files, use them again, etc.

```bash
mkdir demo_output

collect_pipeline_cfgs cfgs/

strun cfgs/calwebb_spec2.cfg my_point_source_lvl2_asn.json --output_dir demo_output --steps.photom.save_results True --steps.cube_build.skip True --steps.extract_1d.skip True
```

This will produce the same output file in the user-defined `--output_dir`



## Further examples

Other notebooks with more complex examples can be found here:

*To be added*

