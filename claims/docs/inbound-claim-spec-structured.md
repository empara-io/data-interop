# Claims EDI Specification

**Inbound | Structured Format**  
Transmission from Partner Organization to Empara

## Revision History

| Version                                                                                                                           | Last Updated       |
|-----------------------------------------------------------------------------------------------------------------------------------|--------------------|
| 1.1                                                                                                                               | September 24, 2025 |
| [1.0](https://github.com/empara-io/data-interop/blob/inbound-claims-structured_v1.0/claims/docs/inbound-claim-spec-structured.md) | August 20, 2025    |

## Table of Contents

- [Introduction](#introduction)
- [Definition of Terms](#definition-of-terms)
- [Data Exchange](#data-exchange)
  - [SFTP Path Structure](#sftp-path-structure)
    - [File Chunking](#file-chunking)
    - [Ad-hoc File Exchange](#ad-hoc-file-exchange)
  - [Naming Conventions](#naming-conventions)
  - [Error Handling](#error-handling)
  - [Miscellaneous Rules](#miscellaneous-rules)
- [Data Types](#data-types)
  - [Datetimes & Timezones](#datetimes--timezones)
  - [Enumerated Types](#enumerated-types)
    - [Claim Type](#claim-type)
    - [Resource Status](#resource-status)
    - [Use](#use)
    - [Payee Type](#payee-type)
    - [Outcome](#outcome)
    - [Decision](#decision)
- [File Types](#file-types)
  - [Claim Requests](#claim-requests)
    - [File Encoding](#file-encoding)
    - [Row Definition](#row-definition)
    - [File Schema](#file-schema)
  - [Claim Responses](#claim-responses)
    - [File Encoding](#file-encoding-1)
    - [Row Definition](#row-definition-1)
    - [File Schema](#file-schema-1)
- [Addendums](#addendums)
  - [Identifier Systems](#identifier-systems)
  - [Code Systems](#code-systems)
  - [Data Nullification](#data-nullification)

## Introduction

The purpose of this document is to outline the standard schemas and semantics for Empara's claims files for trading partners providing claims data. This includes general information and procedures regarding claims data exchange and integration. This document uses the terms "**MUST**", "**MUST NOT**", "**SHOULD**", "**SHOULD NOT**", and "**MAY**" as defined by [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) to indicate requirement levels.

This document specifies data integration requirements for two file types: claim request files and claim response files. It includes details on file transmission protocoles, directory structures, naming conventions, and file types. It also provides the schema required for both file types, and describes how rows in a file correspond to entities within Empara's system, including entity identification and lifecycle statuses.

## Definition of Terms

- **Trading partner**: A business entity that works with Empara to provide claims data. This can be a client of Empara or an organization providing services on behalf of a client.
- **Data provider**: A trading partner that provides data to Empara, such as claims, eligibility, or accumulators, via EDI or an API.
- **The Sender**: A term used in this document to refer to the data provider sending claims file according to this specification.

## Data Exchange

All files **MUST** be exchanged via SFTP. It is **RECOMMENDED** to use an SFTP server hosted by Empara as the exchange point, but connections to a partner-hosted SFTP server can be accommodated if necessary.

### SFTP Path Structure

When using Empara's SFTP server, the following path structure **MUST** be used:

```text
.
├── fromEmpara/
│   ├── ...
└── toEmpara/
    ├── claimRequests/
    │   ├── [prefix]_claim_requests_[YYYYMMDD].tsv
    │   └── ...
    ├── claimResponses/
    │   ├── [prefix]_claim_responses_[YYYYMMDD].tsv
    │   └── ...
    └── ...
```

The directories relevant to this specification are `toEmpara/claimRequests` and `toEmpara/claimResponses`.

#### File Chunking

If a file is expected to exceed 100MB or 1 million rows, it is **RECOMMENDED** to split the file into multiple parts, or "chunks":

```text
.
├── fromEmpara/
│   ├── ...
└── toEmpara/
    ├── claimRequests/
    │   ├── [prefix]_claim_requests_[YYYYMMDD]_01.tsv
    │   ├── [prefix]_claim_requests_[YYYYMMDD]_02.tsv
    │   ├── ...
    │   ├── [prefix]_claim_requests_[YYYYMMDD]_[n].tsv
    │   ├── [prefix]_claim_requests_[YYYYMMDD].ready
    │   └── ...
    ├── claimResponses/
    │   ├── [prefix]_claim_responses_[YYYYMMDD]_01.tsv
    │   ├── [prefix]_claim_responses_[YYYYMMDD]_02.tsv
    │   ├── ...
    │   ├── [prefix]_claim_responses_[YYYYMMDD]_[n].tsv
    │   ├── [prefix]_claim_responses_[YYYYMMDD].ready
    │   └── ...
    └── ...
```

If the sender chooses to split files, this **MUST** be communicated to Empara beforehand, ideally during implementation. The sender **MUST** adopt one convention for file submission (either single or chunked files) and apply it consistently. A mix of both conventions **MUST NOT** be used.

For split files, a `.ready` file with the same base name as the split files (without the `_[n]` index) **MUST** be sent to indicate that all file parts have been transmitted and are ready for processing. This file **SHOULD** be empty, as its contents are not read. Empara's pipelines scans each directory once per day; the scan concludes for the day once the first file (or `.ready` file) is detected.

#### Ad-hoc File Exchange

For file submissions that occur outside of the regular schedule, such as for error corrections or historical data, the following convention **SHOULD** be used:

```text
.
├── fromEmpara/
│   ├── ...
└── toEmpara/
    ├── claimRequests/
    │   ├── [prefix]_claim_requests_[YYYYMMDD]_adhoc_01.tsv
    │   ├── [prefix]_claim_requests_[YYYYMMDD]_adhoc_01_reason.txt
    │   ├── ...
    │   ├── [prefix]_claim_requests_[YYYYMMDD]_adhoc_[n].tsv
    │   ├── [prefix]_claim_requests_[YYYYMMDD]_adhoc_[n]_reason.txt
    │   └── ...
    ├── claimResponses/
    │   ├── [prefix]_claim_responses_[YYYYMMDD]_adhoc_01.tsv
    │   ├── [prefix]_claim_responses_[YYYYMMDD]_adhoc_01_reason.txt
    │   ├── ...
    │   ├── [prefix]_claim_responses_[YYYYMMDD]_adhoc_[n].tsv
    │   ├── [prefix]_claim_responses_[YYYYMMDD]_adhoc_[n]_reason.txt
    │   └── ...
    └── ...
```

The number in the file name is a counter to accommodate multiple ad-hoc files on the same date. A corresponding `_reason.txt` file **SHOULD** be provided to briefly describe the reason for the ad-hoc submission. A simple description is acceptable, though detailed explanations are preferred to support auditing and data lineage.

If ad-hoc files need to be split, the following convention **SHOULD** be applied:

```text
.
├── fromEmpara/
│   ├── ...
└── toEmpara/
    ├── claimRequests/
    │   ├── [prefix]_claim_requests_[YYYYMMDD]_adhoc_01_part_01.tsv
    │   ├── [prefix]_claim_requests_[YYYYMMDD]_adhoc_01_part_02.tsv
    │   ├── ...
    │   ├── [prefix]_claim_requests_[YYYYMMDD]_adhoc_01_part_[m].tsv
    │   ├── [prefix]_claim_requests_[YYYYMMDD]_adhoc_01_reason.txt
    │   ├── ...
    │   ├── [prefix]_claim_requests_[YYYYMMDD]_adhoc_[n]_part_[k].tsv
    │   ├── [prefix]_claim_requests_[YYYYMMDD]_adhoc_[n]_reason.txt
    │   └── ...
    ├── claimResponses/
    │   ├── [prefix]_claim_responses_[YYYYMMDD]_adhoc_01_part_01.tsv
    │   ├── [prefix]_claim_responses_[YYYYMMDD]_adhoc_01_part_02.tsv
    │   ├── ...
    │   ├── [prefix]_claim_responses_[YYYYMMDD]_adhoc_01_part_[m].tsv
    │   ├── [prefix]_claim_responses_[YYYYMMDD]_adhoc_01_reason.txt
    │   ├── ...
    │   ├── [prefix]_claim_responses_[YYYYMMDD]_adhoc_[n]_part_[k].tsv
    │   ├── [prefix]_claim_responses_[YYYYMMDD]_adhoc_[n]_reason.txt
    │   └── ...
    └── ...
```

A `.ready` file is not necessary for ad-hoc submissions, as they are not subject to automated processing. The sender **MUST** notify Empara directly via email whenever an ad-hoc file is submitted.

### Naming Conventions

The file naming convention is `[prefix]_[file_type]_[YYYYMMDD].tsv` for single files, and `[prefix]_[file_type]_[YYYYMMDD]_[k].tsv` for chunked files. The prefix is a unique identifier assigned to the trading partner by Empara during implementation. The `[YYYYMMDD]` portion **MUST** correspond to the date the file is transmitted.

For example, for a company named "Contoso Benefit Administrators (CBA)" with an assigned ID of `contoso`, the file names would be `contoso_claim_requests_20250401.tsv` and `contoso_claim_responses_20250401.tsv` These files **SHOULD** include data for all relevant member groups for the implementation.

If a trading partner has multiple clients with separate data environments, it is **RECOMMENDED** to send separate files for each client to isolate client-specific data characteristics. In this case, the prefixes would be client-specific, such as `datasolutions_acme` and `datasolutions_vandelay`.

### Error Handling

Claims and claim responses may not be processed at times, typically when failing to meet data requirements, or pass validation. The sender **MUST** communicate their preferred strategy for receiving acknowledgments and error reports during implementation.

### Miscellaneous Rules

Once a daily file or set of daily part files have been transmitted, it **MUST NOT** be overwritten. As the pipeline's daily scan concludes upon file detection, any subsequent modifications to a transmitted file on the same day will be ignored. To correct or enhance a submitted file, an ad-hoc file **MUST** be provided as outlined in the section "[Ad-hoc File Exchange](#ad-hoc-file-exchange)".

## Data Types

The following data types are used in this specification as defined below:

| Name       | Definition                                                                                                                                    |
|---------- |--------------------------------------------------------------------------------------------------------------------------------------------- |
| `STRING`   | A sequence of characters; i.e., textual data.                                                                                                 |
| `INTEGER`  | A number that is not a fraction.                                                                                                              |
| `NATURAL`  | A non-negative integer.                                                                                                                       |
| `REAL`     | A real number                                                                                                                                 |
| `MONETARY` | A monetary value, formatted as a number with two decimal places. E.g, `25.10` is read as 25 dollars and ten cents. Unit is assumed to be USD. |
| `DATETIME` | A particular date and time, expressed in an ISO 8601 format; e.g., `2025-01-01T00:00:00Z`                                                     |
| `DATE`     | A particular date, expressed in an ISO 8601 format; e.g., `2025-01-01`                                                                        |
| `ENUM`     | An enumerated value. The various enumerated types and their set of values are described in the subsection below.                              |

### Datetimes & Timezones

Timezone information is critical for accurate data processing. Datetime values **SHOULD** either be converted to UTC, or include a timezone offset before being sent to Empara. For example, if a procedure began at 5:00PM EST on January 5, 2025, the value received by Empara **MUST** be either `2025-01-0122:00:00Z` (UTC conversion) or `2025-01-05T17:00:00-05:00` (UTC offset included).

If the upstream system does not capture timezones for a `DATETIME` field, the sender **MUST** assign a zero time (midnight) and an appropriate timezone offset. For example, if the sender's system operates in EST and only stores a claim received date of `2025-01-05`, this value **MUST** be transmitted as `2025-01-05T05:00:00Z` or `2025-01-05T00:00:00-05:00`. This conversion method **MUST** be applied consistently across all files sent to Empara to ensure proper record matching with related data, such as eligibility files.

### Enumerated Types

#### Claim Type

Indicates the category of claim.

| Value           | Description                                                                                        |
|--------------- |-------------------------------------------------------------------------------------------------- |
| `institutional` | Hospital, clinic, and typically in-patient claims.                                                 |
| `professional`  | Typically outpatient claims from a physician, psychologist, psychiatrist, physical therapist, etc. |
| `pharmacy`      | Claims for pharmacy goods or services.                                                             |
| `oral`          | Dental, denture, or hygiene claims.                                                                |
| `vision`        | Vision claims for services such as glasses and contact lenses.                                     |

#### Resource Status

Indicates the lifecycle state of the data record within Empara's system, as distinct from the lifecycle status of the real-world claim within the adjudication process.

| Value              | Description                                                                                                             |
|------------------ |----------------------------------------------------------------------------------------------------------------------- |
| `active`           | Indicates the claim is currently in-force.                                                                              |
| `entered-in-error` | The claim was sent in error. Used for data nullification. See "[Data Nullification](#data-nullification)" for more information. |

#### Use

Indicates the nature of the request.

| Value               | Description                                                                                                                                                   |
|------------------- |------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `claim`             | A standard claim. A request to an insurer to adjudicate the supplied charges and pay the determined benefit amount.                                           |
| `pre-authorization` | A request to an insurer to adjudicate proposed future charges and potentially reserve funds for payment when claims for the indicated services are submitted. |

#### Payee Type

The party to be reimbursed for the cost of the products or services.

| Value         | Description                                          |
|------------- |---------------------------------------------------- |
| `subscriber`  | The subscriber will be reimbursed.                   |
| `beneficiary` | The beneficiary (member/patient) will be reimbursed. |
| `provider`    | Any benefit payable will be paid to the provider.    |
| `other`       | Any benefit payable will be paid to a third party.   |

#### Outcome

The outcome of processing.

| Value      | Description                                                                                 |
|---------- |------------------------------------------------------------------------------------------- |
| `queued`   | The claim/pre-auth/pre-determination has been received, but processing has not begun.       |
| `complete` | Processing has completed without errors.                                                    |
| `error`    | One or more errors have been detected in the claim.                                         |
| `partial`  | No errors have been detected in the claim, and some of the adjudication has been performed. |

#### Decision

The result of the adjudication.

| Value      | Description                                                               |
|---------- |------------------------------------------------------------------------- |
| `denied`   | The claim or services are not approved for any payment. I.e., 'rejected'. |
| `approved` | The claim or services are approved for the submitted amounts.             |
| `partial`  | The claim or services are approved at an amount less than submitted.      |
| `pending`  | Adjudication is not complete.                                             |

## File Types

The sender **MUST** send two types of files: claim request and claim response files, as outlined below.

### Claim Requests

Claim requests represent submitted claims and convey financial and clinical information, such as the provider, payee, costs, and services rendered.

#### File Encoding

Claim request files **SHOULD** be formatted as tab-separated values (TSV format). If tab delimiters cannot be used, an alternative delimited **MAY** be agreed upon during implementation. If comma delimiters are used, the file extension **SHOULD** be `.csv`. For other delimiters (such as a pipe `|`, for instance), the extension **SHOULD** be `.txt`.

The first line of the file **MUST** be a header row which indicates the order of the fields as columns.

Files **SHOULD** use a UTF-8 character encoding without a byte order mark (BOM) and **SHOULD** use Unix line endings (line feeds only). Any system limitations preventing adherence to this format **MUST** be communicated during implementation.

#### Row Definition

A single row in the claim request file represents one line item on a submitted claim. Information for the entire claim (header-level data) is duplicated on each row for that claim. The header values **MUST** be identical across all line items for a given claim. To change a claim-level value, the sender **MUST** send all line items for that claim with the updated header values.

The following fields constitute the composite primary key of a claim line item:

- `memberGroupId`
- `groupPlanId`
- `subscriberId`
- `memberId`
- `claimId`
- `seq`

All of these fields except `seq` describe the primary key of the claim entity.

Alternatively, the sender **MAY** designate a single-value primary key for the claim with the `_sourceClaimKey` field. The value for this field **MUST** be the exact primary key from the source system's database, and **MUST NOT** be a business identifier or claim number.

If a value for `_sourceClaimKey` is not provided, any change to the fields which are part of the composite primary key (`memberGroupId`, `groupPlanId`, `subscriberId`, `memberId`, or `claimId`) will be processed as a new claim record. Therefore, if any of these values must be changed, all rows sent previously for the claim **MUST** be nullified (see "[Data Nullification](#data-nullification)") before or when the updated values are sent.

#### File Schema

The "Required?" column specifies the minimum fields necessary for schema validation and successful data integration. Fields with a `NO` requirement level may nevertheless be required under the terms of the governing business agreement. The sender **MUST** consult both this technical specification and the requirements outlined by the business agreement to determine the complete set of required data elements before transmission.

Columns corresponding to fields with a `NO` requirement level **MAY** be excluded from the file if they contain no data for every row. However, once a column for a field is included in a transmitted file, it **MUST** be present in all subsequent files, even if the field is no longer populated.

To enhance readability and reduce redundancy, some fields in the schema table are presented in a condensed format. For example, a field named "`item_note_1` - `4`," actually represents four distinct fields, with names: `item_note_1`, `item_note_2`, `item_note_3`, and `item_note_4`. All fields presented with this condensed representation share the same data type, requirement level, and definition. This denormalization is necessary for the schema to adhere to a structured format. As an alternative, a JSON Lines schema is available upon request for senders who would prefer a semi-structured format.

| Name                               | Data Type               | Required? | Description                                                                                                                                                                              |
|---------------------------------- |----------------------- |--------- |---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `_sourceClaimKey`                  | `STRING`                | `NO`      | The primary key of the claim entity (header) within the partner's system. This **MUST NOT** be the claim id/number.                                                                      |
| `claimId`                          | `STRING`                | `YES`     | The unique business identifier for the claim, as assigned or designated by the partner. Typically the claim number.                                                                      |
| `claimNo`                          | `STRING`                | `YES`     | The claim number for tracking, recognized by all parties using the claim.                                                                                                                |
| `seq`                              | `NATURAL`               | `YES`     | A sequential identifier for the line item within a claim (i.e., the line number).                                                                                                        |
| `createdAt`                        | `DATETIME`              | `YES`     | The date and time that the claim was drafted.                                                                                                                                            |
| `memberGroupId`                    | `STRING`                | `YES`     | The ID of the member groups. This **MUST** match the group ID in the eligibility file.                                                                                                   |
| `groupPlanId`                      | `STRING`                | `YES`     | The ID of the benefit plan. This **MUST** match the plan ID in the eligibility file.                                                                                                     |
| `subscriberId`                     | `STRING`                | `YES`     | The ID of the subscriber. This **MUST** match the subscriber ID in the eligibility file.                                                                                                 |
| `memberId`                         | `STRING`                | `YES`     | The ID of the member receiving the product or service. This **MUST** match the member ID in the eligibility file.                                                                        |
| `point`                            | `DATETIME`              | `YES`     | A point in time, typically the date of service, used to determine the applicable coverage period.                                                                                        |
| `header_type`                      | `ENUM(Claim Type)`      | `YES`     | See: "[Claim Type](#claim-type)"                                                                                                                                                         |
| `header_resourceStatus`            | `ENUM(Resource Status)` | `YES`     | See: "[Resource Status](#resource-status)"                                                                                                                                                    |
| `header_use`                       | `ENUM(Use)`             | `YES`     | See: "[Use](#use)"                                                                                                                                                                |
| `header_eventClaimReceived`        | `DATETIME`              | `NO`      | The date when the claim processor (e.g., insurer, TPA) received the claim or pre-authorization.                                                                                          |
| `header_providerName`              | `STRING`                | `YES`     | The display name of the provider responsible for the claim.                                                                                                                              |
| `header_providerIdentifier`        | `STRING`                | `NO`      | The business identifier of the provider (e.g, NPI, TIN). **MUST** conform to the specified identifier system.                                                                            |
| `header_providerIdentifierSystem`  | `STRING`                | `COND`    | **MUST** be provided if `header_providerIdenifier` is present. See ["Identifier Systems"](#identifier-systems)                                                                                   |
| `header_patientId`                 | `STRING`                | `NO`      | The provider-assigned identifier for the member/beneficiary.                                                                                                                             |
| `header_payeeType`                 | `ENUM(Payee Type)`      | `NO`      | See: "[Payee Type](#payee-type)"                                                                                                                                                         |
| `header_patientPaid`               | `MONETARY`              | `NO`      | The amount the patient has already paid to the provider, for the entire claim.                                                                                                           |
| `header_totalCost`                 | `MONETARY`              | `YES`     | The total cost of all items in the claim.                                                                                                                                                |
| `item_providerName`                | `STRING`                | `NO`      | The display name of the provider who rendered the service for this line item.                                                                                                            |
| `item_providerIdentifier`          | `STRING`                | `NO`      | The business identifier of the servicing provider (e.g, NPI, TIN). **MUST** conform to the specified identifier system.                                                                  |
| `item_providerIdentifierSystem`    | `STRING`                | `COND`    | **MUST** be provided if `item_providerIdentifier` is present. See ["Identifier Systems"](#identifier-systems)                                                                                    |
| `item_diagnosisCode_1` - `5`       | `STRING`                | `NO`      | A diagnosis code assigned to the line item (e.g., an ICD-10 code). **MUST** conform to the specified code system.                                                                        |
| `item_diagnosisCodeSystem_1` - `5` | `STRING`                | `COND`    | **MUST** be provided if `item_diagnosisCode_[k]` is given. See: "[Code Systems](#code-systems)"                                                                                            |
| `item_diagnosisCodeDesc_1` - `5`   | `STRING`                | `COND`    | **MUST** be provided if `item_diagnosisCode_[k]` is given. A brief human-readable description of the diagnosis, which will be displayed to end-users of the Empara application.          |
| `item_procedureCode_1` - `5`       | `STRING`                | `NO`      | A procedure performed on the patient, and applicable to the line item (e.g., a HCPCS or CPT code).                                                                                       |
| `item_procedureCodeSystem_1` - `5` | `STRING`                | `COND`    | **MUST** be provided if `item_procedureCode_[k]` is given. See: "[Code Systems](#code-systems)"                                                                                            |
| `item_procedureCodeDesc_1` - `5`   | `STRING`                | `COND`    | **MUST** be provided if `item_procedureCode_[k]` is given. A brief human-readable description of the procedure, which will be displayed to end-users of the Empara application.          |
| `item_productOrServiceCode`        | `STRING`                | `NO`      | The product, service, drug, or other billing code for the item.                                                                                                                          |
| `item_productOrServiceCodeSystem`  | `STRING`                | `COND`    | **MUST** be provided if `item_productOrServiceCode` is given. See: "[Code Systems](#code-systems)"                                                                                         |
| `item_productOrServiceCodeDesc`    | `STRING`                | `COND`    | **MUST** be provided if `item_productOrServiceCode` is given. A brief human-readable description of the product/service, which will be displayed to end-users of the Empara application. |
| `item_modifierCode`                | `STRING`                | `NO`      | Product or service billing modifier.                                                                                                                                                     |
| `item_modifierCodeSystem`          | `STRING`                | `COND`    | **MUST** be provided if `item_modifierCode` is given. See: "[Code Systems](#code-systems)"                                                                                                 |
| `item_modifierCodeDesc`            | `STRING`                | `COND`    | **MUST** be provided if `item_modifierCode` is given. A brief human-readable description of the modifier, which will be displayed to end-users of the Empara application.                |
| `item_servicedAtDate`              | `DATE`                  | `COND`    | The date the service or product was supplied, performed, or completed. **MUST** be provided if `item_servicedAtPeriodStart` is not given.                                                |
| `item_servicedAtPeriodStart`       | `DATETIME`              | `COND`    | The start date of when the service period. **MUST** be provided if `item_servicedAtDate` is not given.                                                                                   |
| `item_servicedAtPeriodEnd`         | `DATETIME`              | `COND`    | The end date of when the service period. **MUST** be provided if `item_servicedAtPeriodStart` start is given                                                                             |
| `item_locationCode`                | `STRING`                | `NO`      | The code for the place of service (e.g., a CMS POS code).                                                                                                                                |
| `item_locationCodeSystem`          | `STRING`                | `COND`    | **MUST** be provided if `item_locationCode` is given. See: "[Code Systems](#code-systems)"                                                                                                 |
| `item_locationCodeDesc`            | `STRING`                | `COND`    | **MUST** be provided if `item_locationCode` is given. A brief human-readable description of the location code, which will be displayed to users on the Empara application.               |
| `item_locationAddressLine1`        | `STRING`                | `NO`      | The first address line of where the product was supplied, or service rendered.                                                                                                           |
| `item_locationAddressLine2`        | `STRING`                | `NO`      | The second address line of where the product was supplied, or service rendered.                                                                                                          |
| `item_locationCity`                | `STRING`                | `COND`    | **MUST** be provided if `item_locationAddressLine1` is given. The city of the address.                                                                                                   |
| `item_locationState`               | `STRING`                | `COND`    | **MUST** be provided if `item_locationAddressLine1` is given. The state of the address.                                                                                                  |
| `item_locationPostalCode`          | `STRING`                | `COND`    | **MUST** be provided if `item_locationAddressLine1` is given. The postal code of the address.                                                                                            |
| `item_quantity`                    | `INTEGER`               | `NO`      | Count of units for the product or service.                                                                                                                                               |
| `item_unitOfMeasure`               | `STRING`                | `NO`      | The unit of measure for the item quantity.                                                                                                                                               |
| `item_unitPrice`                   | `MONETARY`              | `NO`      | Fee, charge, or cost per item.                                                                                                                                                           |
| `item_patientPaid`                 | `MONETARY`              | `NO`      | The amount already paid by the patient to the provider.                                                                                                                                  |
| `item_priceFactor`                 | `REAL`                  | `NO`      | The price scaling factor. Describes discounts and surcharges.                                                                                                                            |
| `item_tax`                         | `MONETARY`              | `NO`      | The total of taxes applicable to the product or service.                                                                                                                                 |
| `item_totalCost`                   | `MONETARY`              | `YES`     | The total cost amount of the line item.                                                                                                                                                  |

### Claim Responses

Claim responses represent a payer's adjudication or authorization response to a claim. For any claim referenced in a claim response, a claim request **SHOULD** already have been sent. A claim request **MAY** be provided without a corresponding claim response.

#### File Encoding

Claim response files **SHOULD** be formatted as tab-separated values (TSV format). If tab delimiters cannot be used, an alternative delimited **MAY** be agreed upon during implementation. If comma delimiters are used, the file extension **SHOULD** be `.csv`. For other delimiters (such as a pipe `|`, for instance), the extension **SHOULD** be `.txt`.

The first line of the file **MUST** be a header row which indicates the order of the fields as columns.

Files **SHOULD** use a UTF-8 character encoding without a byte order mark (BOM) and **SHOULD** use Unix line endings (line feeds only). Any system limitations preventing adherence to this format **MUST** be communicated during implementation.

#### Row Definition

A single row in the claim response file represents the adjudication or authorization for a single line item on a submitted claim. Information for the entire response (header-level data) is duplicated on each row for that claim. The header values **MUST** be identical across all line items for a given response. To change a response-level value, the sender **MUST** send all line items for that response with the updated header values.

The following fields constitute the composite primary key of a claim response item:

- `memberGroupId`
- `groupPlanId`
- `subscriberId`
- `memberId`
- `claimResponseId`
- `seq`

All of these fields except `seq` describe the primary key of the claim response entity.

Alternatively, the sender **MAY** designate their own single-value primary key for the claim response using the `_sourceClaimResponseKey` field. The value for this field **MUST** be the exact primary key used in the source system's database and **MUST NOT** be a business identifier or claim number.

If a value for `_sourceClaimResponseKey` is not provided, any change to the fields which are part of the composite primary key (`memberGroupId`, `groupPlanId`, `subscriberId`, `memberId`, or `claimResponseId`) will be processed as a new claim response record. Therefore, if any of these values must be changed, all rows sent previously for the claim response **MUST** be nullified (see "[Data Nullification](#data-nullification)") before or when the updated values are sent.

#### File Schema

The "Required?" column specifies the minimum fields necessary for schema validation and successful data integration. Fields with a `NO` requirement level may nevertheless be required under the terms of the governing business agreement. The sender **MUST** consult both this technical specification and the requirements outlined by the business agreement to determine the complete set of required data elements before transmission.

Columns corresponding to fields with a `NO` requirement level **MAY** be excluded from the file if they contain no data for every row. However, once a column for a field is included in a transmitted file, it **MUST** be present in all subsequent files, even if the field is no longer populated.

To enhance readability and reduce redundancy, some fields in the schema table are presented in a condensed format. For example, a field named "`item_note_1` - `4`," actually represents four distinct fields, with names: `item_note_1`, `item_note_2`, `item_note_3`, and `item_note_4`. All fields presented with this condensed representation share the same data type, requirement level, and definition. This denormalization is necessary for the schema to adhere to a structured format. As an alternative, a JSON Lines schema is available upon request for senders who would prefer a semi-structured format.

| Name                                                             | Data Type               | Required? | Description                                                                                                                          |
|---------------------------------------------------------------- |----------------------- |--------- |------------------------------------------------------------------------------------------------------------------------------------ |
| `_sourceClaimResponseKey`                                        | `STRING`                | `NO`      | The primary key of the claim entity (header) within the partner's system. This **MUST NOT** be the claim id/number.                  |
| `claimId`                                                        | `STRING`                | `YES`     | The unique business identifier for the claim by the partner. Usually the claim number.                                               |
| `claimNo`                                                        | `STRING`                | `YES`     | The claim number for tracking, recognized by all parties using the claim.                                                            |
| `claimResponseId`                                                | `STRING`                | `NO`      | The unique business identifier for the claim response by the partner. Should be given if claims may have multiple response entities. |
| `seq`                                                            | `NATURAL`               | `YES`     | The identifier for the line item, aka the line number.                                                                               |
| `createdAt`                                                      | `DATETIME`              | `YES`     | The date and time that the claim was drafted.                                                                                        |
| `memberGroupId`                                                  | `STRING`                | `YES`     | The ID of the member group to which the member/patient belongs. Must match the group ID in the eligibility file exactly.             |
| `groupPlanId`                                                    | `STRING`                | `YES`     | The ID of the benefit plan covering the service. Must match the plan ID in the eligibility file exactly.                             |
| `subscriberId`                                                   | `STRING`                | `YES`     | The ID of the subscriber of the plan covering the member/patient. Must match the subscriber ID in the eligibility file exactly.      |
| `memberId`                                                       | `STRING`                | `YES`     | The ID of the member receiving service/goods. Must match the member ID in the eligibility file exactly.                              |
| `point`                                                          | `DATETIME`              | `YES`     | A point in time used to determine the period of coverage the claim falls under. Usually the date of service.                         |
| `header_type`                                                    | `ENUM(Claim Type)`      | `YES`     | See: "[Claim Type](#claim-type)"                                                                                                     |
| `header_resourceStatus`                                          | `ENUM(Resource Status)` | `YES`     | See: "[Resource Status](#resource-status)"                                                                                                |
| `header_use`                                                     | `ENUM(Use)`             | `YES`     | See: "[Use](#use)"                                                                                                            |
| `header_workflowStatusCode`                                      | `STRING`                | `NO`      | See: "[Workflow Status](#workflow-status)"                                                                                                |
| `header_workflowStatusDisplayName`                               | `STRING`                | `COND`    | **MUST** be provided if `header_workflowStatusCode` is given. See: "[Workflow Status](#workflow-status)"                                  |
| `header_workflowStatusDesc`                                      | `STRING`                | `COND`    | **MUST** be provided if `header_workflowStatusCode` is given. See: "[Workflow Status](#workflow-status)"                                  |
| `header_disposition`                                             | `STRING`                | `NO`      | A human readable description of the status of the adjudication.                                                                      |
| `header_outcome`                                                 | `ENUM(Outcome)`         | `YES`     | See: "[Outcome](#outcome)"                                                                                                        |
| `header_decision`                                                | `ENUM(Decision)`        | `COND`    | **MUST** be provided if `header_outcome` is `complete` or `partial`. See: "[Decision](#decision)"                                  |
| `header_payeeType`                                               | `ENUM(Payee Type)`      | `NO`      | See: "[Payee Type](#payee-type)"                                                                                                     |
| `header_paymentTypeCode`                                         | `STRING`                | `NO`      | Code for the type of payment (e.g. complete, partial, etc.)                                                                          |
| `header_paymentTypeCodeSystem`                                   | `STRING`                | `COND`    | **MUST** be provided if `header_paymentTypeCode` is given. See: "[Code Systems](#code-systems)"                                        |
| `header_paymentTypeCodeDesc`                                     | `STRING`                | `COND`    | **MUST** be provided if `header_paymentTypeCode` type code is given. See: "[Code Systems](#code-systems)"                              |
| `header_paymentAdjustment`                                       | `MONETARY`              | `NO`      | Total adjustments to payment unrelated to claim's adjudication.                                                                      |
| `header_paymentAdjustmentReasonCode`                             | `STRING`                | `NO`      | Code for the reason for payment adjustment.                                                                                          |
| `header_paymentAdjustmentReasonCodeSystem`                       | `STRING`                | `COND`    | **MUST** be provided if `header_paymentAdjustmentReasonCode` is given. See: "[Code Systems](#code-systems)"                            |
| `header_paymentAdjustmentReasonCodeDesc`                         | `STRING`                | `COND`    | **MUST** be provided if `header_paymentAdjustmentReasonCode` is given. See: "[Code Systems](#code-systems)"                            |
| `header_paymentDate`                                             | `DATE`                  | `NO`      | Estimated or actual date the payment will be issued.                                                                                 |
| `header_paymentAmount`                                           | `MONETARY`              | `NO`      | Amount payable after adjustment.                                                                                      |
| `header_paymentIdentifier`                                       | `STRING`                | `NO`      | Business identifier for the payment                                                                                                  |
| `item_note_1` - `4`                                              | `STRING`                | `NO`      | Note concerning adjudication. May be displayed to humans.                                                                            |
| `item_networkCode`                                               | `STRING`                | `NO`      | Code indicating if the provider is in network. See: "[Code systems](#code-systems)"                                                    |
| `item_networkCodeSystem`                                         | `STRING`                | `COND`    | **MUST** be provided if `item_networkCode` is given.                                                                                 |
| `item_networkCodeDesc`                                           | `STRING`                | `COND`    | **MUST** be provided if `item_networkCode` is given.                                                                                 |
| `item_reviewOutcomeDecision`                                     | `ENUM(Decision)`        | `COND`    | See: "[Decision](#decision)"                                                                                                       |
| `item_reviewOutcomeReasonCode_1` - `3`                           | `STRING`                | `NO`      | Reason code for the review outcome.                                                                                                  |
| `item_reviewOutcomeReasonCodeSystem_1` - `3`                     | `STRING`                | `COND`    | **MUST** be provided if `item_reviewOutcomeReasonCode_[k]` is given. See: "[Code Systems](#code-systems)"                              |
| `item_reviewOutcomeReasonCodeDesc_1` - `3`                       | `STRING`                | `COND`    | **MUST** be provided if `item_reviewOutcomeReasonCode_[k]` is given. See: "[Code Systems](#code-systems)"                              |
| `item_adjudicationSubmittedAmount`                               | `MONETARY`              | `COND`    | Total amount submitted for the line item. Required if claim processing is complete.                                                  |
| `item_adjudicationCopayAmount`                                   | `MONETARY`              | `NO`      | Member copayment for the line item.                                                                                                  |
| `item_adjudicationCopayReasonCode`                               | `STRING`                | `NO`      | Reason code for copay adjudication.                                                                                                  |
| `item_adjudicationCopayReasonCodeSystem`                         | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationCopayReasonCode` is given. See "[Code Systems](#code-systems)"                               |
| `item_adjudicationCopayReasonDesc`                               | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationCopayReasonCode` is given. See "[Code Systems](#code-systems)"                               |
| `item_adjudicationCoinsuranceAmount`                             | `MONETARY`              | `NO`      | Member coinsurance for the line item.                                                                                                |
| `item_adjudicationCoinsuranceReasonCode`                         | `STRING`                | `NO`      | Reason code for the coinsurance adjudication.                                                                                        |
| `item_adjudicationCoinsuranceReasonCodeSystem`                   | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationCoinsuranceReasonCode` is given. See "[Code Systems](#code-systems)"                         |
| `item_adjudicationCoinsuranceReasonDesc`                         | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationCoinsuranceReasonCode` is given. See "[Code Systems](#code-systems)"                         |
| `item_adjudicationEligibleAmount`                                | `MONETARY`              | `NO`      | The amount of the charge which is considered for adjudication. a.k.a, the allowed amount.                                            |
| `item_adjudicationEligibleReasonCode`                            | `STRING`                | `NO`      | Reason code for the eligible amount adjudication.                                                                                    |
| `item_adjudicationEligibleReasonCodeSystem`                      | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationEligibleReasonCode` is given. See "[Code Systems](#code-systems)"                            |
| `item_adjudicationEligibleReasonDesc`                            | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationEligibleReasonCode` is given. See "[Code Systems](#code-systems)"                            |
| `item_adjudicationDeductibleAmount`                              | `MONETARY`              | `NO`      | Amount deducted from eligible amount prior to adjudication.                                                                          |
| `item_adjudicationDeductibleReasonCode`                          | `STRING`                | `NO`      | Reason code for the deductible adjudication.                                                                                         |
| `item_adjudicationDeductibleReasonCodeSystem`                    | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationDeductibleReasonCode` is given. See "[Code Systems](#code-systems)"                          |
| `item_adjudicationDeductibleReasonDesc`                          | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationDeductibleReasonCode` is given. See "[Code Systems](#code-systems)"                          |
| `item_adjudicationUnallocatedDeductibleAmount`                   | `MONETARY`              | `NO`      | Amount of deductible which could not be allocated to other line items.                                                               |
| `item_adjudicationUnallocatedDeductibleReasonCode`               | `STRING`                | `NO`      | Reason code for the unallocated deductible adjudication.                                                                             |
| `item_adjudicationUnallocatedDeductibleReasonCodeSystem`         | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationUnallocatedDeductibleReasonCode` is given. See "[Code Systems](#code-systems)"               |
| `item_adjudicationUnallocatedDeductibleReasonDesc`               | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationUnallocatedDeductibleReasonCode` is given. See "[Code Systems](#code-systems)"               |
| `item_adjudicationTaxAmount`                                     | `MONETARY`              | `NO`      | Amount of tax.                                                                                                                       |
| `item_adjudicationTaxReasonCode`                                 | `STRING`                | `NO`      | Reason code for tax adjudication.                                                                                                    |
| `item_adjudicationTaxReasonCodeSystem`                           | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationTaxReasonCode` is given. See "[Code Systems](#code-systems)"                                 |
| `item_adjudicationTaxReasonDesc`                                 | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationTaxReasonCode` is given. See "[Code Systems](#code-systems)"                                 |
| `item_adjudicationBenefitAmount`                                 | `MONETARY`              | `NO`      | Amount payable under the coverage.                                                                                                   |
| `item_adjudicationBenefitReasonCode`                             | `STRING`                | `NO`      | Reason code for the benefit adjudication.                                                                                            |
| `item_adjudicationBenefitReasonCodeSystem`                       | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationBenefitReasonCode` is given. See "[Code Systems](#code-systems)"                             |
| `item_adjudicationBenefitReasonDesc`                             | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationBenefitReasonCode` is given. See "[Code Systems](#code-systems)"                             |
| `item_adjudicationMemberResponsibilityAmount`                    | `MONETARY`              | `NO`      | The amount the member is obligated to pay after adjudication.                                                                        |
| `item_adjudicationMemberResponsibilityReasonCode`                | `STRING`                | `NO`      |                                                                                                                                      |
| `item_adjudicationMemberResponsibilityReasonCodeSystem`          | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationMemberResponsibilityReasonCode` is given. See "[Code Systems](#code-systems)"                |
| `item_adjudicationMemberResponsibilityReasonDesc`                | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationMemberResponsibilityReasonCode` is given. See "[Code Systems](#code-systems)"                |
| `item_adjudicationProviderResponsibilityAmount`                  | `MONETARY`              | `NO`      | The amount the provider is obligated to pay after adjudication.                                                                      |
| `item_adjudicationProviderResponsibilityReasonCode`              | `STRING`                | `NO`      |                                                                                                                                      |
| `item_adjudicationProviderResponsibilityReasonCodeSystem`        | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationProviderResponsibilityReasonCode` is given. See "[Code Systems](#code-systems)"              |
| `item_adjudicationProviderResponsibilityReasonDesc`              | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationProviderResponsibilityReasonCode` is given. See "[Code Systems](#code-systems)"              |
| `item_adjudicationCOBResponsibilityAmount`                       | `MONETARY`              | `NO`      | The amount deferred to coordination of benefits.                                                                                     |
| `item_adjudicationCOBResponsibilityReasonCode`                   | `STRING`                | `NO`      |                                                                                                                                      |
| `item_adjudicationCOBResponsibilityReasonCodeSystem`             | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationCOBResponsibilityReasonCode`  is given. See "[Code Systems](#code-systems)"                  |
| `item_adjudicationCOBResponsibilityReasonDesc`                   | `STRING`                | `COND`    | **MUST** be provided if `item_adjudicationCOBResponsibilityReasonCode`  is given. See "[Code Systems](#code-systems)"                  |
| `item_errorCode_1`                                               | `STRING`                | `NO`      | Item processing error.                                                                                                               |
| `item_errorCodeSystem_1`                                         | `STRING`                | `COND`    | **MUST** be provided if `item_errorCode_1` is given. See "[Code Systems](#code-systems)"                                               |
| `item_errorDesc_1`                                               | `STRING`                | `COND`    | **MUST** be provided if `item_errorCode_1` is given. See "[Code Systems](#code-systems)"                                               |
| `header_adjudicationTotalSubmittedAmount`                        | `MONETARY`              | `COND`    | Total amount submitted for the line item. Required if claim processing is complete.                                                  |
| `header_adjudicationTotalCopayAmount`                            | `MONETARY`              | `NO`      | Member copayment for the line item.                                                                                                  |
| `header_adjudicationTotalCopayReasonCode`                        | `STRING`                | `NO`      |                                                                                                                                      |
| `header_adjudicationTotalCopayReasonCodeSystem`                  | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalCopayReasonCode` is given. See "[Code Systems](#code-systems)"                        |
| `header_adjudicationTotalCopayReasonDesc`                        | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalCopayReasonCode` is given. See "[Code Systems](#code-systems)"                        |
| `header_adjudicationTotalCoinsuranceAmount`                      | `MONETARY`              | `NO`      | Member coinsurance for the line item.                                                                                                |
| `header_adjudicationTotalCoinsuranceReasonCode`                  | `STRING`                | `NO`      |                                                                                                                                      |
| `header_adjudicationTotalCoinsuranceReasonCodeSystem`            | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalCoinsuranceReasonCode` is given. See "[Code Systems](#code-systems)"                  |
| `header_adjudicationTotalCoinsuranceReasonDesc`                  | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalCoinsuranceReasonCode` is given. See "[Code Systems](#code-systems)"                  |
| `header_adjudicationTotalEligibleAmount`                         | `MONETARY`              | `NO`      | The amount of the charge which is considered for adjudication. a.k.a, the allowed amount.                                            |
| `header_adjudicationTotalEligibleReasonCode`                     | `STRING`                | `NO`      |                                                                                                                                      |
| `header_adjudicationTotalEligibleReasonCodeSystem`               | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalEligibleReasonCode` is given. See "[Code Systems](#code-systems)"                     |
| `header_adjudicationTotalEligibleReasonDesc`                     | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalEligibleReasonCode` is given. See "[Code Systems](#code-systems)"                     |
| `header_adjudicationTotalDeductibleAmount`                       | `MONETARY`              | `NO`      | Amount deducted from eligible amount prior to adjudication.                                                                          |
| `header_adjudicationTotalDeductibleReasonCode`                   | `STRING`                | `NO`      |                                                                                                                                      |
| `header_adjudicationTotalDeductibleReasonCodeSystem`             | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalDeductibleReasonCode` is given. See "[Code Systems](#code-systems)"                   |
| `header_adjudicationTotalDeductibleReasonDesc`                   | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalDeductibleReasonCode` is given. See "[Code Systems](#code-systems)"                   |
| `header_adjudicationTotalUnallocatedDeductibleAmount`            | `MONETARY`              | `NO`      | Amount of deductible which could not be allocated to other line items.                                                               |
| `header_adjudicationTotalUnallocatedDeductibleReasonCode`        | `STRING`                | `NO`      |                                                                                                                                      |
| `header_adjudicationTotalUnallocatedDeductibleReasonCodeSystem`  | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalUnallocatedDeductibleReasonCode` is given. See "[Code Systems](#code-systems)"        |
| `header_adjudicationTotalUnallocatedDeductibleReasonDesc`        | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalUnallocatedDeductibleReasonCode` is given. See "[Code Systems](#code-systems)"        |
| `header_adjudicationTotalTaxAmount`                              | `MONETARY`              | `NO`      | Amount of tax.                                                                                                                       |
| `header_adjudicationTotalTaxReasonCode`                          | `STRING`                | `NO`      |                                                                                                                                      |
| `header_adjudicationTotalTaxReasonCodeSystem`                    | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalTaxReasonCode` is given. See "[Code Systems](#code-systems)"                          |
| `header_adjudicationTotalTaxReasonDesc`                          | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalTaxReasonCode` is given. See "[Code Systems](#code-systems)"                          |
| `header_adjudicationTotalBenefitAmount`                          | `MONETARY`              | `NO`      | Amount payable under the coverage.                                                                                                   |
| `header_adjudicationTotalBenefitReasonCode`                      | `STRING`                | `NO`      |                                                                                                                                      |
| `header_adjudicationTotalBenefitReasonCodeSystem`                | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalBenefitReasonCode` is given. See "[Code Systems](#code-systems)"                      |
| `header_adjudicationTotalBenefitReasonDesc`                      | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalBenefitReasonCode` is given. See "[Code Systems](#code-systems)"                      |
| `header_adjudicationTotalMemberResponsibilityAmount`             | `MONETARY`              | `NO`      | The amount the member is obligated to pay after adjudication.                                                                        |
| `header_adjudicationTotalMemberResponsibilityReasonCode`         | `STRING`                | `NO`      |                                                                                                                                      |
| `header_adjudicationTotalMemberResponsibilityReasonCodeSystem`   | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalMemberResponsibilityReasonCode` is given. See "[Code Systems](#code-systems)"         |
| `header_adjudicationTotalMemberResponsibilityReasonDesc`         | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalMemberResponsibilityReasonCodeSystem` is given. See "[Code Systems](#code-systems)"   |
| `header_adjudicationTotalProviderResponsibilityAmount`           | `MONETARY`              | `NO`      | The amount the provider is obligated to pay after adjudication.                                                                      |
| `header_adjudicationTotalProviderResponsibilityReasonCode`       | `STRING`                | `NO`      |                                                                                                                                      |
| `header_adjudicationTotalProviderResponsibilityReasonCodeSystem` | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalProviderResponsibilityReasonCode` is given. See "[Code Systems](#code-systems)"       |
| `header_adjudicationTotalProviderResponsibilityReasonDesc`       | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalProviderResponsibilityReasonCode` is given. See "[Code Systems](#code-systems)"       |
| `header_adjudicationTotalCOBResponsibilityAmount`                | `MONETARY`              | `NO`      | The amount deferred to coordination of benefits.                                                                                     |
| `header_adjudicationTotalCOBResponsibilityReasonCode`            | `STRING`                | `NO`      |                                                                                                                                      |
| `header_adjudicationTotalCOBResponsibilityReasonCodeSystem`      | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalCOBResponsibilityReasonCode` is given. See "[Code Systems](#code-systems)"            |
| `header_adjudicationTotalCOBResponsibilityReasonDesc`            | `STRING`                | `COND`    | **MUST** be provided if `header_adjudicationTotalCOBResponsibilityReasonCode` is given. See "[Code Systems](#code-systems)"            |

#### Workflow Status

The workflow status is an optional field that provides a more granular status of the claim within the adjudication process than the outcome and decision fields. The purpose is to convey information unique to the data provider, such as status codes from a previously used claims portal. The code **SHOULD** be a value from the sender's source system.

## Addendums

### Identifier Systems

Identifier systems specify the issuing authority for a given identifier (e.g., a government entity). A canonical URI is provided to specify the system.

| Name                                                           | Relevant Fields      | URI                                                                       |
|-------------------------------------------------------------- |-------------------- |------------------------------------------------------------------------- |
| United States National Provider Identifier (NPI)               | Provider Identifiers | <http://terminology.hl7.org/NamingSystem/npi>                             |
| United States Employer Identification Number (EIN, TIN, USEIN) | Provider Identifiers | <http://terminology.hl7.org/NamingSystem/USEIN>                           |
| NCPDP Provider Identification Number (NCPDP)                   | Provider Identifiers | <http://terminology.hl7.org/CodeSystem/NCPDPProviderIdentificationNumber> |

The sender **MUST NOT** send a system URI unless either:

1. The URI belongs to an [HL7 published artifact](https://terminology.hl7.org/6.5.0/artifacts.html#4) as of the HL7 terminology release v6.5.0
2. The URI is in the HL7 OID registry.
3. The URI is in the table above.
4. The URI has been minted by Empara, and given to the sender.

The sender **MAY** request a canonical URI be minted for their identifier system if it is not recognized by HL7, and they cannot adapt to an HL7 identifier system.

### Code Systems

Code systems specify the standard or issuing authority for a given code.

| Name                                     | Relevant Fields                          | URI                                                          |
|---------------------------------------- |---------------------------------------- |------------------------------------------------------------ |
| ICD-10                                   | Diagnosis Codes                          | <http://hl7.org/fhir/sid/icd-10>                             |
| HCFA Procedure Codes (HCPCS)             | Procedure Codes, Product or Service Code | <http://terminology.hl7.org/CodeSystem/HCPCS-all-codes>      |
| Current Procedural Terminology (CPT®)    | Procedure Codes                          | <http://www.ama-assn.org/go/cpt>                             |
| Network Type Codes                       | Network Code                             | <http://terminology.hl7.org/CodeSystem/benefit-network>      |
| FHIR Claim Decision Reason Codes         | Outcome Reason Codes                     | <http://hl7.org/fhir/claim-outcome>                          |
| X12 Service Review Decision Reason Codes | Outcome Reason Codes                     | <https://x12.org/codes/service-review-decision-reason-codes> |
| FHIR Adjudication Reason Codes           | Adjudication Reason Codes                | <http://terminology.hl7.org/CodeSystem/adjudication-reason>  |
| X12 Claim Adjustment Reason Codes        | Adjudication Reason Codes                | <https://x12.org/codes/claim-adjustment-reason-codes>        |
| FHIR Adjudication Error Codes            | Error Code                               | <http://terminology.hl7.org/CodeSystem/adjudication-error>   |
| X12 Error Reason Codes                   | Error Code                               | <https://x12.org/codes/error-reason-codes>                   |

The sender **MUST NOT** send a system URI unless either:

1. The URI belongs to an [HL7 published artifact](https://terminology.hl7.org/6.5.0/artifacts.html#3) as of the HL7 terminology release v6.5.0
2. The URI is in the HL7 OID registry.
3. The URI is in the table above.
4. The URI has been minted by Empara, and given to the sender.

The sender **MAY** request a canonical URI be minted for their code system if it is not recognized by HL7, and they cannot adapt to an HL7 code system.

Because Empara does not (at present) maintain a comprehensive internal library of all possible code descriptions, the sender **MUST** supply a human-readable description for every code provided, using the corresponding `...Desc` field in the schemas.

### Data Nullification

If a claim or claim response is sent in error and cannot be directly updated, all previously sent line items for that claim **MUST** be resent with the `header_resourceStatus` set to `entered-in-error`. This also applies when a claim number or member ID is changed.
