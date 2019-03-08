#  BigDataProcessor

## Summary


The BigDataProcessor (BDP) enables interactive browsing of Terabyte sized Tiff and Hdf5 based image data, employing ImageJ’s VirtualStack class (LINK), where only the currently displayed image plane is loaded into RAM. 
BDC facilitates serveral common household data processing routines aimed at shortening the (pre-)processing time spent on large image datasets. BDC supports channel (chromatic), and spatial drift correction, cropping and saving of big image data including binning and bit-depth conversion. 

**Example use cases:**  
Browsing large image data in a familiar ImageJ/FIJI environment.
Perform (near) interactive tracking of cells in TB sized image data.
Crop and resave ROI in TB-size timelapses.  
Drift correct 3D-images.
Align chromatic shift between image channels in timelapses. 




## Background/Overview

Inspection and manipulation of TB sized image data as produced by light-sheet and electron microscopy poses several challenges. Even  loading the data from disc into RAM is very time consuming or not possible. Is is thus necessary to employ lazy-loading strategies that, e.g., only load the currently visible fraction of the data set into RAM (see e.g., BigDataViewer [1]).

The BigDataProcessor (BDP) enables fast lazy-loading and interactive browsing of Terabyte sized Tiff and Hdf5 based image data employing ImageJ’s VirtualStack class (LINK), where only the currently displayed image plane is loaded into RAM. The data is presented within the popular ImageJ/FIJI software (Hyperstack Viewer) well known throughout the life sciences. All ImageJ measurement tools, such as line profile or regions of interest based measurements are available. In fact, essentially all of ImageJ's functionality is available, however one has to pay attention, because some operations will attempt to copy the data into RAM, which, of course, will fail is the data are too big. 
In addition to viewing big image data, the BDP supports cropping and saving of big image data including binning and bit-depth conversion. This functionality is useful, because raw microscopy data is often not in an ideal state for image analysis. For example, only part of the acquired data may be of actual interest, either because larger fields of view have been acquired to compensate for unpredictable sample motion, or scientifically interesting phenomena have only occurred in specific parts of the imaged sample. Moreover, pixel density and bit-depth can be unnecessarily high, e.g., because camera based microscope systems with fixed pixel size and bit-depth have been used. Or the raw data file-format might simply not be compatible with the analysis software.

In addition to conventional "static" cropping, the BDC also provides an "adaptive" cropping functionality, based on center-of-mass or cross-correlation based tracking. This is useful when the sample moves during acquisition (e.g., due to microscope drift or biological motility), rendering a static crop of the image suboptimal.

Finally, chromatic shifts can be interactively corrected by specifying x and y pixel offsets.

## Supported file formats

Currently, for both reading and writing we support Tiff and hdf5 based image data. To our knowledge those are currently the most popular (open-source) file formats.

### Reading

For reading, the BDC supports pattern matching based file parsing to accommodate different naming schemes, specifying z-slice, channels, and time-points.

Example use-cases include:

- Luxendo hdf5 based light-sheet data.
- Leica Tiff based light-sheet data.
- Electron microscopy Tiff based data.
- Custom-built light-sheet microscope Tiff based data.

### Writing

The BDC supports writing to Tiff and Hdf5 files. For Tiff writing one can choose between one file per plane or one file per channel and time-point. For Hdf5 writing, an Imaris compatible multi-resolution file format is supported with channels and time-points in separate files, linked together by one "header" hdf5 file.

## Installation

The Big Data Converter runs as a PlugIn within Fiji.

