# Drupal 11 upgrade report

Branch `drupal-11`. Data collected 2026-09-28 from packages.drupal.org, Packagist and GitHub. Full build with the `composer_site.json` overlay verified 2026-09-29 (see [Build verification](#build-verification-2026-09-29)).

## Summary

| | Before | After |
|---|---|---|
| drupal/core-recommended | 10.6.10 | **11.4.8** (`^11.4`) |
| PHP | `^7.4 \|\| ^8` | `>=8.3` |
| drush/drush | 12.5.3 | 13.8.0 (`^13`) |
| phpunit/phpunit (dev) | `^9.6` | `^11` |
| Template version | 2.2.4 | 3.0.0 |

Other `composer.json` changes needed for Drupal 11:

- `drupal/hal ^2.0` and `drupal/rdf ^3.0@beta` are now required directly. Both modules are enabled in `config/sync/core.extension.yml` but were removed from core in Drupal 11. rdf is held at 3.0.0-beta2 because `drupal/jsonld` 3.0.5 requires `drupal/rdf ^3.0@beta`, and rdf 4.0.0 cannot be installed with it.
- `webflo/drupal-finder ^1.3` is now required. `scripts/composer/ScriptHandler.php` uses `DrupalFinder\DrupalFinder`, which came in through Drush 12 but is not a dependency of Drush 13.
- `symfony/runtime` and `php-http/discovery` were added to `config.allow-plugins`. Drupal 11.4 core requires `symfony/runtime`, and drupal/recommended-project 11.4 allows both.

`composer.lock` now covers `composer.json` plus the merged `composer_site.json` overlay. It installs cleanly, and every patch applies.

## Constraint style (2026-09-29)

Caret constraints in `composer.json` and `composer_site.json` use two parts (`^2.0`, not `^2.0.23`). 46 constraints were shortened. A two-part caret keeps the same upper bound, so `composer update` resolves the same releases: after the change `composer update --lock` reported nothing to modify, and all 341 locked versions are unchanged. Each shortened constraint was checked against its source (packages.drupal.org, Packagist, or the GitHub tags): the `X.Y` line exists, and its newest stable release is the one locked.

Shortening lowers the minimum, so older releases become allowed. For drupal.org packages that is safe, because each release's metadata carries its `drupal/core` requirement and Composer refuses the Drupal 10-only ones. `digitalutsc/drupal_hero_banner` is safe for a similar reason: its 1.0.0–1.0.2 tags require `drupal/image_widget_crop ^2.4`, and 2.4.0 supports only `^8 || ^9 || ^10`.

**Four constraints keep three parts**, because their `composer.json` declares no `drupal/core` requirement and the two-part form would admit Drupal 10-only releases that Composer cannot reject. Drupal then refuses to enable them, which is the "incompatible with this version of Drupal core" error seen on 2026-09-29.

| Package | Constraint | Releases a two-part caret would admit | Their `core_version_requirement` |
|---|---|---|---|
| islandora_lite/islandora_breadcrumbs | ^1.0.2 | 1.0.0, 1.0.1 | `^8 \|\| ^9 \|\| ^10` |
| drupal/group_concat | ^1.0.2 | 1.0.1 | `^8 \|\| ^9 \|\| ^10` |
| islandora_lite/rest_translation_util | ^1.1.1 | 1.1.0 | `^8.8 \|\| ^9 \|\| ^10` |
| discoverygarden/dgi_fixity | ^1.4.6 | v1.4.0, v1.4.1 | `^9 \|\| ^10` |

These can move to two parts once the modules declare `drupal/core` in their `composer.json`, or once the old releases are no longer a concern.

## Build verification (2026-09-29)

A complete build (`composer.json` + `composer_site.json`) was checked against this report. The build was verified at the Composer and filesystem level only; no running site was available, so `drush updb`, `drush cim` and enabling modules on Drupal 11 are **not yet tested**.

| Check | Result |
|---|---|
| `composer.lock` | 341 packages (316 + 25 dev). `composer validate`: valid; the only warnings are for the intentional exact pins (group, islandora_workbench_integration, timeline3, jsoneditor) |
| Lock vs `composer.json` + overlay | in sync (no lock-hash warning) |
| Versions in this report | all match the lock, including every "After" and "Locked" value below |
| Change from the committed lock (`10a1608`) | 4 added (`digitalutsc/drupal_hero_banner` 1.0.3, `drupal/image_widget_crop` 3.0.0, `drupal/crop` 2.6.0, `drupal/imce` 3.1.5); 14 changed (facets_year_range 1.0.1→1.0.4, islandora_breadcrumbs 1.0.1→1.0.2, group_concat 1.0.1→1.0.2, rest_translation_util dev-main→1.1.1, and ten symfony/* components →v7.4.20); none removed |
| Blockers removed | `drupal/getjwtonlogin`, `drupal/media_revisions_ui`, `digitalutsc/advanced_search_tips` are absent from the lock |
| `composer.json` patches (cweagans) | 11 of 11 recorded as applied in each package's `PATCHES.txt`; 12 of 12 after `islandora_iiif_hocr_d11.patch` was added the same day |
| `composer_site.json` patches (`applyDrupalPatches`) | none defined, so nothing to apply |
| `rm -rf web/modules/contrib/islandora` | done; directory absent |
| `patch-files` (`rest_oai_pmh` `mods.html.twig`) | replaced with the `islandora_lite_installation` `main` copy, as designed. **The local reference copy `assets/templates/mods.html.twig` has drifted from it:** the local copy maps `Family`→`family` and uses `loop.index0`; the installed remote copy has no `Family` mapping. Decide which is correct and update the other |
| `settings.php` scaffold append | present: `config_sync_directory = ../config/sync`, `file_private_path = sites/default/private` |
| `core_version_requirement` of all 161 contrib modules and themes in `web/` | only `islandora_iiif_hocr` (`^9 \|\| ^10`) excluded Drupal 11; after `islandora_iiif_hocr_d11.patch` none do |
| Loading every contrib class on the isle-dc Drupal 11.4 / PHP 8.4 runtime (4,248 classes, each in its own process) | one incompatible declaration: `media_thumbnails_video` `VideoExtendedFormatter::viewElements()`, fixed by `media_thumbnails_video_d11.patch` (see Blockers). No other fatal errors. 49 PHP 8.4 "implicitly nullable parameter" deprecations, not fatal until PHP 9: structure_sync (most), controlled_access_terms, views_timelinejs, context_ui, google_analytics, json_field_processor, color, search_api_glossary, and the `codementality/flysystem-stream-wrapper` library |
| `composer audit --locked` | no security advisories |
| Host `composer install` | fails on the host (PHP 8.5.8, no `ext-imagick`): `drupal/media_thumbnails_pdf` 2.0.1 requires `ext-imagick`. Build inside the container, or pass `--ignore-platform-req=ext-imagick` on the host |

## Upgraded drupal.org modules (composer.json)

★ marks a major-version jump. Test these before deploying and run `drush updb`.

| Module | Before | After | Notes | Upgraded version links | Module page |
|---|---|---|---|---|---|
| ableplayer | 3.4.2 | 3.4.3 | patch still applies | [3.4.3](https://www.drupal.org/project/ableplayer/releases/3.4.3) | [ableplayer](https://www.drupal.org/project/ableplayer) |
| advanced_search | 2.4.2 | 2.4.5 | patch still applies | [2.4.5](https://www.drupal.org/project/advanced_search/releases/2.4.5) | [advanced_search](https://www.drupal.org/project/advanced_search) |
| advancedqueue | 1.6.0 | 1.7.0 | | [8.x-1.7](https://www.drupal.org/project/advancedqueue/releases/8.x-1.7) | [advancedqueue](https://www.drupal.org/project/advancedqueue) |
| better_social_sharing_buttons ★ | 4.1.0 | 5.0.0 | rewritten with OOP hooks and partial templates; **patch re-rolled** (see Patches). 5.x post_update converts the `services` setting format. | [5.0.0](https://www.drupal.org/project/better_social_sharing_buttons/releases/5.0.0) | [better_social_sharing_buttons](https://www.drupal.org/project/better_social_sharing_buttons) |
| devel | 5.4.0 | 5.5.0 | | [5.5.0](https://www.drupal.org/project/devel/releases/5.5.0) | [devel](https://www.drupal.org/project/devel) |
| filemime ★ | 1.13.0 | 2.0.2 | 2.x requires `^11.2` | [2.0.2](https://www.drupal.org/project/filemime/releases/2.0.2) | [filemime](https://www.drupal.org/project/filemime) |
| filter_perms | 2.0.2 | 2.0.3 | | [2.0.3](https://www.drupal.org/project/filter_perms/releases/2.0.3) | [filter_perms](https://www.drupal.org/project/filter_perms) |
| fontawesome ★ | 2.26.0 | 3.0.0 | 2.x supports only `^9.4 \|\| ^10`, so 3.x is required for D11 | [3.0.0](https://www.drupal.org/project/fontawesome/releases/3.0.0) | [fontawesome](https://www.drupal.org/project/fontawesome) |
| geolocation | 3.14.0 | 3.15.0 | 4.0.0 exists, but **controlled_access_terms 2.6.0 requires `geolocation ^3.2`** | [8.x-3.15](https://www.drupal.org/project/geolocation/releases/8.x-3.15) | [geolocation](https://www.drupal.org/project/geolocation) |
| imagemagick ★ | 4.0.2 | 5.0.1 | 5.x requires `^11.3` | [5.0.1](https://www.drupal.org/project/imagemagick/releases/5.0.1) | [imagemagick](https://www.drupal.org/project/imagemagick) |
| islandora_mirador ★ | 2.4.2 | 3.0.1 | patch still applies | [3.0.1](https://www.drupal.org/project/islandora_mirador/releases/3.0.1) | [islandora_mirador](https://www.drupal.org/project/islandora_mirador) |
| jsonld | 3.0.2 | 3.0.5 | | [3.0.5](https://www.drupal.org/project/jsonld/releases/3.0.5) | [jsonld](https://www.drupal.org/project/jsonld) |
| jwt | 2.3.1 | 2.4.0 | | [2.4.0](https://www.drupal.org/project/jwt/releases/2.4.0) | [jwt](https://www.drupal.org/project/jwt) |
| media_file_delete | 1.3.1 | 1.3.2 | | [1.3.2](https://www.drupal.org/project/media_file_delete/releases/1.3.2) | [media_file_delete](https://www.drupal.org/project/media_file_delete) |
| rest_oai_pmh | 2.3.2 | 2.3.3 | `patch-files` still overwrites `mods.html.twig` (verified in the build; the local `assets/templates` copy differs from the one installed, see Build verification) | [2.3.3](https://www.drupal.org/project/rest_oai_pmh/releases/2.3.3) | [rest_oai_pmh](https://www.drupal.org/project/rest_oai_pmh) |
| search_api_solr | 4.3.10 | 4.4.0 | 4.4 requires `^11.3`. Regenerate/redeploy the Solr config set after the upgrade. | [4.4.0](https://www.drupal.org/project/search_api_solr/releases/4.4.0) | [search_api_solr](https://www.drupal.org/project/search_api_solr) |
| term_condition | 2.0.4 | 2.0.5 | | [2.0.5](https://www.drupal.org/project/term_condition/releases/2.0.5) | [term_condition](https://www.drupal.org/project/term_condition) |
| views_bulk_operations | 4.4.5 | 4.4.8 | | [4.4.8](https://www.drupal.org/project/views_bulk_operations/releases/4.4.8) | [views_bulk_operations](https://www.drupal.org/project/views_bulk_operations) |
| views_flipped_table ★ | 2.0.3 | 3.0.0 | patch still applies | [3.0.0](https://www.drupal.org/project/views_flipped_table/releases/3.0.0) | [views_flipped_table](https://www.drupal.org/project/views_flipped_table) |

**Held back:** `drupal/facets` stays at **2.0.10**. facets 3.0.7 supports D11, but two packages require `facets ^2`:
- `drupal/advanced_search` 2.4.5, the latest release;
- our `islandora_lite/facets_year_range`.

The remaining drupal.org modules in `composer.json` were already on their latest D11-compatible release. Their constraints only name the current minor line (for example `^3.6` for admin_toolbar 3.6.3):
- admin_toolbar 3.6.3, advancedqueue_runner 2.0.5, archive_list_contents 2.0.0, citation_select 2.1.1, config_update 2.0.0-alpha4, context/context_ui 5.0.0-rc2
- controlled_access_terms 2.6.0, csvfile_formatter 1.0.26, field_group 4.0.0, field_permissions 1.5.0, fullcalendar_solr 1.0.2, group 3.3.5 (4.x is alpha)
- json_field 1.7.0, json_field_processor 1.0.x-dev, media_library_edit 3.0.5, media_thumbnails 2.0.0 (and its _jp2, _pdf, _tiff, _video submodules), migrate_plus 6.0.10, migrate_source_csv 3.8.0
- openseadragon 3.0.1, pathauto 1.15.0, pdf 1.3.0, replaywebpage 1.0.1, restui 1.22.0, search_api_location 1.0.0-alpha4, taxonomy_manager 2.0.23
- triplestore_indexer 2.1.2, views_bulk_edit 3.0.1, views_data_export 1.10.0, views_field_view 1.0.0, views_timelinejs 4.2.2

## Upgraded drupal.org modules (composer_site.json)

`composer_site.json` is untracked. Its constraints were updated in place.

| Module | Before | After | Notes | Upgraded version links | Module page |
|---|---|---|---|---|---|
| anchor_link | ^3.0 | ^3.0 | | [3.0.6](https://www.drupal.org/project/anchor_link/releases/3.0.6) | [anchor_link](https://www.drupal.org/project/anchor_link) |
| backup_migrate | ^5.0 | ^5.1 | | [5.1.5](https://www.drupal.org/project/backup_migrate/releases/5.1.5) | [backup_migrate](https://www.drupal.org/project/backup_migrate) |
| bootstrap_barrio | ^5.5 | ^5.5 | | [5.5.20](https://www.drupal.org/project/bootstrap_barrio/releases/5.5.20) | [bootstrap_barrio](https://www.drupal.org/project/bootstrap_barrio) |
| color ★ | ^1.0 | ^2.0@alpha | 2.0.0-alpha1 is the only release that supports D11 | [2.0.0-alpha1](https://www.drupal.org/project/color/releases/2.0.0-alpha1) | [color](https://www.drupal.org/project/color) |
| drupal_libcal | 1.x-dev@dev | ^1.0 | first stable release | [1.0.2](https://www.drupal.org/project/drupal_libcal/releases/1.0.2) | [drupal_libcal](https://www.drupal.org/project/drupal_libcal) |
| embed | ^1.7 | ^1.10 | | [8.x-1.10](https://www.drupal.org/project/embed/releases/8.x-1.10) | [embed](https://www.drupal.org/project/embed) |
| extlink ★ | ^1.7 | ^3.0 | 3.x requires `^11.3` | [3.0.0](https://www.drupal.org/project/extlink/releases/3.0.0) | [extlink](https://www.drupal.org/project/extlink) |
| file_extractor ★ | ^4.1 | ^5.0 | 5.x requires `^11.4` | [5.0.1](https://www.drupal.org/project/file_extractor/releases/5.0.1) | [file_extractor](https://www.drupal.org/project/file_extractor) |
| geofield ★ | ^1.57 | ^10.3 | new version scheme | [10.3.4](https://www.drupal.org/project/geofield/releases/10.3.4) | [geofield](https://www.drupal.org/project/geofield) |
| google_analytics | ^4.0 | ^4.0 | | [4.0.3](https://www.drupal.org/project/google_analytics/releases/4.0.3) | [google_analytics](https://www.drupal.org/project/google_analytics) |
| google_tag | ^2.0 | ^2.0 | | [2.0.9](https://www.drupal.org/project/google_tag/releases/2.0.9) | [google_tag](https://www.drupal.org/project/google_tag) |
| leaflet | ^10.2 | ^10.4 | | [10.4.12](https://www.drupal.org/project/leaflet/releases/10.4.12) | [leaflet](https://www.drupal.org/project/leaflet) |
| masquerade | ^2.0@RC | ^2.2 | | [8.x-2.2](https://www.drupal.org/project/masquerade/releases/8.x-2.2) | [masquerade](https://www.drupal.org/project/masquerade) |
| memcache | ^2.5 | ^2.8 | | [8.x-2.8](https://www.drupal.org/project/memcache/releases/8.x-2.8) | [memcache](https://www.drupal.org/project/memcache) |
| metatag | ^2.1 | ^2.2 | | [2.2.0](https://www.drupal.org/project/metatag/releases/2.2.0) | [metatag](https://www.drupal.org/project/metatag) |
| monolog | ^3.0 | ^3.1 | | [3.1.0](https://www.drupal.org/project/monolog/releases/3.1.0) | [monolog](https://www.drupal.org/project/monolog) |
| oembed_providers ★ | ^2.1 | ^3.0 | | [3.0.0](https://www.drupal.org/project/oembed_providers/releases/3.0.0) | [oembed_providers](https://www.drupal.org/project/oembed_providers) |
| quickedit ★ | ^1.0 | ^2.0 | 1.x supports only `^9.4 \|\| ^10` | [2.0.1](https://www.drupal.org/project/quickedit/releases/2.0.1) | [quickedit](https://www.drupal.org/project/quickedit) |
| recaptcha / recaptcha_v3 | ^3.2 / ^2.0 | ^3.5 / ^2.0 | | [8.x-3.5](https://www.drupal.org/project/recaptcha/releases/8.x-3.5) / [2.0.5](https://www.drupal.org/project/recaptcha_v3/releases/2.0.5) | [recaptcha](https://www.drupal.org/project/recaptcha) / [recaptcha_v3](https://www.drupal.org/project/recaptcha_v3) |
| redirect | ^1.9 | ^1.13 | | [8.x-1.13](https://www.drupal.org/project/redirect/releases/8.x-1.13) | [redirect](https://www.drupal.org/project/redirect) |
| views_entity_form_field | ^1.1 | ^1.2 | 2.x is alpha | [8.x-1.2](https://www.drupal.org/project/views_entity_form_field/releases/8.x-1.2) | [views_entity_form_field](https://www.drupal.org/project/views_entity_form_field) |
| views_url_path_arguments | ^1.2 | ^1.7 | | [8.x-1.7](https://www.drupal.org/project/views_url_path_arguments/releases/8.x-1.7) | [views_url_path_arguments](https://www.drupal.org/project/views_url_path_arguments) |
| webform | ^6.2 | ^6.3 | | [6.3.1](https://www.drupal.org/project/webform/releases/6.3.1) | [webform](https://www.drupal.org/project/webform) |

Only the constraint style changed for these; they were already on their latest D11-compatible release:
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
| born-digital/islandora_iiif_hocr | 2.0.7 | 2.0.7 | yes, through `islandora_iiif_hocr_d11.patch` (no upstream D11 release) |

## digitalutsc modules

**Update (2026-09-29):** new digitalutsc releases add Drupal 11 to `core_version_requirement`, so the constraints below were bumped and `composer update -W` was run for just these packages. That update also moved the transitive symfony/* 7.4 components to v7.4.20.
- islandora_breadcrumbs 1.0.2
- group_concat 1.0.2
- rest_translation_util 1.1.1
- hero_banner 1.0.3
- facets_year_range 1.0.4

hero_banner is back in `composer_site.json`, and it pulls in `drupal/image_widget_crop` 3.0.0, `drupal/crop` 2.6.0 and `drupal/imce` 3.1.5. advanced_search_tips has no new release and is still blocked.

"core req" is `core_version_requirement` from the module's `.info.yml`. "Blocks composer" means the package cannot be installed alongside Drupal 11 at all. In the "Locked" column, "(overlay)" marks packages required by `composer_site.json`; since the 2026-09-29 build they are in `composer.lock` too.

| Package | Repo | Constraint | Locked | Latest tag → core req | Default branch → core req | D11 ready |
|---|---|---|---|---|---|---|
| digitalutsc/group_solr | [group_solr](https://github.com/digitalutsc/group_solr) | ^1.0 | 1.0.2 | 1.0.2 → `^8.8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes |
| digitalutsc/islandora_group | [islandora_group](https://github.com/digitalutsc/islandora_group) | ^2.0 | 2.0.4 | 2.0.4 → `^8.8 \|\| ^9 \|\| ^10 \|\| ^11` | 2.x → same | yes |
| digitalutsc/private_files_adapter | [private_files_adapter](https://github.com/digitalutsc/private_files_adapter) | ^1.0 | 1.0.0 | 1.0.0 → `^8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes |
| digitalutsc/media_fits | [media_fits](https://github.com/digitalutsc/media_fits) | ^1.0 | 1.0.2 | 1.0.2 → `^8.8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes |
| islandora/islandora_iiif | [islandora_iiif](https://github.com/digitalutsc/islandora_iiif) | ^1.0 | 1.1.0 | 1.1.0 → `^9 \|\| ^10 \|\| ^11` | 1.x → same | yes |
| islandora_lite/ableplayer_extend | [ableplayer_extend](https://github.com/digitalutsc/ableplayer_extend) | ^1.0 | 1.0.0 | 1.0.0 → `^9.3 \|\| ^10 \|\| ^11` | main → same | yes |
| islandora_lite/facets_year_range | [facets_year_range](https://github.com/digitalutsc/facets_year_range) | 1.0.4 | 1.0.4 | 1.0.4 → `^8.9 \|\| ^9.2 \|\| ^10 \|\| ^11` | main → same | yes (pin bumped from 1.0.1 on 2026-09-29) |
| islandora_lite/islandora_breadcrumbs | [islandora_breadcrumbs](https://github.com/digitalutsc/islandora_breadcrumbs) | ^1.0.2 | 1.0.2 | 1.0.2 → `^8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes (since 1.0.2) |
| islandora_lite/islandora_object_thumbnail | [islandora_object_thumbnail](https://github.com/digitalutsc/islandora_object_thumbnail) | ^1.0 | 1.0.2 | 1.0.2 → `^8.8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes |
| islandora_lite/islandora_search_processor | [islandora_search_processor](https://github.com/digitalutsc/islandora_search_processor) | ^1.0 | 1.0.1 | 1.0.1 → `^9.2 \|\| ^10.0 \|\| ^11` | main → same | yes |
| islandora_lite/islandora_site_name | [islandora_site_name](https://github.com/digitalutsc/islandora_site_name) | ^1.1 | 1.1.3 | 1.1.3 → `^8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes |
| islandora_lite/termwithuri_condition | [termwithuri_condition](https://github.com/digitalutsc/termwithuri_condition) | ^1.0 | 1.0.1 | 1.0.1 → `^9 \|\| ^10 \|\| ^11` | main → same | yes |
| islandora_lite/islandora_iiif_hocr_extend | [islandora_iiif_hocr_extend](https://github.com/digitalutsc/islandora_iiif_hocr_extend) | ^1.0 | 1.0.5 | 1.0.5 → `^8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes |
| digitalutsc/drupal_hero_banner | [hero_banner](https://github.com/digitalutsc/hero_banner) | ^1.0 | 1.0.3 (overlay) | 1.0.3 → `^8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes (since 1.0.3, which requires `image_widget_crop ^3` and `field_group ^4`; 1.0.2 and earlier block composer) |
| digitalutsc/advanced_search_tips | [advanced_search_tips](https://github.com/digitalutsc/advanced_search_tips) | ^1.2 | not installed (removed) | 1.2.1 → `^8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | **no, blocks composer**: requires `drupal/fontawesome ^2`, and fontawesome 2.x is D10-only |
| islandora_lite/serve_iiif_file | [serve_iiif_file](https://github.com/digitalutsc/serve_iiif_file) | ^1.0@beta | 1.0.0 (overlay) | 1.0.0 → `^8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes |
| drupal/jsonld_markup | [jsonld_markup](https://github.com/digitalutsc/jsonld_markup) | ^1.0@beta | 1.0.0-beta4 (overlay) | 1.0.0-bata5 → `>=8` | main → same | yes (open-ended constraint). The latest tag is misspelled "bata5", which Composer cannot parse, so it resolves to 1.0.0-beta4 |
| islandora_lite/relation_extend | [relation_extend](https://github.com/digitalutsc/relation_extend) | ^1.0 | 1.0.1 (overlay) | 1.0.1 → `^10 \|\| ^11` | main → same | yes |
| drupal/group_concat | [group_concat](https://github.com/digitalutsc/group_concat) | ^1.0.2 | 1.0.2 (overlay) | 1.0.2 → `^8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes (since 1.0.2) |
| islandora_lite/advanced_search_default_sort_override | [advanced_search_default_sort_override](https://github.com/digitalutsc/advanced_search_default_sort_override) | ^1.0 | 1.0.4 (overlay) | 1.0.4 → `^10 \|\| ^11` | main → same | yes |
| islandora_lite/xmlsitemap_extend | [xmlsitemap_extend](https://github.com/digitalutsc/xmlsitemap_extend) | ^1.0 | 1.0.1 (overlay) | 1.0.1 → `^10.3 \|\| ^11` | main → same | yes |
| digitalutsc/temporary_downloadable_media | [temporary_downloadable_media](https://github.com/digitalutsc/temporary_downloadable_media) | ^1.0 | 1.0.2 (overlay) | 1.0.2 → `^10 \|\| ^11` | main → same | yes |
| digitalutsc/leafletjs | [leafletjs](https://github.com/digitalutsc/leafletjs) | ^1.1 | 1.1.2 (overlay) | 1.1.2 → `^9 \|\| ^10 \|\| ^11` | main → same | yes |
| borisay/assign_calc | [assign_calc](https://github.com/digitalutsc/assign_calc) | ^1.0 | 1.0.1 (overlay) | 1.0.1 → `^8 \|\| ^9 \|\| ^10 \|\| ^11` | utsc → same | yes |
| islandora_lite/rest_translation_util | [rest_translation_util](https://github.com/digitalutsc/rest_translation_util) | ^1.1.1 | 1.1.1 (overlay) | 1.1.1 → `^8.8 \|\| ^9 \|\| ^10 \|\| ^11` | main → same | yes (since 1.1.1; was dev-main) |
| drupal/language_switcher_popup | [language_switcher_popup](https://github.com/digitalutsc/language_switcher_popup) | ^1.0 | 1.0.3 (overlay) | 1.0.3 → `^9 \|\| ^10 \|\| ^11` | main → same | yes |
| digitalutsc/static_metadata_records | [static_metadata_records](https://github.com/digitalutsc/static_metadata_records) | ^1.0 | 1.0.0 (overlay) | 1.0.0 → `^10 \|\| ^11` | main → same | yes |
| digitalutsc/selective_revisions | [selective_revisions](https://github.com/digitalutsc/selective_revisions) | dev-main | dev-main (overlay) | no tags | main → `^10.3 \|\| ^11` | yes |
| digitalutsc/media_file_relocate | [media_file_relocate](https://github.com/digitalutsc/media_file_relocate) | dev-main | dev-main (overlay) | no tags | main → `^10.3 \|\| ^11` | yes |

Composer also moved `digitalutsc/islandora_group` from 2.0.2 to 2.0.4 and `digitalutsc/media_fits` from 1.0.1 to 1.0.2. Their constraints were not changed; these releases already satisfied them.

## Blockers (kept, not resolved)

These were intentionally left in place. **With `composer_site.json` merged in, `composer update` fails** until the first group is resolved.

**Blocks composer (cannot be installed with Drupal 11):**

| Package | File | Reason |
|---|---|---|
| drupal/getjwtonlogin | composer_site.json | latest 2.0.3 requires `drupal/core ^8 \|\| ^9 \|\| ^10`; no D11 release or branch |
| drupal/media_revisions_ui | composer_site.json | latest 2.1.0 requires `^9.3 \|\| ^10`; the dev-3.x branch is `^10.1` |
| digitalutsc/advanced_search_tips (ours) | composer_site.json | requires `drupal/fontawesome ^2` |
| ~~digitalutsc/drupal_hero_banner (ours)~~ | composer_site.json | **resolved 2026-09-29**: 1.0.3 requires `image_widget_crop ^3` and supports D11, and it was re-added |

Verified: with only these four removed, `composer update -W` resolves with the overlay merged in.

**Update (2026-09-28):** these four were then removed from the local `composer_site.json` to get a working D11 build. `composer update -W` with the overlay now succeeds, and every patch applies. Sites that have these modules enabled must uninstall them on Drupal 10 before upgrading, or wait for D11-compatible releases.

**Update (2026-09-29):** hero_banner 1.0.3 was re-added, and `composer update` still succeeds with the overlay. getjwtonlogin, media_revisions_ui and advanced_search_tips remain removed.

**Resolved by a local patch (2026-09-29): islandora_iiif_hocr.** 2.0.7 declares `^9 \|\| ^10`, and there is no newer upstream release. `config/sync/core.extension.yml` enables it and `views.view.search_in_hocr.yml` depends on it, so a fresh Drupal 11 install from this config stopped with "module 'islandora_iiif_hocr' is incompatible with this version of Drupal core".

`assets/patches/islandora_iiif_hocr_d11.patch` is now the **only** patch for this package. It is one patch against upstream 2.0.7 that combines the Lite changes from `islandora_iiif_hocr_lite.patch` (kept unchanged in `assets/patches/` for 2.x sites) with the Drupal 11 change `core_version_requirement: ^10 || ^11`. The D11 part is the same change as the open upstream PR [Born-Digital-US/islandora_iiif_hocr#3](https://github.com/Born-Digital-US/islandora_iiif_hocr/pull/3) ("Declare d11 compatibility", by an Islandora maintainer). Remove that part once upstream tags a release with it. How it was checked:

- Code review against Drupal 11.4: no removed APIs. `drupal-check -d` reports no deprecations; its seven findings are pre-existing and unrelated to D11 (a `getIndexes()` call in the unrouted `SettingsForm`, and malformed or relative class names in doc comments).
- `git apply` (no fuzz, as in the ISLE container) and macOS `patch` both apply it, on the installed code and on pristine upstream 2.0.7 plus the Lite patch.
- `composer install` applies it on its own (`PATCHES.txt` lists only this patch). The resulting module code is byte-identical to the earlier two-patch result (Lite then D11), and the lock is unchanged apart from its hash.
- On the isle-dc Drupal 11.4 site, Drupal reports the module compatible, and `drush en islandora_iiif_hocr` enabled it with `islandora_iiif` and `termwithuri_condition`. Its views style plugin is discovered and no errors were logged. The optional `search_in_hocr` view was skipped because that site had no `default_solr_index_islandora_lite` index yet; a full config import creates both.
- Not yet tested: an hOCR search through Mirador, which needs indexed hOCR content.

**Resolved by a local patch (2026-09-29): media_thumbnails_video.** 2.0.2 declares `^9.3 || ^10 || ^11`, but fails on Drupal 11.4, which added a native `: array` return type to core's `FileVideoFormatter::viewElements()` and three required constructor parameters. Rebuilding the isle-dc site stopped with:

```
PHP Fatal error:  Declaration of Drupal\media_thumbnails_video\Plugin\Field\FieldFormatter\VideoExtendedFormatter::viewElements(...) must be compatible with Drupal\file\Plugin\Field\FieldFormatter\FileVideoFormatter::viewElements(...): array
```

Upstream fixed it only on the untagged `2.1.x` branch, which requires `>=11.4.0` (the `2.0.x` branch now requires `<11.4`). `assets/patches/media_thumbnails_video_d11.patch` applies that fix to 2.0.2: upstream commits `08a1c8d` (return type) and `8df07a1` (constructor and `create()` pass `current_user`, the image style storage and `entity_field.manager` to the parent), and sets `core_version_requirement: ^11.4`. `drupal/media_thumbnails_video` is pinned to `2.0.2` in `composer.json` so the patch keeps matching. Drop the patch and the pin once upstream tags a 2.1.x release. Verified on the isle-dc Drupal 11.4 site: `drush cr` completes, the class loads, and Drupal constructs the formatter with the new constructor. `ableplayer`'s formatters also extend core file formatters, but they extend `FileMediaFormatterBase`, whose constructor and `viewElements()` did not change, and they don't override the constructor.

Clones of the `drupal-11` branch made before commit `83ce365` (2026-09-29) have the older `composer.json`, `composer_site.json` and `composer.lock`, which lock islandora_breadcrumbs 1.0.1, facets_year_range 1.0.1 and rest_translation_util dev-main. Those do not enable on Drupal 11; pull and run `composer install`.

Resolved on 2026-09-29 by new releases: islandora_breadcrumbs 1.0.2, facets_year_range 1.0.4, group_concat 1.0.2, and rest_translation_util 1.1.1.

**Pre-release only.** These are D11-compatible but have no stable release:
- color 2.0.0-alpha1
- rdf 3.0.0-beta2
- cer, entity_reference_purger, term_reference_change, ultimate_cron, views_arg_entity_field (beta)
- recogito_integration, search_api_location, config_update (alpha)
- context (rc)

## Core modules removed in Drupal 11 that this config uses

| Module | Enabled in config/sync | Replacement |
|---|---|---|
| hal | yes | `drupal/hal ^2.0` (contrib) |
| rdf | yes (`rdf.mapping.*` config) | `drupal/rdf ^3.0@beta` (contrib) |

Nothing else in `core.extension.yml` was removed from core in D11 (checked: action, book, forum, statistics, tracker, color, quickedit, tour, ckeditor). The `color` and `quickedit` modules requested by `composer_site.json` are contrib.

`action` is not enabled in `config/sync`, but the build still installs the contrib `drupal/action` 0.2.2 into `web/modules/contrib/action`. It is a dependency of `drupal/islandora` 2.19.0, which Composer resolves and the post-install `rm -rf web/modules/contrib/islandora` then deletes. The `action` directory is left behind; it is harmless while the module stays disabled.

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
| drupal/media_thumbnails_video 2.0.2 | **media_thumbnails_video_d11.patch** | **new**: upstream 2.1.x fix for Drupal 11.4 (see Blockers) |
| discoverygarden/islandora_hocr v1.4.3 | islandora_hocr_lite.patch | applies |
| born-digital/islandora_iiif_hocr 2.0.7 | islandora_iiif_hocr_lite.patch → **islandora_iiif_hocr_d11.patch** | **merged**: one patch with the Lite changes plus `core_version_requirement: ^10 \|\| ^11` (see Blockers) |
| mjordan/islandora_workbench_integration v1.2.1 | workbench_integration.patch → **workbench_integration_d11.patch** | **re-rolled**: the old patch's context contains `version: "1.2.0"`, which only applied with fuzz (macOS `patch`); GNU patch 2.8 / `git apply` in the ISLE container reject it |

`better_social_sharing_d11.patch` re-implements the "preferred URL field" feature for the 5.x code base:
- `BetterSocialSharingButtonsHooks::preprocessBetterSocialSharingButtons()` overrides `items.page_url` with the configured link field's URL. Every partial template and the copy-link button (`data-page-url`) then use it, so the old template/JS changes are no longer needed.
- The settings form keeps the `use_url_field` → `preferred_url` setting.
- `preferred_url` was added to the config schema.

The four Drupal 11 patches are referenced by **local path** (`assets/patches/better_social_sharing_d11.patch`, `assets/patches/workbench_integration_d11.patch`, `assets/patches/islandora_iiif_hocr_d11.patch`, `assets/patches/media_thumbnails_video_d11.patch`), so the `drupal-11` branch installs on its own; the 2026-09-29 builds applied all four from those paths. The other nine patches keep the repo convention of `raw.githubusercontent.com/.../refs/heads/2.x/assets/patches/...` URLs. After this branch is merged into `2.x`, switch the four entries to that URL form and run `composer update --lock`. A `2.x` URL for any of them does not resolve before the merge.

The old `better_social_sharing.patch`, `workbench_integration.patch` and `islandora_iiif_hocr_lite.patch` stay in `assets/patches/` for 2.x sites.

## Upgrading an existing site

1. PHP 8.3+ in the container, and `ext-imagick` (media_thumbnails_pdf requires it). Run Composer inside that container; a host PHP without `ext-imagick` refuses to install the lock unless you pass `--ignore-platform-req=ext-imagick`.
2. Resolve or remove the three remaining composer blockers in `composer_site.json`: getjwtonlogin, media_revisions_ui, and advanced_search_tips.
3. islandora_iiif_hocr no longer needs uninstalling: `islandora_iiif_hocr_d11.patch` makes it enable on D11. Our own modules (islandora_breadcrumbs, group_concat, rest_translation_util, hero_banner, facets_year_range) now have D11 releases, and `composer update` picks them up.
4. `composer update -W`
5. `drush updb -y && drush cr`
6. Re-save the search_api_solr server and redeploy the Solr config set (search_api_solr 4.4).
7. `drush cex -y` and review the `config/sync` diff. Expect changes from the major bumps: better_social_sharing_buttons 5, fontawesome 3, islandora_mirador 3, views_flipped_table 3, oembed_providers 3, extlink 3, file_extractor 5, imagemagick 5, and filemime 2.
8. hero_banner 1.0.3 brings in `image_widget_crop`, `crop` and `imce`. Enable them if the site uses hero_banner's crop widget, then export config.

Steps 5 to 8 have not been run against this build yet; the 2026-09-29 verification stopped at the Composer build.
