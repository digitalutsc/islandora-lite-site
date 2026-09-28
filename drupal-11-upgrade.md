# Drupal 11 upgrade report

Branch `drupal-11`. Data collected 2026-09-28 from packages.drupal.org, Packagist and GitHub.

## Summary

| | Before | After |
|---|---|---|
| drupal/core-recommended | 10.6.10 | **11.4.8** (`^11.4`) |
| PHP | `^7.4 \|\| ^8` | `>=8.3` |
| drush/drush | 12.5.3 | 13.8.0 (`^13`) |
| phpunit/phpunit (dev) | `^9.6` | `^11` |
| Template version | 2.2.4 | 3.0.0 |

Other `composer.json` changes needed for Drupal 11:

- `drupal/hal ^2.0.5` and `drupal/rdf ^3.0@beta` are now required directly. Both modules are enabled in `config/sync/core.extension.yml` but were removed from core in Drupal 11. rdf is held at 3.0.0-beta2 because `drupal/jsonld` 3.0.5 requires `drupal/rdf ^3.0@beta`, and rdf 4.0.0 cannot be installed with it.
- `webflo/drupal-finder ^1.3` is now required. `scripts/composer/ScriptHandler.php` uses `DrupalFinder\DrupalFinder`, which came in through Drush 12 but is not a dependency of Drush 13.
- `symfony/runtime` and `php-http/discovery` were added to `config.allow-plugins`. Drupal 11.4 core requires `symfony/runtime`, and drupal/recommended-project 11.4 allows both.

`composer.lock` was regenerated from `composer.json` only, without `composer_site.json`. It installs cleanly, and every patch applies.

## Upgraded drupal.org modules (composer.json)

★ marks a major-version jump. Test these before deploying and run `drush updb`.

| Module | Before | After | Notes |
|---|---|---|---|
| ableplayer | 3.4.2 | 3.4.3 | patch still applies |
| advanced_search | 2.4.2 | 2.4.5 | patch still applies |
| advancedqueue | 1.6.0 | 1.7.0 | |
| better_social_sharing_buttons ★ | 4.1.0 | 5.0.0 | rewritten with OOP hooks and partial templates; **patch re-rolled** (see Patches). 5.x post_update converts the `services` setting format. |
| devel | 5.4.0 | 5.5.0 | |
| filemime ★ | 1.13.0 | 2.0.2 | 2.x requires `^11.2` |
| filter_perms | 2.0.2 | 2.0.3 | |
| fontawesome ★ | 2.26.0 | 3.0.0 | 2.x supports only `^9.4 \|\| ^10`, so 3.x is required for D11 |
| geolocation | 3.14.0 | 3.15.0 | 4.0.0 exists, but **controlled_access_terms 2.6.0 requires `geolocation ^3.2`** |
| imagemagick ★ | 4.0.2 | 5.0.1 | 5.x requires `^11.3` |
| islandora_mirador ★ | 2.4.2 | 3.0.1 | patch still applies |
| jsonld | 3.0.2 | 3.0.5 | |
| jwt | 2.3.1 | 2.4.0 | |
| media_file_delete | 1.3.1 | 1.3.2 | |
| rest_oai_pmh | 2.3.2 | 2.3.3 | `patch-files` still overwrites `mods.html.twig` |
| search_api_solr | 4.3.10 | 4.4.0 | 4.4 requires `^11.3`. Regenerate/redeploy the Solr config set after the upgrade. |
| term_condition | 2.0.4 | 2.0.5 | |
| views_bulk_operations | 4.4.5 | 4.4.8 | |
| views_flipped_table ★ | 2.0.3 | 3.0.0 | patch still applies |

**Held back:** `drupal/facets` stays at **2.0.10**. facets 3.0.7 supports D11, but two packages require `facets ^2`:
- `drupal/advanced_search` 2.4.5, the latest release;
- our `islandora_lite/facets_year_range`.

