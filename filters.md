# AdPresso Filter Hooks

Filter hooks (`apply_filters()`) available in the AdPresso plugin for developers to extend and customize its behavior.

## Entities & Lifecycle

### `adpresso_duplicate_post_status`
Filters the default post status used when duplicating an entity (Ad, Group, or Placement).
- **Default:** `'draft'`
- **Parameters:**
  - `string $status`: The default status.
  - `WP_Post $post`: The original post object.
- **Example:**
```php
add_filter( 'adpresso_duplicate_post_status', function( $status, $post ) {
    return 'pending';
}, 10, 2 );
```

### `adpresso_entity_quick_links`
Filters the quick links displayed in the admin list tables for each entity.
- **Parameters:**
  - `array $links`: Array of link data.
  - `AdPresso\Entities\Entity $entity`: The entity object.
- **Example:**
```php
add_filter( 'adpresso_entity_quick_links', function( $links, $entity ) {
    $links['view_stats'] = [
        'label' => 'View Stats',
        'url'   => admin_url( 'admin.php?page=adpresso-stats&id=' . $entity->get_id() ),
    ];
    return $links;
}, 10, 2 );
```

### `adpresso_post_type_ad_args`
### `adpresso_post_type_group_args`
### `adpresso_register_post_type_placement`
Filters the registration arguments for AdPresso custom post types.
- **Parameters:**
  - `array $args`: The registration arguments.
- **Example:**
```php
add_filter( 'adpresso_post_type_ad_args', function( $args ) {
    $args['show_in_rest'] = true;
    return $args;
} );
```

### `adpresso/ads/fields/base`
### `adpresso/groups/fields/base`
### `adpresso/placements/fields/base`
Filters the base fields (settings) for ads, groups, or placements.
- **Parameters:**
  - `array $fields`: The base fields configuration.
- **Example:**
```php
add_filter( 'adpresso/ads/fields/base', function( $fields ) {
    $fields['internal_note'] = [
        'label' => 'Internal Note',
        'type'  => 'textarea',
    ];
    return $fields;
} );
```

### `adpresso_placement_type_fields`
Filters the fields for a specific placement type.
- **Parameters:**
  - `array $fields`: The fields.
  - `string $type_id`: The type ID.
- **Example:**
```php
add_filter( 'adpresso_placement_type_fields', function( $fields, $type_id ) {
    if ( $type_id === 'sidebar' ) {
        $fields['sticky'] = [ 'label' => 'Sticky', 'type' => 'checkbox' ];
    }
    return $fields;
}, 10, 2 );
```

### `adpresso_placement_type_compatible_conditions`
Filters the list of targeting condition IDs a Placement Type supports in the edit modal's Condition Builder. Return `null` (the default) for no restriction; return an array of condition IDs to narrow the catalog to only those. Embedded per-type into the same field config `dependent_fields` uses, so the React edit modal can react live to a Placement Type change without a REST refetch.
- **Parameters:**
  - `array|null $compatible_conditions`: `null` for no restriction, or an array of allowed condition IDs.
  - `string $type_id`: The placement type ID.
- **Example:**
```php
add_filter( 'adpresso_placement_type_compatible_conditions', function( $compatible_conditions, $type_id ) {
    if ( $type_id === 'my_custom_type' ) {
        return [ 'device', 'browser_language' ];
    }
    return $compatible_conditions;
}, 10, 2 );
```

