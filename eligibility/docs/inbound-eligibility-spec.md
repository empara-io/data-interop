# Eligibility EDI Specification

**Inbound | Structured Format**  
Transmission from Partner Organization to Empara

## Revision History

| Version | Last Updated     |
| ------- | ---------------- |
| 1.4.1   | November 3, 2026 |
| 1.3     | October 2, 2024  |

**Eligibility Transmission from Partner Organization to Empara**

## Overview

The eligibility file is used to communicate member participation in one or more group benefit plans. The file should include data for all participating members, with separate records for the subscriber and any eligible dependents. This document outlines the file structure, data schema, and provides examples of supported business scenarios with their expected data values.

## File Structure

### Format Rules

- Fields are required to be tab delimited.
- A header row must be present.

### File Name Convention

During implementation, Empara will provide the 'EligibilityProviderCode' to be used in the file name. Eligibility files must be tab-delimited with either a .tsv or .txt extension. Files sent by the provider must follow one of these naming conventions:

- **File per Member Group (preferred)**
  - Eligibility records are divided into multiple files based on the member group.
  - The `MemberGroupId` must correspond to the identifier used in the Eligibility Provider's source system.
  - **Filename:** `<EligibilityProviderCode>_<MemberGroupId>_elig_YYYYMMDD.tsv`

- **File per Eligibility Provider**
  - A single file contains the eligibility records for all member groups.
  - **Filename:** `<EligibilityProviderCode>_elig_YYYYMMDD.tsv`

## Transmission

Empara offers a secure file transfer protocol (SFTP) site for file exchanges, with sign-in credentials provided during implementation. The SFTP site is configured with drop folders for eligibility, accumulators, and claims.

Below is an example of the SFTP site structure:

- **SFTP Root Folder**
  - `fromEmpara`
  - `toEmpara`
    - `accumulators`
    - `claims`
    - `eligibility`
      - `sample_group123_elig_20230101.tsv`
      - `sample_group123_elig_20230102.tsv`
      - `sample_group123_elig_20230103.tsv`
      - `sample_group978_elig_20230101.tsv`
      - `sample_group978_elig_20230102.tsv`
      - `sample_group978_elig_20230103.tsv`

## Data Schema

Each eligibility file must contain the following fields:

