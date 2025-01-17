# OpenBIS REST API
The NephrESA OpenBIS offers a dedicated REST interface to enable communication between RedCAP and openBIS, which can be
found on [GitHub](https://github.com/NantiaL/openBIS-API).
The repository is currently set to private, please contact [Nantia Leonidou](mailto:nantia.leonidou@dkfz-heidelberg.de)
for access.
Currently, the API offers these endpoints:
1. `/help/`: Lists all available endpoints.
2. `/ids/`: Lists all OpenBIS permids matching the supplied filter criteria.
3. `/data/`: Provides a single object matching the supplied filter criteria in the OpenBIS JSON format.
4. `/upload/`: Takes a list of attributes and inserts them into OpenBIS as new objects.
## Authentication
The API requires valid OpenBIS credentials, which are used to process the requests within OpenBIS.
Authentication is handled through the [WWW-Authenticate](https://datatracker.ietf.org/doc/html/rfc2617) protocol using
the `NephrESA` basic realm.
Invalid or missing credentials are answered with a 401 status code. 
> The code provided for this API does not encrypt any communication paths on its own.
> Any custom instances of this interface have to ensure security by encrypting all communications between the client,
> the interface, and OpenBIS, e.g. by using SSL or only communicating locally within a secure network.
> It is highly recommended to always use SSL, even within secure networks, because WWW-Authenticate might transmit
> unencrypted passwords otherwise.
{style="warning"}
## OpenBIS JSON Format
The JSON format used by this API always follows the same structure.
OpenBIS objects are formatted as simple JSON objects using the attribute codes as keys, and the corresponding values as
objects, e.g.
```JSON
{
  "NEPHRO_PATIENT_ID": "N_001",
  "DATE": "2019-09-07T15:50",
  "NEPHRO_SMOKING": false,
  "NEPHRO_HB": 5.3,
  "NEPHRO_SEX": "MALE",
  "NEPHRO_AGE": 62
}
```
Where possible, trailing commas should be omitted, and dates should be encoded as strings according to
[ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) for maximum compatibility.
Empty values can either be omitted altogether or represented as `null` values.

In some cases, multiple objects have to be transmitted at once.
In these cases, they should ideally be sent as a single JSON array containing one JSON object per OpenBIS object to
reduce the server load:
```JSON
[
  {
      "NEPHRO_PATIENT_ID": "N_001",
      "DATE": "2019-09-07T15:50",
      "NEPHRO_SMOKING": false,
      "NEPHRO_HB": 5.3,
      "NEPHRO_SEX": "MALE",
      "NEPHRO_AGE": 62
  },
  {
    "NEPHRO_PATIENT_ID": "N_002",
    "DATE": "2019-09-08T15:50",
    "NEPHRO_SMOKING": true,
    "NEPHRO_HB": 7.3,
    "NEPHRO_SEX": "FEMALE",
    "NEPHRO_AGE": null // Can be omitted
  }
]
```
If the object type is not obvious from context, all objects of the same type should be combined into a single list,
which is then added to the transmitted containing JSON object as the value of an attribute named like the object type.
```JSON
{
  "SMART_PATIENT_INFO":
  [
    {
      "NEPHRO_PATIENT_ID": "N_001",
      "DATE": "2019-09-07T15:50",
      ...
    },
    ...
  ],
  "NEPHRO_ADVERSE_EVENT":
  [
    {
      "NEPHRO_PATIENT_ID": "N_002",
      "DATE": "2019-09-08T15:50",
      ...
    },
    ...
  ]
}
```
## API Endpoints
Most endpoints support both `GET` and `POST` requests.
For `GET` requests, all required information has to be encoded using the URL, while `POST` ignores anything in the URL
and only reads the request body.
All allowed arguments for endpoints are listed in their respective `/help/` entry, and unless stated otherwise, they are
usually simple string or number objects.
### Upload Endpoint
The upload endpoint only supports `POST` requests.
The objects to be saved in OpenBIS have to be supplied as the `data` argument using the JSON format described above,
using the variant with explicit object type names.

This endpoint has a 30-second rate limit (Blocked with status code `429`) and automatically blocks repeated requests
with a `409` status code for security reasons.
Because duplicates are identified using hashes, hash collisions can potentially lead to mistakenly blocked genuine
requests.