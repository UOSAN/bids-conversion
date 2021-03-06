# bids-conversion
Convert DICOMS to BIDS. More information about BIDS specifications can be found here.

## BIDS conversion
Convert DICOMS to [BIDS](http://bids.neuroimaging.io/). More information about BIDS specifications can be found [here](http://bids.neuroimaging.io/bids_spec1.0.2.pdf).

### Key scripts
**`convert_bids_config.sh`**
This script specifies the study-specific variables and paths to use in `convert_bids.sh`. **This script must be modified by the user**

**`batch_convert_bids.sh`**
The script creates a conversion job for each subject in `subject_list.txt` and submits the job using sbatch. **This script must be modified by the user**

**`convert_bids.sh`**
This script will convert dicoms, create the BIDS files structure, and move and rename converted files to BIDS specification. It will also check the number of files and print errors to the error log.

**`createJson_config.sh`**
This script specifies the study-specific variables and paths to use in `createJson.sh`. **This script must be modified by the user**

**`batch_createJson.sh`**
This script creates a conversion job for each subject in `subject_list.txt` and submits the job using sbatch. **This script must be modified by the user**

**`createJson.sh`**
This script will check whether the subject-specific MRI acquisition details match the overall group specifications. If they differ, the script will create subject-specific Json files with appropriate parameters for subsequent use in BIDS apps.

**`extract_dicom_fields.py`**
This script was built by Jolinda (LCNI) to extract some relevant info to build the Json script. It is called upon by `convert_bids.sh`.

### Metadata
This folder contains group-level information that needs to be modified. These files will be copied to the bids directory by `convert_bids.sh`. 

**`dataset_description.json`**
This metadata file specifies the dataset name and BIDS version. **This script must be modified by the user**

**`T1w.json`**
This metadata file specifies the anatomical scan parameters and is necessary to run many BIDS apps. This info can be found in the scan protocol. This file is not necessary for simply converting the files to BIDS format. **This script must be modified by the user**

**`task-<insert name>.json`**
This metadata file specifies the functional scan parameters and is necessary to run many BIDS apps. A seperate file must be created for each functional task and resting state scan. This info can be found in the scan protocol. This file is not necessary for simply converting the files to BIDS format. **This script must be modified by the user**

**`phasediff.json`**
This metadata file specifies the scan parameters for fieldmaps and what functional sequences it should be applied to. This file is necessary for many BIDS apps to appropriately apply fieldmap correction. This info can be found in the scan protocol. This file is not necessary for simply converting the files to BIDS format. **This script must be modified by the user**

### Output
For each subject, a BIDS file structure will be created in `$bidsdir` as specified in `convert_bids_config.sh`. DICOMS will then be converted to niftis, renamed according to the BIDS specification, and copied into the newly created BIDS file structure. 

All DICOMS will be converted to `$niidir` as specified in `convert_bids_config.sh` and these files will be checked before renaming and moving them to `$bidsdir`. Errors will be printed to `errorlog.txt`. More detailed individual output and error logs are written to `$OUTPUTDIR` as specified in `batch_convert_bids.sh`.

For each subject, a session and task (i.e. func, anat) specific json file will be created only if the acquisition parameters do not match the overall group parameters. In addition, all subject and session specific json files for fmaps will be created for all participants (regardlesso of whether these parameters match the group json file), as is required by some BIDS apps.  

When errors are found, the following rules are employed:
1. **Anatomical images:** If there are multiple anatomical images, rename and copy all files to `$bidsdir`. These structural images will have `run-01` and `run-02` appended to their names to distinguish them.
2. **Fieldmaps:** If there are more than 2 fieldmaps (ap + pa) for epi fieldmaps, or if there are more than 3 (phase + magnitude images) for traditional fieldmaps, no fieldmaps will be renamed or copied to `$bidsdir`. 
3. **DTI:** If there are not exactly 2 files (rl + lr), no images will be renamed or copied to `$bidsdir`. 
4. **Resting state:** If there are more than 2 run with the same name, no images will be renamed or copied to `$bidsdir`. 
5. **Functional task runs:** If there are multiple functional task runs, the largest task run will be renamed and copied to `$bidsdir`. 
