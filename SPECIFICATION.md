# Common Lifecycle Enumeration (CLE) Specification v1.0.0

## Overview

The Common Lifecycle Enumeration (CLE) provides a standardized format for communicating software component lifecycle events in a machine-readable format. This specification defines the JSON schema and requirements for CLE documents.

## Requirements Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

## Types of Work

The following types of work are commonly associated with software projects regardless of whether they are open source or not.

- `Marketing`: Promoting and advertising a software project to potential users.
- `Substantial Modifications`: Making substantial changes to a software project that are not considered bug fixes or security fixes, such as adding new features or functionality.
- `Bug Fixes`: Addressing and resolving issues or defects in a software project.
- `Security Fixes`: A distinct type of bug fix focused on security vulnerabilities that is useful to differentiate from other types of bug fixes.
- `Distribution`: The process of making a software project available for use by others.
- `Documentation`: Writing and updating documentation for a software project to help users understand how to use it.

## Schema Definition

### JSON Schema

CLE is formally specified by a `Draft 2020-12` JSON Schema.

Each published version of the specification is accompanied by a versioned meta-schema at a stable URI:

```
https://cle.example.com/schema/cle-<major>.<minor>.<patch>.schema.json
```

Every CLE document MUST include a top-level `$schema` field whose value is the URI of the meta-schema for the version it follows.

The $schema field serves two purposes:
- Validation – It allows tooling to validate that the document is structurally correct and conforms to the CLE specification.
- Version Signaling – It identifies the version of the CLE specification the document follows, so consumers can parse and interpret it accordingly.

### Top-level Fields