### `adpresso_placement_type_hidden_sections`
Filters the list of edit-modal section/box IDs to hide entirely for a Placement Type (e.g. the Layout box makes no sense for a type that's never rendered into a WordPress page's DOM). Returns an empty array by default (nothing hidden). Embedded per-type the same way as `adpresso_placement_type_compatible_conditions`, so the React edit modal reacts live to a Placement Type change without a REST refetch.
- **Parameters:**
  - `array $hidden_sections`: Section/box IDs to hide (see `Entity::get_edit_ui_sections()`'s `id`, e.g. `'adpresso_layout'`).
  - `string $type_id`: The placement type ID.
- **Example:**
```php
add_filter( 'adpresso_placement_type_hidden_sections', function( $hidden_sections, $type_id ) {
    if ( $type_id === 'my_custom_type' ) {
        $hidden_sections[] = 'adpresso_layout';
    }
    return $hidden_sections;
}, 10, 2 );
```

### `adpresso_placement_fallback_fields`
Filters the fallback fields for placements when they are in a specific state.
- **Parameters:**
  - `array $fields`: Fallback fields array.
- **Example:**
```php
add_filter( 'adpresso_placement_fallback_fields', function( $fields ) {
    $fields['custom_fallback'] = [ 'label' => 'Custom Fallback', 'type' => 'select', 'options' => [] ];
    return $fields;
} );
```

### `adpresso_placement_rest_response`
Filters a Placement's fully-assembled REST response data right before it's sent to the edit modal - e.g. to add type-specific fields, or to strip out UI data (like `condition_builder_data` entries) that don't apply to a particular placement type.
- **Parameters:**
  - `array $data`: The prepared response data.
  - `AdPresso\Entities\Placement $placement`: This placement.
- **Example:**
```php
add_filter( 'adpresso_placement_rest_response', function( $data, $placement ) {
    if ( $placement->get_type() === 'my_custom_type' ) {
        $data['my_custom_field'] = 'some computed value';
    }
    return $data;
}, 10, 2 );
```

---

## Rendering & Output

### `adpresso.output.wrapper`
Applied to the final HTML output of a placement.
- **Parameters:**
  - `string $html`: The HTML output.
  - `AdPresso\Entities\Placement $placement`: The placement entity.
- **Example:**
```php
add_filter( 'adpresso.output.wrapper', function( $html, $placement ) {
    return '<!-- Start Placement ' . $placement->get_id() . ' -->' . $html . '<!-- End -->';
}, 10, 2 );
```

### `adpresso.output.ad.wrapper`
Applied to the HTML output of an individual Ad.
- **Parameters:**
  - `string $html`: The Ad HTML.
  - `AdPresso\Ads\Types\AbstractType $ad_type`: The Ad type instance.
- **Example:**
```php
add_filter( 'adpresso.output.ad.wrapper', function( $html, $ad_type ) {
    return '<div class="adpresso-ad-item">' . $html . '</div>';
}, 10, 2 );
```

### `adpresso.output.placeholder`
Filters the placeholder of a placement rendered in the browser (JS mode): a `<script type="text/{prefix}-tmpl">` tag whose body is the placement's JS config as JSON. The frontend script replaces it with the ad, so keep the tag, its attributes and its body intact - add to the HTML rather than replacing it.
- **Parameters:**
  - `string $html`: The placeholder HTML.
  - `AdPresso\Entities\Placement $placement`: The placement entity.
- **Example:**
```php
add_filter( 'adpresso.output.placeholder', function( $html, $placement ) {
    return '<div class="my-ad-slot">' . $html . '</div>';
}, 10, 2 );
```

### `adpresso_inline_placeholder_config`
Whether a JS-mode placeholder carries its config in its body. With it, markup loaded after the page (AJAX, REST, cached fragments) renders on its own; without it, the config only comes from `window.adpressoData`. Meant as an emergency switch, e.g. for an optimizer that alters script contents.
- **Parameters:**
  - `bool $inline`: Default `true`.
  - `AdPresso\Entities\Placement $placement`: The placement entity.
- **Example:**
```php
add_filter( 'adpresso_inline_placeholder_config', '__return_false' );
```

### `adpresso_dynamic_content_support`
Loads AdPresso's client scripts on the page even when the page itself renders no ad in the browser, because content loaded after page load (infinite scroll, "load more", AJAX streams) can bring placements of its own. Insert that content with `AdPresso.content.insert()` or call `AdPresso.content.scan()` on it (see [JavaScript API & Events](javascript.md)). Jetpack Infinite Scroll, Ajax Load More, BuddyPress/BuddyBoss and PeepSo turn this on by themselves.
- **Parameters:**
  - `bool $supported`: Default `false`.
- **Example:**
```php
add_filter( 'adpresso_dynamic_content_support', function( $supported ) {
    return $supported || is_singular( 'post' ); // The theme loads the next article on scroll.
} );
```

### `adpresso_ssr_output_ad_context`
### `adpresso_ssr_output_group_context`
Filters the context data passed to an ad or group's output method during Server-Side Rendering (SSR).
- **Parameters:**
  - `array $context`: Context information.
  - `AdPresso\Entities\Ad|AdPresso\Entities\Group $item`: The entity.
- **Example:**
```php
add_filter( 'adpresso_ssr_output_ad_context', function( $context, $ad ) {
    $context['is_mobile'] = wp_is_mobile();
    return $context;
}, 10, 2 );
```

### `adpresso_placement_js_config`
Filters the configuration data passed to the frontend JavaScript for a specific placement and item.
- **Parameters:**
  - `array $config`: The configuration data.
  - `AdPresso\Entities\Placement $placement`: The placement entity.
  - `AdPresso\Entities\Entity $item`: The ad or group being rendered.
- **Example:**
```php
add_filter( 'adpresso_placement_js_config', function( $config, $placement, $item ) {
    $config['refreshInterval'] = 30000; // 30 seconds
    return $config;
}, 10, 3 );
```

### `adpresso_ad_type_{type}_frontend_config`
Filters the frontend configuration for specific ad types (e.g., `image`, `html`, `dummy`).
- **Parameters:**
  - `array $config`: The config data.
  - `AdPresso\Entities\Ad $ad`: The ad entity.
- **Example:**
```php
add_filter( 'adpresso_ad_type_image_frontend_config', function( $config, $ad ) {
    $config['lazyLoad'] = true;
    return $config;
}, 10, 2 );
```

### `adpresso_html_ad_allow_shortcodes`
Determines if shortcodes should be processed in HTML ads - in SSR output and in the content shipped for JS-rendered ads and Headless Delivery (rendered once per request, as that content leaves PHP).
- **Parameters:**
  - `bool $allow`: Default `true`.
  - `AdPresso\Entities\Ad $ad`: The ad entity.
- **Example:**
```php
add_filter( 'adpresso_html_ad_allow_shortcodes', function( $allow, $ad ) {
    // Disable shortcodes for a specific ad
    if ( $ad->get_id() === 123 ) return false;
    return $allow;
}, 10, 2 );
```

### `adpresso_content_filter_priority`
Filters the priority of the `the_content` filter used for automatic injection.
- **Default:** `100`
- **Example:**
```php
add_filter( 'adpresso_content_filter_priority', function() {
    return 10; // Run earlier
} );
```

### `adpresso_dom_injectable_types`
Filters the list of placement types that support DOM injection.
- **Example:**
```php
add_filter( 'adpresso_dom_injectable_types', function( $types ) {
    $types[] = 'my_custom_type';
    return $types;
} );
```

---

## Admin UI & Assets

### `adpresso_list_table_columns`
Filters the columns displayed in AdPresso list tables.
- **Parameters:**
  - `array $columns`: The columns array.
- **Example:**
```php
add_filter( 'adpresso_list_table_columns', function( $columns ) {
    $columns['id'] = 'ID';
    return $columns;
} );
```

### `adpresso_list_table_filters`
Filters the available filters in the list table.
- **Parameters:**
  - `array $filters`: The filters array.
- **Example:**
```php
add_filter( 'adpresso_list_table_filters', function( $filters ) {
    $filters['author'] = [ 'label' => 'Author', 'type' => 'select', 'options' => [] ];
    return $filters;
} );
```

### `adpresso_list_table_bulk_actions`
Filters the bulk actions available in the list table.
- **Parameters:**
  - `array $bulk_actions`: The bulk actions array.
- **Example:**
```php
add_filter( 'adpresso_list_table_bulk_actions', function( $actions ) {
    $actions['export_csv'] = 'Export to CSV';
    return $actions;
} );
```

### `adpresso_ad_metabox_classes`
### `adpresso_group_metabox_classes`
### `adpresso_placement_metabox_classes`
Filters the CSS classes applied to metaboxes in the admin.
- **Example:**
```php
add_filter( 'adpresso_ad_metabox_classes', function( $classes ) {
    $classes['my-custom-box'] = 'custom-metabox-style';
    return $classes;
} );
```

### `adpresso_script_dependencies`
Filters the dependencies for AdPresso scripts.
- **Parameters:**
  - `array $deps`: Dependencies.
  - `string $handle`: Script handle.
- **Example:**
```php
add_filter( 'adpresso_script_dependencies', function( $deps, $handle ) {
    if ( $handle === 'adpresso-admin' ) {
        $deps[] = 'jquery-ui-sortable';
    }
    return $deps;
}, 10, 2 );
```

### `adpresso_localize_admin_scripts`
Filters data localized for admin scripts.
- **Example:**
```php
add_filter( 'adpresso_localize_admin_scripts', function( $data ) {
    $data['my_param'] = 'my_value';
    return $data;
} );
```

### `adpresso_disguisable_assets`
Filters assets that can be disguised/obfuscated to avoid adblockers.
- **Example:**
```php
add_filter( 'adpresso_disguisable_assets', function( $assets ) {
    $assets[] = 'custom-tracker.js';
    return $assets;
} );
```

---

## Settings & Tools

### `adpresso_settings_tabs`
Filters the registered settings tabs.
- **Parameters:**
  - `array $tabs`: Array of `SettingsTabInterface` objects.
- **Example:**
```php
add_filter( 'adpresso_settings_tabs', function( $tabs ) {
    // Modify the tabs array
    return $tabs;
} );
```

### `adpresso_tools_tabs`
Filters the tabs on the AdPresso Tools page.
- **Parameters:**
  - `array $tabs`: Array of tab labels.
- **Example:**
```php
add_filter( 'adpresso_tools_tabs', function( $tabs ) {
    $tabs['my_tab'] = 'My Custom Tool';
    return $tabs;
} );
```

### `adpresso_settings_tab_{tab_id}_options`
### `adpresso_tab_{tab_id}_options`
### `adpresso_tab_{tab_id}_sections`
Dynamic filters to modify options and sections for specific settings tabs.
- **Example:**
```php
add_filter( 'adpresso_tab_general_options', function( $options, $tab ) {
    $options['custom_field'] = [ 'label' => 'Custom Field', 'type' => 'text' ];
    return $options;
}, 10, 2 );
```

### `adpresso_adblocker_tab_sections`
### `adpresso_adblocker_tab_options`
Filters specifically for the Adblocker settings tab.

### `adpresso_get_setting`
Filters a setting value when retrieved via the internal settings manager.
- **Parameters:**
  - `mixed $value`: The value.
  - `string $key`: Setting key.
  - `mixed $default`: Default value.
- **Example:**
```php
add_filter( 'adpresso_get_setting', function( $value, $key ) {
    if ( $key === 'custom_api_key' ) return decrypt_value( $value );
    return $value;
}, 10, 2 );
```

### `adpresso_importer_classes`
Filters the list of available importer classes.
- **Example:**
```php
add_filter( 'adpresso_importer_classes', function( $classes ) {
    $classes[] = 'My_Custom_Importer';
    return $classes;
} );
```

### `adpresso_export_settings`
Filters the data to be exported.
- **Example:**
```php
add_filter( 'adpresso_export_settings', function( $data ) {
    unset( $data['sensitive_data'] );
    return $data;
} );
```

---

## Targeting & Logic

### `adpresso_is_globally_disabled`
Filters the result of the global disable check for AdPresso.
- **Parameters:**
  - `array $result`: Array with 'disabled' (bool) and 'reason' (string).
- **Example:**
```php
add_filter( 'adpresso_is_globally_disabled', function( $result ) {
    if ( is_user_logged_in() && current_user_can( 'subscriber' ) ) {
        $result['disabled'] = true;
        $result['reason'] = 'Ads disabled for subscribers';
    }
    return $result;
} );
```

### `adpresso_requires_frontend_logic`
Filters whether a placement or entity requires frontend (JS) logic.
- **Parameters:**
  - `bool $requires`: The current requirement status.
  - `AdPresso\Entities\Placement $placement`: The placement entity.
  - `AdPresso\Entities\Entity $item`: The ad or group item.
- **Example:**
```php
add_filter( 'adpresso_requires_frontend_logic', function( $requires, $placement, $item ) {
    // Force JS rendering for a specific placement
    if ( $placement && $placement->get_slug() === 'js-only-placement' ) return true;
    return $requires;
}, 10, 3 );
```

### `adpresso_item_requires_consent`
Filters whether an Ad/Group item requires consent (must be JS-rendered instead of SSR) - the decision `AdPresso\Modules\Privacy\ConsentPolicy::item_requires_consent()` makes for the Privacy module's `adpresso_requires_frontend_logic` callback. The same result is sent to the browser as the item's `requires_consent` flag, so the JS consent gate follows your override too. It is also written into fallback item configs, but the JS consent gate currently only checks the placement's primary item (or its fallback when there is no primary item). Only fires when the Privacy module is active and the item passed backend validation; it can fire several times per item and request, with `$placement` `null` for fallback items.
- **Parameters:**
  - `bool $requires_consent`: The policy's decision so far.
  - `AdPresso\Entities\Entity $item`: The ad or group item.
  - `AdPresso\Entities\Placement|null $placement`: The placement the item is being resolved for, if known.
- **Example:**
```php
add_filter( 'adpresso_item_requires_consent', function( $requires_consent, $item, $placement ) {
    // Treat a custom ad type as always privacy-safe.
    if ( $item instanceof \AdPresso\Entities\Ad && $item->get_type() === 'my_safe_type' ) return false;
    return $requires_consent;
}, 10, 3 );
```

### `adpresso_privacy_exceptional_ad_types`
Filters the ad types that never need consent on their own (no third-party code) - `ConsentPolicy` applies it for SSR and for the `requires_consent` flag the JS consent gate reads. An ad with custom ad code always needs consent, whatever its type.
- **Parameters:**
  - `string[] $types`: Default `[ 'image', 'dummy' ]`.
- **Example:**
```php
add_filter( 'adpresso_privacy_exceptional_ad_types', function( $types ) {
    $types[] = 'my_first_party_type';
    return $types;
} );
```

### `adpresso_ad_display_limit_reached`
Filters whether an ad has already reached its "Display Only Once" limit for the current request. Only consulted when the ad's own `display_only_once` meta is enabled; default `false` (no listener means the limit is never enforced, e.g. on a Free install). `AdPresso\Pro\Pipeline\Ad_Rendered_Tracker` answers this on Pro.
- **Parameters:**
  - `bool $reached`: Whether the limit was reached so far. Only ever change `false` to `true`, never override another listener's `true` back to `false`.
  - `int $ad_id`: The ad's ID.
- **Example:**
```php
add_filter( 'adpresso_ad_display_limit_reached', function( $reached, $ad_id ) {
    return $reached || my_custom_tracker()->already_shown( $ad_id );
}, 10, 2 );
```

### `adpresso_ad_link_masking_enabled`
Filters whether an ad's target link should be cloaked/masked. Default `false` - only an actually-loaded link-masking implementation (Pro's `AdPresso\Pro\Infrastructure\LinkRedirector`) can turn this on, so a stale/lapsed-Pro setting can never cloak into a masked URL that nothing redirects.
- **Parameters:**
  - `bool $is_cloaked`: Whether the ad's link should be cloaked so far. Default `false`.
  - `AdPresso\Entities\Ad $ad`: The ad entity.
