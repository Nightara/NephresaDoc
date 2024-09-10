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
For further instructions on how to use them, please refer to the [openBIS documentation](https://openbis.readthedocs.io/en/20.10.x/user-documentation/advance-features/excel-import-service.html#property-type).
