# Historical Member ID Supplemental File Specification

**Inbound | Supplemental Format**  
Transmission from Partner Organization to Opyn Engage

## Revision History

| Version | Last Updated    |
| --------| ----------------|
| 1.0     | August 6, 2026  |

## Overview

The Historical Member ID file is an **optional** supplemental file that data providers may supply to backfill `memberId` and `subscriberId` values that predate, or fall outside of, the current eligibility feed. This is useful if historical claim data will be provided that references these older `memberId`'s. This file is typically provided once during initial implementation, but may be re-sent when onboarding a new member group or when the need for historical data is identified.

## File Structure

### Format Rules

- Fields are required to be tab delimited.
- A header row must be present.

### File Name Convention

Files must be tab-delimited with either a `.tsv` or `.txt` extension. Files sent by the provider must follow one of these naming conventions:

- **File per Member Group**  
  - **Filename:** `<EligibilityProviderCode>_<MemberGroupId>_hist_member_ids_YYYYMMDD.tsv`

- **File per Eligibility Provider (all groups)**  
  - A single file contains historical ID records for all member groups.
  - **Filename:** `<EligibilityProviderCode>_hist_member_ids_YYYYMMDD.tsv`

### Transmission

Files should be placed in the existing `toEmpara/eligibility/` SFTP folder alongside standard eligibility files. No new drop folder is required.

## Data Schema

Each Historical Member ID file must contain the following fields.

| Column | Type | Required | Min | Max | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `planAdministratorId` | STRING | Conditional | | | The mutually agreed upon Plan Administrator ID. Only required when sending data for multiple plan administrators in the file. |
| `memberGroupId` | STRING | N | | | Identifier for the member group/employer as assigned by the eligibility provider. When provided, group-scoped keys are also generated. |
| `memberId` | STRING | Y | | | The historical member ID as previously assigned by the eligibility provider. |
| `subscriberId` | STRING | N | | | The historical subscriber ID. Required when the member was a dependent and the subscriber ID differs from `memberId`. |
| `ssn` | STRING | Conditional | 9 | 9 | The member's SSN, without dashes. Used to match to an existing member record. Required for the subscriber when `personNumber` is not provided. Always optional for dependents. |
| `subscriberSsn` | STRING | Conditional | 9 | 9 | The subscriber's SSN. Used as part of the two-step dependent fallback match. |
| `personNumber` | STRING | Conditional | | | Immutable identifier for the person associated to this eligibility record. An alternative to `ssn` for providers that use person-number-based eligibility. Required for the subscriber when `ssn` is not provided. Always optional for dependents. Must be unique within the scope of the member group (`memberGroupId`) |
| `subscriberPersonNumber` | STRING | Conditional | | | Immutable identifier for the person designated as the subscriber on this eligibility record. Required for the subscriber when `subscriberSsn` is not provided. Must be the same for all dependents associated with a subscriber. |
| `firstName` | STRING | Y | | | The member's first name. Required to disambiguate dependents during subscriber fallback matching. |
| `lastName` | STRING | Y | | | The member's last name. Required to disambiguate dependents during subscriber fallback matching. |
| `birthdate` | DATE | Y | | | ISO-8601 format (`YYYY-MM-DD`). Required to disambiguate dependents during subscriber fallback matching. |
| `personCode` | STRING | N | | | Identifier for a person within a family/contract (often a sequence number such as 01, 02, 03). When provided, used as a tie-breaker during dependent disambiguation. Should be consistent with the values used in the standard eligibility file. |
| `effectivePeriodStart` | TIMESTAMP | N | | | ISO-8601 UTC format (`YYYY-MM-DDTHH:MM:SSZ`). Start of the period for which this ID was valid. |
| `effectivePeriodEnd` | TIMESTAMP | N | | | ISO-8601 UTC format (`YYYY-MM-DDTHH:MM:SSZ`). End of the period for which this ID was valid. |

_\* At least one of `ssn` or `personNumber` must be provided per row._

## Business Rules

- A single file may contain records for multiple plan administrators.
- A single file may contain records for multiple member groups.
- A person may appear multiple times in the file — one row per historical ID pair.
- **At least one of `ssn` or `personNumber` must be provided per row.** Rows with neither will be quarantined.

