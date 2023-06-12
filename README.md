# Enrichment Analysis
FIJI macros to perform Enrichment Analysis.

# Installation
Copy the code (from browser or download as .txt first) and paste it in your macro window as: 
     Plugins > New > Macro
you may need to specify the language in Language tab of the Macro window, select IJ1 Macro, and save it as a .ijm.

# How to use 
I have made this macro to assess spatial correlation between 2 fluorescent signals with different structures.
Based on our publication, we used regions of interest (ROIs) from widefield fluorescent z-stack image files (.tif) with 2 channels.

The 1st channel is for an elongated structure like a filament (e.g. Actin) and the second channel is for a round structure like a cluster (e.g. Immune-Complex).
Run `Sharp Ruffle` macro on the ROI-image stack to identify the 3 sharpest frames. Duplicate these 3 frames (2 channels) and run the `Enrichment Analysis` macro to identify the spatial coerrelation of the 2 signals.



## `Sharp Ruffle`

The majority of Actin signal segmentation tools are designed to detect linear actin structures, while SharpRuffle is made to detect curly actin structures (e.g. Ruffles). 
The first step to detect the sharpest z-stack frames of the filament was to detect structural edges in the image. Edge detection is a method that reduces noise in an image without removing significant details (edges, lines) that are important for the image. This filtering method is based on anisotropic diffusion, also known as Perona-Malik diffusion. Anisotropic diffusion is the process of producing blurred images of the original image and a convolution matrix, termed a kernel. In contrast with isotropic diffusion (e.g. Gaussian blurring) each resulting image is not only a blurred version of the previous one but a combination of the original image and a filter that examines the local content for any orientation anisotropy (Perona and Malik, 1990). 

*to be added*
Figure 1. Pipeline analysis of the sharpest F-actin z-slices. Steps of image analysis to find the sharpest frames of the ROI z-stack, immune complexes (magenta) and F-actin (Phalloidin, green). Scale bar: 0.5 µm.

In detail, initially a series of successively increased blurred versions of the image slices are generated using a 2D isotropic Gaussian filter (Gaussian Blur) based on a smoothing parameter σ (Gaussian kernel, here, σ= [3]) (Weickert, 1998). This filtering allows the calculation of the intensity gradient of each image and its directionality and is known in image analysis as the Sobel operator (or “Find Edges” in FIJI). A high gradient between two pixels suggests an edge. On the contrary, a low gradient suggests the absence of an edge and Gaussian Blurring is applied there. The outcome of anisotropic diffusion filtering allows the boundaries of the structure to remain sharp.

The Canny-Deriche operator (Canny, 1986) is applied to the resulting edge image (or map) to thin the edge structures by filtering the local intensity maximums. This operator applies a hysteresis thresholding to the detected local maximums to remove noise. The hysteresis thresholding allows for double thresholding, where the pixel intensity values below the bottom threshold are removed, the ones above the top threshold are preserved, and the intensity values falling between the two thresholds are preserved if only spatially associated with the above threshold. This process is performed using the image_edges.jar plugin (written originally by Thomas Boudier and repackaged by Joris Meys) and generates an edge detection map of every frame in the z-stack of the region of interest. 

The three frames of the edge detection map with the highest integrated values are the sharpest detected frames. Depending on the cellular location you are interested in you may need to choose the Sharpest frames in the respective z-stack slices, for example the sharpest frames for the actin cytoskeleton signal may correspond to thw ventral side (podosomes) or to the dorsal (F-actin ruffles) of the cell. 

## `Enrichment Analysis`

We are interested in the fraction of immune complexes (round objects) that associate with F-actin ruffles (elongated object), rather than fluorescence values themselves. This relies on the fact that F-actin and immune complex fluorescence can either partially or fully overlap, making a signal-based analysis unsuitable. Further, a pure object-based analysis would also not be a right fit, as it would require similarity between the morphology of the two structures. To resolve this issue, we developed a hybrid approach that merges intensity- and object-based association approaches.

Inspired by the work of Sun et al., 2020, our semi-automated method calculates the intensity ratio of membrane-bound immune complexes that colocalise with actin filaments, and the total immune complex intensity regardless of their F-actin association. Based on this normalised quantification we can measure the enrichment of immune complexes on F-actin ruffles in what will from now on be referred to as enrichment ratio. Immune complexes and F-actin ruffles that fully or partially overlap are considered associated. The enrichment analysis uses the MIP slice that has been generated by the three sharpest sequential frames of an ROI z-stack building on our segmentation pipeline. Once the masks of the actin filament channel and the immune complex channel have been generated, a few post-processing steps must take place to calculate the enrichment ratio. The immune complexes mask is used for immune complex identification using the “Analyse particles” command in FIJI by “Analyse particles” each immune complex is assigned a number, which is automatically recorded in the “ROI manager” table. Using these asigned numbers, we can identify each immune complex on the immune-complex raw intensity image and find their raw integrated density.

*to be added*

Figure 2. Pipeline for F-actin ruffle enrichment analysis. Fluorescence staining with immune complexes (magenta) and Phalloidin (F-actin). Scale bar: 0.5 µm.

To further identify the relative positions of immune complexes with respect to F-actin, the actin mask is inverted and thus represents the membrane space between the actin structures. By projecting of the number-assigned immune complex ROIs (from ROI manager) onto the F-actin mask and inverted F-actin mask (or membrane) we can identify the immune complexes that are “on” and “off” F-actin. Retrieving the raw integrated densities of immune complexes allows us to calculate the enrichment ratio:
                                                                                     R=∑I(c,on)/∑I(c,on+off)                          

where R is the enrichment ratio (R∈[0,1]), I(c,on) is the raw integrated density of immune complexes that overlap with actin filaments, and I(c,on+off) is the raw integrated density of all immune complexes in the ROI.

### Things to keep in mind

* Intensity threshold values, smoothing parameter σ values, filters like Watershed, etc, are things you have to match to your own samples.
* In case of 0 signal from the elongated structure channel the pseudocolor will appear as green, in this example as if the immune-complexes are fully on actin, this is a color representation and nothing more, the intensity values of the raw ICs (round object) image is what matters.

### References
- Canny, J., 1986. A Computational Approach to Edge Detection. IEEE Trans. Pattern Anal. Mach. Intell. PAMI-8, 679–698. https://doi.org/10.1109/TPAMI.1986.4767851
- Iliopoulou, M., 2022. Subcapsular sinus macrophage sensing of extracellular matrix rigidity alters membrane topography and immune complex mobility. bioRxiv 518873; https://doi.org/10.1101/2022.12.02.518873
- Perona, P., Malik, J., 1990. Scale-space and edge detection using anisotropic diffusion. IEEE Trans. Pattern Anal. Machine Intell. 12, 629–639. https://doi.org/10.1109/34.56205
- Sun, X., Phua, D.Y.Z., Axiotakis, L., Smith, M.A., Blankman, E., Gong, R., Cail, R.C., Espinosa de Los Reyes, S., Beckerle, M.C., Waterman, C.M., Alushin, G.M., 2020. Mechanosensing through direct binding of tensed F-actin by LIM domains (preprint). Biophysics. https://doi.org/10.1101/2020.03.06.979245
- Weickert, J., 1998. Anisotropic Diﬀusion in Image Processing 184.

- The description is part of my PhD thesis "Biophysical characteristics of antigen presentation by subcapsular sinus macrophages to B cells" and I will include a link once uploaded.
