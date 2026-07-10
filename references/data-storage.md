# Data Storage

## ArkData Choices

Use ArkData according to data shape and lifecycle:

- Preferences: Lightweight key-value settings, flags, user choices, small app state.
- KV Store: Key-value data requiring database-like behavior, optional distributed capabilities.
- RelationalStore: Structured relational data, queries, transactions, and larger persistent datasets.
- DataShare: Cross-application data provider/consumer scenarios.
- Distributed data objects/database: Cross-device synchronization scenarios.

Application-created databases usually live in the app sandbox and are deleted when the app is uninstalled unless cloud/backup features are configured.

## Practical Selection

Use Preferences for:

- Theme mode
- Last selected tab
- Small settings
- Onboarding completion

Use RelationalStore for:

- Notes
- Transactions
- User-created records
- Search/filter/sort-heavy datasets

Use KV/distributed capabilities only when key-value semantics or cross-device synchronization are real requirements.

## Development Guidance

- Keep storage code out of ArkUI `build()` methods.
- Initialize storage in a service/helper layer, not directly in every component.
- Use explicit model types for rows/records.
- Handle async errors and migration failures.
- Keep storage permissions and backup behavior visible in module/profile configuration.

## Validation

When changing data code:

- Compile with hvigor.
- Run the app in DevEco if possible.
- Test first launch, upgrade/migration path, empty state, corrupted/missing data handling, and uninstall/reinstall expectations.
