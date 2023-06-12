# Enrichment Analysis
FIJI macros to perform Enrichment Analysis.

# Installation
Copy the code (from browser or download as .txt first) and paste it in your macro window as: 
     Plugins > New > Macro
you may need to specify the language in Language tab of the Macro window, select IJ1 Macro, and save it as a .ijm.

# How to use 
I have used this macro to assess spatial correlation between 2 fluorescent signals with different structures. Based on our publication we use regions of interest (ROIs) from widefield fluorescent z-stack image files (.tif) with 2 channels.

The 1st channel is an elongated structure like a filament (e.g. Actin) and the second channel is a round structure like a cluster (e.g. Immune-Complex).

## `Sharp Ruffle`
The majority of Actin signal segmentation tools are designed to detect linear actin structures, while SharpRuffle is made to detect curly actin structures (e.g. Ruffles). 
The first step to detect the sharpest z-stack frames of the filament was to detect structural edges in the image. Edge detection is a method that reduces noise in an image without removing significant details (edges, lines) that are important for the image. This filtering method is based on anisotropic diffusion, also known as Perona-Malik diffusion. Anisotropic diffusion is the process of producing blurred images of the original image and a convolution matrix, termed a kernel. In contrast with isotropic diffusion (e.g. Gaussian blurring) each resulting image is not only a blurred version of the previous one but a combination of the original image and a filter that examines the local content for any orientation anisotropy (Perona and Malik, 1990). 

![Picture1](https://g![Picture1](https://github.com/maroiliop/Enrichment-Analysis/assets/136265557/cce3e317-1fb1-44c2-a7a7-0e6fb64acddd)
ithub.com/maroiliop/Enrichment-Analysis/assets/136265557/f3ac05b2-407b-43b7-9035-feef6bdb0488)
Figure 1. Pipeline analysis of the sharpest F-actin z-slices. Steps of image analysis to find the sharpest frames of the ROI z-stack, immune complexes (magenta) and F-actin (Phalloidin, green). Scale bar: 0.5 µm.

In detail, initially a series of successively increased blurred versions of the image slices are generated using a 2D isotropic Gaussian filter (Gaussian Blur) based on a smoothing parameter σ (Gaussian kernel, here, σ= [3]) (Weickert, 1998). This filtering allows the calculation of the intensity gradient of each image and its directionality and is known in image analysis as the Sobel operator (or “Find Edges” in FIJI). A high gradient between two pixels suggests an edge. On the contrary, a low gradient suggests the absence of an edge and Gaussian Blurring is applied there. The outcome of anisotropic diffusion filtering allows the boundaries of the structure to remain sharp.

The Canny-Deriche operator (Canny, 1986) is applied to the resulting edge image (or map) to thin the edge structures by filtering the local intensity maximums. This operator applies a hysteresis thresholding to the detected local maximums to remove noise. The hysteresis thresholding allows for double thresholding, where the pixel intensity values below the bottom threshold are removed, the ones above the top threshold are preserved, and the intensity values falling between the two thresholds are preserved if only spatially associated with the above threshold. This process is performed using the image_edges.jar plugin (written originally by Thomas Boudier and repackaged by Joris Meys) and generates an edge detection map of every frame in the z-stack of the region of interest. 

The three frames of the edge detection map with the highest integrated values are the sharpest detected frames. Depending on the cellular location you are interested in you may need to choose the Sharpest frames in the respective z-stack slices, for example the sharpest frames for the actin cytoskeleton signal may correspond to thw ventral side (podosomes) or to the dorsal (F-actin ruffles) of the cell. 

##`Enrichment Analysis`

References
