# 3.1.1 Repository Context

The `beets` repository is an open-source music library management system designed to help users organize and maintain large digital music collections. It offers features for importing, renaming, organizing and querying audio files in addition to automatically tagging music files using metadata from external databases like MusicBrainz. It is mainly utilized via a command-line interface and has a sizable plugin ecosystem that expands its capabilities.

Music lovers, collectors, archivists, and sophisticated users who keep organized music libraries across local storage systems are the target audience. Thousands of songs and albums in various formats, editions and releases are managed by several users. Beets automates file management and metadata rectification because manually managing such collections is laborious and prone to mistakes.

The repository covers the fields of metadata management and digital media organization. Finding duplicate content during imports is one of its main problems. Because users frequently re-import albums, download updated information, or keep numerous versions of the same release, duplicate detection is crucial. Unwanted album and track skipping, replacement or merger might result from incorrect duplicate detection. As a result, the importer subsystem needs to accommodate various library organization techniques while striking a balance between automation and flexibility. Additionally, the project places a strong emphasis on extensibility, enabling users to modify import behavior using configuration files and plugins.

---

# 3.1.2 Pull Request Description

PR #4199 introduces configurable duplicate detection keys for the beets importer system. Duplicate detection relied on hardcoded metadata fields prior to this modification. Fields like `albumartist` and `album` were used to identify albums and `artist` and `title` were used to identify tracks. Users who kept several editions or variations of the same music release encountered difficulties with this fixed approach which was effective for modest libraries.

For instance, a user may possess a digital, remastered and vinyl version of the same album. These versions might be mistakenly labeled as duplicates even though the user intended to maintain all of them since the importer only examined a small number of fields. Additionally, specific duplicate matching techniques and configurable metadata fields were not supported in the earlier implementation.

The PR adds a new configuration option named `duplicate_keys` to address this issue. For both individual tracks and albums, users can now specify which metadata fields should be used for duplicate detection. The approach uses dynamically created queries based on defined fields in place of hardcoded comparisons. Additionally, it allows for singleton imports and flexible characteristics.

Duplicate detection becomes more accurate and flexible for sophisticated music library procedures as a result. By maintaining the original fields as default settings, the new design maintains backward compatibility and guarantees that current users are unaffected unless they specifically modify the configuration.

---

# 3.1.3 Acceptance Criteria

- ✓ When importing albums using the default configuration, the importer should continue using `albumartist` and `album` as duplicate detection keys.

- ✓ When a user defines custom `duplicate_keys` in the configuration, the importer should use those fields instead of hardcoded values.

- ✓ When flexible/custom metadata fields are included in `duplicate_keys`, the importer should correctly evaluate duplicates using those fields.

- ✓ When importing singleton tracks, the importer should apply the configured `item` duplicate keys instead of album-level matching.

- ✓ When two albums differ in at least one configured duplicate field, the importer should treat them as separate entries.

- ✓ When an album is fully re-imported using the same file paths, the importer should not incorrectly flag it as a duplicate.

- ✓ The implementation should preserve backward compatibility for users who do not modify the default configuration.

- ✓ The system should generate valid AND-based database queries for all configured duplicate fields.

- ✓ Unit tests should verify duplicate behavior for both default and custom configurations.

- ✓ Documentation should clearly describe the new `duplicate_keys` configuration option and its default behavior.

---

# 3.1.4 Edge Cases

## Edge Case 1: Missing Metadata Fields

If a configured duplicate field is missing or contains `None`, the importer should avoid crashing and should handle the comparison safely.

---

## Edge Case 2: Flexible Attribute Matching

If users include custom metadata fields that are not part of the default schema, duplicate detection should still function correctly using temporary model objects or computed values.

---

## Edge Case 3: Re-importing Existing Files

If the same album is re-imported from identical file paths, the importer should recognize it as a replacement operation rather than a duplicate conflict.

---

## Edge Case 4: Large Metadata Combinations

If many fields are included in `duplicate_keys`, query generation should remain efficient and should not produce malformed database queries.

---

## Edge Case 5: Singleton vs Album Imports

The importer should correctly separate album-level duplicate detection from singleton track duplicate detection.

---

# 3.1.5 Initial Prompt

You are contributing to the `beets` music library management repository. The goal is to improve the importer subsystem by implementing configurable duplicate detection keys for albums and individual tracks.

Currently, duplicate detection relies on hardcoded metadata fields. Album imports use fields such as `albumartist` and `album`, while singleton track imports use `artist` and `title`. This approach is too restrictive for users managing multiple editions or variations of the same release.

Implement a new configuration option called `duplicate_keys` inside the import configuration section. The configuration should support separate key lists for albums and items, for example:

```yaml
duplicate_keys:
    album: albumartist album
    item: artist title
```
Instead than utilizing hardcoded comparisons, the importer must dynamically construct duplicate detection queries using the configured attributes. The solution should enable flexible/custom metadata attributes during duplicate checks and add reusable query-generation aids for model field matching. Item-level duplicate keys should be used for singleton imports, and the implementation must maintain backward compatibility by using the default fields. The importer should create AND-based duplication queries over all configurable fields and refrain from mistakenly identifying duplicates when the same files are being re-imported. It is necessary to modify the current importer logic so that it uses metadata dictionaries rather than fixed tuples.

Comprehensive test coverage should also be part of the implementation. Include unit tests for singleton imports, flexible metadata fields, customizable duplicate keys, default duplicate behavior, and duplicate differentiation behavior. The tests should confirm that re-importing identical file paths does not result in erroneous duplicate detections and that albums that differ in at least one specified field are handled as distinct entries.

When putting the functionality into practice, take into account edge cases like missing metadata values, flexible attributes that aren't included in the default schema, a lot of duplicate fields, re-import situations and distinct duplication handling techniques for album imports and singleton imports. Updates to the documentation and changelog outlining the new duplicate_keys configuration option and its anticipated behavior should also be included of the final implementation.
