![Testing - main](https://github.com/TRI-AMDD/beep/workflows/Testing%20-%20main/badge.svg)
[![Coverage Status](https://coveralls.io/repos/github/TRI-AMDD/beep/badge.svg?branch=master)](https://coveralls.io/github/TRI-AMDD/beep?branch=master)

# Battery Evaluation and Early Prediction (BEEP)

BEEP is a set of tools designed to support Battery Evaluation and Early Prediction of cycle life corresponding to the research of the [d3batt program](https://d3batt.mit.edu/) and the [Toyota Research Institute](http://www.tri.global/accelerated-materials-design-and-discovery/).


BEEP enables parsing and handing of electrochemical battery cycling data
via data objects reflecting cycling run data, experimental protocols,
featurization, and modeling of cycle life.  Currently beep supports 
arbin, maccor and biologic cyclers.

We are currently looking for experienced python developers to help us improve this package and implement new features.
Please contact any of the maintainers for more information.

# Table of Contents
1. [Installation](#installation)
2. [Environment](#environment)
3. [Testing](#testing)
4. [Using scripts](#using-scripts)
5. [Data requirements](#data-requirements)
6. [How to cite](#how-to-cite)

## Installation
Use `pip install beep` to install.

If you want to develop BEEP, clone the repo via git and use 
pip (or `python setup.py develop`)  for an editable install:

```bash
git clone git@github.com:ToyotaResearchInstitute/BEEP.git
cd BEEP
pip install -e .[tests]
```
## Environment
To configure the use of AWS resources its necessary to set the environment variable `BEEP_ENV`. For most users `'dev'`
is the appropriate choice since it assumes that no AWS resources are available. 
```.env
export BEEP_ENV='dev'
```
For processing file locally its necessary to configure the folder structure 
```.env
export BEEP_PROCESSING_DIR='/path/to/beep/data/'
```

## Testing
You can use pytest for running unittests. In order to run tests the environment variable
needs to be set (i.e. `export BEEP_ENV='dev'`)

```bash
pytest beep
```

## Using scripts

The standard installation procedure above should install and link console scripts
with currently available BEEP functionality.  Each BEEP script takes a JSON string
as input in order to provide flexibility and more facile automation.  They are documented
below:

### collate
The `collate` script takes no input, and operates by assuming the BEEP_PROCESSING_DIR (default `/`)
has subdirectories `/data-share/raw_cycler_files` and `data-share/renamed_cycler_files/FastCharge`.

The script moves files from the `/data-share/raw_cycler_files` directory, parses the metadata,
and renames them according to a combination of protocol, channel number, and date, placing them in
`/data-share/renamed_cycler_files`.

The script output is a json string that contains the following fields:

* `fid` - The file id used internally for renaming
* `filename` - full paths for raw cycler filenames
* `strname` - the string name associated with the file (i. e. scrubbed of `csv`)
* `file_list` - full paths for the new, renamed, cycler files
* `protocol` - the cycling protocol corresponding to each file
* `channel_no` - the channel number corresponding to each file
* `date` - the date corresponding to each file

Example:
```bash
$ collate
```
```json
{
    "mode": "events_off",
    "fid": [0, 
            1, 
            2],
    "strname": ["2017-05-09_test-TC-contact", 
                "2017-08-14_8C-5per_3_47C", 
                "2017-12-04_4_65C-69per_6C"],
    "file_list": ["/data-share/renamed_cycler_files/FastCharge/FastCharge_0_CH33.csv", 
                  "/data-share/renamed_cycler_files/FastCharge/FastCharge_1_CH44.csv", 
                  "/data-share/renamed_cycler_files/FastCharge/FastCharge_2_CH29.csv"],
    "protocol": [null, 
               "8C(5%)-3.47C", 
               "4.65C(69%)-6C"],
    "date": ["2017-05-09", 
             "2017-08-14", 
             "2017-12-04"],
    "channel_no": ["CH33", 
                   "CH44", 
                   "CH29"],
    "filename": ["/data-share/raw_cycler_files/2017-05-09_test-TC-contact_CH33.csv", 
                 "/data-share/raw_cycler_files/2017-08-14_8C-5per_3_47C_CH44.csv", 
                 "/data-share/raw_cycler_files/2017-12-04_4_65C-69per_6C_CH29.csv"]
}
```

### validate
The validation script, `validate`, runs the validation procedure contained
in `beep.validate` on renamed files according to the output of `rename` above.
It also updates a general json validation record in `/data-share/validation/validation.json`.

The input json must contain the following fields

* `file_list` - the list of filenames to be validated
* `mode` - mode for events i.e. 'test' or 'run'
* `run_list` - list of run_ids for each of the files, used by the database for linking data

The output json will have the following fields:

* `validity` - a list of validation results, e. g. `["valid", "valid", "invalid"]`
* `file_list` - a list of full path filenames which have been processed

Example:
```bash
$ validate '{
    "mode": "events_off",
    "run_list": [1, 20, 34],
    "strname": ["2017-05-09_test-TC-contact", 
                "2017-08-14_8C-5per_3_47C", 
                "2017-12-04_4_65C-69per_6C"],
    "file_list": ["/data-share/renamed_cycler_files/FastCharge/FastCharge_0_CH33.csv", 
                  "/data-share/renamed_cycler_files/FastCharge/FastCharge_1_CH44.csv", 
                  "/data-share/renamed_cycler_files/FastCharge/FastCharge_2_CH29.csv"],
    "protocol": [null, 
               "8C(5%)-3.47C", 
               "4.65C(69%)-6C"],
    "date": ["2017-05-09", 
             "2017-08-14", 
             "2017-12-04"],
    "channel_no": ["CH33", 
                   "CH44", 
                   "CH29"],
    "filename": ["/data-share/raw_cycler_files/2017-05-09_test-TC-contact_CH33.csv", 
                 "/data-share/raw_cycler_files/2017-08-14_8C-5per_3_47C_CH44.csv", 
                 "/data-share/raw_cycler_files/2017-12-04_4_65C-69per_6C_CH29.csv"]
}'
```
```json
{"validity": ["invalid",
              "invalid",
              "valid"],
 "file_list": ["/data-share/renamed_cycler_files/FastCharge/FastCharge_0_CH33.csv", 
               "/data-share/renamed_cycler_files/FastCharge/FastCharge_1_CH44.csv", 
               "/data-share/renamed_cycler_files/FastCharge/FastCharge_2_CH29.csv"],
}
```

### structure

The `structure` script will run the data structuring on specified filenames corresponding
to validated raw cycler files.  It places the structured datafiles in `/data-share/structure`.

The input json must contain the following fields:
* `file_list` - a list of full path filenames which have been processed
* `validity` - a list of boolean validation results, e. g. `[True, True, False]`
* `mode` - mode for events i.e. 'test' or 'run'
* `run_list` - list of run_ids for each of the files, used by the database for linking data

The output json contains the following fields:

* `invalid_file_list` - a list of invalid files according to the validity
* `file_list` - a list of files which have been structured into processed_cycler_runs

Example:
```bash
$ structure '{
    "mode": "events_off",
    "run_list": [1, 20, 34],
    "validity": ["invalid", "invalid", "valid"], 
    "file_list": ["/data-share/renamed_cycler_files/FastCharge/FastCharge_0_CH33.csv", 
                  "/data-share/renamed_cycler_files/FastCharge/FastCharge_1_CH44.csv", 
                  "/data-share/renamed_cycler_files/FastCharge/FastCharge_2_CH29.csv"]}'
```
```json
{
  "invalid_file_list": ["/data-share/renamed_cycler_files/FastCharge/FastCharge_0_CH33.csv", 
                       "/data-share/renamed_cycler_files/FastCharge/FastCharge_1_CH44.csv"], 
  "file_list": ["/data-share/structure/FastCharge_2_CH29_structure.json"],
}
```

### featurize
The `featurize` script will generate features according to the methods
contained in beep.generate_features.  It places output files corresponding to 
features in `/data-share/features/`.

The input json must contain the following fields

* `file_list` - a list of processed cycler runs for which to generate features
* `mode` - mode for events i.e. 'test' or 'run'
* `run_list` - list of run_ids for each of the files, used by the database for linking data

The output json file will contain the following:

* `file_list` - a list of filenames corresponding to the locations of the features

Example:
```bash
$ featurize '{
    "mode": "events_off",
    "run_list": [1, 20, 34],
    "invalid_file_list": ["/data-share/renamed_cycler_files/FastCharge/FastCharge_0_CH33.csv", 
                          "/data-share/renamed_cycler_files/FastCharge/FastCharge_1_CH44.csv"], 
    "file_list": ["/data-share/structure/FastCharge_2_CH29_structure.json"]
}'
```
```json
{
  "file_list": ["/data-share/features/FastCharge_2_CH29_full_model_features.json"]}
```

### run_model
The `run_model` script will generate a model and create predictions
based on the features previously generated by the generate_features.
It stores its outputs in `/data-share/predictions/`

The input json must contain the following fields
* `file_list` - list of files corresponding to model features
* `mode` - mode for events i.e. 'test' or 'run'
* `run_list` - list of run_ids for each of the files, used by the database for linking data

The output json will contain the following fields
* `file_list` - list of files corresponding to model predictions

Example:
```bash
$ run_model '{
    "mode": "events_off",
    "run_list": [34],
    "file_list": ["/data-share/features/FastCharge_2_CH29_full_model_features.json"]
}'
```
```json
{
  "file_list": ["/data-share/predictions/FastCharge_2_CH29_full_model_predictions.json"],
}
```

## Data requirements

BEEP automatically parses and structures data based on specific outputs from various 
battery cyclers. The following column headers are required for downstream processing of each 
cycler.

### Arbin

Arbin data files are of the form `name_of_file_CHXX.csv` with an associated metadata file `name_of_file_CHXX_Metadata.csv`


##### Cycler Data

| Column name | Required |   Explanation  | Unit |  Data Type |
|-------------|----------|-------------|------|------------|
| `data_point` |   | index of this data point  |  |   `int32` |
| `test_time` |   |  time of data point relative to start | seconds  |   `float32` |
| `datetime` | ✓  |  time of data point relative to epoch time | seconds  |   `float32` |
| `step_time` |   |  elapsed time counted from the starting point of present active step |  seconds |   `float32` |
| `step_index` | ✓  | currently running step number in the active schedule  |   |   `int16` |
| `cycle_index` | ✓  | currently active test cycle number  |   |   `int32` |
| `current` | ✓  | measured value of present channel current  |  A |   `float32` |
| `voltage` | ✓  | measured value of present channel voltage  |  V |   `float32` |
| `charge_capacity` | ✓  | cumulative value of present channel charge capacity  | Ah  |   `float64` |
| `discharge_capacity` | ✓  | cumulative value of present channel discharge capacity  | Ah  |   `float64` |
| `charge_energy` | ✓  | cumulative value of present channel charge energy  |  Wh  |   `float64` |
| `discharge_energy` | ✓  | cumulative value of present channel discharge energy   | Wh  |   `float64` |
| `dv/dt` |   | the first-order change rate of voltage  |  V/s |   `float32` |
| `internal_resistance` | ✓  | calculated internal resistance  |  Ohm |   `float32` |
| `temperature` | ✓  | cell temperature | °C |   `float32` |


##### Metadata

| Field name | Required |
|------------|-------------|
| `test_id`  |  |
| `device_id`  |  |
| `iv_ch_id`  |  |
| `first_start_datetime`  |  |
| `schedule_file_name`  |  |
| `item_id`  |  |
| `resumed_times`  |  |
| `last_end_datetime`  |  |
| `databases`  |  |
| `grade_id`  |  |
| `has_aux`  |  |
| `has_special`  |  |
| `schedule_version`  |  |
| `log_aux_data_flag`  |  |
| `log_special_data_flag`  |  |
| `rowstate`  |  |
| `canconfig_filename`  |  |
| `m_ncanconfigmd5`  |  |
| `value`  |  |
| `value2`  |  |



## How to cite
If you use BEEP, please cite this article:

> P. Herring, C. Balaji Gopal, M. Aykol, J.H. Montoya, A. Anapolsky, P.M. Attia, W. Gent, J.S. Hummelshøj, L. Hung, H.-K. Kwon, P. Moore, D. Schweigert, K.A. Severson, S. Suram, Z. Yang, R.D. Braatz, B.D. Storey, SoftwareX 11 (2020) 100506.
[https://doi.org/10.1016/j.softx.2020.100506](https://doi.org/10.1016/j.softx.2020.100506)

