# UKE Data Flow
For the initial setup of NephrESA at the UKE, the data flow and data analysis will work as follows:
- UKE collects all necessary patient data in REDCap.
- The B200 OpenBis offers a public, secure REST interface for data upload and data retrieval that can be accessed by REDCap.
- The university of Freiburg offers a public, secure MATLAB server with an interface allowing access to the NephrESA model.
- Data flow is initiated by UKE, who converts the REDCap data into a format compatible with the [](Data-Format-Definitions.md), and uploads them to the B200 OpenBis.
- The B200 OpenBis stores all data submitted by UKE and establishes a connection to Freiburg to run predictions on the NephrESA model, which are also stored in OpenBis.
- Finally, B200 OpenBis forwards the predictions to REDCap, either as direct response to an upload request, or as response to a separate request.