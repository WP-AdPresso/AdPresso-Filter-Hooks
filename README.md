# AdPresso Plugin Hooks Documentation

This document provides a comprehensive list of Filter and Action hooks available in the AdPresso plugin for developers to extend and customize the plugin's behavior.

## Filter Hooks

### Entities & Lifecycle

#### `adpresso_duplicate_post_status`
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

#### `adpresso_entity_quick_links`
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

#### `adpresso_post_type_ad_args`
#### `adpresso_post_type_group_args`
#### `adpresso_register_post_type_placement`
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

#### `adpresso/ads/fields/base`
#### `adpresso/groups/fields/base`
#### `adpresso/placements/fields/base`
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

#### `adpresso_placement_type_fields`
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

#### `adpresso_placement_fallback_fields`
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

---

### Rendering & Output

#### `adpresso.output.wrapper`
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

#### `adpresso.output.ad.wrapper`
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

#### `adpresso.output.placeholder`
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

#### `adpresso_ssr_output_ad_context`
#### `adpresso_ssr_output_group_context`
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

#### `adpresso_placement_js_config`
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

#### `adpresso_ad_type_{type}_frontend_config`
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

#### `adpresso_html_ad_allow_shortcodes`
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

#### `adpresso_content_filter_priority`
Filters the priority of the `the_content` filter used for automatic injection.
- **Default:** `100`
- **Example:**
```php
add_filter( 'adpresso_content_filter_priority', function() {
    return 10; // Run earlier
} );
```

#### `adpresso_dom_injectable_types`
Filters the list of placement types that support DOM injection.
- **Example:**
```php
add_filter( 'adpresso_dom_injectable_types', function( $types ) {
    $types[] = 'my_custom_type';
    return $types;
} );
```

---

### Admin UI & Assets

#### `adpresso_list_table_columns`
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

#### `adpresso_list_table_filters`
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

#### `adpresso_list_table_bulk_actions`
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

#### `adpresso_ad_metabox_classes`
#### `adpresso_group_metabox_classes`
#### `adpresso_placement_metabox_classes`
Filters the CSS classes applied to metaboxes in the admin.
- **Example:**
```php
add_filter( 'adpresso_ad_metabox_classes', function( $classes ) {
    $classes['my-custom-box'] = 'custom-metabox-style';
    return $classes;
} );
```

#### `adpresso_script_dependencies`
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

#### `adpresso_localize_admin_scripts`
Filters data localized for admin scripts.
- **Example:**
```php
add_filter( 'adpresso_localize_admin_scripts', function( $data ) {
    $data['my_param'] = 'my_value';
    return $data;
} );
```

#### `adpresso_disguisable_assets`
Filters assets that can be disguised/obfuscated to avoid adblockers.
- **Example:**
```php
add_filter( 'adpresso_disguisable_assets', function( $assets ) {
    $assets[] = 'custom-tracker.js';
    return $assets;
} );
```

---

### Settings & Tools

#### `adpresso_settings_tabs`
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

#### `adpresso_tools_tabs`
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

#### `adpresso_settings_tab_{tab_id}_options`
#### `adpresso_tab_{tab_id}_options`
#### `adpresso_tab_{tab_id}_sections`
Dynamic filters to modify options and sections for specific settings tabs.
- **Example:**
```php
add_filter( 'adpresso_tab_general_options', function( $options, $tab ) {
    $options['custom_field'] = [ 'label' => 'Custom Field', 'type' => 'text' ];
    return $options;
}, 10, 2 );
```

#### `adpresso_adblocker_tab_sections`
#### `adpresso_adblocker_tab_options`
Filters specifically for the Adblocker settings tab.

#### `adpresso_get_setting`
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

#### `adpresso_importer_classes`
Filters the list of available importer classes.
- **Example:**
```php
add_filter( 'adpresso_importer_classes', function( $classes ) {
    $classes[] = 'My_Custom_Importer';
    return $classes;
} );
```

#### `adpresso_export_settings`
Filters the data to be exported.
- **Example:**
```php
add_filter( 'adpresso_export_settings', function( $data ) {
    unset( $data['sensitive_data'] );
    return $data;
} );
```

---

### Targeting & Logic

#### `adpresso_is_globally_disabled`
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

#### `adpresso_requires_frontend_logic`
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

#### `adpresso_get_capability`
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

#### `adpresso_debug_mode_user_cap`
Filters the capability required to see debug information.
- **Default:** `'manage_options'`
- **Example:**
```php
add_filter( 'adpresso_debug_mode_user_cap', function() {
    return 'edit_posts';
} );
```

#### `adpresso_privacy_module_active`
Filters whether the privacy module is considered active.
- **Example:**
```php
add_filter( 'adpresso_privacy_module_active', function( $active ) {
    return true; // Force enable
} );
```