| Column                     | Type      | Required    | Min | Max | Range of Values                                                                         | Description                                                                                                                                                                                                                                                                                                      |
| :------------------------- | :-------- | :---------- | :-- | :-- | :-------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `planAdministratorId`      | STRING    | Conditional |     |     |                                                                                         | The mutually agreed upon Plan Administrator ID. Only required when sending data for multiple plan administrators in the file.                                                                                                                                                                                    |
| `memberGroupId`            | STRING    | Y           |     |     |                                                                                         | Unique identifier for the member group/employer as assigned by the eligibility provider.                                                                                                                                                                                                                         |
| `groupPlanId`              | STRING    | Y           |     |     |                                                                                         | Unique identifier for the benefit plan as assigned by the eligibility provider.                                                                                                                                                                                                                                  |
| `coverageTier`             | STRING    | N           |     |     | `subscriberOnly`, `subscriberAndFamily`, `subscriberAndSpouse`, `subscriberAndChildren` | Required only if the plan is divided into coverage tiers, and the implementation calls for the presence of this data.                                                                                                                                                                                            |
| `subscriberId`             | STRING    | Y           |     |     |                                                                                         | Unique identifier for the subscriber as assigned by the eligibility provider.                                                                                                                                                                                                                                    |
| `memberId`                 | STRING    | Y           |     |     |                                                                                         | Unique identifier for the member as assigned by the eligibility provider.                                                                                                                                                                                                                                        |
| `coverageStartDate`        | TIMESTAMP | Y           |     |     |                                                                                         | ISO-8601 UTC format (`YYYY-MM-DDTHH:MM:SSZ`).                                                                                                                                                                                                                                                                    |
| `coverageEndDate`          | TIMESTAMP | N           |     |     |                                                                                         | ISO-8601 UTC format (`YYYY-MM-DDTHH:MM:SSZ`).                                                                                                                                                                                                                                                                    |
| `medicareIsPrimary`        | BOOLEAN   | N           |     |     | `true`, `false`                                                                         |                                                                                                                                                                                                                                                                                                                  |
| `cobraIsActive`            | BOOLEAN   | N           |     |     | `true`, `false`                                                                         |                                                                                                                                                                                                                                                                                                                  |
| `ssn`                      | STRING    | Conditional | 9   | 9   |                                                                                         | The SSN of the member, without dashes. Required when `personNumber` is not provided.                                                                                                                                                                                                                             |
| `subscriberSsn`            | STRING    | Conditional | 9   | 9   |                                                                                         | The SSN of the subscriber, without dashes. Required when `subscriberPersonNumber` is not provided.                                                                                                                                                                                                               |
| `personNumber`             | STRING    | Conditional |     |     |                                                                                         | Immutable identifier for the person associated to this eligibility record. An alternative to `ssn` for providers that use person-number-based eligibility. Required when `ssn` is not provided. Must be unique within the scope of the member group (`memberGroupId`); it is not required to be globally unique. |
| `subscriberPersonNumber`   | STRING    | Conditional |     |     |                                                                                         | Immutable identifier for the person designated as the subscriber on this eligibility record. Required when `subscriberSsn` is not provided. Must be the same for all dependents associated with a subscriber.                                                                                                    |
| `firstName`                | STRING    | Y           |     |     |                                                                                         | The member's first given name.                                                                                                                                                                                                                                                                                   |
| `lastName`                 | STRING    | Y           |     |     |                                                                                         | The member's family name.                                                                                                                                                                                                                                                                                        |
| `middleName`               | STRING    | N           |     |     |                                                                                         | The member's middle name.                                                                                                                                                                                                                                                                                        |
| `suffix`                   | STRING    | N           |     |     |                                                                                         | The member's suffix. Sr., Jr., etc.                                                                                                                                                                                                                                                                              |
| `gender`                   | STRING    | N           |     |     | `male`, `female`, `other`, `unknown`                                                    | The member's gender.                                                                                                                                                                                                                                                                                             |
| `birthdate`                | DATE      | Y           |     |     |                                                                                         | ISO-8601 format (`YYYY-MM-DD`).                                                                                                                                                                                                                                                                                  |
| `relationshipToSubscriber` | STRING    | Y           |     |     | `self`, `spouse`, `child`, `other`                                                      | The relationship of the member to the subscriber.                                                                                                                                                                                                                                                                |
| `personCode`               | STRING    | N           |     |     |                                                                                         | Identifier for a person within a family/contract (often a sequence number such as 01, 02, 03). If provided, must be unique within the family. Assigned by the eligibility provider.                                                                                                                              |
| `addressLine1`             | STRING    | Y           |     |     |                                                                                         |                                                                                                                                                                                                                                                                                                                  |
| `addressLine2`             | STRING    | N           |     |     |                                                                                         |                                                                                                                                                                                                                                                                                                                  |
| `city`                     | STRING    | Y           |     |     |                                                                                         |                                                                                                                                                                                                                                                                                                                  |
| `state`                    | STRING    | Y           | 2   | 2   |                                                                                         |                                                                                                                                                                                                                                                                                                                  |
| `postalCode`               | STRING    | Y           | 5   | 10  |                                                                                         |                                                                                                                                                                                                                                                                                                                  |
| `country`                  | STRING    | N           | 2   | 2   |                                                                                         |                                                                                                                                                                                                                                                                                                                  |
| `homePhone`                | STRING    | N           | 9   | 10  |                                                                                         |                                                                                                                                                                                                                                                                                                                  |
| `cellPhone`                | STRING    | N           | 9   | 10  |                                                                                         |                                                                                                                                                                                                                                                                                                                  |
| `workPhone`                | STRING    | N           | 9   | 10  |                                                                                         |                                                                                                                                                                                                                                                                                                                  |
| `emailAddress`             | STRING    | N           | 3   | 255 |                                                                                         |                                                                                                                                                                                                                                                                                                                  |
| `memberGroupStartDate`     | TIMESTAMP | N           |     |     |                                                                                         | ISO-8601 UTC format (`YYYY-MM-DDTHH:MM:SSZ`).                                                                                                                                                                                                                                                                    |
| `memberGroupEndDate`       | TIMESTAMP | N           |     |     |                                                                                         | ISO-8601 UTC format (`YYYY-MM-DDTHH:MM:SSZ`).                                                                                                                                                                                                                                                                    |
| `transactionDate`          | TIMESTAMP | Y           |     |     |                                                                                         | The timestamp the record is sent/transacted. ISO-8601 UTC format (`YYYY-MM-DDTHH:MM:SSZ`).                                                                                                                                                                                                                       |
| `externalMemberId`         | STRING    | N           |     |     |                                                                                         | Unique identifier for the member which was provided by Empara in augmented eligibility implementations.                                                                                                                                                                                                          |
| `externalSubscriberId`     | STRING    | N           |     |     |                                                                                         | Unique identifier for the member which was provided by Empara in augmented eligibility implementations.                                                                                                                                                                                                          |

