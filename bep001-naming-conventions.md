## **PROPOSAL**: Introduce additional key-value pairs for anatomical imaging data.

**Justification**: There are a bunch of additional pieces of metadata that are required for some quantitative MRI scans.
Here we adjust the template for anatomical MRI files in section 4.1.x and define additional key-value pairs.

*What follows is the text from the current BIDS specification with the suggested additions inclued.*
*The text is also shown in "code diff" format to make it easy to see the proposed changes.*

---

### Anatomy imaging data

Anatomical neuroimaging files should be saved in the `anat` directory folling this template:

```Text
sub-<participant_label>/[ses-<session_label>/]
    anat/
        sub-<participant_label>[_ses-<session_label>][_indexable_metadata-<index>][_acq-<label>][_part-<mag/phase>][_ce-<label>][_rec-<label>][_run-<index>]_<modality_label>.nii[.gz]
        sub-<participant_label>[_ses-<session_label>][_indexable_metadata-<index>][_acq-<label>][_part-<mag/phase>][_ce-<label>][_rec-<label>][_run-<index>][_mod-<label>]_defacemask.nii[.gz]
```

The key-value pairs in the template are used to distinguish acquisitions from each other. They are defined below. Please note that diffusion imaging data is stored elsewhere (see
below).

#### Run key (`run-<value>`)

If several scans of the same modality and with identical acquisition parameters are acquired they MUST be indexed with the `run-<label>` key-value pair.
Only integers are allowed as run labels.
For example files will be named `_run-1`, `_run-2`, `_run-3` etc. 

When there is only one scan of a given type the `run` key MAY be omitted.

#### Acquisition key (`acq-<value>`)

The OPTIONAL `acq-<label>` key/value pair corresponds to a custom label the user MAY use to distinguish a different set of parameters used for **acquiring** the same modality.
The value of the `acq-<label>` is free text and user defined.
The values must be consistent across subjects and sessions.

For example this should be used when a study includes two T1w images - one full brain low resolution and and one restricted field of view but high resolution.
In such case two files could have the following names: `sub-01_acq-highres_T1w.nii.gz` and `sub-01_acq-lowres_T1w.nii.gz`.
Note that the user is free to choose any other label than `highres` and `lowres` as long as they are consistent across subjects and sessions.

Another example use is when different sequences are used to record the same modality (e.g. RARE and FLASH both for T1w).
This field can be used to make the distinction between the two sequences.

If the grouping logic of several scans of the same modality is (entirely or partially) bound up with a metadata field (which varies between scans), `acq-<label>` SHOULD be included in the file name.
An example of this use case is if the varying entries of the same metadata field are categorical.

While the BIDS developers have chosen to allow free text entries for the `acq-` value options, they acknowledge a risk this decision may lead lead to many labels for the same information in BIDS formatted datasets.



To enable a unified naming convention while combining several scans of the same modality intended to create quantitative map(s), the following labels SHOULD be included in the filename where applicable:

| Method Name | Labels           | Respective metadata fields   |
|-------------|------------------|------------------------------|
| MTR         | MTon, MToff      | MTC (Sequence variant attr.) |
| MTS         | MTon, MToff, T1w | MTC (Sequence variant attr.) |
| MPM         | MTon, MToff, T1w | MTC (Sequence variant attr.) |

#### Contrast enhanced key (`ce-<value>`)

The OPTIONAL `ce-<label>` key/value pair can be used to distinguish sequences using different **contrast enhanced** images. 
The label is the name of the contrast agent.
The key `ContrastBolusIngredient` MAY be also be added in the
JSON file, with the same label.

#### Reconstruction algorithm key (`rec-<value>`)

The OPTIONAL `rec-<label>` key/value pair can be used to distinguish different **reconstruction** algorithms (for example ones using motion correction).

#### Part key (`part-<value>`)

The OPTIONAL `part-<label>` key/value pair can be used to distinguish the magnitude or phase **parts** of an acquisition.
Only "mag" and "phase" are valid labels for the `part-` key.

If this key is not included, scans are assumed to be magnitude images.

####

#### `indexable_metadata-<index>` key-value pair

