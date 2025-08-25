# Accumulator EDI Specification

**Inbound | Structured Format**  
Transmission from Partner Organization to Empara

## Revision History

| Version | Last Updated    |
|---------|-----------------|
| 1.0     | June 12, 2024   |
| 1.1     | August 22, 2025 |

## Overview

The accumulator file is used to communicate the amounts that have accrued toward a member's obligations and protections, such as their progress towards a deductible or out-of-pocket limit. This document provides details on the required file structure and data schema.

## Data Exchange

All files must be exchanged via the secure file transfer protocol (SFTP). It is recommended to use an SFTP service hosted by Empara as the exchange point, but connections to a partner-hosted SFTP server can be accommodated if necessary.

### SFTP Path Structure

Below is an example of the SFTP site's structure:

```text
.
├── fromEmpara/
│   ├── ...
└── toEmpara/
    ├── accumulators/
    │   ├── [prefix]_accum_[YYYYMMDD].tsv
    │   └── ...
    └── ...
```

The directory relevant to this specification is `toEmpara/accumulators`.

### File Chunking

If a file is expected to exceed 100MB or 1 million rows, it is recommended to split the file into multiple parts, or "chunks":

```text
.
├── fromEmpara/
│   ├── ...
└── toEmpara/
    ├── accumulators/
    │   ├── [prefix]_accum_[YYYYMMDD]_01.tsv
    │   ├── [prefix]_accum_[YYYYMMDD]_02.tsv
    │   ├── ...
    │   ├── [prefix]_accum_[YYYYMMDD]_[n].tsv
    │   ├── [prefix]_accum_[YYYYMMDD].ready
    │   └── ...
    └── ...
```

If the sender chooses to split files, this must be communicated to Empara beforehand, ideally during implementation. The sender must adopt one convention for file submission (either single or chunked files) and apply it consistently. A mix of both conventions must not be used.

For split files, a .ready file with the same base name as the split files (without the _[n] index) must be sent to indicate that all file parts have been transmitted and are ready for processing. This file should be empty, as its contents are not read. Empara's pipelines scans each directory once per day; the scan concludes for the day once the first file (or .ready file) is detected.

#### Ad-hoc File Exchange

For file submissions that occur outside of the regular schedule, such as for error corrections or historical data, the following convention should be used:

```text
.
├── fromEmpara/
│   ├── ...
└── toEmpara/
    ├── accumulators/
    │   ├── [prefix]_accum_[YYYYMMDD]_adhoc_01.tsv
    │   ├── [prefix]_accum_[YYYYMMDD]_adhoc_01_reason.txt
    │   ├── ...
    │   ├── [prefix]_accum_[YYYYMMDD]_adhoc_[n].tsv
    │   ├── [prefix]_accum_[YYYYMMDD]_adhoc_[n]_reason.txt
    │   └── ...
    └── ...
```

The number in the file name is a counter to accommodate multiple ad-hoc files on the same date. A corresponding _reason.txt file SHOULD be provided to briefly describe the reason for the ad-hoc submission. A simple description is acceptable, though detailed explanations are preferred to support auditing and data lineage.

If ad-hoc files need to be split, the following convention SHOULD be applied:

```text
.
├── fromEmpara/
│   ├── ...
└── toEmpara/
    ├── accumulators/
    │   ├── [prefix]_accum_[YYYYMMDD]_adhoc_01_part_01.tsv
    │   ├── [prefix]_accum_[YYYYMMDD]_adhoc_01_part_02.tsv
    │   ├── ...
    │   ├── [prefix]_accum_[YYYYMMDD]_adhoc_01_part_[m].tsv
    │   ├── [prefix]_accum_[YYYYMMDD]_adhoc_01_reason.txt
    │   ├── ...
    │   ├── [prefix]_accum_[YYYYMMDD]_adhoc_[n]_part_[k].tsv
    │   ├── [prefix]_accum_[YYYYMMDD]_adhoc_[n]_reason.txt
    │   └── ...
    └── ...
```