_\* At least one of `ssn` or `personNumber` must be provided per row. Rows with neither will be quarantined._

### Member Identification

Empara supports two methods of identifying a person on an eligibility record. A provider uses one method consistently for the duration of an implementation; the choice is agreed during implementation.

- **SSN-based.** Populate `ssn` and `subscriberSsn`. Leave `personNumber` and `subscriberPersonNumber` empty.
- **Person-number-based.** Populate `personNumber` and `subscriberPersonNumber`. Leave `ssn` and `subscriberSsn` empty.

When using person numbers:

- `personNumber` must be **unique within the member group (`memberGroupId`)**. It is not required to be unique across member groups, and Empara does not treat the same value in two different member groups as the same person.
- `personNumber` must be **immutable** for the life of the person's relationship with the member group. Reassigning a person number to a different individual, or issuing a new person number to the same individual, will be interpreted as a different person.
- `subscriberPersonNumber` must be identical on every record belonging to the same family, including the subscriber's own record.

`personCode` is not a person identifier. It is a position within a family/contract (`01`, `02`, `03`) and is only unique within a family.

### Multiple Plan Administrators

A provider that sends eligibility on behalf of more than one plan administrator must populate `planAdministratorId` on every record. Each value is agreed with Empara during implementation and must be registered before the first file containing it is sent; records carrying an unrecognized `planAdministratorId` cannot be matched and will be quarantined.

Providers sending data for a single plan administrator may omit the column.

## Business Rules / Supported Scenarios

The following are examples of supported eligibility scenarios.

### Scenario 1: New Enrollment

To enroll a member in a benefit plan, add an eligibility record to the next file with the plan's ID and the coverage start date.

**Example**

To enroll Dan Jump (member of Northwind Exports, group ID 6000) in plan 6041 starting 1/1, the file `sample_6000_elig_20231230.tsv` would look like this:

| Column                     | Row 1                  |
| :------------------------- | :--------------------- |
| `memberGroupId`            | `6000`                 |
| `groupPlanId`              | `6041`                 |
| `subscriberId`             | `V1StGXR8Z501`         |
| `memberId`                 | `V15tGXR8Z501`         |
| `coverageStartDate`        | `2024-01-01T05:00:00Z` |
| `coverageEndDate`          |                        |
| `medicareIsPrimary`        | `false`                |
| `cobraIsActive`            | `false`                |
| `ssn`                      | `123451111`            |
| `subscriberSsn`            | `123451111`            |
| `firstName`                | `Dan`                  |
| `lastName`                 | `Jump`                 |
| `middleName`               |                        |
| `suffix`                   |                        |
| `gender`                   | `male`                 |
| `birthdate`                | `1974-01-01`           |
| `relationshipToSubscriber` | `self`                 |
| `personCode`               | `01`                   |
| `addressLine1`             | `1234 Main St`         |
| `addressLine2`             |                        |
| `city`                     | `Tampa`                |
| `state`                    | `FL`                   |
| `postalCode`               | `33602`                |
| `country`                  | `US`                   |
| `homePhone`                |                        |
| `cellPhone`                | `813-555-1234`         |
| `workPhone`                |                        |
| `emailAddress`             | `djump@northwind.fake` |
| `memberGroupStartDate`     | `2010-05-01T05:00:00Z` |
| `memberGroupEndDate`       |                        |
| `transactionDate`          | `2023-12-30T05:00:00Z` |

### Scenario 2: Update Member Info

To update a member's information, send an eligibility record with their current coverage details and the updated personal information.

**Example**

If Dan Jump from the previous scenario moved, the file `sample_6000_elig_20240323.tsv` would appear as follows:

