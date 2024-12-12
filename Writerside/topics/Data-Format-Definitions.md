# Data Format Definitions
All patient-related data used in NephrESA should be stored in a systematic and system-independent table format as much
as possible.
To ensure compatibility across all systems, the following requirements have to be met by any instance of stored data:
- Any data entry has to contain the corresponding Study Name, Study ID, Patient ID, Patient Disease, and, if applicable, a Timestamp describing the time point of data acquisition as accurately as possible. This set of attributes is referred to as identification info of any object.
- The Study Name has to be unique for each study, can be in any format, but should be human-readable as much as possible. The Study Name of a study is chosen when the first set of data from the corresponding study is entered into B200 OpenBis, and has to be used exactly as defined there from then on.
- The Study ID is a unique one-letter alias for the Study Name, and is chosen together with the Study Name. A list of all Study IDs in use can be found in [Glossary: Study IDs](Glossary.md#study-ids)
- The Patient ID is a project-wide unique pseudonym matching the RegEx pattern `\w_\d+` used to identify a patient of any study. The number part of the Patient ID can be chosen freely, but the letter prefix has to match the corresponding Study ID the data is coming from. Like the Study Name and Study ID, a Patient ID is chosen when data for a patient is first entered into B200 OpenBis and should not be changed afterward.
- The Timestamp is used to identify the time point of when the corresponding data was taken from the patient, not when it was entered into the system. Valid formats for the Timestamp are any commonly used time formats (Timestamp, Datetime, etc.) and "time point in hours", which is defined as the time (In hours) between the event in question and the time of the first injection of the patient (Defined as time point 0). Negative numbers mean the event happened before time point 0, while positive numbers describe events happening after time point 0. Where possible, time point in hours is the preferred format.

In OpenBis, the identification info is stored in these attributes:

-|-|-
**Attribute**|**OpenBis Identifier**|**Data Type**
Study Name|STUDY_NAME|Varchar
Study ID|STUDY_ID|Varchar
Patient ID|NEPHRO_PATIENT_ID|Varchar
Timepoint|TIMEPOINT_HOURS|Real
Date|DATE|Date
Disease*|DISEASE|Varchar

*The Disease attribute is only stored in `SMART_PATIENT_INFO`, and can be looked up using the Patient ID field.

## Patient-related Data Types
As a table-based data repository, B200 OpenBis defines several "object types" to hold data.
A complete list of all available types can be found in the [openBIS documentation](https://openbis.readthedocs.io/en/20.10.x/user-documentation/advance-features/excel-import-service.html#property-type)
in the section "property types", but for the most part, these data types are being used within the NephrESA project:

-|-
**Data Type**|**Description**
INTEGER|Whole number
REAL|Floating-point decimal number
VARCHAR|Generic text
BOOLEAN|TRUE or FALSE
CONTROLLEDVOCABULARY|One element from a predefined list of text values
DATE|A datetime object with timezone information and a precision of seconds
TIMESTAMP|A Unix-style timestamp with a precision of seconds
TEXT|Long-format text

An attribute is uniquely identified within OpenBis via its attribute code, referred to as the OpenBis Identifier in this
documentation.
For programmatic access, it is usually safer to rely on these unique identifiers, as multiple attributes within OpenBis
might have the same display name.
Since attributes can and should be reused across OpenBis, not all OpenBis Identifiers follow the same naming principle,
but in general, many OpenBis Identifiers used in NephrESA start with the prefix `NEPHRO_`.

To ensure uniformity across all platforms involved in NephrESA, all patient-related data stored in B200 OpenBis always
follows these conventions, unless specified otherwise:
- Attributes are always stored in a uniform unit described by the attribute itself, and the values themselves pure numeric values without units wherever possible.
- Attributes that generally don't apply to the measurement in question, like values that are simply not measured by the measurement method in question (E.g. blood pressure for lab measurements), are left empty.
- Attributes that can be measured by the method in question but were not measured are set to an out-of-bounds value depending on the value in question (E.g. -1 for concentration, or `Integer.MIN_VALUE` for values that can be negative).
- The detection limits of values (If known) are stored in the attribute description, and values beyond that range (Or values marked as beyond detection limits) are clipped to the corresponding limit.

### Patient Info
A Patient Info object (OpenBis type code `SMART_PATIENT_INFO`) describes a single patient within the NephrESA project.
This object contains the identification info for the corresponding patient, as well as these pieces of demographic
information:

-|-|-|-|-
**Attribute**|**OpenBis Identifier**|**Data Type**|**Unit**|**Description**
Age|AGE_PATIENT|Integer|Years| |
Sex|GENDER_PATIENT|Integer|N/A|0 (male), 1 (female), or 2 (other)
Weight|NEPHRO_WEIGHT_PATIENT|Real|kg| |
Height|NEPHRO_HEIGHT_PATIENT|Real|cm| |
BMI|DZL_BMI|Real|kg/m²|Body-Mass-Index|

Patient Info is meant to be an abstract representation of a patient within a study, so any patient within a single study
can never have more than one Patient Info object associated with them.
As with the Patient ID, a patient participating in multiple studies is required to have exactly one Patient Info object
per study.
Furthermore, the demographic data contained within a Patient Info should not be updated to represent the current state
of the patient, but rather be kept at the initial value.
To track changes in e.g. weight or BMI, other objects like `NEPHRO_WEIGHT` or `NEPHRO_OBSERVABLES` should be used
instead.

### Adverse Event
An Adverse Event object (OpenBis type code `NEPHRO_ADVERSE_EVENT`) represents a single medical event happening to a
patient, like a recorded infarction or patient death.
In addition to the identification info, an Adverse Event also contains these attributes:
> Deprecation Notice: `NEPHRO_ADVERSE_EVENT` does not use `TIMEPOINT_HOURS` or `DATE` as part of the identification
> info at the moment, but its own `NEPHRO7_START-DATE_APPLICATION-DATE` attribute instead.
> This is deprecated and will be fixed in a future patch.
{style="warning"}

-|-|-
**Attribute**|**OpenBis Identifier**|**Data Type**|**Description**
Adverse Event|ADVERSE_EVENT|Varchar|A broad classification of the event, written in all uppercase
Diagnosis|NEPHRO_DIAGNOSIS|Varchar|A detailed description of the event, usually copied directly from the diagnosis
ICD-10 Code|ICD10|Varchar|The ICD-10-GM code of the event, if known
Diagnosis Certainty|DIAGNOSIS_CERTAINTY|Controlled Vocabulary|Diagnosis certainty code according to ICD-10-GM, if known
Start Date / Application Date|NEPHRO7_START-DATE_APPLICATION-DATE|Timestamp|Replacement for `DATE`

### Medication
A Medication object (OpenBis type code `NEPHRO_MEDICATION`) represents a single application of medication, usually an
ESA/iron injection or blood transfusion.
In addition to the identification info, a Medication object also contains these attributes:

-|-|-|-
**Attribute**|**OpenBis Identifier**|**Data Type**|**Description**
Data Source|NEPHRO_DATA_SOURCE|Varchar|Currently unused
Injection Method|INJECTION_METHOD|Controlled Vocabulary|A two-letter expression for the application method of the medication, e.g. OR (oral), IV (intravenous), or SC (subcutaneous)
Dose|DOSE|Real|The dose of medication provided to the patient, in a unit specified by "Unit of Measurement"
Unit of Measurement|DOSE_UNIT|Varchar|The unit of the number in "Dose"
Medication Type|NEPHRO_MEDICATION_TYPE|Varchar|A broad classification of the active ingredient, e.g. CERA, EPOALPHA, or VITAMIN_D
Medication Name|NEPHRO_MEDICATION_NAME|Varchar|The full name of the medication
PZN|NEPHRO_PZN|Integer|The PZN identifying the medication, if known

### Observables
An Observables objects (OpenBis type code `NEPHRO_OBSERVABLES`) represents a single measurement of observable patient
parameters.
Because the data of NephrESA comes from many different sources, a single measurement may contain any combination of
parameters with varying degrees of overlap.
It was decided that creating one or even multiple separate object type for every study or data set within the project is
not feasible, and all data should be merged into as few object types as possible.
Because of the uniformity and high overlap of iron and hemo observables, these two sets of attributes were separated
into their own object types as described in [](#hemo-observables) and
[](#iron-observables), respectively.
If any new attribute that is not already present in this or any other object type is required, it should by default be
added to the Observables object type first.
If any given subset of attributes ends up having a very high density or required very often, it should be considered to
extract them from the Observables object type into a separate Observables types, similar to [](#hemo-observables) or
[](#iron-observables).
For a current list of attributes of the Observables object type, please refer to the [](B200-OpenBis.md).
> Deviation from the defined standard: Due to the high sparsity within the Observables attributes, it was infeasible to
> set all numerical attributes to -1 or `INTEGER.MAX_VALUE` if the original data had no values in the corresponding
> fields as defined in [](#patient-related-data-types).
> Instead, attributes without values are left empty by default, unless the import process defines the source data of a
> specific Observable object as being expected to have these values, in which case empty values are filled with a
> suitable default value according to the standard.
> This will not be fixed.
{style="note"}

### Hemo Observables
A Hemo Observables object (OpenBis type code `NEPHRO_HEMO_OBSERVABLES`) represents a single measurement of hemo-related
observable patient parameters.
Like most Observables object types, this object is not fundamentally different from `NEPHRO_OBSERVABLES`, and was
merely extracted from the generic Observables type for performance reasons.
Its additional attributes are:

-|-|-|-
**Attribute**|**OpenBis Identifier**|**Data Type**|**Unit**|**Description**
Data Source|NEPHRO_DATA_SOURCE|Varchar|N/A|CLINICAL_LAB or LAB if the measurements were made in a clinical or other laboratory (Respectively), DIALYSIS if they were taken by a dialysis machine, or BGA if they were generated from a blood gas analysis
Hemoglobin|NEPHRO_HEMOGLOBIN_I|Real|g/dl|Hemoglobin concentration
Eryhrocyte Concentration|NEPHRO_UKE_ERYTHRO|Real|billion/l| |
Total Hypochromic Erythrocyte Fraction|NEPHRO_RBC_HYPO|Real|%| |
Hematocrit Concentration|NEPHRO_UKE_HEMATOCRIT|Real|%| |
Reticulocytes|NEPHRO_UKE_RETICULOCYTES|Real|billion/l|Reticulocyte concentration, usually not used in combination with `NEPHRO_RETICULOCYTES`
Reticulocytes|NEPHRO_RETICULOCYTES|Real|%|Reticulocyte concentration, usually not used in combination with `NEPHRO_UKE_RETICULOCYTES`
Reticulocytes, Hkt-corrected|NEPHRO_UKE_RETRICULOCYTES|Real|%|Reticulocyte concentration, usually not used in combination with `NEPHRO_RETICULOCYTES`
Reticulocytes Reduction Index (RPI)|NEPHRO_RETRICULOCYTES_REDUCTION_INDEX|Real|N/A| |

### Iron Observables
An Iron Observables object (OpenBis type code `NEPHRO_IRON_OBSERVABLES`) represents a single measurement of iron-related
observable patient parameters.
Like most Observables object types, this object is not fundamentally different from `NEPHRO_OBSERVABLES`, and was
merely extracted from the generic Observables type for performance reasons.
Its additional attributes are:

-|-|-|-
**Attribute**|**OpenBis Identifier**|**Data Type**|**Unit**|**Description**
Transferrin|NEPHRO_TRANSFERRIN|Real|g/l|Transferrin concentration
Transferrin Saturation|NEPHRO_TRANSFERRIN_SATURATION|Real|%| |
Soluble Transferrin Receptor|NEPHRO_STFR|Real|g/l|Soluble transferrin receptor concentration
Iron|NEPHRO_IRON|Real|ug/l|Iron concentration
NTBI|SMART_NTBI|Real|ug/l|Non-transferrin-bound iron concentration
FeS|NEPHRO_FE_S|Real|ug/l|Iron sulfite concentration
Ferritin|NEPHRO_FERRITIN|Real|ug/l|Ferritin concentration
CRP|NEPHRO_CRP|Real|mg/l|C-reactive protein concentration. Lower detection limit of 0.6 mg/l
Erythroferron|ERFE|Real|ug/l|Erythroferron concentration
Hamp|HAMP|Real|ug/l|Hepcidin concentration
Total Iron Binding Capacity|NEPHRO_TIBC|Real|umol/l| |

### ISAS Sample
ISAS Samples (OpenBis type code `ISAS_SAMPLE`) contain information on protein concentration in patient blood plasma.
In addition to the identification info, these objects contain the following fields:

> Deprecation Notice: `ISAS_SAMPLE` does not use `TIMEPOINT_HOURS` as part of the identification info at the moment, but
> the non-mandatory and less precise `ISAS_WEEK` field instead.
> Due to this, `DATE` is mandatory for these objects.
> This is deprecated and will be fixed in a future patch.
{style="warning"}

-|-|-|-
**Attribute**|**OpenBis Identifier**|**Data Type**|**Unit**|**Description**
Week|ISAS_WEEK|Integer|Weeks|The week this measurement was taken in, relative to the first week as defined in `TIMEPOINT_HOURS`
Protein Name|CALIBRATOR_PROTEIN_NAME|Varchar|N/A|Full protein name, including intra/extra domains
Replicate Name|REPLICATE_NAME|Varchar|N/A| |
Concentration|CONCENTRATION|Real|mg/l| |

### Nephro Weight
A Nephro Weight object (OpenBis type code `NEPHRO_WEIGHT`) represents a weight measurement before and after dialysis.

> Deprecation Notice: `NEPHRO_WEIGHT` does not use `TIMEPOINT_HOURS` as part of the identification info at the moment.
> Due to this, `DATE` is mandatory for these objects.
> `NEPHRO_WEIGHT_POST_HD` is of type Varchar, but should be Real.
> This is deprecated and will be fixed in a future patch.
{style="warning"}

-|-|-|-
**Attribute**|**OpenBis Identifier**|**Data Type**|**Unit**|**Description**
Weight (Prior HD)|NEPHRO_WEIGHT_PRIOR_HD|Real|kg|Patient weight before hemodialysis (wet weight)
Weight (Post HD)|NEPHRO_WEIGHT_POST_HD|Varchar|kg|Patient weight after hemodialysis (dry weight)

## Other Data Types
Some data types within OpenBis are not directly related to patients, and thus don't follow the structure defined in
[](#patient-related-data-types).

### ISAS NIST
ISAS NIST objects (OpenBis type code `ISAS_NIST`) contain reference values for [](#isas-sample) objects [certified by
NIST](https://www.nist.gov/glossary-term/20016).

> Deprecation Notice: `ISAS_NIST` has multiple identification info fields that are not in use and will not be listed
> here.
> They will be removed in a future patch.
{style="warning"}

-|-|-|-
**Attribute**|**OpenBis Identifier**|**Data Type**|**Unit**|**Description**
Protein Name|CALIBRATOR_PROTEIN_NAME|Varchar|N/A|Full protein name, including intra/extra domains
Replicate Name|REPLICATE_NAME|Varchar|N/A| |
Concentration|CONCENTRATION|Real|mg/l| |

### ISAS LoD and LLoQ
An ISAS LoD and LLoQ object (OpenBis type code `ISAS_LOD_LLOQ`) specifies the limit of detection and lower limit of
quantification for a given protein.
Its attributes are:

-|-|-|-
**Attribute**|**OpenBis Identifier**|**Data Type**|**Unit**|**Description**
Protein Name|CALIBRATOR_PROTEIN_NAME|Varchar|N/A|Full protein name, including intra/extra domains
LOD|ISAS_LOD|Real|mg/l|Limit of detection for the given protein
LLOQ|ISAS_LLOQ|Real|mg/l|Lower limit of quantification for the given protein
