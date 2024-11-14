# B200 OpenBis
The B200 OpenBis, accessible via [https://b200-openbis3.dkfz.de/](https://b200-openbis3.dkfz.de/openbis/webapp/eln-lims/),
is the central data repository of the NephrESA project.
All data used or generated within the project has to be stored within this OpenBis, and any data transfer between
partners should make use of OpenBis whenever feasible.

## Technical Details
The data repository runs using OpenBis version 20.10 (current subversion: 20.10.7.2) as provided by the ETH Zürich
Scientific IT Services.

## Administration and Login
The primary system administrator of B200 OpenBis is [Florian Tichawa](mailto:f.tichawa@dkfz-heidelberg.de) of the
Klingmüller group.

Login and user management is handled by the DKFZ IT infrastructure and supervised by the Klingmüller group.
To get access to the B200 OpenBis, every user is required to have their own user account within the DKFZ infrastructure.
Non-DKFZ members of the project can request a temporary collaboration account by contacting
[Marcel Schilling](mailto:m.schilling@dkfz-heidelberg.de) or [Florian Tichawa](mailto:f.tichawa@dkfz-heidelberg.de).
After registration, existing DKFZ accounts or collaboration accounts have to be granted access to OpenBis, after which
the DKFZ credentials can be used to access OpenBis via
[web interface](https://b200-openbis3.dkfz.de/openbis/webapp/eln-lims/) or the [](#openbis-api).

## OpenBis API
The OpenBis API is accessible via [](https://b200-openbis3.dkfz.de/openbis/openbis/rmi-application-server-v3) for the
application server interface or [](https://b200-openbis3.dkfz.de/datastore_server/rmi-data-store-server-v3) for the
datastore server interface, using the same credentials as for the web interface.
In Python and Java, the developers of OpenBis offer fully implemented libraries handling the interaction with the API.
For further instructions on how to use them, please refer to the [openBIS documentation](https://openbis.readthedocs.io/en/latest/software-developer-documentation/apis/).

## OpenBis Data Storage Structure
> This section is a detailed description of the storage structure of NephrESA data within the B200 OpenBis.
> Readers are advised to familiarize themselves with OpenBis terminology as used in the
> [openBIS documentation](https://openbis.readthedocs.io/en/latest/software-developer-documentation/apis/) before using
> this section.

> Before version 20, OpenBis used a different terminology for its data type: Objects, Object Types, and Collections were
> known as Samples, Sample Types, and Experiments in version 19 and earlier.
> In version 20, these terms can be used interchangeably, but the old terminology might be phased out eventually.
{style="warning"}

The majority of patient-related data used within NephrESA is stored within the B200 OpenBis, either as structured
Objects or as unstructured Data Sets.
All data entries and data sets related to NephrESA are exclusively stored in the `NEPHRESA` space.
The Phase I (`/NEPHRESA/PHASE_I`) Project mostly contains legacy Data Sets and Attachments, the majority of which have
been transferred into the more structured Object-based Phase II Project.
The Phase I data will not be deleted for reference purposes, but it is strongly advised to not use the data for anything
but consistency checks.

The Phase II (`/NEPHRESA/PHASE_II`) Project contains only structured data, formatted into the Object Types as described
in [](Data-Format-Definitions.md).
Within Phase II, the data is further sorted into multiple Collections based on the type and source of data:
- Plasma sample data provided by ISAS can be found in ISAS_Figures of Merit_Plasma (`/NEPHRESA/PHASE_II/PHASE_II_EXP_1`).
- Clinical data from the Bruchsal clinic (Study B) can be found in NEPHRESA_DATABASE_BRUCHSAL (`/NEPHRESA/PHASE_II/NEPHRESA_DATABASE_BRUCHSAL`).
- Clinical data from Hamburg UKE (Study N) can be found in PATIENT_DATABASE_HAMBURG (`/NEPHRESA/PHASE_II/PATIENT_DATABASE_HAMBURG`).
- Clinical data of CKD patients (Study I) can be found in PATIENT_DATABASE_CKD (`/NEPHRESA/PHASE_II/PATIENT_DATABASE_CKD`).
- Clinical data of healthy patients (Studies Z, G, H, F) can be found in PATIENT_DATABASE_HEALTHY (`/NEPHRESA/PHASE_II/NEPHRESA_DATABASE_HEALTHY`).
- Clinical data of cancer patients (Studies T, S, L, A, E, W) can be found in PATIENT_DATABASE_LUNG_CANCER (`/NEPHRESA/PHASE_II/NEPHRESA_DATABASE_LUNG_CANCER`).
- Laboratory depletion data can be found in PRECLINICAL_DEPLETION_DATA (`/NEPHRESA/PHASE_II/PRECLINICAL_DEPLETION_DATA`).
