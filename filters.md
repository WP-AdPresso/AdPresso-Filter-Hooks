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
Filters the placeholder HTML when a placement has no active ad to show.
- **Parameters:**
  - `string $html`: The placeholder HTML.
  - `AdPresso\Entities\Placement $placement`: The placement entity.
- **Example:**
```php
add_filter( 'adpresso.output.placeholder', function( $html, $placement ) {
    return '<div class="advertise-here"><a href="/advertise">Your Ad Here</a></div>';
}, 10, 2 );
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
Determines if shortcodes should be processed in HTML ads.
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
Filters the capability required for a specific action.
- **Parameters:**
  - `string $cap`: The capability.
  - `string $action`: The action being performed.
- **Example:**
```php
add_filter( 'adpresso_get_capability', function( $cap, $action ) {
    if ( $action === 'manage_ads' ) return 'edit_posts';
    return $cap;
}, 10, 2 );
```

### `adpresso_debug_mode_user_cap`
Filters the capability required to see debug information.
- **Default:** `'manage_options'`
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