#### `adpresso_condition_general_page_type_options`
#### `adpresso_condition_page_template_post_types`
#### `adpresso_condition_parent_page_post_types`
#### `adpresso_condition_post_id_post_types`
Filters used in various targeting conditions to define available options or restricted post types.

#### `adpresso_browser_lang_condition_languages`
Filters the list of available languages for the browser language targeting condition.

---

### Pro & Advanced

#### `adpresso/revisions/meta_keys`
#### `adpresso/revisions/light_meta_keys`
Filters meta keys to be tracked in revisions.
- **Example:**
```php
add_filter( 'adpresso/revisions/meta_keys', function( $keys ) {
    $keys[] = '_my_custom_meta';
    return $keys;
} );
```

#### `adpresso_global_fallbacks`
Filters the global fallback configuration.

#### `adpresso_data_for_js`
Filters the global data object passed to frontend JavaScript.

---

## Action Hooks

### Lifecycle & Boot

#### `adpresso.loaded`
Fires after AdPresso has finished booting.
- **Parameters:**
  - `AdPresso\Plugin $plugin`: Main plugin instance.
- **Example:**
```php
add_action( 'adpresso.loaded', function( $plugin ) {
    // Plugin is ready
} );
```

#### `adpresso_register_entities`
Action to register custom entities during plugin boot.
- **Example:**
```php
add_action( 'adpresso_register_entities', function( $registrar ) {
    // Register custom entities here
} );
```

#### `adpresso_ad_types_registered`
#### `adpresso_group_types_registered`
#### `adpresso_placement_types_registered`
Action fired after core types are registered.
- **Example:**
```php
add_action( 'adpresso_ad_types_registered', function( $registry ) {
    $registry->register( new My_Custom_Ad_Type() );
} );
```

#### `adpresso_register_conditions`
Allows developers to register custom targeting conditions.
- **Example:**
```php
add_action( 'adpresso_register_conditions', function( $registry ) {
    $registry->register( new My_Custom_Condition() );
} );
```

#### `adpresso_register_assets`
Fires when assets should be registered.

#### `adpresso_load_integrations`
Fires when integrations should be loaded.

#### `adpresso_load_rest_controllers`
Fires when REST controllers should be registered.

#### `adpresso_pro_booted`
Fires after AdPresso Pro has booted.

---

### Admin & Settings Actions

#### `adpresso-admin-menu-init`
Fires when the admin menu is being initialized.
- **Example:**
```php
add_action( 'adpresso-admin-menu-init', function( $admin_menu ) {
    // Add custom menu items
} );
```

#### `adpresso/settings/form/before`
#### `adpresso/settings/form/{tab_id}/end`
#### `adpresso/settings/form/{tab_id}/after`
Actions fired around the settings form.
- **Example:**
```php
add_action( 'adpresso/settings/form/general/after', function() {
    echo '<p>Additional info for general settings</p>';
} );
```

#### `adpresso/settings/page/{page_id}/before`
#### `adpresso/settings/page/{page_id}/after`
Actions fired around specific settings pages.

#### `adpresso/settings/section/{section_id}/before`
#### `adpresso/settings/section/{section_id}/after`
Actions fired around specific settings sections.

#### `adpresso/settings/field/{field_id}/before`
#### `adpresso/settings/field/{field_id}/after`
Actions fired around individual settings fields.
- **Example:**
```php
add_action( 'adpresso/settings/field/ad_name/before', function( $field ) {
    echo '<p>Choose a unique name for your ad.</p>';
} );
```

#### `adpresso_tools_render_tab_{tab_id}`
Fires to render a specific tool tab.
- **Example:**
```php
add_action( 'adpresso_tools_render_tab_my_tab', function() {
    echo '<h2>My Custom Tool</h2>';
} );
```

#### `adpresso_after_settings_tab_registered`
Fires after a settings tab has been registered.

---

### Database & Operations

#### `adpresso_after_duplicate_entity`
Action fired after an entity has been successfully duplicated.
- **Parameters:**
  - `int $new_post_id`: ID of the new post.
  - `int $original_id`: ID of the original post.
- **Example:**
```php
add_action( 'adpresso_after_duplicate_entity', function( $new_id, $old_id ) {
    // Handle custom meta duplication
}, 10, 2 );
```

#### `adpresso_after_register_ad_post_type`
Fires after the ad post type is registered.

#### `adpresso_clear_page_cache`
Fires when page cache should be cleared (e.g., after saving an ad).
- **Example:**
```php
add_action( 'adpresso_clear_page_cache', function() {
    if ( function_exists( 'w3tc_flush_all' ) ) w3tc_flush_all();
} );
```
