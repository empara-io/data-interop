# Accumulator EDI Specification

**Inbound | Structured Format**  
Transmission from Partner Organization to Empara

## Revision History

| Version | Last Updated    |
| --------| ----------------|
| 1.0     | June 12, 2024   |

## Overview

The accumulator file is used to communicate the amounts that have accrued toward a member's obligations and protections, such as their progress towards a deductible or out-of-pocket limit. This document provides details on the required file structure and data schema.

## File Structure

### Format and Rules

- There should be one member group per file.
- Fields are required to be tab delimited.
- A header row must be part of the file.

### File Name Convention

Empara will provide the designated 'Accumulator ProviderCode' to be used in the file name during the implementation process. Files submitted by the provider must adhere to the following naming convention:

`<Accumulator ProviderCode>_<MemberGroupId>_acum_YYYYMMDD.tsv`

## Transmission

Empara offers a secure file transfer protocol (SFTP) site for file exchanges. The necessary sign-in credentials will be securely provided during implementation. The SFTP site is set up by default with specific drop folders for eligibility, accumulators, and claims.

Below is an example of the SFTP site's structure:

```text
SFTP Root Folder
|- fromEmpara
|- toEmpara
   |- accumulators
      |- sample_group123_acum_20230101.tsv
      |- sample_group123_acum_20230102.tsv
      |- sample_group123_acum_20230103.tsv
      |- sample_group978_acum_20230101.tsv
      |- sample_group978_acum_20230102.tsv
      |- sample_group978_acum_20230103.tsv
   |- claims
   |- eligibility
```

## Data Schema

Each accumulator file must include the following fields:

| Column | Type | Required | Min | Max | Range of Values | Description |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **subscriberld** | STRING | N | | | | The ID of the subscriber with whom the member is associated. |
| **memberld** | STRING | Y | | | | The ID of the member to whom the accumulator is applied. |
| **personCode** | STRING | N | | | | The person code of the member. |
| **memberGroupld** | STRING | Y | | | | The identifier for the member group. |
| **groupPlanld** | STRING | Y | | | | The identifier for the benefit plan. |
| **lastName** | STRING | N | | | | The last name of the member. |
| **firstName** | STRING | N | | | | The first name of the member. |
| **transactionDate** | DATE | N | | | | If not provided, this is inferred from the file name. |
| **type** | STRING | Y | | | `familyDeductible`, `familyMaxPay`, `individualDeductible`, `individualMaxPay` | The type of accumulator. |
| **coverageType** | STRING | Y | | | `medical`, `pharmacy` | The type of coverage. |
| **applicability** | STRING | N | | | `inNetwork`, `outOfNetwork` | The network to which the accumulator applies. If no value is supplied, it implies universal applicability. |
| **value** | DECIMAL (2) | Y | | | | A monetary value with two digits of precision. |
