# 3.1.1 Repository Context

The `beets` repository is an open-source music library management system designed to help users organize and maintain large digital music collections. The application automatically tags music files using metadata from external databases such as MusicBrainz and provides tools for importing, renaming, organizing, and querying audio files. It is primarily used through a command-line interface and supports a large plugin ecosystem that extends its functionality.

The intended users are music enthusiasts, collectors, archivists, and advanced users who maintain structured music libraries across local storage systems. Many users manage thousands of songs and albums in different formats, editions, and releases. Since manually organizing such collections is time-consuming and error-prone, beets automates metadata correction and file management.

The repository addresses the domain of digital media organization and metadata management. One of its key challenges is identifying duplicate content during imports. Duplicate detection is important because users often re-import albums, download updated metadata, or maintain multiple versions of the same release. Incorrect duplicate detection can lead to unwanted skipping, replacement, or merging of albums and tracks. Therefore, the importer subsystem must balance automation with flexibility while supporting different library organization strategies. The project also emphasizes extensibility, allowing users to customize import behavior using configuration files and plugins.

---

# 3.1.2 Pull Request Description

PR #4199 introduces configurable duplicate detection keys for the beets importer system. Before this change, duplicate detection used hardcoded metadata fields. Albums were identified using fields such as `albumartist` and `album`, while tracks were identified using `artist` and `title`. This fixed approach worked for simple libraries but caused problems for users who stored multiple editions or variations of the same music release.

For example, a user might have a vinyl edition, remastered edition, and digital edition of the same album. Since the importer only checked a limited set of fields, these versions could incorrectly be marked as duplicates even though the user wanted to keep all of them. The previous implementation also lacked support for flexible metadata fields and custom duplicate-matching strategies.

The PR solves this problem by adding a new configuration option called `duplicate_keys`. Users can now define which metadata fields should be used during duplicate detection for albums and individual tracks. The implementation replaces hardcoded comparisons with dynamically generated queries based on configured fields. It also supports flexible attributes and singleton imports.

As a result, duplicate detection becomes customizable and more accurate for advanced music library workflows. The new implementation preserves backward compatibility by keeping the original fields as default values, ensuring existing users are not affected unless they explicitly change the configuration.

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
The importer must dynamically build duplicate detection queries using the configured fields instead of relying on hardcoded comparisons.

## Requirements
- Add reusable query-generation helpers for model field matching.
- Support flexible/custom metadata attributes during duplicate checks.
- Ensure singleton imports use item-level duplicate keys.
- Preserve backward compatibility by keeping the existing fields as defaults.
- Avoid falsely detecting duplicates when the same files are being re-imported.
- Ensure duplicate queries use AND-based matching across all configured fields.
- Update importer logic to work with metadata dictionaries instead of fixed tuples.
## Testing Requirements
- Add unit tests for default duplicate behavior.
- Add tests for custom duplicate keys.
- Add tests for flexible metadata fields.
- Add tests for singleton imports.
- Verify that albums differing in at least one configured field are treated as separate entries.
- Verify that re-importing identical file paths does not create false duplicates.
## Edge Cases to Consider
- Missing metadata values
- Flexible attributes not present in default schemas
- Large numbers of duplicate fields
- Re-import scenarios
- Singleton and album imports using different duplicate strategies