**Required Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `$schema` | string | URI identifying the [JSON Schema](https://json-schema.org/) document that describes the version of the CLE schema to use (e.g., "https://TODO/cle.v1.0.0.json"). CLE is built on JSON Schema draft 2020-12. |
| `identifier` | string or array[string] | Component identifier(s) in the PURL format. When an array is provided, each identifier alias must identify the exact same bits from the same software but distributed differently. A software component that is distributed or packaged by a third party should not have an identifier in the array. Identifiers MUST NOT include version strings but MAY include qualifiers. |
| `updatedAt` | string | ISO 8601 timestamp indicating when this CLE document was last updated. |
| `events` | array | Ordered array of Event objects representing the component's lifecycle events. MUST be ordered by ID in descending order (newest events with highest IDs first). |

**Additional Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `definitions` | object | Container for reusable policy definitions that can be referenced throughout the document. |
| `index` | string | URL pointing to the index file that lists all CLE pages for this component. Only allowed to be present when pagination is used; and is required in that case. |
| `next` | string | URL pointing to the next CLE page containing newer events (higher event IDs). When pagination is used, this field is required unless this is the final document. Otherwise, it MUST be absent. |

### Definitions Object

The definitions object allows specification of reusable policies and calculations that can be referenced by events.

**Support**

The support object defines the support policies provided for a specific version or version range of a component. This may include first-party manufacturer support or third-party support options endorsed by the manufacturer.

Support policies are immutable once defined - the semantics and meaning of a support policy MUST NOT change over time for a given support policy ID. This ensures consistent interpretation of support commitments across the component's lifecycle.

Support definitions are optional unless there is an event that requires them.

| Field | Type | Description |
|-------|------|-------------|
| `support` | array[object] | List of support policies. |

Each support policy object has the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | **Required.** Unique identifier for the support policy. Once assigned, the semantics of this support policy MUST remain constant. |
| `description` | string | **Required.** Human readable description of the policy |
| `url` | string | **Optional.** URL to detailed documentation about this support policy |

Example support policy definition:
```json
{
  "definitions": {
    "support": [
      {
        "id": "standard",
        "description": "Standard product support policy",
        "url": "https://example.com/support/standard"
      },
      {
        "id": "extended",
        "description": "Extended support with priority response times",
        "url": "https://example.com/support/extended"
      },
      {
        "id": "premium",
        "description": "Premium support with 24/7 coverage and dedicated team",
      }
    ]
  }
}
```

### Event Object

The base object that represents a discrete lifecycle event. All events share these common fields.

Events are immutable once created - the content and meaning of a specific event MUST NOT change after it has been published. Additionally, the ordering of events across shards MUST NOT change.

**Required Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | A unique, auto-incrementing integer identifier for the event. IDs must be assigned in ascending order as events are added. |
| `type` | string | The type of lifecycle event. MUST be one of the defined Event Types. |
| `effective` | string | The time when the event takes effect, as an ISO 8601 formatted timestamp in UTC (ending in "Z"). |
| `published` | string | The time when the event was first published, as an ISO 8601 formatted timestamp in UTC (ending in "Z"). |

### Event Types

#### released
*Category: Version Event*

Indicates when a component version is released and available for use.

**Additional Required Fields:**

- version

**Additional Optional Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `license` | string | License identifier that summarizes the license as declared in the component's metadata. |

Example:
```json
{
  "id": 1,
  "type": "released",
  "effective": "2024-01-01T00:00:00Z",
  "version": "1.0.0",
  "license": "MIT",
  "published": "2023-06-01T00:00:00Z"
}
```

#### endOfDevelopment
*Category: Version Event*

The manufacturer or maintainer ceases work on `Substantial Modifications` for a specific version or version range of a component or service. `Security Fixes` and `Bug Fixes` will continue to be provided for this specific version or version range until `endOfSupport` is declared, but no new features or enhancements will be added.

> TL;DR: Ceasing Substantial Modifications.

**Additional Required Fields:**

- versions
- supportId

Example:
```json
{
  "id": 2,
  "type": "endOfDevelopment",
  "effective": "2024-01-01T00:00:00Z",
  "versions": [
    {
      "version": "1.0.0"
    },
    {
      "range": "vers:npm/>=1.0.0|<2.0.0"
    }
  ],
  "supportId": "standard",
  "published": "2023-06-01T00:00:00Z"
}
```

#### endOfSupport
*Category: Version Event*

The manufacturer or maintainer ceases providing `Security Fixes` and `Bug Fixes` for a specific version or version range of a component or service.

The `supportId` field MUST be included and used to specify which support policy is ending, referencing a support policy defined in the definitions section.

An `endOfSupport` event should only exist when a support definition object exists for the entire indicated version range.

> TL;DR: Ceasing Security Fixes and Bug Fixes.

**Additional Required Fields:**

- versions
- supportId

Example:
```json
{
  "id": 3,
  "type": "endOfSupport",
  "effective": "2024-01-01T00:00:00Z",
  "versions": ["1.0.0"],
  "supportId": "guaranteed",
  "published": "2023-06-01T00:00:00Z"
}
```

#### endOfLife
*Category: Version Event*

The manufacturer or maintainer formally ceases all work (including `Distribution`, `Substantial Modifications`, `Bug Fixes`, `Security Fixes`, `Documentation`, and `Maintenance`) for a specific version or version range of a component. No further updates, support, or distribution will be provided for this specific version or version range. The component is considered retired.

> TL;DR: Ceasing all work.

**Additional Required Fields:**

- versions

Example:
```json
{
  "id": 4,
  "type": "endOfLife",
  "effective": "2024-01-01T00:00:00Z",
  "versions": [
    {
      "range": "vers:npm/>=1.0.0|<2.0.0"
    }
  ],
  "published": "2023-06-01T00:00:00Z"
}
```

### endOfDistribution
*Category: Version Event*

The manufacturer or maintainer ceases distribution of a specific version or version range of a component or service. This should only be used when the manufacturer has control over the distribution of the component or service - e.g. if the manufacturer publishes to a registry, they likely do not have this control, and thus would not provide this event.

**Additional Required Fields:**

- versions

Example:
```json
{
  "id": 5,
  "type": "endOfDistribution",
  "effective": "2024-01-01T00:00:00Z",
  "versions": [
    {
      "version": "1.0.0"
    }
  ],
  "published": "2023-06-01T00:00:00Z"
}
```

#### endOfMarketing
*Category: Version Event*

The manufacturer or maintainer ceases marketing and promotion of a specific version or version range of a component or service. The component or service may still be available, and existing support policies may remain in effect, but the manufacturer will no longer seek new customers or promote its use.

> TL;DR: Ceasing marketing and promotion.

**Additional Required Fields:**

- versions

Example:
```json
{
  "id": 6,
  "type": "endOfMarketing",
  "effective": "2024-01-01T00:00:00Z",
  "versions": [
    {
      "range": "vers:npm/>=1.0.0|<2.0.0"
    }
  ],
  "published": "2023-06-01T00:00:00Z"
}
```

#### supersededBy
*Category: Version Event*

Indicates when a version of a component is superseded by another version of a component. This should only exist for components in which version progression is not implicit, for example, a component is not using semantic versioning.

**Additional Required Fields:**
- `supersededByVersion`: Plain version string that supersedes it (e.g., "2.0.0")

**Additional Optional Fields:**

- versions

Example:
```json
{
  "id": 7,
  "type": "supersededBy",
  "effective": "2024-01-01T00:00:00Z",
  "versions": [
    {
      "version": "1.0.0"
    }
  ],
  "supersededByVersion": "2.0.0",
  "published": "2023-06-01T00:00:00Z"
}
```

#### componentRenamed
*Category: Component Event*

Indicates when a component is renamed.

**Additional Required Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `identifiers` | array[object] | Array of identifier objects specifying the new identifiers for the component. Each object must have `type` field set to "PURL" and `value` field containing a valid Package URL (PURL) string. |

**Additional Optional Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `description` | string | Human-readable description of the event. SHOULD be clear and concise, and SHOULD explain why the event is occurring. |
| `references` | array[string] | List of URLs to supporting documentation. SHOULD point to authoritative sources. |

Example:
```json
{
  "id": 8,
  "type": "componentRenamed",
  "effective": "2024-01-01T00:00:00Z",
  "published": "2023-06-01T00:00:00Z",
  "description": "The component has been renamed to 'new-component', because the product has been acquired by a new company",
  "references": [
    "https://example.com/component-renamed"
  ],
  "identifiers": [
      {
      "type": "PURL",
      "value": "pkg:npm/new-component"
    }
  ]
}
```

#### withdrawn
*Category: Meta Event*

Indicates that a previously published event is being withdrawn or revoked. This is used in a prepend-only event model where events cannot be modified, only new events can be added. When an event is withdrawn, it should be ignored during processing as if it never existed.

**Additional Required Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `eventId` | integer | The ID of the event being withdrawn. Must reference an existing event ID. |

**Additional Optional Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `references` | array[string] | List of URLs to supporting documentation explaining the withdrawal. |
| `reason` | string | Human-readable explanation for why the event is being withdrawn. SHOULD be clear and concise. |

Example:
```json
{
  "id": 9,
  "type": "withdrawn",
  "effective": "2024-02-01T00:00:00Z",
  "published": "2024-02-01T00:00:00Z",
  "eventId": 3,
  "reason": "The endOfSupport event was published in error. Support continues until 2025-01-01.",
  "references": [
    "https://example.com/support-extension-announcement"
  ]
}
```

## Event Categories

The CLE specification supports three distinct categories of events: `Version Events`, `Component Events`, and `Meta Events`.

The distinction between these categories is important as it affects how tools and systems should process and interpret the events. `Version Events` are typically used for lifecycle management and support planning, `Component Events` are crucial for maintaining proper component identification and traceability over time, and `Meta Events` are used to manage the event history itself.

### Version Events

Events that affect specific versions or ranges of versions of a component. These events use either the `version` or `range` field to specify which versions are affected. Version Events include:

- `released`
- `endOfDevelopment`
- `endOfSupport`
- `endOfLife`
- `endOfDistribution`
- `endOfMarketing`
- `supersededBy`

### Component Events

Events that affect the component itself and may impact how the component is identified or referenced. These events often affect all versions of a component from the effective date forward. Component Events include:

- `componentRenamed`: Changes how the component is identified

### Meta Events

Events that affect other events in the history. These are used in the prepend-only event model to manage the event stream. Meta Events include:

- `withdrawn`: Revokes a previously published event

## Event Processing Rules

CLE uses a prepend-only event model where:

1. **Immutability**: Once an event is published, it cannot be modified. New events must be added to correct or update information.
2. **Ordering**: Events MUST be ordered by ID in descending order (newest events with highest IDs first).
3. **ID Assignment**: Event IDs MUST be assigned as auto-incrementing integers in the order events are added (not by effective date).
4. **Processing Order**: When processing events, consumers should process them in reverse order (oldest to newest by ID) to build the correct state.
5. **Withdrawn Events**: When a `withdrawn` event is encountered:
   - The event referenced by `eventId` should be ignored as if it never existed
   - The withdrawn event itself remains in the history for audit purposes
   - Any dependent events or calculations based on the withdrawn event should be recalculated

## Pagination

CLE supports pagination to handle components with extensive event histories. When a CLE file reaches the maximum limit of 100,000 events, it must be split into multiple pages.

### Pagination Rules

1. **Page Size Limit**: A single CLE page MUST NOT exceed 100,000 events. This is a hard limit to ensure reasonable file sizes and processing performance.

2. **Event ID Continuity**: Event IDs MUST be globally unique and incrementing across all pages. IDs do not restart for each page.

3. **Event Ordering**: Within each page, events MUST be ordered by ID in descending order (newest events with highest IDs first).

4. **Page Chaining**: Pages are linked using the `next` field, which points to the page containing newer events (higher event IDs).

5. **Index File**: When pagination is used, an index file SHOULD be provided via the `index` field to enable efficient discovery of all pages.

### CLE Index Schema

The CLE index file provides a directory of all pages for a component. It follows this structure:

```json
{
  "$schema": "https://TODO/cle-index.v1.0.0.json",
  "pages": [
    {
      "url": "https://example.com/component/cle-page-1.json",
      "firstEventId": 1,
      "lastEventId": 100000
    },
    {
      "url": "https://example.com/component/cle-page-2.json",
      "firstEventId": 100001,
      "lastEventId": 150000
    }
  ]
}
```

**Index Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `$schema` | string | **Required.** URI identifying the CLE index schema version. |
| `pages` | array[object] | **Required.** Array of page descriptor objects, ordered by event ID ranges. |

**Page Descriptor Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `url` | string | **Required.** URL of the CLE page file. |
| `firstEventId` | integer | **Required.** The lowest event ID in this page. |
| `lastEventId` | integer | **Required.** The highest event ID in this page. |

### Pagination Examples

**Example: Page 1 (oldest events)**
```json
{
  "$schema": "https://TODO/cle.v1.0.0.json",
  "index": "https://example.com/component/cle-index.json",
  "next": "https://example.com/component/cle-page-2.json",
  "events": [
    {
      "id": 100000,
      "type": "released",
      "effective": "2022-01-01T00:00:00Z",
      "version": "5.0.0",
      "published": "2022-01-01T00:00:00Z"
    },
    // ... more events ...
    {
      "id": 1,
      "type": "released",
      "effective": "2020-01-01T00:00:00Z",
      "version": "1.0.0",
      "published": "2020-01-01T00:00:00Z"
    }
  ]
}
```

**Example: Page 2 (newer events)**
```json
{
  "$schema": "https://TODO/cle.v1.0.0.json",
  "index": "https://example.com/component/cle-index.json",
  "events": [
    {
      "id": 150000,
      "type": "released",
      "effective": "2024-06-01T00:00:00Z",
      "version": "10.0.0",
      "published": "2024-06-01T00:00:00Z"
    },
    // ... more events ...
    {
      "id": 100001,
      "type": "released",
      "effective": "2022-01-02T00:00:00Z",
      "version": "5.0.1",
      "published": "2022-01-02T00:00:00Z"
    }
  ]
}
```

Note: Page 2 does not have a `next` field because it contains the newest events.

### Processing Paginated CLE Files

When processing paginated CLE files:

1. Start with any page (typically discovered via the index file or a known entry point).
2. Process events within the current page according to standard processing rules.
3. To process all events, use the index file to discover all pages, or follow `next` links to traverse pages.
4. Event IDs are globally unique, so `withdrawn` events can reference events on any page.

## Distribution

TODO: think about the possible distribution methods around GitHub, TEA, etc.

## Example

```json
{
  "$schema": "https://TODO/cle.v1.0.0.json",
  "events": [
    {
      "id": 5,
      "type": "withdrawn",
      "effective": "2021-01-15T00:00:00Z",
      "published": "2021-01-15T00:00:00Z",
      "eventId": 2,
      "reason": "The endOfSupport date was incorrect. Support actually ended on 2021-01-01.",
      "references": [
        "https://example.com/support-correction"
      ]
    },
    {
      "id": 4,
      "type": "endOfSupport",
      "effective": "2021-01-01T00:00:00Z",
      "published": "2021-01-01T00:00:00Z",
      "versions": [
        {
          "range": "vers:npm/>=1.0.0|<2.0.0"
        }
      ],
      "supportId": "standard"
    },
    {
      "id": 3,
      "type": "componentRenamed",
      "effective": "2020-01-01T00:00:00Z",
      "published": "2020-01-01T00:00:00Z",
      "description": "The component has been renamed to 'new-component', because the product has been acquired by a new company",
      "identifiers": [
        {
          "type": "PURL",
          "value": "pkg:npm/new-component"
        }
      ],
      "references": [
        "https://example.com/component-renamed"
      ]
    },
    {
      "id": 2,
      "type": "endOfSupport",
      "effective": "2020-01-01T00:00:00Z",
      "published": "2020-01-01T00:00:00Z",
      "versions": [
        {
          "range": "vers:npm/>=1.0.0|<2.0.0"
        }
      ],
      "supportId": "standard"
    },
    {
      "id": 1,
      "type": "released",
      "effective": "2019-01-01T00:00:00Z",
      "published": "2019-01-01T00:00:00Z",
      "version": "1.0.0",
      "license": "MIT"
    }
  ]
}
```

## Use Cases

The CLE specification aims to address several common use cases in software component lifecycle management. The following use cases are supported in version 1.0.0:

### v1.0.0 Supported Use Cases:
- General Availability: Track when new versions of a component are released and available for use
- End of Support/End of Life: Communicate when versions will no longer receive updates or support
- Component Renaming: Handle cases where a component's identifiers change (e.g., package rename)

### Future Use Cases:
- Complex License Changes: Handling license changes for previously released versions. A maintainer may change the license for a previously released version, we may use a `licenseChange` event with a range of versions to handle this use case.
- Component Bundling/Unbundling: Track when components are bundled into or extracted from larger packages
- Component Acquisition: Handle cases where components change ownership
- Extended Support: Support for third-party extended support offerings
- Third Party Claims: Handling CLE from a third party perspective making claims about a component which they have no operational control over.
- Component Forking: Track when components are forked into new projects
- Export Restrictions: Handle cases where components become restricted in certain regions
- Security Status Changes: Track when components are marked as compromised or unsafe

These use cases will be addressed in future versions of the specification based on community feedback and requirements.

## References

- Package URL (PURL) Specification: https://github.com/package-url/purl-spec
- Version Range (VERS) Specification: https://github.com/package-url/purl-spec/blob/master/VERSION-RANGE-SPEC.rst