The remaining drupal.org modules in `composer.json` were already on their latest D11-compatible release. Their constraints were tightened only to the current version:
- admin_toolbar 3.6.3, advancedqueue_runner 2.0.5, archive_list_contents 2.0.0, citation_select 2.1.1, config_update 2.0.0-alpha4, context/context_ui 5.0.0-rc2
- controlled_access_terms 2.6.0, csvfile_formatter 1.0.26, field_group 4.0.0, field_permissions 1.5.0, fullcalendar_solr 1.0.2, group 3.3.5 (4.x is alpha)
- json_field 1.7.0, json_field_processor 1.0.x-dev, media_library_edit 3.0.5, media_thumbnails 2.0.0 (and its _jp2, _pdf, _tiff, _video submodules), migrate_plus 6.0.10, migrate_source_csv 3.8.0
- openseadragon 3.0.1, pathauto 1.15.0, pdf 1.3.0, replaywebpage 1.0.1, restui 1.22.0, search_api_location 1.0.0-alpha4, taxonomy_manager 2.0.23
- triplestore_indexer 2.1.2, views_bulk_edit 3.0.1, views_data_export 1.10.0, views_field_view 1.0.0, views_timelinejs 4.2.2

## Upgraded drupal.org modules (composer_site.json)

`composer_site.json` is untracked. Its constraints were updated in place.

| Module | Before | After | Notes |
|---|---|---|---|
| anchor_link | ^3.0 | ^3.0.6 | |
| backup_migrate | ^5.0 | ^5.1.5 | |
| bootstrap_barrio | ^5.5 | ^5.5.20 | |
| color ★ | ^1.0 | ^2.0@alpha | 2.0.0-alpha1 is the only release that supports D11 |
| drupal_libcal | 1.x-dev@dev | ^1.0.2 | first stable release |
| embed | ^1.7 | ^1.10 | |
| extlink ★ | ^1.7 | ^3.0 | 3.x requires `^11.3` |
| file_extractor ★ | ^4.1 | ^5.0.1 | 5.x requires `^11.4` |
| geofield ★ | ^1.57 | ^10.3.4 | new version scheme |
| google_analytics | ^4.0 | ^4.0.3 | |
| google_tag | ^2.0 | ^2.0.9 | |
| leaflet | ^10.2 | ^10.4.12 | |
| masquerade | ^2.0@RC | ^2.2 | |
| memcache | ^2.5 | ^2.8 | |
| metatag | ^2.1 | ^2.2 | |
| monolog | ^3.0 | ^3.1 | |
| oembed_providers ★ | ^2.1 | ^3.0 | |
| quickedit ★ | ^1.0 | ^2.0.1 | 1.x supports only `^9.4 \|\| ^10` |
| recaptcha / recaptcha_v3 | ^3.2 / ^2.0 | ^3.5 / ^2.0.5 | |
| redirect | ^1.9 | ^1.13 | |
| views_entity_form_field | ^1.1 | ^1.2 | 2.x is alpha |
| views_url_path_arguments | ^1.2 | ^1.7 | |
| webform | ^6.2 | ^6.3.1 | |

Constraints were tightened only for these; they were already on their latest D11-compatible release:
- block_class, cer (5.0.0-beta4), contact_block, dblog_filter, ds (3.37; 5.x is alpha and D10-only), entity_reference_purger (beta8), form_mode_control
- jquery_once, media_file_delete, recogito_integration (alpha1), search_api_glossary, structure_sync, term_merge, term_reference_change (beta5)
- textarea_limit, translatable_menu_link_uri, ultimate_cron (beta1), views_arg_entity_field (beta1), views_geojson, webform_validation

## Upgraded third-party GitHub modules

| Package | Before | After | D11 |
|---|---|---|---|
| discoverygarden/islandora_hocr | v1.4.1 | v1.4.3 | yes (`^10 \|\| ^11`), patch still applies |
| mjordan/islandora_workbench_integration | v1.2.0 | v1.2.1 | yes, patch still applies |
| discoverygarden/dgi_fixity | v1.4.6 | v1.4.6 | yes, already latest |
| islandora/views_nested_details (Islandora-Labs) | 1.1.1 | 1.1.1 | yes, already latest |
| born-digital/islandora_iiif_hocr | 2.0.7 | 2.0.7 | **no**, see Blockers |

## digitalutsc modules (not changed, report only)

"core req" is `core_version_requirement` from the module's `.info.yml`. "Blocks composer" means the package cannot be installed alongside Drupal 11 at all.

