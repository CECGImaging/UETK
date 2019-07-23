# Unified ECGI Toolkit (UETK)

The Unified ECGI Toolkit (UETK) is a framework to combine many of the varied Electrocaridgraphic Imaging (ECGI) techniques and methods.  It is designed to enable users to perform as many as possible of the preprocessing steps needed to perform ECGI, together with the ECGI techniques, such as: processing of body surface recordings, signal averaging, spatial interpolation, and others.  The UETK also allows users to exchange or run in parallel different methods for each of these steps and many types of ECGI techniques.

The UETK is a series of networks and pipelines that runs within SCIRun (https://github.com/SCIInstitute/SCIRun), a problem solving Environment.  The flexibility of SCIRun allows for many different methods to be quickly implemented and combined together.  The UETK also builds upon the development of other tools built with ECGI in mind: The Foward/Inverse toolkit (https://github.com/SCIInstitute/FwdInvToolkit) and PFEIFER (http://sci.utah.edu/cibc-software/pfeifer.html).  The UETK also leverages the EDGAR database (https://edgar.sci.utah.edu), a set of ECGI data compiled by the Consortium for ECG Imaging (CEI) (https://ecg-imaging.org), to provide example data, a consistent format, and direct comparison to ground truth data.

## Prerequisites

To run the Unified ECGI Toolkit (UETK) users will need to download and install SCIRun (https://github.com/SCIInstitute/SCIRun/releases), and to download PFEIFER (v 1.2.1 or later) (https://github.com/SCIInstitute/PFEIFER/releases) and add the PFEIFER source folder and its subfolders to the MATLAB path variable.  A valid MATLAB license and installation is required to run PFEIFER. Users should also download the Foward/Inverse Toolkit (https://github.com/SCIInstitute/FwdInvToolkit/releases) for a broader range of techniques for ECGI.  Users should also clone or download this UETK repo.

Running the UTEK will require running some python packages in SCIRun. These packages include:

 - matlab engine
 - numpy
 - scipy
 
 Instructions on getting these packages installed in SCIRun can be found here (http://sciinstitute.github.io/scirun.pages/python.html)
 
 