- **Example:**
```php
add_filter( 'adpresso_ad_link_masking_enabled', function( $is_cloaked, $ad ) {
    // Force cloaking on for every ad site-wide.
    return true;
}, 10, 2 );
```

### `adpresso_global_gates_active`
Filters whether any global, visitor-dependent gate (e.g. "Show ads for specific user roles", the Pro "Disable ads for bots" setting) is currently configured. Only ever inspects saved setting values, never the current request - used to decide whether the frontend's single, page-wide global-gates AJAX check is worth making at all.
- **Parameters:**
  - `bool $active`: Whether a gate is active so far.
- **Example:**
```php
add_filter( 'adpresso_global_gates_active', function( $active ) {
    return $active || my_custom_gate_is_configured();
} );
```

### `adpresso_global_gate_blocks_visitor`
Filters whether the current visitor is blocked by any global gate. Only ever fires inside the uncached AJAX request that answers the frontend's global-gates check - never during normal page rendering - which is what makes it safe to inspect the actual current visitor/request here.
- **Parameters:**
  - `bool $blocked`: Whether the visitor is blocked so far (should only ever change `false` to `true`, never override another gate's block back to `false`).
- **Example:**
```php
add_filter( 'adpresso_global_gate_blocks_visitor', function( $blocked ) {
    return $blocked || my_custom_gate_blocks_current_visitor();
} );
```

### `adpresso_get_capability`
Filters the capability required for a specific action. Every permission check in AdPresso (admin menus, REST routes, AJAX handlers, post type capabilities) runs through this filter; Pro maps the actions to its own `adpresso_*` capabilities.
- **Parameters:**
  - `string $cap`: The capability.
  - `string $action`: The action being performed.
- **Actions:**
  - `edit_ad`, `edit_group`, `edit_placement`, `delete_ad`, `delete_group`, `delete_placement`, `duplicate_ad`, `duplicate_group`, `duplicate_placement` (default `edit_posts`)
  - `edit_others_ad`, `edit_others_group`, `edit_others_placement` (default `edit_others_posts`)
  - `allow_php_execution` (default `unfiltered_html`; also requires the `ADPRESSO_ALLOW_PHP_EVAL` constant)
  - `view_analytics`, `manage_settings`, `use_tools`, `create_debug_token`, `view_debug` (default `manage_options`)
- **Example:**
```php
add_filter( 'adpresso_get_capability', function( $cap, $action ) {
    if ( $action === 'view_debug' ) return 'edit_posts';
    return $cap;
}, 10, 2 );
```

### `adpresso_user_can_unfiltered_html`
Filters whether a user is trusted with unfiltered HTML. AdPresso filters ad code at save time and at render time (by the entity's author) unless this returns true. Pro uses it on multisite to trust whole roles, which WordPress otherwise reserves for super admins. `DISALLOW_UNFILTERED_HTML` short-circuits before the filter runs.
- **Parameters:**
  - `bool $allowed`: Result of `user_can( $user_id, 'unfiltered_html' )`.
  - `int $user_id`: The user (for rendering: the entity author).
- **Example:**
```php
add_filter( 'adpresso_user_can_unfiltered_html', function( $allowed, $user_id ) {
    return $allowed || user_can( $user_id, 'my_trusted_advertiser' );
}, 10, 2 );
```

### `adpresso_debug_mode_user_cap`
Legacy filter for the capability required to see debug information. Applied on top of `adpresso_get_capability( 'view_debug' )`; prefer that filter for new code.
- **Default:** `adpresso_get_capability( 'view_debug' )`
- **Example:**
```php
add_filter( 'adpresso_debug_mode_user_cap', function() {
    return 'edit_posts';
} );
```

### `adpresso_privacy_module_active`
Filters whether the privacy module is considered active.
- **Example:**
```php
add_filter( 'adpresso_privacy_module_active', function( $active ) {
    return true; // Force enable
} );
```

### `adpresso_condition_general_page_type_options`
### `adpresso_condition_page_template_post_types`
### `adpresso_condition_parent_page_post_types`
### `adpresso_condition_post_id_post_types`
### `adpresso_condition_taxonomy_excluded_taxonomies`
### `adpresso_condition_taxonomy_archive_options`
Filters used in various targeting conditions to define available options or restricted post types. `adpresso_condition_taxonomy_excluded_taxonomies` excludes taxonomy slugs from the auto-generated per-custom-taxonomy conditions (Category and Tag always get their own dedicated conditions regardless). `adpresso_condition_taxonomy_archive_options` filters the list of taxonomies offered by the "Taxonomy Archive" condition (Meta group) - checks whether the current page is the archive of one of the selected taxonomies; General Pages' own "Category Archive"/"Tag Archive" entries are hidden in favor of this condition but still evaluate correctly for conditions saved before the change.

### `adpresso_browser_lang_condition_languages`
Filters the list of available languages for the browser language targeting condition.

---

## Pro & Advanced

### `adpresso/revisions/meta_keys`
### `adpresso/revisions/light_meta_keys`
Filters meta keys to be tracked in revisions.
- **Example:**
```php
add_filter( 'adpresso/revisions/meta_keys', function( $keys ) {
    $keys[] = '_my_custom_meta';
    return $keys;
} );
```

### `adpresso_global_fallbacks`
Filters the global fallback configuration.

### `adpresso_data_for_js`
Filters the global data object passed to frontend JavaScript.

## Tracking

### `adpresso_enable_fast_tracking`
Opts into `fast-track.php`, a SHORTINIT-based tracking endpoint used instead of the `adpresso/v1/track` REST route for impression/click beacons. Off by default. The REST route is always kept as an automatic fallback (tracking.js falls back to it on any failure), so this is safe to toggle at any time. Equivalent to defining `ADPRESSO_FAST_TRACKING` as `true` in `wp-config.php`.
- **Default:** `false`
- **Example:**
```php
add_filter( 'adpresso_enable_fast_tracking', '__return_true' );
```

---