| Package | Repo | Constraint | Locked | Latest tag → core req | Default branch → core req | D11 ready |
|---|---|---|---|---|---|---|
| digitalutsc/group_solr | [group_solr](https://github.com/digitalutsc/group_solr) | ^1.0 | 1.0.2 | 1.0.2 → `^8.8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes |
| digitalutsc/islandora_group | [islandora_group](https://github.com/digitalutsc/islandora_group) | ^2.0 | 2.0.4 | 2.0.4 → `^8.8 \|\| ^9 \|\| ^10 \|\| ^11` | 2.x → same | yes |
| digitalutsc/private_files_adapter | [private_files_adapter](https://github.com/digitalutsc/private_files_adapter) | ^1.0 | 1.0.0 | 1.0.0 → `^8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes |
| digitalutsc/media_fits | [media_fits](https://github.com/digitalutsc/media_fits) | ^1.0 | 1.0.2 | 1.0.2 → `^8.8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes |
| islandora/islandora_iiif | [islandora_iiif](https://github.com/digitalutsc/islandora_iiif) | ^1.0 | 1.1.0 | 1.1.0 → `^9 \|\| ^10 \|\| ^11` | 1.x → same | yes |
| islandora_lite/ableplayer_extend | [ableplayer_extend](https://github.com/digitalutsc/ableplayer_extend) | ^1.0 | 1.0.0 | 1.0.0 → `^9.3 \|\| ^10 \|\| ^11` | main → same | yes |
| islandora_lite/facets_year_range | [facets_year_range](https://github.com/digitalutsc/facets_year_range) | **1.0.1** | 1.0.1 | 1.0.4 → `^8.9 \|\| ^9.2 \|\| ^10 \|\| ^11` | main → same | **pinned 1.0.1 is not** (`^8.9 \|\| ^9.2 \|\| ^10`). 1.0.4 is. Bump the pin to 1.0.4. |
| islandora_lite/islandora_breadcrumbs | [islandora_breadcrumbs](https://github.com/digitalutsc/islandora_breadcrumbs) | ^1.0 | 1.0.1 | 1.0.1 → `^8 \|\| ^9 \|\| ^10` | main → same | **no** (installs, but Drupal refuses to enable it) |
| islandora_lite/islandora_object_thumbnail | [islandora_object_thumbnail](https://github.com/digitalutsc/islandora_object_thumbnail) | ^1.0 | 1.0.2 | 1.0.2 → `^8.8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes |
| islandora_lite/islandora_search_processor | [islandora_search_processor](https://github.com/digitalutsc/islandora_search_processor) | ^1.0 | 1.0.1 | 1.0.1 → `^9.2 \|\| ^10.0 \|\| ^11` | main → same | yes |
| islandora_lite/islandora_site_name | [islandora_site_name](https://github.com/digitalutsc/islandora_site_name) | ^1.1 | 1.1.3 | 1.1.3 → `^8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes |
| islandora_lite/termwithuri_condition | [termwithuri_condition](https://github.com/digitalutsc/termwithuri_condition) | ^1.0 | 1.0.1 | 1.0.1 → `^9 \|\| ^10 \|\| ^11` | main → same | yes |
| islandora_lite/islandora_iiif_hocr_extend | [islandora_iiif_hocr_extend](https://github.com/digitalutsc/islandora_iiif_hocr_extend) | ^1.0 | 1.0.5 | 1.0.5 → `^8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes |
| digitalutsc/drupal_hero_banner | [hero_banner](https://github.com/digitalutsc/hero_banner) | ^1.0 | (site) | 1.0.2 → `^8 \|\| ^9 \|\| ^10` | main → same | **no, blocks composer**: 1.0.2 requires `drupal/image_widget_crop ^2.4` (D10-only), and 1.0.0–1.0.1 require `field_group ^3.4` |
| digitalutsc/advanced_search_tips | [advanced_search_tips](https://github.com/digitalutsc/advanced_search_tips) | ^1.2 | (site) | 1.2.1 → `^8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | **no, blocks composer**: requires `drupal/fontawesome ^2`, and fontawesome 2.x is D10-only |
| islandora_lite/serve_iiif_file | [serve_iiif_file](https://github.com/digitalutsc/serve_iiif_file) | ^1.0@beta | (site) | 1.0.0 → `^8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes |
| drupal/jsonld_markup | [jsonld_markup](https://github.com/digitalutsc/jsonld_markup) | ^1.0@beta | (site) | 1.0.0-bata5 → `>=8` | main → same | yes (open-ended constraint; tag name has a typo, "bata5") |
| islandora_lite/relation_extend | [relation_extend](https://github.com/digitalutsc/relation_extend) | ^1.0 | (site) | 1.0.1 → `^10 \|\| ^11` | main → same | yes |
| drupal/group_concat | [group_concat](https://github.com/digitalutsc/group_concat) | ^1.0 | (site) | 1.0.1 → `^8 \|\| ^9 \|\| ^10` | main → same | **no** (installs, but Drupal refuses to enable it) |
| islandora_lite/advanced_search_default_sort_override | [advanced_search_default_sort_override](https://github.com/digitalutsc/advanced_search_default_sort_override) | ^1.0 | (site) | 1.0.4 → `^10 \|\| ^11` | main → same | yes |
| islandora_lite/xmlsitemap_extend | [xmlsitemap_extend](https://github.com/digitalutsc/xmlsitemap_extend) | ^1.0 | (site) | 1.0.1 → `^10.3 \|\| ^11` | main → same | yes |
| digitalutsc/temporary_downloadable_media | [temporary_downloadable_media](https://github.com/digitalutsc/temporary_downloadable_media) | ^1.0 | (site) | 1.0.2 → `^10 \|\| ^11` | main → same | yes |
| digitalutsc/leafletjs | [leafletjs](https://github.com/digitalutsc/leafletjs) | ^1.1 | (site) | 1.1.2 → `^9 \|\| ^10 \|\| ^11` | main → same | yes |
| borisay/assign_calc | [assign_calc](https://github.com/digitalutsc/assign_calc) | ^1.0 | (site) | 1.0.1 → `^8 \|\| ^9 \|\| ^10 \|\| ^11` | utsc → same | yes |
| islandora_lite/rest_translation_util | [rest_translation_util](https://github.com/digitalutsc/rest_translation_util) | dev-main | (site) | 1.1.0 → `^8.8 \|\| ^9 \|\| ^10` | main → same | **no** (installs, but Drupal refuses to enable it) |
| drupal/language_switcher_popup | [language_switcher_popup](https://github.com/digitalutsc/language_switcher_popup) | ^1.0 | (site) | 1.0.3 → `^9 \|\| ^10 \|\| ^11` | main → same | yes |
| digitalutsc/static_metadata_records | [static_metadata_records](https://github.com/digitalutsc/static_metadata_records) | ^1.0 | (site) | 1.0.0 → `^10 \|\| ^11` | main → same | yes |
| digitalutsc/selective_revisions | [selective_revisions](https://github.com/digitalutsc/selective_revisions) | dev-main | (site) | no tags | main → `^10.3 \|\| ^11` | yes |
| digitalutsc/media_file_relocate | [media_file_relocate](https://github.com/digitalutsc/media_file_relocate) | dev-main | (site) | no tags | main → `^10.3 \|\| ^11` | yes |

Composer also moved `digitalutsc/islandora_group` from 2.0.2 to 2.0.4 and `digitalutsc/media_fits` from 1.0.1 to 1.0.2. Their constraints were not changed; these releases already satisfied them.

## Blockers (kept, not resolved)

These were intentionally left in place. **With `composer_site.json` merged in, `composer update` fails** until the first group is resolved.

**Blocks composer (cannot be installed with Drupal 11):**

| Package | File | Reason |
|---|---|---|
| drupal/getjwtonlogin | composer_site.json | latest 2.0.3 requires `drupal/core ^8 \|\| ^9 \|\| ^10`; no D11 release or branch |
| drupal/media_revisions_ui | composer_site.json | latest 2.1.0 requires `^9.3 \|\| ^10`; the dev-3.x branch is `^10.1` |
| digitalutsc/advanced_search_tips (ours) | composer_site.json | requires `drupal/fontawesome ^2` |
| digitalutsc/drupal_hero_banner (ours) | composer_site.json | requires `drupal/image_widget_crop ^2.4` (D10-only) |

Verified: with only these four removed, `composer update -W` resolves with the overlay merged in.

**Update (2026-09-28):** these four were then removed from the local `composer_site.json` to get a working D11 build. `composer update -W` with the overlay now succeeds, and every patch applies. Sites that have these modules enabled must uninstall them on Drupal 10 before upgrading, or wait for D11-compatible releases.

**Installs, but Drupal will not enable it on D11.** The `core_version_requirement` in `.info.yml` excludes 11:

| Package | File | core_version_requirement |
|---|---|---|
| born-digital/islandora_iiif_hocr 2.0.7 | composer.json | `^9 \|\| ^10`, including after `islandora_iiif_hocr_lite.patch`; there is no newer upstream release |
| islandora_lite/islandora_breadcrumbs (ours) | composer.json | `^8 \|\| ^9 \|\| ^10` |
| islandora_lite/facets_year_range 1.0.1 (ours) | composer.json | `^8.9 \|\| ^9.2 \|\| ^10` (1.0.4 fixes this) |
| drupal/group_concat (ours) | composer_site.json | `^8 \|\| ^9 \|\| ^10` |
| islandora_lite/rest_translation_util (ours) | composer_site.json | `^8.8 \|\| ^9 \|\| ^10` |

**Pre-release only.** These are D11-compatible but have no stable release:
- color 2.0.0-alpha1
- rdf 3.0.0-beta2
- cer, entity_reference_purger, term_reference_change, ultimate_cron, views_arg_entity_field (beta)
- recogito_integration, search_api_location, config_update (alpha)
- context (rc)

## Core modules removed in Drupal 11 that this config uses

| Module | Enabled in config/sync | Replacement |
|---|---|---|
| hal | yes | `drupal/hal ^2.0.5` (contrib) |
| rdf | yes (`rdf.mapping.*` config) | `drupal/rdf ^3.0@beta` (contrib) |

Nothing else in `core.extension.yml` was removed from core in D11 (checked: action, book, forum, statistics, tracker, color, quickedit, tour, ckeditor). The `color` and `quickedit` modules requested by `composer_site.json` are contrib.

## Patches

| Package | Patch | Status |
|---|---|---|
| drupal/ableplayer 3.4.3 | ableplayer_fixentitydisplayissue.patch | applies |
| drupal/advanced_search 2.4.5 | advanced_search_issue_96.patch | applies |
| drupal/better_social_sharing_buttons 5.0.0 | better_social_sharing.patch → **better_social_sharing_d11.patch** | **re-rolled** |
| drupal/controlled_access_terms 2.6.0 | controlled_access_term_reversed_typerelation.patch | applies (unchanged version) |
| drupal/group 3.3.5 | view_batch_export_issue.patch | applies (unchanged version) |
| drupal/islandora_mirador 3.0.1 | islandora_mirador_lite.patch | applies |
| drupal/media_thumbnails 2.0.0 | media_thumbnails_march_17_2025.patch | applies (unchanged version) |
| drupal/views_flipped_table 3.0.0 | accessibility_views_flipped_table_convert_to_layout_table.patch | applies |
| discoverygarden/islandora_hocr v1.4.3 | islandora_hocr_lite.patch | applies |
| born-digital/islandora_iiif_hocr 2.0.7 | islandora_iiif_hocr_lite.patch | applies (unchanged version) |
| mjordan/islandora_workbench_integration v1.2.1 | workbench_integration.patch | applies |

`better_social_sharing_d11.patch` re-implements the "preferred URL field" feature for the 5.x code base:
- `BetterSocialSharingButtonsHooks::preprocessBetterSocialSharingButtons()` overrides `items.page_url` with the configured link field's URL. Every partial template and the copy-link button (`data-page-url`) then use it, so the old template/JS changes are no longer needed.
- The settings form keeps the `use_url_field` → `preferred_url` setting.
- `preferred_url` was added to the config schema.

The patch URL in `composer.json` follows the existing convention (`raw.githubusercontent.com/.../refs/heads/2.x/assets/patches/better_social_sharing_d11.patch`). **It only resolves after this branch is merged into `2.x`.** Until then, `composer install` from the committed `composer.json` fails on that patch. For local testing, point the entry at `assets/patches/better_social_sharing_d11.patch`.

The old `better_social_sharing.patch` is still in `assets/patches/` for 2.x sites.

## Upgrading an existing site

1. PHP 8.3+ in the container, and `ext-imagick` (media_thumbnails_pdf requires it).
2. Resolve or remove the four composer blockers in `composer_site.json`.
3. Uninstall modules that won't enable on D11, or update them first: islandora_iiif_hocr, islandora_breadcrumbs, group_concat, rest_translation_util, and bump facets_year_range to 1.0.4. Do this **on Drupal 10**, before upgrading core.
4. `composer update -W`
5. `drush updb -y && drush cr`
6. Re-save the search_api_solr server and redeploy the Solr config set (search_api_solr 4.4).
7. `drush cex -y` and review the `config/sync` diff. Expect changes from the major bumps: better_social_sharing_buttons 5, fontawesome 3, islandora_mirador 3, views_flipped_table 3, oembed_providers 3, extlink 3, file_extractor 5, imagemagick 5, and filemime 2.
