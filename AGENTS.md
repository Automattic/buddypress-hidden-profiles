# BuddyPress Hidden Profiles

Lets site admins hide BuddyPress member profiles from non-admins: hidden members are left out of directories and searches, and their profile pages return a 404.

## Project Knowledge

| Property | Value |
|----------|-------|
| **Main file** | `buddypress-hidden-profiles.php` |
| **Text domain** | `buddypress-hidden-profiles` |
| **Namespace** | `Automattic\BuddyPressHiddenProfiles` |
| **Composer package** | `automattic/buddypress-hidden-profiles` |
| **Filter prefix** | `buddypress_hidden_profiles_` |
| **Requires PHP** | 8.2+ |
| **Requires WP** | 6.6+ |
| **Requires** | BuddyPress or BuddyBoss Platform |
| **Distribution** | GitHub releases only; not on WordPress.org |

`BuddyPress_Hidden_Profiles` is the entry point: on `bp_loaded` it wires up three classes in `src/`:

- **`Hidden_Users`**: who is hidden, who may see them, and the cache.
- **`Visibility`**: the BuddyPress hooks that keep hidden users out of profile pages, member lists and the REST API. Reads everything from `Hidden_Users`.
- **`Profile_Setting`**: the admin checkbox on profile screens.

Do not rename these without a migration, because they are stored data or public API:

- **`profile_visibility` user meta, value `hidden`**: what marks a profile as hidden. The README tells admins to set it with WP-CLI.
- **`bp_hidden_user_ids` cache key, in the global `buddypress_hidden_profiles` group**: the README tells admins to delete it by hand.
- **The two filters**, `buddypress_hidden_profiles_is_hidden` and `buddypress_hidden_profiles_additional_hidden_ids`, which other code uses to extend the plugin.

## Commands

```bash
composer cs                  # Check coding standards (PHPCS)
composer cs-fix              # Fix what PHPCS can
composer lint                # PHP syntax lint
composer test                # Integration tests (requires wp-env)
composer test:integration-ms # Integration tests, multisite
npx wp-env start             # WordPress with BuddyPress active
```

## Conventions

See `CONTRIBUTING.md` for setup, checks and the contribution workflow. Key points:

- **Commits**: Every commit must be signed; `develop` and `main` reject unsigned commits. Use a [Conventional Commits](https://www.conventionalcommits.org/) prefix (`fix:`, `feat:`, `test:`, `docs:`, `ci:`, `chore:` and so on) with an imperative subject of about 50 characters, and a body that explains why the change was needed rather than what changed. One concern per commit.
- **PRs**: Open against `develop`. The title describes the outcome in sentence case, without a conventional commit prefix. The description explains the problem first, then how the PR solves it. PRs land as merge commits (squash and rebase merging are disabled), so every commit message reaches `develop` as written.
- **Tests**: Integration tests only, in `tests/Integration/`, one file per feature, extending the shared `TestCase` there for its fixtures. There is no unit suite, because the plugin is thin glue around WordPress and BuddyPress.
- **Code style**: WordPress VIP coding standards via PHPCS, configured in `.phpcs.xml.dist`. Tabs for indentation.
- **i18n**: All user-facing strings must use the `buddypress-hidden-profiles` text domain.

## Common Pitfalls

- **The wp-env BuddyPress URL**: wp-env names a plugin's directory after its ZIP file, and the test bootstrap requires `plugins/buddypress/bp-loader.php`. Keep the URL ending in `buddypress.zip`.
- **Clearing the cache**: anything that changes who is hidden must clear `bp_hidden_user_ids`, or directories keep showing stale results for up to a day. Changes to the `profile_visibility` meta clear it already.
- **One source of truth**: member lists and `is_hidden()` both read `Hidden_Users::get_hidden_user_ids()`. Keep it that way, or a user can be missing from lists while their profile still loads.
- **Release ZIP contents**: the release workflow builds the ZIP with `rsync --exclude-from=.distignore`. Add new development-only files to both `.distignore` and `.gitattributes`.
