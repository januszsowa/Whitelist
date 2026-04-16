# Whitelist Notes

This repository now uses `whitelist.yaml` as the source of truth for folders that may contain user-owned data in forensic images.

## Axes

- `user_files` covers direct file-bearing locations such as `Documents`, `Pictures`, `DCIM`, shared media roots, downloads, and synced file stores.
- `app_data` covers application and profile locations that may contain user-created databases, preferences, browser history, mail stores, backups, and logs.
- `core` paths are the first-pass scan targets within each category.
- `secondary` paths often contain user artifacts but also more noise.
- `exclusions` are pruned unless a higher-level rule explicitly overrides them.

## Matching Strategy

- Normalize path separators before comparing.
- Expand home-directory and user-profile tokens.
- Strip mount prefixes so canonical paths still match when images are mounted under analyst-specific roots.
- Keep wildcard support for user-specific roots such as `C:/Users/*`, `/home/*`, app-container UUIDs, and package names.
- Resolve common mobile aliases such as Android shared-storage variants before evaluating patterns.

## Validation Notes

- iOS system application data often lives under `/private/var/mobile/Library`, not inside every app container.
- iOS paths should be matched case-sensitively; macOS should follow the actual filesystem mode instead of assuming case-insensitive behavior.
- Android shared downloads should account for the real on-disk folder name `Download`; `Downloads` is kept only as a compatibility alias.
- Legacy mobile layouts are kept in `secondary` so older extractions are covered without polluting the primary scan set.
- Android external `Android/data/*/files` paths are treated as `user_files`, while private `/data/...` sandboxes stay under `app_data`.

## Extension Rule

When adding a new path, decide first whether it is a direct file store or an app-private artifact store. Prefer the most specific folder that still captures the target evidence. If a folder mostly holds caches, logs, or binaries, keep it out of `user_files/core` and add it only to `app_data/secondary` when there is a clear forensic reason.