If the grouping logic of several scans of the same modality is (entirely or partially) bound up with a metadata field (which varies between scans), `indexable_metadata-<index>` SHOULD be included in the file name.
This is applicable if the varying entries of the same metadata field are enumerable.
Unlike other key-value pairs, the key tag of the `indexable_metadata-<index>` is mutable depending on the metadata field that varies between several scans of the same modality intended to create quantitative map(s).

The table lists some example "indexable_metadata" keys:

| Key tag | Value list | Associated metadata |
|---------|------------|---------------------|
| fa      | 1,2,... N  | FlipAngle           |
| inv     | 1,2,... N  | InversionTime       |
| echo    | 1,2,... N  | EchoTime            |
| tsl     | 1,2,... N  | SpinLockTime        |

Only integers are allowed as run labels.

```

A naming convention that can logically group parametrically linked anatomy imaging datasets can be achieved by combining following key/value pairs:

1. Suffix (a.k.a modality_label)               (REQUIRED)
2. [\_indexable\_metadata-<index>]             (RECOMMENDED)
3. [\_acq-<label>]                             (RECOMMENDED)
4. [\_part-<mag/phase>]                        (OPTIONAL)


### Suffix

Quantitative MRI (qMRI) techniques can be referred by i) sequence names, ii) outputs they generate or iii) special acronyms. As a result, a list of qMRI methods (values) cannot be semantically linked to a single key. To free this field from a key tag, names of the qMRI methods are stored in the `suffix` (a.k.a `<modality_label>`). If several scans of the same modality are intended to create a quantitative map, they MUST be labeled by `suffix`.

| Suffix  | Method Name                              | Output Name                    |
|---------|------------------------------------------|--------------------------------|
| VFA     | Variable Flip Angle                      | T1Map                          |
| IRT1    | Inversion Recovery                       | T1Map                          |
| MP2RAGE | Magnetization Prepared 2 Gradient Echoes | T1Map, UNIT1                   |
| MESET2  | Multi-Echo Spin Echo                     | MWF, T2Map                     |
| MTR     | Magnetization Transfer Ratio             | MTRmap                         |
| MTS     | Magnetization Transfer Saturation        | MTsat, T1map                   |
| MPM     | Multi-Parameter Mapping (hMRI)           | R1map, R2starmap, MTsat, PDmap |
| SESL    | Spin Echo-Spin Lock                      | T1Rmap                         |



### `acq-<label>` key-value pair

If the grouping logic of several scans of the same modality is (entirely or partially) bound up with a metadata field (which varies between scans), `acq-<label>` SHOULD be included in the file name. This is applicable if the varying entries of the same metadata field are categorical. Note that value of the `acq-<label>` is free form. However, to enable a unified naming convention while combining several scans of the same modality intended to create quantitative map(s), following labels SHOULD be included in the filename where applicable:

| Method Name | Labels           | Respective metadata fields   |
|-------------|------------------|------------------------------|
| MTR         | MTon, MToff      | MTC (Sequence variant attr.) |
| MTS         | MTon, MToff, T1w | MTC (Sequence variant attr.) |
| MPM         | MTon, MToff, T1w | MTC (Sequence variant attr.) |

### `part-<mag/phase>` key-value pair

See (0008,0008) Image Type. Vendor variant?

### Description of the quantitative maps 

| Name      | Description                                  | Units |
|-----------|----------------------------------------------|-------|
| T1map     | Longitudinal relaxation time map             | ms    |
| T2map     | True transverse relaxation time map          | ms    |
| T2starmap | Observed transverse relaxation time map      | ms    |
| R1map     | Longitudinal relaxation rate map             | 1/ms  |
| R2map     | Transverse relaxation rate map               | 1/ms  |
| R2starmap | Observed transverse relaxation rate map      | 1/ms  |
| MTRmap    | Magnetization transfer ratio map             | %     |
| MTsat     | Magnetization transfer saturation map        | %     |
| PDmap     | Proton density map                           | N/A   |
| T1Rmap    | T1-rho (T1 relaxation in rotating frame) map | ms    |
| UNIT1     | Homogeneous (flat) T1w image                 | N/A   |
