# Common Lifecycle Enumeration (CLE) Specification v1.0.0

## Overview

The Common Lifecycle Enumeration (CLE) provides a standardized format for communicating software component lifecycle events in a machine-readable format. This specification defines the JSON schema and requirements for CLE documents.

## Requirements Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

## Schema Definition

### Top-level Fields

**Required Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `events` | array | Ordered array of Event objects representing the component's lifecycle events. MUST be chronologically ordered by effective date. |

### Event Object

The base object that represents a discrete lifecycle event. All events share these common fields.

**Required Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | The type of lifecycle event. MUST be one of the defined Event Types. |
| `effective` | string | The time when the event takes effect, as an RFC3339-formatted timestamp in UTC (ending in "Z"). |
| `published` | string | The time when the event was first published, as an RFC3339-formatted timestamp in UTC (ending in "Z"). |
| `modified` | string | The time when the event was last modified, as an RFC3339-formatted timestamp in UTC (ending in "Z"). |

**Optional Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `description` | string | Human-readable description of the event. SHOULD be clear and concise. |
| `references` | array[string] | List of URLs to supporting documentation. SHOULD point to authoritative sources. |

### Event Types

#### generalAvailability
*Category: Version Event*

Indicates when a component version is released and available for use.

**Additional Required Fields:**
- `version`: VERS-formatted single version (e.g., "vers:npm/1.0.0")

Example:
```json
{
  "type": "generalAvailability",
  "effective": "2023-01-01T00:00:00Z",
  "version": "vers:npm/1.0.0",
  "published": "2023-01-01T00:00:00Z",
  "modified": "2023-01-01T00:00:00Z",
  "description": "Version 1.0.0 is now generally available",
  "references": [
    "https://example.com/release-notes"
  ]
}
```

#### endOfDevelopment
*Category: Version Event*

Indicates when the manufacturer ceases development of a component or service.

**Additional Required Fields:**
- `range`: VERS-formatted version range (e.g., "vers:npm/>=1.0.0|<2.0.0")

Example:
```json
{
  "type": "endOfDevelopment",
  "effective": "2024-01-01T00:00:00Z",
  "range": "vers:npm/>=1.0.0|<2.0.0",
  "published": "2023-06-01T00:00:00Z",
  "modified": "2023-06-01T00:00:00Z",
  "description": "Development of 1.x versions will cease"
}
```

#### endOfSupport
*Category: Version Event*

Indicates when the manufacturer ceases any and all support of a component or service. This point in time marks a transfer of risk from the manufacturer to the consuming organisation or user of the component or service, encompassing all cybersecurity knowledge and known vulnerabilities, with no further assistance provided by the manufacturer.

**Additional Required Fields:**
- `range`: VERS-formatted version range (e.g., "vers:npm/>=1.0.0|<2.0.0")

Example:
```json
{
  "type": "endOfSupport",
  "effective": "2024-01-01T00:00:00Z",
  "range": "vers:npm/>=1.0.0|<2.0.0",
  "published": "2023-06-01T00:00:00Z",
  "modified": "2023-06-01T00:00:00Z",
  "description": "All 1.x versions will no longer receive support after January 1st, 2024",
  "references": [
    "https://example.com/end-of-support"
  ]
}
```

#### endOfGuaranteedSupport
*Category: Version Event*

Indicates when the manufacturer no longer provides assured services such as technical assistance, user training, repairs, spare parts, and software updates for a component or service. Beyond this period, any support offered is discretionary and may incur extra costs or have restrictions.

**Additional Required Fields:**
- `range`: VERS-formatted version range (e.g., "vers:npm/>=1.0.0|<2.0.0")

Example:
```json
{
  "type": "endOfGuaranteedSupport",
  "effective": "2024-01-01T00:00:00Z",
  "range": "vers:npm/>=1.0.0|<2.0.0",
  "published": "2023-06-01T00:00:00Z",
  "modified": "2023-06-01T00:00:00Z",
  "description": "Guaranteed support for 1.x versions will end"
}
```

#### endOfLife
*Category: Version Event*

Indicates when the manufacturer stops distribution of a product after its defined useful life and formally notifies users. It marks the conclusion of the product's lifecycle, following a structured EOL process.

**Additional Required Fields:**
- `range`: VERS-formatted version range (e.g., "vers:npm/>=1.0.0|<2.0.0")

Example:
```json
{
  "type": "endOfLife",
  "effective": "2024-01-01T00:00:00Z",
  "range": "vers:npm/>=1.0.0|<2.0.0",
  "published": "2023-06-01T00:00:00Z",
  "modified": "2023-06-01T00:00:00Z",
  "description": "1.x versions have reached end of life"
}
```

#### endOfProduction
*Category: Version Event*

Indicates when the manufacturer stops producing a component, often due to newer versions, unavailable parts, market changes, or strategic shifts. Existing units may still be available in warehouses, distribution channels, and for use, but no new units will be manufactured.

**Additional Required Fields:**
- `range`: VERS-formatted version range (e.g., "vers:npm/>=1.0.0|<2.0.0")

Example:
```json
{
  "type": "endOfProduction",
  "effective": "2024-01-01T00:00:00Z",
  "range": "vers:npm/>=1.0.0|<2.0.0",
  "published": "2023-06-01T00:00:00Z",
  "modified": "2023-06-01T00:00:00Z",
  "description": "Production of 1.x versions will cease"
}
```

