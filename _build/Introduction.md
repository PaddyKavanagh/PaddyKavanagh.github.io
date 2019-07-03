---
redirect_from:
  - "/introduction"
interact_link: content/Introduction.ipynb
kernel_name: python3
has_widgets: false
title: 'Home'
prev_page:
  url: 
  title: ''
next_page:
  url: /Level1_Detector1Pipeline
  title: 'Ramps-to-slopes (CALDETECTOR1)'
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


# JWST Calibration Pipeline Build 7.3 for MIRI Team

## Introduction

This Jupyter book provides instructions on running the JWST Calibration Pipeline and individual pipeline steps. Information in this book is collated from the official JWST Calibration Pipeline documentation (see link below), MIRI CDP documentation, MIRI Team Wiki, etc. The goal is to provide information, user instructions, examples and issues for each pipeline and steps (except coronography). More detailed information on the pipeline can be found here:

<https://jwst-pipeline.readthedocs.io/en/latest/>

If you have or find any other issues installing and running the pipeline you can report them on the MIRICLE Bugzilla page (you need to have/set up a Bugzilla account) here:

<http://www.miricle.org/bugzilla/>

Select the 'Pipeline support' option in the component menu when filing the new report 


### Pipeline stages

There are three different stages of processing for the JWST observing modes:

- Stage 1: Detector-level corrections and ramp fitting for individual exposures

- Stage 2: Instrument-mode calibrations for individual exposures

- Stage 3: Combining data from multiple exposures within an observation

Stage 1 corrections are applied nearly universally for all instruments and modes. Stage 2 is divided into separate modules for imaging and spectroscopic modes. Stage 3 is divided into five separate modules for imaging, spectroscopic, coronagraphic, Aperture Masking Interferometry (AMI), and Time Series Observation (TSO) modes.

This book is split into chapters according to these stages and application to relevant MIRI modes.




## Installation

This book is specifically for Build 7.3 of the JWST Calibration Pipeline. To install this version you must have an Anaconda installation:

<https://www.anaconda.com/>

To install, open a terminal:

```bash
conda create -n jwst_env --file <URL>
conda activate jwst_env
```

where `<URL>` is dependent on the operating system:
    
Linux: <http://ssb.stsci.edu/releases/jwstdp/0.13.7/latest-linux>

OS X: <http://ssb.stsci.edu/releases/jwstdp/0.13.7/latest-osx>





## Calibration Reference Data System (CRDS)

The CRDS is the calibration file server for the JWST Calibration Pipeline. When running the pipeline, the calibration steps will use the `crds` task to fetch appropriate files from the CRDS and store them locally. Therefore, it is important that you have this setup correctly.

### CRDS configuration

To use the CRDS server, one must set the `CRDS_PATH` and `CRDS_SERVER_URL` environment variables which tell the `crds` task where to find and store the calibration files. Set these (e.g., in your bash login file): 

```bash
export CRDS_PATH=/path/to/your/crds
export CRDS_SERVER_URL="https://jwst-crds.stsci.edu"
```

The reference file mapping is set using the `CRDS_CONTEXT` environment variable. For Build 7.3, this should be set to 535 and will contain the CRDS-ingested MIRI CDP7 files: 

```bash
export CRDS_CONTEXT='jwst_0535.pmap'
```

Note that the calibration files can be large (~GB) and may take some time to download. However, the `crds` task will only fetch calibration files from the server if they are not on your system so they will only be downloaded once. Using the CRDS server will require the least amount of CRDS file storage on your system. 



## Pipeline output

Unless the user specifies an output filename, the various pipeline modules will use the input root along with a set of predetermined suffixes to set the outputs. The output suffix will always replace the input suffix. The list of default suffixes is:

|Product|Suffix| 
|:---|:---|
|uncalibrated level 1B|uncal|
|level 2A ramp|ramp|
|level 2A count rate image|rate|
|level 2A count rate per integration|rateints|
|optional fitting results from ramp_fit step|fitopt|
|level 2B background-subtracted image|bsub|
|level 2B background-subtracted image per integration|bsubints|
|fully calibrated level 2B image|cal|
|fully calibrated level 2B images per integration|calints|
|1D extracted spectrum|x1d|
|1D extracted spectra per integration|x1dints|
|Resampled image|i2d|
|Resampled spectrum|s2d|
|Resampled 3D IFU cube|s3d|





## Running the JWST pipeline

The JWST pipeline can be run either from the command line or from within python. The examples in this book will demonstrate both options.

The instructions and tutorials for the pipelines and the steps can be found under the following chapters:

* Ramps-to-slopes


