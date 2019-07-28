# TCGA-GBM Dataset Patient Pickle File Generation

This repository was created to log and report all the steps and stages taken in the pre-training
phase of the brain tumor deep learning detection module. In a nutshell, the python notebooks in 
this repository iterates over all the patient dicom files and their correspondant segmentation
files and stores saves the image arrays in a separate file for every patient. This was done in a number 
of stages. These were:

- **Stage #1:** Storing the patient scans and their respective segmentation scans in a pickle file for every patient.
- **Stage #2:** Sampling all the pickle file data to have a shape of (256,256) as a perpping generalization phase prior
to using these images in the network.

## Stage One: Dataset iteration and storage of patient data in pickle files
In this stage it first is important to understand how the final data file is supposed to look like in order to be able to understand the procedure taken to build it.
The pickle file is named with the patient ID. For example: *TCGA-xx-xxxx.pickle*.
The pickle file components is an array composed of 5 arrays. The first 4 arrays are the original scans
of the patient corresponding to the available scan segmentations. These scan types are mainly **T1, T2, FLAIR and POST**. Each of the 4 arrays of the scan types contain inside it the slices. The 5th array is composed of 4 arrays inside it each of which coressponds to one of the original scan arrays. To put it into picture, consider the following illustration:

<div align="center">
<img src = "media/pickle-ds.jpg" width="525" height="525">
</div>

The proceure in which the generation of the patient pickle files is coded in the `Generalization.ipynb` in the following steps:

- Patient folders in the segmentation dataset directory are placed in an array to iterate over them.
- Iterating over every patient, a list of the **corrected** dicom segmentation filename is generated.
- Creating a `data_array` which will contain all 5 arrays of the patient (4 for original scans and 1 for segmentation).
- Retreiving the slices from the scans that have segmentation only. (Filtering was through the unique scan ID).
- For every scan an array was generated, and the slices were iterated upon. Using `pixel_array` attribute from the dicom object, we fetched the slice 2D array and appended it to the scan array.
- The corresponding segmentation file of the scan was accessed, and slice 2D arrays were stored.
- The scan array is appended to the `data_array` for every scan and the segmentation array is appended last prior to closing and saving the pickle file.

### Problems faced at stage one
- **Patient Multiple Visits**  
Two patients had more than one visit for the MRI scanning, each with a different set of scans unlike most patients where only one set of scans was available in the dataset.  
To resolve this, a helper function `multipleVisits` was declared to return the folder name/visit instance conaining the needed scans corresponding to those from the segmentation.
- **DICOM files sorting**  
The dicom files for the slices were numbered. However, the numbered filenames was not of correct order of alignment. Ther is a DICOM object attribute called `InstanceNumber` containing the number of the slice in the correct alignment.  
To resolve this, a helper function `dicomSort` was declared to return a dictionary where the key is the InstanceNumber and the value for each key is the corresponding dcm filename.
- **Retreiving the segmenation for every scan**  
The segmentation for every scan is not stored in differnt dcm file for every slice like the original scans, but instead all the slices segmentation is stored in one dcm file.
We have 3 types of tumor segments:
    - Edema
    - Non-Enhancing Tumor
    - Tumor Core.  
    To resolve this, a helper function `segmentationArray` was declared to create a unified label, summing up all the 3 tumor segments we have to retreive the array corresponding to the slice at hand and finally return the unified label.

**Important Observation:** Not all the 2D arrays are of shape (256,256). This led us to go through stage two of the pickle file grooming for the training phase i.e. Sampling all the arrays to have a (256,256) slice and segmentation arrays.


## Stage Two: Sampling pickle file arrays to a unified shape
There is a total of 17/29 patients with arrays in the shape of (256,256). The remaining 12 patients had varied array shapes. The anomalous shapes were:
- (512, 512)
- (256, 192)
- (352, 352)
- (320, 320)
- (384, 320)
- (256, 224)
- (256, 208)  

The `Sampling Implementation.ipynb` contains a helper function that takes in the pickle file of the patient with the anomalous shape and creates another pickle file for that patient with resized arrays of shape (256,256) using the `cv2.resize` function.  
The helper function declared is called `downSample`. The section where the function is implemented only generates pickle files for the 12 anomalous patients, meaning that we need to copy the remaining 17 from the original pickle file directory that was created at stage one.

## Next Steps
- Order the scan types in the pickle file (prior to generation and sampling)
- choose 3 patients for validation and 23 for training.
| Training | Validation |
| -------- | ---------- |
| 30x26x256x256x4 | 30x3x256x256x4 |
| 30x26x256x256x1 | 30x3x256x256x1 |
- Build the model for training

## References
- [TCIA TCGA-GBM Dataset](https://wiki.cancerimagingarchive.net/display/Public/TCGA-GBM)
- [TCGA-GBM Segmentation Dataset](https://app.box.com/s/sljtgos3u2j2q33cn51se0c6scmtpng6)  
**Important Observation** The Segmentation dataset hyperlink in the references is updated with segmentations to 102 and not only 29 patients.