#### endOfMarketing
*Category: Version Event*

Indicates when the manufacturer will cease actively promoting or advertising a component or service. While the component or service may still be available for purchase and support, it will no longer receive active marketing efforts from the manufacturer.

**Additional Required Fields:**
- `range`: VERS-formatted version range (e.g., "vers:npm/>=1.0.0|<2.0.0")

Example:
```json
{
  "type": "endOfMarketing",
  "effective": "2024-01-01T00:00:00Z",
  "range": "vers:npm/>=1.0.0|<2.0.0",
  "published": "2023-06-01T00:00:00Z",
  "modified": "2023-06-01T00:00:00Z",
  "description": "Marketing of 1.x versions will end"
}
```

#### supersededBy
*Category: Version Event*

Indicates when a version of a component is superseded by another version of a component.

**Additional Required Fields:**
- `version`: VERS-formatted version being superseded (e.g., "vers:npm/1.0.0")
- `supersededByVersion`: VERS-formatted version that supersedes it (e.g., "vers:npm/2.0.0")

Example:
```json
{
  "type": "supersededBy",
  "effective": "2024-01-01T00:00:00Z",
  "version": "vers:npm/1.0.0",
  "supersededByVersion": "vers:npm/2.0.0",
  "published": "2023-06-01T00:00:00Z",
  "modified": "2023-06-01T00:00:00Z",
  "description": "Version 1.0.0 is superseded by version 2.0.0"
}
```

#### componentRenamed
*Category: Component Event*

Indicates when a component has been renamed with new identifiers.

**Additional Required Fields:**
- `identifiers`: Array of new identifiers for the component

**Additional Optional Fields:**
- `range`: VERS-formatted version range (e.g., "vers:npm/>=1.0.0|<2.0.0")
- `version`: VERS-formatted single version (e.g., "vers:npm/1.0.0")

Example:
```json
{
  "type": "componentRenamed",
  "effective": "2024-01-01T00:00:00Z",
  "published": "2023-12-01T00:00:00Z",
  "modified": "2023-12-01T00:00:00Z",
  "description": "The component has been renamed to 'new-component'",
  "range": "vers:npm/>=1.0.0",
  "identifiers": [
    {
      "type": "PURL",
      "value": "pkg:npm/new-component"
    }
  ],
  "references": [
    "https://example.com/rename-announcement"
  ]
}
```

## Event Categories

The CLE specification supports two distinct categories of events: `Version Events` and `Component Events`.

The distinction between these categories is important as it affects how tools and systems should process and interpret the events. `Version Events` are typically used for lifecycle management and support planning, while `Component Events` are crucial for maintaining proper component identification and traceability over time.

### Version Events

Events that affect specific versions or ranges of versions of a component. These events use either the `version` or `range` field to specify which versions are affected. Version Events include:

- `generalAvailability`
- `endOfDevelopment`
- `endOfSupport`
- `endOfGuaranteedSupport`
- `endOfLife`
- `endOfProduction`
- `endOfMarketing`
- `supersededBy`

### Component Events

Events that affect the component itself and may impact how the component is identified or referenced. These events often affect all versions of a component from the effective date forward. Component Events include:

- `componentRenamed`: Changes how the component is identified

## Extended Support

TODO: think about how we would represent extended support in the CLE.

## Distribution

TODO: think about the possible distribution methods around GitHub, TEA, etc.

## Example

```json
{
  "events": [
    {
      "type": "generalAvailability",
      "effective": "2019-01-01T00:00:00Z",
      "version": "vers:npm/1.0.0",
      "published": "2019-01-01T00:00:00Z",
      "modified": "2019-01-01T00:00:00Z"
    },
    {
      "type": "endOfSupport",
      "effective": "2020-01-01T00:00:00Z",
      "range": "vers:npm/>=1.0.0|<2.0.0",
      "references": [
        "https://example.com/end-of-support"
      ],
      "published": "2020-01-01T00:00:00Z",
      "modified": "2020-01-01T00:00:00Z"
    },
    {
      "type": "componentRenamed",
      "effective": "2020-01-01T00:00:00Z",
      "description": "The component has been renamed to 'new-component'.",
      "identifiers": [
        {
          "type": "PURL",
          "value": "pkg:npm/new-component"
        }
      ],
      "published": "2020-01-01T00:00:00Z",
      "modified": "2020-01-01T00:00:00Z"
    }
  ]
}
```

## Use Cases

The CLE specification aims to address several common use cases in software component lifecycle management. The following use cases are supported in version 1.0.0:

v1.0.0 Supported Use Cases:
- General Availability: Track when new versions of a component are released and available for use
- End of Support: Communicate when versions will no longer receive updates or support
- Component Renaming: Handle cases where a component's identifiers change (e.g., package rename)

Future Use Cases:
- Component Bundling/Unbundling: Track when components are bundled into or extracted from larger packages
- Component Acquisition: Handle cases where components change ownership
- Extended Support: Support for third-party extended support offerings
- Component Forking: Track when components are forked into new projects
- Export Restrictions: Handle cases where components become restricted in certain regions
- Security Status Changes: Track when components are marked as compromised or unsafe

These use cases will be addressed in future versions of the specification based on community feedback and requirements.

## References

- Package URL (PURL) Specification: https://github.com/package-url/purl-spec
- Version Range (VERS) Specification: https://github.com/package-url/purl-spec/blob/master/VERSION-RANGE-SPEC.rst