- Please install [Fiji](fiji.sc)
- Within Fiji, please enable the [Update Site](https://imagej.net/Update_Sites):
  - [X] EMBL-CBA
```diff
 - what is the correct path? more info..
```

## Starting Big Data Converter

The Big Data Converter can be found in Fiji's menu:

- [Fiji > Plugins > BigDataTools > Big Data Converter]
 


 ---

## User Guide
Below we explain the functionality, different options and available settings in the ImageJ/FIJI plugins BigDataConverter, DataStreamingTools and BigDataTracker.


### Loading Tab
 
**Image Data Folder:** Sets the path to the image data.  
```diff
 - Files needs to have the same size ?
```
**File Naming scheme:**  Under this drop down menu, a number of common file storage conventions are found. E.g single plane tiffs that have the naming convention _.*--C<c>--T<t>.tif_. _None_, reads all files in folder. _Channels from sub-folders_, reads all data from all subfolders given in path and treat them as channels. None (Default)

**Only load files matching:** Accepts a string that limits the selection to only files containing this string in the chosen folder (.* Default)   

**Hdf5 data set name:** Defines the path to the data within the .h5 files when reading hdf5 datasets. None (Default)

### Channel shift correction

**chromatic shift in pixels for each channel** Defined shifts between  [x1,y1,z1;x2,y2,z2;...,xn,yn,zn]  (0,0,0;0,0,0 Default)


### Cropping Tab
x/y extend is given by a rectangle ROI ( the leftmost icon of the Fiji GUI list)

**z-min,z-max[ slices]**
Comma separated boundaries in z to define a crop region to crop out a subset of original stream.
1,all (Default)

**t-min,t-max[ slices]**
Comma separated boundaries in time, to define a cropped subset of original 
stream.   
1,all (Default)

**Crop**
Executes the crop command and shows the new Stream with the cropped dimensions.


### Motion Correction Tab
TRACKING panel:

**Set x&y from ROI** , **Set Z**,  Region size x,y,z In pixels  200,200,30 (Default)   

**Maximal displacement between subsequent frames:** Maximally allowed distance (x,y,z distance) between two timepoints in track.
150,150,150 (Default)

**Down-sample: dx,dy,xz,dt [pixels,frames]:**  Sets the down-sampling of the data used for tracking in dx (pixels) dy (pixel)  dz (pixels)  dt (frames)**  1,1,1,1 (Default)    
**Track Length [frames]:** Tracking length of desired track, starting from the current position in stream window. Length is given in (int) frames. 3  (Default)
**Process Region: Intenstity Gating [min, max]:** Used for setting an intensity gating between two [min, max] values  

**Process Region Filter:** A setting for tracking method _center of mass_. For example useful if strong objects passes by in vicinity of the tracked object or to remove noise that could affect the tracking 
Enhance image features  Allow image features to be enhanced using a pre-filtering step based on threshold – where the variance is calculated before?  (how are these set?) .   (Default, No Filter )

**Process Region Show Rgions[#]:** Number of regions shown (  Default, 3) 

**Tracking method** The tracking is performed eith used  _correlation_ (Default) which is described [2] or _Center of Mass_.  updates 
*Correlation tracking* calculates the shift between subsequent images that maximizes the sum of the pixel-wise product of both images, thereby trying to match bright pixels in image1 onto bright pixels in image2. One issue with this approach is the situation when, e.g. image2 features a really bright region that was not present already in image1. The correlation will be maximal if this really bright region falls onto some bright region of image1, even if pixel-wise match is not perfect. In other words the correlation has the tendency to simply find a shift that puts the two brightest regions of both images on top of each other, even if they don't look exactly the same. That is why we implemented the "threshold" image feature enhancement method, which replaces all pixel values by 0 or a constant, thereby counteracting the overly strong weight of very bright image regions. 
 

**Track** Executes tracking based on selected method and settings.
 
**Interrupt Tracking**  

**Show tracking regions**    
Tracking method Selection of which tracking method to use  – correlation,  [ref.]  and  - center of mass  calculating 

Stop tracking Stops the current tracking calculations.

TRACKING TABLE:
**Results Table**
Show table Opens a interactive tracking box where 
Shows

**Save table**  Saves the tracking table to a  the format can be 

**Clear all tracks**  Remove all tracks in memory 

VIEW TRACKED OBJECTS panel

**Resize Tracked Regions [factor]**:  (  Default, 1) 
**View** Opens a new stream with of the  

### Saving Tab

**Save As:**
Sets save settings for stream to be saved in one of the supported fileformats. Currently four filetypes/formats to save the data are supported, Tiff stacks (.tif) Tiff planes (.tif), hdf5 (.h5) and The Imaris open format (.h5).

**Binnings [pixels]** : x1,y1,z1;x2,y2,z2;.. 
Bin down the stream in x,y,z. given by xi,yi,zi (where i is number of channel). For multichannel data, using only one triplet of values performs same binning for all channels.   
1,1,1 (Default)

**Save volume data**
Needs to be checked to save stacks when save as stacks and save as planes are pressed, uncheck for example when only projections needs to be saved. checked (Default) 

**Save projections** 
Saves xyz-projections of the data in one combined stack. unchecked (Default)

**LZW compression** When checked each tiff slice will be LZW compressed Strips ?? what settings?. Only valid for saving .tif data. 
unchecked (Default)

**8-bit conversion** Type conversion to 8-bit.  Possibility to customizable set 
set upper and lower bounds of the conversion
unchecked (Default) , ‘255=’  65535 (Default)  ‘0=’ 0 (Default)

**16-bit conversion**  Type conversion to 16-bit.    if data is something else ..
unchecked (Default)
Possibility to customizable set 
set upper and lower bounds of the conversion
unchecked (Default) , ‘Min=’  0 (Default)  ‘Max=’ 255 (Default)
```diff
 - consistency!?  Max Min both places?
```

Save as stacks Choose this option to execute saving data as stacks with given preferences.   

Save as planes Choose this option to execute saving data as separate planes/slices with given preferences.   
Stop saving Cancels an ongoing saving task.


### Chromatic Shifts

**Chromatic shifts**  in pixels for each channel [x channel1, y channel1, z channel1; x channel2, y channel2, z channel2;..  ]. Adjusts the value (double) of a pixelshift to be applied to each separate channel in the stream. Will not be executed until apply shifts in pressed. 0,0,0; 0,0,0 (Default)
**Apply shifts**   executes the shift and updates the stream.

### Oblique (viewing)  ( Available only in BigDataProcessor 2. )

**Use Oblique viewing** Choose this option to execute viewing of data  with a oblique angle and other given preferences.  Have to be active (green) in order to be able to browse in the oblique view.  

**Update** Choose this option to execute viewing of data with given preferences.  The button Use Oblique viewing have to be active (green) in order to be able to update. 

**Y shearing** Along Y shearing? unchecked means false i.e. in X-direction. 
Unchecked, i.e. false (Default)

**Magnification**  Set the magnification of the detection objective. E.g. for a 40X objective, type 40.
40 (Default)

**Pixelsize** Pixelsize on Camera. Set the size of the pixels (a.k.a pixelpitch) on the camera in micrometers.  
6.5 (Default)    

**CameraBackwardStackAcquisition?** Check this box if stack acquired in opposite direction?
Unchecked, i.e. false (Default)
**Stack stepsize:**  Set the step between planes in an image stack, given in micrometers.
2 (Default)
**Objective Angle** Set the angle α in degrees between the acquisition direction and the detection objective. 45 (Default)

### Miscellaneous
**I/O threads**  Set the magnification of the detection objective. E.g. for a 40X objective, type 40.
5 (Default)
**Chunks [ny]** Sets the chunk size of .h5 block. Valid only when saving .h5 data.  10 (Default) 

**Verbose logging** Set the magnification of the detection objective. E.g. for a 40X objective, type 40.
Unchecked (Default)
Report an issue By clicking the user is linked to the issue section of the github repository of the plugin and can favorable report an issue there.  



Report issue By clicking this link the user is linked to the issue section of the github repository of the plugin and can favorable report an issue there.

### References

[1] Pietzsch, T., Saalfeld, S., Preibisch, S. & Tomancak, P. BigDataViewer: visualization and processing for large image data sets. Nature Methods 12, 481–483 (2015).
