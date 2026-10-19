# AdPresso Action Hooks

Action hooks (`do_action()`) available in the AdPresso plugin for developers to extend and customize its behavior.

## Lifecycle & Boot

### `adpresso.loaded`
Fires after AdPresso has finished booting.
- **Parameters:**
  - `AdPresso\Plugin $plugin`: Main plugin instance.
- **Example:**
```php
add_action( 'adpresso.loaded', function( $plugin ) {
    // Plugin is ready
} );
```

### `adpresso_register_entities`
Action to register custom entities during plugin boot.
- **Example:**
```php
add_action( 'adpresso_register_entities', function( $registrar ) {
    // Register custom entities here
} );
```

### `adpresso_ad_types_registered`
### `adpresso_group_types_registered`
### `adpresso_placement_types_registered`
Action fired after core types are registered.
- **Example:**
```php
add_action( 'adpresso_ad_types_registered', function( $registry ) {
    $registry->register( new My_Custom_Ad_Type() );
} );
```

### `adpresso_register_conditions`
Allows developers to register custom targeting conditions.
- **Example:**
```php
add_action( 'adpresso_register_conditions', function( $registry ) {
    $registry->register( new My_Custom_Condition() );
} );
```

### `adpresso_ad_saved`
Fires once an ad's post AND its `meta_data` are both fully persisted for a save - unlike core's `save_post` (which fires from inside `AdRepository::save()`'s `wp_insert_post()` call, before that same save's `meta_data` is written), so listeners see the values just submitted, not whatever was there before this save.
- **Parameters:**
  - `int $post_id`: The ad's post ID.
  - `int $previous_image_id`: The ad's `image_id` meta value before this save (`0` if none/new ad) - lets a listener detect an image change and act on the old value before it's gone.
- **Example:**
```php
add_action( 'adpresso_ad_saved', function( $post_id, $previous_image_id ) {
    // React to a fully-saved ad, e.g. compare against $previous_image_id
}, 10, 2 );
```

### `adpresso_register_assets`
Fires when assets should be registered.

### `adpresso_virtual_placement_resolved`
Fires once a block/shortcode-rendered ("virtual") placement has resolved to JS-mode output during the current request. Lets code that gates a frontend script by scanning `Runtime::get_processed_data()` on `wp_enqueue_scripts` (which runs before `the_content`, i.e. before any virtual placement exists) re-check now that virtual placement data is available. No parameters.

### `adpresso_ad_rendered`
Fires once an ad flagged `display_only_once` has actually been rendered this request. `AdPresso\Pro\Pipeline\Ad_Rendered_Tracker::mark_rendered()` listens on this to back the `adpresso_ad_display_limit_reached` filter.
- **Parameters:**
  - `int $ad_id`: The ad's ID.
- **Example:**
```php
add_action( 'adpresso_ad_rendered', function( $ad_id ) {
    my_custom_tracker()->mark_shown( $ad_id );
} );
```

### `adpresso_load_integrations`
Fires when integrations should be loaded.

### `adpresso_load_rest_controllers`
Fires when REST controllers should be registered.

### `adpresso_pro_booted`
Fires after AdPresso Pro has booted.

---

## Admin & Settings Actions

### `adpresso-admin-menu-init`
Fires when the admin menu is being initialized.
- **Example:**
```php
add_action( 'adpresso-admin-menu-init', function( $admin_menu ) {
    // Add custom menu items
} );
```

### `adpresso/settings/form/before`
### `adpresso/settings/form/{tab_id}/end`
### `adpresso/settings/form/{tab_id}/after`
Actions fired around the settings form.
- **Example:**
```php
add_action( 'adpresso/settings/form/general/after', function() {
    echo '<p>Additional info for general settings</p>';
} );
```

### `adpresso/settings/page/{page_id}/before`
### `adpresso/settings/page/{page_id}/after`
Actions fired around specific settings pages.

### `adpresso/settings/section/{section_id}/before`
### `adpresso/settings/section/{section_id}/after`
Actions fired around specific settings sections.

### `adpresso/settings/field/{field_id}/before`
### `adpresso/settings/field/{field_id}/after`
Actions fired around individual settings fields.
- **Example:**
```php
add_action( 'adpresso/settings/field/ad_name/before', function( $field ) {
    echo '<p>Choose a unique name for your ad.</p>';
} );
```

### `adpresso_tools_render_tab_{tab_id}`
Fires to render a specific tool tab.
- **Example:**
```php
add_action( 'adpresso_tools_render_tab_my_tab', function() {
    echo '<h2>My Custom Tool</h2>';
} );
```

### `adpresso_after_settings_tab_registered`
Fires after a settings tab has been registered.

---

## Database & Operations

### `adpresso_after_duplicate_entity`
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

### `adpresso_after_register_ad_post_type`
Fires after the ad post type is registered.

### `adpresso_clear_page_cache`
Fires when page cache should be cleared (e.g., after saving an ad).
- **Example:**
```php
add_action( 'adpresso_clear_page_cache', function() {
    if ( function_exists( 'w3tc_flush_all' ) ) w3tc_flush_all();
} );
```

### `adpresso_invalidate_pipeline_cache`
Call this to invalidate AdPresso's internal pipeline snapshot cache (the persistent cache of raw Ad/Group/Placement data) from Pro or third-party code that changes ad data outside the normal save/delete hooks AdPresso already listens to.
- **Example:**
```php
do_action( 'adpresso_invalidate_pipeline_cache' );
```
