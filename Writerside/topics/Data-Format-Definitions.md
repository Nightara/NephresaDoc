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

The Disease attribute is only stored in `SMART_PATIENT_INFO`, and can be looked up using the Patient ID field.

## OpenBis Data Types
As a table-based data repository, B200 OpenBis defines several "sample types" to hold data.
A complete list of all available types can be found in the [openBIS documentation](https://openbis.readthedocs.io/en/20.10.x/user-documentation/advance-features/excel-import-service.html#property-type) in the section "property types", but for the most part, these data types are being used within the NephrESA project:

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

To ensure uniformity across all platforms involved in NephrESA, all data stored in B200 OpenBis always follows these
conventions, unless specified otherwise:
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

### Adverse Events
An Adverse Events object (OpenBis type code `NEPHRO_ADVERSE_EVENTS`) represents a single medical event happening to a
patient, like a recorded infarction or patient death.
In addition to the identification info, an Adverse Event also contains these attributes:
> Deprecation Notice: `NEPHRO_ADVERSE_EVENTS` does not use `TIMEPOINT_HOURS` or `DATE` as part of the identification info at the
moment, but its own `NEPHRO7_START-DATE_APPLICATION-DATE` attribute instead.
This is deprecated and will be fixed in a future patch.
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

### Hemo Observables
A Hemo Observables object (OpenBis type code `NEPHRO_HEMO_OBSERVABLES`) represents a single measurement of hemo-related
observable patient parameters.
Like most Observables object types, this object is not fundamentally different from `NEPHRO_OBSERVABLES`, and was
merely extracted from the generic Observables type for performance reasons.
Its additional attributes are:

-|-|-|-
**Attribute**|**OpenBis Identifier**|**Data Type**|**Unit**|**Description**
Data Source|NEPHRO_DATA_SOURCE|Varchar|N/A|LAB if the measurements were made in a laboratory, DIALYSIS if they were taken by a dialysis machine, or BGA if they were generated from a blood gas analysis
Hemoglobin|NEPHRO_HEMOGLOBIN_I|Real|g/dl|Hemoglobin concentration
Eryhrocyte Concentration|NEPHRO_UKE_ERYTHRO|Real|billion/l| |
Total Hypochromic Erythrocyte Fraction|NEPHRO_RBC_HYPO|Real|%| |
Hematocrit Concentration|NEPHRO_UKE_HEMATOCRIT|Real|%| |
Reticulocytes|NEPHRO_UKE_RETICULOCYTES|Real|billion/l|Reticulocyte concentration, usually not used in combination with `NEPHRO_RETICULOCYTES`
Reticulocytes|NEPHRO_RETICULOCYTES|Real|%|Reticulocyte concentration, usually not used in combination with `NEPHRO_UKE_RETICULOCYTES`
Reticulocytes, Hkt-corrected|NEPHRO_UKE_RETRICULOCYTES|Real|%|Reticulocyte concentration, usually not used in combination with `NEPHRO_RETICULOCYTES`
Reticulocytes Reduction Index (RPI)|NEPHRO_RETRICULOCYTES_REDUCTION_INDEX|Real|N/A| |

### Iron Observables
...

### Observables
...