| Column                     | Row 1                  |
| :------------------------- | :--------------------- |
| `memberGroupId`            | `6000`                 |
| `groupPlanId`              | `6041`                 |
| `subscriberId`             | `V1StGXR8Z501`         |
| `memberId`                 | `V15tGXR8Z501`         |
| `coverageStartDate`        | `2024-01-01T05:00:00Z` |
| `coverageEndDate`          |                        |
| `medicareIsPrimary`        | `false`                |
| `cobraIsActive`            | `false`                |
| `ssn`                      | `123451111`            |
| `subscriberSsn`            | `123451111`            |
| `firstName`                | `Dan`                  |
| `lastName`                 | `Jump`                 |
| `middleName`               |                        |
| `suffix`                   |                        |
| `gender`                   | `male`                 |
| `birthdate`                | `1974-01-01`           |
| `relationshipToSubscriber` | `self`                 |
| `personCode`               | `01`                   |
| `addressLine1`             | `9786 Broad St`        |
| `addressLine2`             |                        |
| `city`                     | `Tampa`                |
| `state`                    | `FL`                   |
| `postalCode`               | `33611`                |
| `country`                  | `US`                   |
| `homePhone`                |                        |
| `cellPhone`                | `813-555-1234`         |
| `workPhone`                |                        |
| `emailAddress`             | `djump@northwind.fake` |
| `memberGroupStartDate`     | `2010-05-01T05:00:00Z` |
| `memberGroupEndDate`       |                        |
| `transactionDate`          | `2024-03-23T05:00:00Z` |

**Note:** This process applies only to a member's personal information and coverage status, not their coverage details (`memberGroupId`, `groupPlanId`, `coverageStartDate`, etc.). To modify a member's coverage, you must first terminate their existing coverage and then enroll them in the new one. If a member who was in a previous file is not included in a new file, their data and status will not change.

### Scenario 3: Terminate Coverage and (optionally) Enroll in a New Plan

To disenroll a member from a benefit plan, send an eligibility record with their previous coverage information and set the `coverageEndDate` to the date of termination.

**Example**

To end Dan Jump's coverage at the close of the year, the file `sample_6000_elig_20241201.tsv` would have a row like Row 1 in the example below. If Dan is also enrolling in a new plan for 2025, a second row, similar to Row 2, should be included with the new plan details.

| Column                     | Row 1 (Old Plan)       | Row 2 (New Plan)       |
| :------------------------- | :--------------------- | :--------------------- |
| `memberGroupId`            | `6000`                 | `6000`                 |
| `groupPlanId`              | `6041`                 | `6042`                 |
| `subscriberId`             | `V1StGXR8Z501`         | `V1StGXR8Z501`         |
| `memberId`                 | `V15tGXR8Z501`         | `V15tGXR8Z501`         |
| `coverageStartDate`        | `2024-01-01T05:00:00Z` | `2025-01-01T05:00:00Z` |
| `coverageEndDate`          | `2025-01-01T04:59:00Z` |                        |
| `medicareIsPrimary`        | `false`                | `false`                |
| `cobraIsActive`            | `false`                | `false`                |
| `ssn`                      | `123451111`            | `123451111`            |
| `subscriberSsn`            | `123451111`            | `123451111`            |
| `firstName`                | `Dan`                  | `Dan`                  |
| `lastName`                 | `Jump`                 | `Jump`                 |
| `middleName`               |                        |                        |
| `suffix`                   |                        |                        |
| `gender`                   | `male`                 | `male`                 |
| `birthdate`                | `1974-01-01`           | `1974-01-01`           |
| `relationshipToSubscriber` | `self`                 | `self`                 |
| `personCode`               | `01`                   | `01`                   |
| `addressLine1`             | `9786 Broad St`        | `9786 Broad St`        |
| `addressLine2`             |                        |                        |
| `city`                     | `Tampa`                | `Tampa`                |
| `state`                    | `FL`                   | `FL`                   |
| `postalCode`               | `33611`                | `33611`                |
| `country`                  | `US`                   | `US`                   |
| `homePhone`                |                        |                        |
| `cellPhone`                | `813-555-1234`         | `813-555-1234`         |
| `workPhone`                |                        |                        |
| `emailAddress`             | `djump@northwind.fake` | `djump@northwind.fake` |
| `memberGroupStartDate`     | `2010-05-01T05:00:00Z` | `2010-05-01T05:00:00Z` |
| `memberGroupEndDate`       |                        |                        |
| `transactionDate`          | `2024-12-01T05:00:00Z` | `2024-12-01T05:00:00Z` |
