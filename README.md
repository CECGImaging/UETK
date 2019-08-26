# Unified ECGI Toolkit (UETK)

The Unified ECGI Toolkit (UETK) is a framework to combine many of the varied Electrocaridgraphic Imaging (ECGI) techniques and methods.  It is designed to enable users to perform as many as possible of the preprocessing steps needed to perform ECGI, together with the ECGI techniques, such as: processing of body surface recordings, signal averaging, spatial interpolation, and others.  The UETK also allows users to exchange or run in parallel different methods for each of these steps and many types of ECGI techniques.

The UETK is a series of networks and pipelines that runs within [SCIRun](https://github.com/SCIInstitute/SCIRun), a problem solving Environment.  The flexibility of SCIRun allows for many different methods to be quickly implemented and combined together.  The UETK also builds upon the development of other tools built with ECGI in mind: [The Foward/Inverse toolkit](https://github.com/SCIInstitute/FwdInvToolkit) and [PFEIFER](http://sci.utah.edu/cibc-software/pfeifer.html).  The UETK also leverages the [EDGAR database](https://edgar.sci.utah.edu), a set of ECGI data compiled by the [Consortium for ECG Imaging (CEI)](https://ecg-imaging.org), to provide example data, a consistent format, and direct comparison to ground truth data.

## Prerequisites

To run the Unified ECGI Toolkit (UETK) users will need to download and install [SCIRun](https://github.com/SCIInstitute/SCIRun/releases), and to download [PFEIFER (v 1.2.1 or later)](https://github.com/SCIInstitute/PFEIFER/releases) and add the PFEIFER source folder and its subfolders to the MATLAB path variable.  A valid MATLAB license and installation is required to run PFEIFER. Users should also download the [Foward/Inverse Toolkit](https://github.com/SCIInstitute/FwdInvToolkit/releases) for a broader range of techniques for ECGI.  Users should also clone or download this UETK repo.

Running the UTEK will require running some python packages in SCIRun. These packages include:

 - matlab engine
 - numpy
 - scipy
 
Instructions on getting these packages installed in SCIRun can be found here (http://sciinstitute.github.io/scirun.pages/python.html).  For help, you may contact the SCIRun users mailing list: [scirun-users@sci.utah.edu](scirun-users@sci.utah.edu).
 
Some of the examples require datasets from the [EDGAR database](https://edgar.sci.utah.edu).  Not all the datasets are needed, but it will be easier to run the examples if the directory structure of the database and the experiments is maintained.  Each dataset should be within one of four directories:

 - Human_Cardiac_Mapping
 - Human_Pacing_Site
 - InSitu_Animal
 - Torso_Tank
 
For example, the `example_nijmegen.srn5` and other example networks use the "Nijmegen-2004-12-09" dataset, which should be located withing the "Human_Pacing_Site" directory.  It is easiest to download to whole dataset to maintain the correct internal directory structure.  The other dataset that is used by multiple example networks, such as `example_cage-tank.srn5` is the "Utah-2002-05-15" cage-tank dataset, which should be placed in the "Torso_Tank" directory.

_*To run the examples with the EDGAR data, set the SCIRun Data path (found in the preferences window) to the directory containing the EDGAR datasets.*_


## Running UETK

The UETK is designed to be a modular workflow that can allow researchers take their data from recording to ECGI solutions and validation in a single instance. Running the UETK is similar to running the [Forward/Inverse Toolkit](https://github.com/SCIInstitute/FwdInvToolkit): the two toolkits use the same set of modules to perform ECG imaging and have a suite of example networks to help users begin using each of the techniques.  However, there are some extra pipeline steps in the UETK to facilitate integration of preprocessing and other often ignored aspects of ECGI.  The main steps of the UETK are: PFEIFER processing, signal processing, geometry processing, forward modeling, ECGI, and validation.

### PFEIFER processing

The main component of the UETK that differs from the Forward/Inverse Toolkit is the integration with PFIEFER.  PFEIFER is a Matlab package that is used to process raw cardiac signals and can be run in SCIRun throught the InterfaceWithPython module using the matlab engine in python.  After installing the relevant packages as described [above](#prerequisites), add an InterfaceWithPython module.  In the `Top-level Script` tab, use this code to launch Matlab:
```
import matlab.engine
eng =  matlab.engine.start_matlab() if (not 'eng' in vars()) else eng
```
Launching the Matlab engine in the `Top-level Script` tab will enable the engine to remain open until SCIRun is closed, and will save time if the network is run multiple times.  In the main `Code` tab, launch PFEIFER with:
```
eng.PFEIFER('runScriptFile', nargout=0)
mf = eng.eval("findobj(allchild(0.0),'tag','PROCESSINGSCRIPTMENU')")
eng.workspace['mainFigure']=mf
eng.eval('uiwait(mainFigure)',nargout=0)
```
Which will launch the PFEIFER UI and wait to execute the rest of the SCIRun network until the PFEIFER window closes.  The user can then process the BSPM signals as needed, as explained breifly in the [examples](#), and more extensively in the [PFEIFER documentation](http://sci.utah.edu/devbuilds/pfeifer_docs/PfeiferDocumentation.pdf).  Once the processing is done once, the parameters and settings can be saved and loaded.  To run PFEIFER with saved settings, launch PFEIFER with the parameter files as arguments:
```
eng.PFEIFER('runScriptFile',SCRIPTDATA_filename,PROCESSINGDATA_filename, nargout=0)
```

Due to the size of data that can come from multichannel recordings, PFEIFER will save the processed data to disk. In order to use the processed signals in the rest of the pipeline, they will have to be loaded into SCIRun.  This can be accomplished many ways, but they all involve getting the list of output file names and other parameters relevant to the processing.  The script settings, including the output filenames, are save in the SCRIPTDATA and PROCESSINGDATA files, which are also saved as global variables in PFEIFER.  To pass them out of PFEIFER, declare these as global variables before running pfiefer:
```
eng.eval("global SCRIPTDATA PROCESSINGDATA",nargout = 0)
```
Then access the variables in python by assigning them to a pytthon variable after PFEIFER runs:
```
SCRIPTDATA = eng.workspace['SCRIPTDATA']
```
The output filenames can then be reconstructed from the output directory and the input filenames.  See the [example networks](#uetk-examples) for various ways to accomplish this.

_*Note: it may be necessary to convert signals into a format pfeifer can process.  For most formats, this is possible in PFEIFER using the `File Converter` tool in the Workbench window.*_


### Signal processing

In addition to PFEIFER, the UETK can perform additional signal processing.  While there are some limited features that can be done through native SCIRun modules, a more extensive set of processing can be performed with the InterfaceWithPython module.  For example, to perform a lowpass bbutterworth filter, use the following python code in the module:
```
from scipy.signal import butter, filtfilt
fs = 1000
cutoff = 100
order = 6

nyq = 0.5 * fs
normal_cutoff = cutoff / nyq
b, a = butter(order, normal_cutoff,  btype='low', analog=False)


BSPM = matrixInput1
BSPM_new=[]
for k in range(len(BSPM)):
  signal = np.array(BSPM[k])
  new_signal = filtfilt(b, a, signal)
  BSPM_new.append(new_signal.tolist())

matrixOutput1 = BSPM_new
```

Matlab processing tools can also be used in the InterfaceWithPython using the [Matlab code block](http://sciinstitute.github.io/scirun.pages/python.html), or through the [Matlab python interface](http://www.mathworks.com/help/matlab/matlab_external/install-the-matlab-engine-for-python.html)

### Geometry processing

SCIRun has many modules designed to facilitate geometric processing, such as remeshing, clipping, interpolation, projection and registration.  These can be combined in many ways to affect the ECGI calculation.  One particularly useful example is interpolating over bad recordings.  This is accomplished by clipping out the electrode location identified as bad (which can be done in PFEIFER), then spatially interpolating over the empty region.  See the `bad_leads_interpolate.srn5` example network for details.

### Forward Modeling

The UETK allows for users to use different formulations of the forward model that ECGI attempts to inverse.  These implementations of the forward problem include BEM, FEM, and MFS and are describe in the [Forward/Inverse Toolkit](https://github.com/SCIInstitute/FwdInvToolkit).  For some cases (BEM) it is possible to compute the forward matrix quickly and it can be done at run-time.  However, in some cases it may be necessary to compute the forward matrix seperately and load it in at run-time.  Many datasets in the EDGAR repository contain precomputed forward matrices.

### ECG Imaging

The UETK can use any of the ECGI methods implemented in the [Forward/Inverse Toolkit](https://github.com/SCIInstitute/FwdInvToolkit).  Presently, the main method used in the examples is Tikhonov regularization, yet users can substitute or run in parallel any other method.  Some of the implementations will require the use of Python or Matlab.

### Validation

For datasets that include ground truth data, it is also possible to provide a validation step into the UETK pipeline.  The `example_cage_tank.srn5` example network includes ground truth data with the ECGI solution for visual comparison.  SCIRun provides 3D visualization tools and a 2D plotter to visualize the temporal recordings of the singals. Quantitative comparison is also possible with the CalculatedFieldData and EvaluateLinearAlgebraBinary modules, among others.  Quantitative comparison is illustrated with the `example_nijmegen.srn5` example network, though there is no ground truth for this dataset.


## UETK Examples

  - `Tank_simple.srn5` - Simplest example in the UETK, including the PFEIFER processing and ECGI with Tikhonov regularization.  This example is designed to work with the "Utah-2002-05-15" cage-tank dataset from the EDGAR repository.  This example can run with or without existing SCRIPTDATA.mat and PROCESSINGDATA.mat files.  To run with existing files, either save the files in the "Utah-2002-05-15" directory or modify the CreateString modules to include the proper relative path.
  - `preprocessing_steps.srn5` - Highlights the use of Python to perfom filtering in addition to PFEIFER.  This example shows how to implement a low pass butterworth filter on all the BSPM channels.  This network is designed to run with the "Utah-2002-05-15" cage-tank dataset from the EDGAR repository.  It requires existing SCRIPTDATA.mat and PROCESSINGDATA.mat which can be the same as those generated from `Tank_simple.srn5`.  Locate the files with the ImportFieldsFromMatlab module file dialogue.
  - `bad_leads_interpolate.srn5` - Highlights one way to perform interpolation over bad leads, yet also includes low pass filtering.  This network specifically removes the bad leads identified with PFIEFER, then uses linear interpolation to replace the signals.  This network is designed to run with the "Utah-2002-05-15" cage-tank dataset from the EDGAR repository.  It requires existing SCRIPTDATA.mat and PROCESSINGDATA.mat which can be the same as those generated from `Tank_simple.srn5`.  Locate the files with the ImportFieldsFromMatlab module file dialogue.
  - `singal_averaging.srn5` -  Highlights an implementation of signal averaging performed using the PFEIFER beat autofeducializer and simple averaging in Python.  This network also shows how to build the forward matrix using BEM.  This example is designed to run with the "Nijmegen-2004-12-09" dataset and can run with or without existing SCRIPTDATA.mat and PROCESSINGDATA.mat files.  To run with existing files, either save the files in the "Nijmegen-2004-12-09/Interventions/" directory or modify the PrintStringIntoString modules to include the proper relative path.  The "Nijmegen-2004-12-09" data may need to be converted to the PFEIFER format.
  - `example_cage_tank.srn5` - This example illustrates how to use each of the major steps of the previous examples together.  It includes PFEIFER processing (with existing settings files), lowpass filtering, signal averaging (no multiple beats in this dataset), bad lead interpolation, forward matrix calculation, ECGI with Tikhonov regularization, and qualitative validation to the ground truth dataset.  This network is designed to run with the "Utah-2002-05-15" cage-tank dataset from the EDGAR repository.  It requires existing SCRIPTDATA.mat and PROCESSINGDATA.mat which can be the same as those generated from `Tank_simple.srn5`.  Locate the files with the ImportFieldsFromMatlab module file dialogue.
  - `example_nijmegen.srn5` - This example illustrates how to use each of the major steps of the previous examples together.  It includes PFEIFER processing (with existing settings files), lowpass filtering, signal averaging, bad lead interpolation, forward matrix calculation, ECGI with Tikhonov regularization, and qualitative and quantitative comparison between solutions with PFEIFER processing and signal averaging.  This network is designed to run with the "Nijmegen-2004-12-09" cage-tank dataset from the EDGAR repository.  It requires existing SCRIPTDATA.mat and PROCESSINGDATA.mat which can be the same as those generated from `singal_averaging.srn5`.  Locate the files with the ImportFieldsFromMatlab module file dialogue.

### Running examples without existing PFEIFER files

#### Utah-2002-05-15"

Load the `Tank_simple.srn5` example network and make sure that the SCIRun Data path is set to the EDGAR database location and run the network.  The PFEIFER UI should appear with `DATA ORGANIZATION` window showing.  Press the `Save` buttton for the Script and Processing data files to choose a location to save the settings files.  It may be easier if they are located with the EDGAR data. Choose the input directory, which should be the `Torso` folder in one of the Interventions provided in the dataset.  Choose or create a folder to save the processed beats (MATLAB output directory).  Next, create a new `RUNGROUP` by clicking on the dropdown menu and selecting `NEW RUNGROUP`.  Name the Run group after the intervention and select the files that are associated with it (all the files in the Torso folder).  Create a new `GROUP` by selecting the `NEW GROUP` option from the second dropdown menu.  Uncheck the `Use Mappingfile` option, name the group `torso`, enter lead numbers `1:192`, and the filename extension `-ts`.  You may also choose some Bad Leads if you are running the bad lead interpolation example.  Now open the `WORKBENCH` window.  Alter the settings on the left side as desired.  No change is required, nor are any of the processing steps.  After running the script once, unchecking the `User Interaction` option will run the previously chosen values. Choose one or more of the files to process from the list of files and click `RUN SCRIPT`.  Manually choose the beat window, baseline regions, and fiducials (right click to add a new one and use the menu on the right to change types).  Once completed, close one of the PFEIFER windows to resume ECGI calculations.

#### Nijmegen-2004-12-09

Load the `singal_averaging.srn5` example network and make sure that the SCIRun Data path is set to the EDGAR database location and run the network.  Running PFEIFER will be very similar to the "Utah-2002-05-15" dataset, with some significant changes.  Before the "Nijmegen-2004-12-09" dataset can be run, it needs to be converted to the PFEIFER format.  In the `WORKBENCH` window, select `FILE CONVERTER` and choose one of the interventions as the input directory. Create a new directory for the converted files and set that as the output.  Select all the files and click `CONVERT SELECTED FILES` and close the `FILE CONVERTER` window.  Now use these converted files as the input to PFEIFER, similar to the "Utah-2002-05-15" dataset.  The main differences are: the number of leads (`1:65`) and autofeducializing should be on to perform signal averaging.