A .ready file is not necessary for ad-hoc submissions, as they are not subject to automated processing. The sender MUST notify Empara directly via email whenever an ad-hoc file is submitted.

### Naming Conventions

The file naming convention is `[prefix]_[file_type]_[YYYYMMDD].tsv` for single files, and `[prefix]_[file_type]_[YYYYMMDD]_[k].tsv` for chunked files. The prefix is a unique identifier assigned to the trading partner by Empara during implementation. The [YYYYMMDD] portion MUST correspond to the date the file is transmitted.

For example, for a company named "Contoso Benefit Administrators (CBA)" with an assigned ID of contoso, the file names would be `contoso_accum_20250401.tsv` and `contoso_accum_20250401.tsv` These files should include data for all relevant member groups for the implementation.

If a trading partner has multiple clients with separate data environments, it is recommended to send separate files for each client to isolate client-specific data characteristics. In this case, the prefixes would be client-specific, such as `datasolutions_acme` and `datasolutions_vandelay`.

### Error Handling

Accumulators may not be processed at times, typically when failing to meet data requirements, or pass validation. The sender MUST communicate their preferred strategy for receiving acknowledgments and error reports during implementation.

### Miscellaneous Rules

Once a daily file or set of daily part files have been transmitted, it MUST NOT be overwritten. As the pipeline's daily scan concludes upon file detection, any subsequent modifications to a transmitted file on the same day will be ignored. To correct or enhance a submitted file, an ad-hoc file MUST be provided as outlined in the section "Ad-hoc File Exchange".

## File Format

### File Encoding

Files should be tab-separated values (TSV format). If tab delimiters cannot be used, an alternative delimiter may be agreed upon during implementation. If comma delimiters are used, the file extension should be `.csv`. For other delimiters (for instance, the pipe delimiter `|`), the extension should be `.txt`.

The first line of the file must be a header row which indicates the order of the fields as columns.

Files should use a UTF-8 character encoding without a the order mark (BOM) and should use Unix line endings (line feeds only). Any system limitations preventing adherence to this format must be communicated during implementation.

### Data Schema

Each accumulator file must include the following fields:

| Column            | Type        | Required | Min | Max | Range of Values                                                                         | Description                                                                                                |
|:------------------|:------------|:---------|:----|:----|:----------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------|
| `subscriberId`    | STRING      | N        |     |     |                                                                                         | The ID of the subscriber with whom the member is associated.                                               |
| `memberId`        | STRING      | Y        |     |     |                                                                                         | The ID of the member to whom the accumulator is applied.                                                   |
| `personCode`      | STRING      | N        |     |     |                                                                                         | The person code of the member.                                                                             |
| `memberGroupId`   | STRING      | Y        |     |     |                                                                                         | The identifier for the member group.                                                                       |
| `groupPlanId`     | STRING      | Y        |     |     |                                                                                         | The identifier for the benefit plan.                                                                       |
| `lastName`        | STRING      | N        |     |     |                                                                                         | The last name of the member.                                                                               |
| `firstName`       | STRING      | N        |     |     |                                                                                         | The first name of the member.                                                                              |
| `transactionDate` | DATE        | N        |     |     |                                                                                         | If not provided, this is inferred from the file name.                                                      |
| `type`            | STRING      | Y        |     |     | `familyDeductible`<br/>`familyMaxPay`<br/>`individualDeductible`<br/>`individualMaxPay` | The type of accumulator.                                                                                   |
| `coverageType`    | STRING      | Y        |     |     | `medical`<br/>`pharmacy`                                                                | The type of coverage.                                                                                      |
| `applicability`   | STRING      | N        |     |     | `inNetwork`<br/>`outOfNetwork`                                                          | The network to which the accumulator applies. If no value is supplied, it implies universal applicability. |
| `value`           | DECIMAL (2) | Y        |     |     |                                                                                         | A monetary value with two digits of precision.                                                             |
