# General Guideline for MRI file processing
## All the analysis are currently conducted on the sylvian server.
## The project folder should be backed up to our qilab server once every week.
* All the dicoms files should be saved at `/home/qigroup/Documents/projects/{projectname}/dicoms/`
* The EP2D_DIFF_SMS_ABCD_TENSOR_* series are not currently convertable by heudiconv. Please mv this folder to ./tensor/{subj}/.
* All the nifti files should be saved at `/home/qigroup/Documents/projects/{projectname}/bids/`
* Backup all the dicoms to our lab server: `/data/project/{projectname}`
* Create a folder for freesurfer outputs at `/home/qigroup/Documents/projects/{projectname}/surface`
* Make a symbolic link for the surface folder at ./bids/derivatives/freesurfer:`ln -s`

## To convert dicoms to BIDS formatted nifti
### Make sure docker is running on the imac
### On the terminal, type:
```
cd /home/nas/projects/{projectname}/
docker run --rm -it --entrypoint=bash -v $(pwd):/data nipy/heudiconv:latest
```
Now you will be inside of the container. Type:
```
cd /data
```
The first step is to run a dry pass (no conversion), which will stack and group the dicoms into series.
```
heudiconv -d /data/dicoms/{subject}/*/*/*.IMA -s subjectID -f convertall -c none -o /data/bids
```
Within /output/.heudiconv/subjectID/info, you will find a dicominfo.tsv. We will convert this file to specify bids format.

### create the heuristic file at /data/projectname_heuristic.py
example here: <http://nipy.org/heudiconv/#22>
for most updated bids specification: <http://bids.neuroimaging.io/bids_spec.pdf>

### run the conversion
## anything in all caps must be edited
```
rm -r -f /data/niftis/*
heudiconv -d /data/data/mri/imaging/dicoms/{subject}/*/*/*.IMA -s SUBJECTID -f /data/data/mri/imaging/PROJECTNAME_heuristics.py -c dcm2niix -b -o /data/bids
```

## Run freesurfer
### first define the FREESURFER directories
```
export FREESURFER_HOME=/usr/local/freesurfer/bin/freesurfer
source $FREESURFER_HOME/SetUpFreeSurfer.sh
export SUBJECTS_DIR=/home/qigroup/Documents/projects/projectname/freesurfer
```
### if the subject has T2w scans, type the following in the terminal (replace subjid with the real subject id)
```
recon-all -subject subjid -i /home/qigroup/Documents/projects/projectname/bids/subjid/anat/subjid_T1w.nii.gz -T2 /home/qigroup/Documents/projects/projectname/bids/subjid/anat/subjid_T2w.nii.gz -T2pial -all
```
### if the subject does not have T2 scan, type the following in the terminal
```
recon-all -autorecon-all -subject subjid -i /home/qigroup/Documents/projects/projectname/bids/subjid/anat/subjid_T1w.nii.gz
```
## Edit Freesurfer outputs
### Use the freesurfer editing GUI
`python /home/qigroup/Documents/projects/blast/scripts/edit_freesurfer_surface.py`
### For more documentation and tips, see the lab wiki: <https://github.com/Qlab-UDel/qlab/wiki/FreeSurfer-Overview> and the official wiki: <http://ftp.nmr.mgh.harvard.edu/fswiki/FreeSurferWiki>

###To validate BIDS files, see the BIDS validator: <http://incf.github.io/bids-validator/>