## Examples

### Scenario 1: Single Plan Administrator using person-number-based eligibility

A Plan Administrator that uses person-number-based eligibility sends historical IDs for a subscriber and a dependent from group `G001`.

**Filename:** `acme_G001_hist_member_ids_20260101.tsv`

| Column | Row 1 (Subscriber) | Row 2 (Dependent) |
| :--- | :--- | :--- |
| `memberGroupId` | `G001` | `G001` |
| `memberId` | `OLD-M-1001` | `OLD-M-1002` |
| `subscriberId` | `OLD-S-5001` | `OLD-S-5001` |
| `ssn` | | |
| `subscriberSsn` | | |
| `personNumber` | `PN-9001` | |
| `subscriberPersonNumber` | `PN-9001` | `PN-9001` |
| `firstName` | `Jane` | `Alex` |
| `lastName` | `Smith` | `Smith` |
| `birthdate` | `1980-03-15` | `2010-07-22` |
| `personCode` | `01` | `02` |
| `effectivePeriodStart` | `2020-01-01T05:00:00Z` | `2019-06-01T05:00:00Z` |
| `effectivePeriodEnd` | `2022-12-31T05:00:00Z` | `2021-05-31T05:00:00Z` |

### Scenario 2: Multiple Plan Administrators using person-number-based eligibility

A Data Provider (`companyx`) sending data for multiple Plan Administrators that use person-number-based eligibility sends historical IDs for a subscriber and a dependent from groups `G001` & `G002`.

**Filename:** `companyx_hist_member_ids_20260101.tsv`

| Column | Row 1 (Subscriber) | Row 2 (Dependent) | Row 3 (Subscriber) | Row 4 (Dependent) |
| :--- | :--- | :--- | :--- | :--- |
| `planAdministratorId` | `acme` | `acme` | `contoso` | `contoso` |
| `memberGroupId` | `G001` | `G001` | `G102` | `G102` |
| `memberId` | `OLD-M-1001` | `OLD-M-1002` | `OLD-M-2001` | `OLD-M-2002` |
| `subscriberId` | `OLD-S-5001` | `OLD-S-5001` | `OLD-S-6001` | `OLD-S-6001` |
| `ssn` | | | | |
| `subscriberSsn` | | | | |
| `personNumber` | `PN-9001` | | `PN-8001` | `PN-8002` |
| `subscriberPersonNumber` | `PN-9001` | `PN-9001` | `PN-8001` | `PN-8001` |
| `firstName` | `Jane` | `Alex` | `Bob` | `Susan` |
| `lastName` | `Smith` | `Smith` | `Jones` | `Jones` |
| `birthdate` | `1980-03-15` | `2010-07-22` | `1980-03-15` | `2010-07-22` |
| `personCode` | `01` | `02` | `01` | `02` |
| `effectivePeriodStart` | `2020-01-01T05:00:00Z` | `2019-06-01T05:00:00Z` | `2020-01-01T05:00:00Z` | `2019-06-01T05:00:00Z` |
| `effectivePeriodEnd` | `2022-12-31T05:00:00Z` | `2021-05-31T05:00:00Z` | `2020-01-01T05:00:00Z` | `2019-06-01T05:00:00Z` |

### Scenario 3: Single Plan Administrator using SSN-based eligibility

A provider that uses SSN-based eligibility sends a historical ID for a subscriber from group `L001`.

**Filename:** `northwind_L001_hist_member_ids_20260101.tsv`

| Column | Row 1 |
| :--- | :--- |
| `planAdministratorId` | `northwind` |
| `memberGroupId` | `L001` |
| `memberId` | `OLD-M-2001` |
| `subscriberId` | `OLD-S-6001` |
| `ssn` | `123456789` |
| `subscriberSsn` | `123456789` |
| `personNumber` | |
| `subscriberPersonNumber` | |
| `firstName` | `Dan` |
| `lastName` | `Jump` |
| `birthdate` | `1974-01-01` |
| `personCode` | |
| `effectivePeriodStart` | `2018-01-01T05:00:00Z` |
| `effectivePeriodEnd` | `2020-12-31T05:00:00Z` |
