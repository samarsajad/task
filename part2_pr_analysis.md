# Selected PR 1: beetbox/beets — PR #4199

## PR Link
https://github.com/beetbox/beets/pull/4199

## Title
**Allow to configure which fields are used to find duplicates**

---

# PR Summary

This pull request improves the duplicate detection system used during music imports in the beets music library manager. Before this change, duplicate detection relied on a mostly fixed set of metadata fields such as album artist and album title. This created problems for users managing multiple editions or formats of the same release. For example, vinyl, CD, remastered, or reissued versions of the same album could incorrectly be treated as duplicates even when users wanted to keep them separately.

The PR introduces a configurable option called `duplicate_keys`, allowing users to define exactly which metadata fields should be considered when determining duplicates. This gives users much greater flexibility and precision in organizing their music libraries. The feature also extends support to flexible attributes and singleton imports, making duplicate detection more adaptable for advanced workflows and custom metadata setups.

---

# Technical Changes

## Files / Components Modified

### `beets/importer.py`
- Updated duplicate detection logic
- Added support for configurable duplicate matching fields
- Added singleton handling support

### `beets/dbcore/db.py`
- Added reusable query-generation helpers
- Added `field_query` and `all_fields_query` utilities
- Refactored AND-based duplicate query construction

### `docs/reference/config.rst`
- Added documentation for the new `duplicate_keys` configuration option

### `docs/changelog.rst`
- Added changelog entry for the feature

### `beets/config_default.yaml`
- Added default `duplicate_keys` configuration:

```yaml
duplicate_keys:
    album: albumartist album
    item: artist title
## Implementation Approach
The implementation introduces a configuration-driven mechanism for duplicate detection. Instead of hardcoding which metadata fields define a duplicate album or track, the importer now reads a configurable list of fields from duplicate_keys. During import, the system dynamically constructs matching queries based on these configured fields.
To support this behavior, the PR refactors parts of the importer and model logic. Query generation was moved into reusable model-level functionality so the same matching logic could be shared across albums, items, and singleton imports. The contributor also introduced temporary model objects to support flexible attributes, allowing custom metadata fields to participate in duplicate matching.
The implementation additionally improves maintainability by reducing repeated query-building logic and centralizing duplicate comparison behavior. Several iterations of the PR refined handling for edge cases such as missing metadata values (None fields), flexible attributes, and duplicate detection for singleton items.
The solution balances configurability with backward compatibility because users who do not define duplicate_keys can continue using the default behavior. Extensive tests and documentation updates were added to ensure the feature integrates cleanly into existing import workflows.

## Potential Impact
This PR affects the import pipeline, metadata matching logic, and duplicate management behavior throughout the beets system. Users with complex music collections benefit significantly because they can preserve multiple versions of the same release without accidental duplicate filtering.
The change also impacts configuration management, database query construction, and importer workflows. Since duplicate detection is a core part of library imports, the PR improves flexibility while slightly increasing the complexity of query generation and metadata handling. Plugin developers and advanced users gain more control over custom metadata workflows.
