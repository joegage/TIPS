# TIPS

Welcome to the *Tassel Image-based Phenotyping System*, aka *Tassel Image Processing Software*, aka *TIPS*!  See below for relevant information and help.

## Downloading and testing TIPS

*TIPS is now available on CyVerse!* The original version of TIPS is available on CyVerse, as well as an updated version that automatically segments the tassel from the background in a single image (no 'blank' background image required). Benefits of running your tassel analysis on CyVerse include:

* High throughput, distributed computing.
* Repeatability - analysis are easily replicable and repeatable.
* No wrangling with Matlab versions - it just works.
* Free storage of images and results on the CyVerse Data Store

To run your analyses, first make an account on the CyVerse Discovery Environment: https://de.cyverse.org/de/

Then, contact me (Joseph dot L dot Gage at gmail dot com) for access to the TIPS algorithm.

## Running TIPS on your own images
If you really want to run TIPS on your own computer, use the code in this repository.  You may run into compatibility issues with different versions of MATLAB (TIPS was developed under 2014b).

### Inputs
TIPS was written to process tassels photographed in front of a white background.  The algorithm expects to be passed two images: one with the tassel in it (foreground), and one without the tassel (background), as well as an output path and prefix.  An optional final argument containing a vector of parameter values can be provided as well.  Formally, the arguments taken by TIPS are (in order):

* **foreground**: path to the image with the tassel in it. *e.g., './myTasselImages/tassel1_foreground.jpg'*
* **background**: path to the image without the tassel in it. *e.g., './myTasselImages/tassel1_background.jpg'*
* **out**: path and prefix for output.  Suffixes will be appended to this input by TIPS. *e.g., './output/tassel1'* will produce *'./output/tassel1_processed.png' and './output/tassel1_out.txt'*
* **params** (optional): vector of 8 parameters, explained in detail below.

TIPS should work with images of any format accepted by MATLAB's `imread` function, but .jpg is recommended. The background image helps isolate the tassel faithfully.  Depending on the amount of noise in your imaging setup, you may be able to get away with using the same background image with all foreground images.

The optional parameters that can be used to fine-tune TIPS are outlined below, in the order in which they should appear in the vector passed to TIPS.m.  The default values were chosen because they worked well with the set of images used when developing TIPS - your mileage may vary.

1. **gThresh**: [*default 0.08*] TIPS uses the MATLAB's `graythresh` function to binarize the image.  If the threshold that `graythresh` uses to binarize is < **gThresh**, the image will throw an error.  This is in place to help avoid analyzing images with no tassel in the foreground, bad contrast, or reflection.  The default was determined empirically using a set of faulty and acceptable images.
* **padSize**: [*default 200*] The amount of padding added around the edges of the original and binarized tassel image to prevent certain operations from falling off the edge of the images.
* **smoothSigma**: [*default 55*] TIPS uses a Gaussian kernel to smooth the binary tassel image.  This is the standard deviation of that kernel.
* **smoothKernelDim**: [*default 15*] This is used to determine the size of the Gaussian kernel.  The kernel is square with dimension **smoothKernelDim** * 2 + 1.
* **skelTol**: [*default 10e-8*] Tolerance parameter passed to MATLAB's `csaps` function.
* **skelMinBranch**: [*default 75*] After the tassel has been skeletonized, any branches with length < **skelMinBranch** will be ignored during spline fitting.
* **spikeWidth**: [*default 301*] When searching for the lowest branch, TIPS integrates the binary image along the longest spline (corresponding to the tassel spike) within a window of **spikeWidth** centered on the spline.
* **spikeTol**: [*default 0.2*] The position where the derivative of the integral along the spike spline (see above) becomes > **spikeTol** is identified as the lowest branch point.

### Outputs
TIPS returns two files, named using the path and prefix specified in the `'out'` input argument.  For a path and prefix of *./myTasselImages/tassel1* TIPS will return:

* Processed Image: *./myTasselImages/tassel1_processed.png* is a binary image with some traits illustrated in color:
  + Tassel base denoted by a magenta square
  + Lowest branch point denoted by a blue circle
  + Skeleton endpoints denoted by red '+' signs
  + Splines from base to skeleton endpoints denoted by green lines
  + Spline from lowest branch point to spike tip denoted by blue line
  + Straight line from first branch point to spike tip in red line
  + Convex hull in cyan

* Phenotypic measurements: *./myTasselImages/tassel1_out.txt* is a one line, tab-separated text file with measured phenotypes and other information **in the following order**:
  + Foreground image name, as passed to TIPS, in single quotes.
  + Area: sum of pixels in the binarized tassel image.
  + Branch Number: maximum number of times the binary tassel is intercepted by circles of varying size with origin at the lowest branch point.
  + Tassel Length: length of the line integral from the lowest branch point to the spike tip.
  + Tortuosity: euclidean distance from first branch point to spike tip divided by the tassel length.
  + Compactness: area inside a convex hull around the tassel, divided by the tassel area.
  + Fractal Dimension: a measure of complexity, calculated by the box-counting method.
  + Skeleton Length: sum of the pixels that comprise the tassel skeleton.
  + Perimeter Length: sum of the pixels that comprise the tassel perimeter.
  + Error message: if there was no error for the image, this will just be an empty set of single quotes.
  
