# Data Format Definitions
All patient-related data used in NephrESA should be stored in a systematic and system-independent table format as much
as possible.
To ensure compatibility across all systems, the following requirements have to be met by instance of stored data:
- Any data entry has to contain the corresponding Study Name, Study ID, Patient ID, and a Timestamp describing the timepoint of when the data was acquired as accurately as possible.
- The Study Name has to be unique for each study, can be in any format, but should be human-readable as much as possible. The Study Name of a study is chosen when the first set of data from the corresponding study is entered into B200 OpenBis, and has to be used exactly as defined there from then on.
- The Study ID is a unique one-letter alias for the Study Name, and is chosen together with the Study Name.
- The Patient ID is a project-wide unique pseudonym matching the RegEx pattern `\w_\d+` used to identify a patient of any study. The number part of the Patient ID can be choosen freely, but the letter prefix has to match the corresponding Study ID the data is coming from. Like the Study Name and Study ID, a Patient ID is chosen when data for a patient is first entered into B200 OpenBis and should not be changed afterward.
- The Timestamp is used to identify the time point of when the corresponding data was taken from the patient, **not** when it was entered into the system. Valid formats for the Timestamp are any commonly used time formats (Timestamp, Datetime, etc.) and "time point in hours", which is defined as the time (In hours) between the event in question and the time of the first injection of the patient (Defined as time point 0). Negative numbers mean the event happened before time point 0, while positive numbers describe events happening after time point 0. Where possible, time point in hours is the preferred format.

## OpenBis Data Types
As a table-based data repository, B200 OpenBis defines several "sample types" to hold data.
In general, data stored in B200 OpenBis follows these conventions, in addition to the general conventions listed above:
- Attributes are always stored in a uniform unit described by the attribute itself, and the values themselves pure numeric values without units wherever possible.
- Attributes that generally don't apply to the measurement in question, like values that are simply not measured by the measurement method in question (E.g. blood pressure for lab measurements), are left empty.
- Attributes that can be measured by the method in question but were not measured are set to an out-of-bounds value depending on the value in question (E.g. -1 for concentration, or `Integer.MIN_VALUE` for values that can be negative).
- The detection limits of values (If known) are stored in the attribute description, and values beyond that range (Or values marked as beyond detection limits) are clipped to the corresponding limit.

### Patient Info
...
