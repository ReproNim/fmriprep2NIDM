# fmriprep2NIDM example

### Step 1: Run fmriprep
Our friends at **INDI** have run the *fmriprep* tool on much of their publically hosted data, includeing the ABIDE and
ADHD-200 datasets. These *fmriprep* results are available on the **INDI** AWS S3 bucket. The 'path' for such an
example fmriprep run for sub-0050118 of the ABIDE dataset is:

  s3://fcp-indi/data/Projects/ABIDE/Outputs/fmriprep/fmriprep/sub-0050118

There are many example results there.

### Step 2: Represent the results of the tool in a CSV file
The fmriprep processing generates descriptors of each time point of a fMRI run. For the summary we want to include
as metadata about the run, we want to generate a summary (i.e. average, standard deviation, etc.) of a select set
of features from the complete set generated by *fmriprep*.

We use a tool called *summarize_confounds* (thanks Christian!). It currently lives at:
[SumarizeFmriprep](https://github.com/ReproNim/SumarizeFmriprep). More details about how the summarizer works is provided in
its documentation.

This tool was used to generate the *ABIDE_fmriprep_results.csv* file shared above, by running it on all the
ABIDE fmriprep results available at s3://fcp-indi/data/Projects/ABIDE/Outputs/fmriprep.

### Step 3: Generate a dictionary for the results as a CSV file
Given that the *summarize_confounds* process generates a specific list of features, we need a dictionary for
files generated in this fashion. The dictionary includes information about the contents of each column including
name, description, value type, range, units, semantic annotation, etc. Since this is 'fixed' by the output of
the summarizer, other users do note need to generate this file themselves, but rather just need to fetch this file
from some source, either here, or more officially, probably shared with the summarizer tool.

### Step 4: Generate a tool description file
Tool description files have the following required fields that describe the invocation of a particular software tool:

* title
* description
* version
* url
* cmdline
* platform
* ID

See the example file provided above for a complete example description. In general, a user does not need to generate 
this file de novo themselves; they should be abble to access a template version of this file, and update (or check) 
the entries for the 'version', 'cmdline' and 'platform' as meets the invocation used. The remaining entries should be 
a constant for a specific tool. This template can be found here, or perhaps more officially, from the particular tool's 
page or a registry of supported tools that ReproNim may create.

### Step 5: Use *PyNIDM* and the  3 CSV files to generate the NIDM representation  
*PyNIDM* is available [here](https://github.com/incf-nidash/PyNIDM). We can create the NIDM representation of these 
results either as a 'stand alone' NIDM file, or attach it to the NIDM representation of the source imaging data that 
was used.

#### Stand-alone NIDM results
After installing PyNIDM, one can use the csv2nidm tool to perform the conversion:

```console
>  csv2nidm -csv /Your_Path_To/ABIDE_fmriprep_results_v2.csv \
   -csv_map /Your_Path_To/fmriprep_data_dictionary_v3.csv -no_concepts \
   -derivative /Your_Path_To/fmriprep_software_metadata.csv
```

In this command, the '-csv' flag indicates the full path to CSV file to convert that contains your ***processing results*** 
(in this example, the summarized results of the fmriprep analysis. The '-csv_map' is followed by the full path to 
user-supplied CSV-version of ***data dictionary*** containing the following required columns: source_variable, label,
description, valueType, measureOf, isAbout(For multiple isAbout entries, use a ';' to separate them in a single column
within the csv file dataframe), unitCode, minValue, maxValue. The '-derivative', if set, indicates CSV file provided 
above is derivative data which includes columns 'ses','task','run' which will be used to identify the subject scan 
session, run, and verify against the task if an existing nidm file is provided and was made from bids (bidsmri2nidm).
Otherwise these additional columns (ses, task,run) will be ignored. After the '-derivative' flag one must provide 
the ***software metadata*** CSV file. Additionally, the '-no_concepts' flag specifies that no concept associations will 
be asked of the user.

#### Add derivitives to existing NIDM file
If you happen to have a NIDM file for the MRI images upon which your fmriprep was run, you can add these derivitives to that
NIDM file. The command is similar to the above stand-alone version except that we provide the '-nidm' flag followed by the 
NIDM (.ttl) file you want to add to.  We provide an example ABIDE site NIDM for the OHSU site [here](TTLs/OHSU_nidm.ttl).

```console
>  csv2nidm -nidm /Your_Path_To/OHSU_nidm.ttl -csv /Your_Path_To/ABIDE_fmriprep_results_v2.csv \
   -csv_map /Your_Path_To/fmriprep_data_dictionary_v3.csv -no_concepts \
   -derivative /Your_Path_To/fmriprep_software_metadata.csv
```
### Next steps
So, you have a NIDM file now, what can you do with it? We can query (ask questions of) it.

* What subjects do you have?

```console
>  pynidm query -nl ~/GitHub/fmriprep2NIDM/TTLs/OHSU_nidm.ttl -u /subjects
```

> Subject UUID                            Source Subject ID
> ------------------------------------  -------------------
> c77a326a-a169-11ec-b1dd-003ee1ce9545                50155
> 
> cc22f39c-a169-11ec-b1dd-003ee1ce9545                50169
> 
> c426355a-a169-11ec-b1dd-003ee1ce9545                50143

* What do we know about any given subject?

```console
>  pynidm query -nl ~/GitHub/fmriprep2NIDM/TTLs/OHSU_nidm.ttl -u /subjects
```





