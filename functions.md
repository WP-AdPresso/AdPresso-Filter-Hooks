# AdPresso PHP Functions

Public PHP functions of the AdPresso plugin for themes and other plugins.

### `adpresso_render_in_post_context()`
Renders a post with its ads in a REST or AJAX endpoint of your own - e.g. the next article for infinite scroll. For the duration of the callback the post is the main query and AdPresso's pipeline has run for it, so conditions, content injection and widgets behave as on the article's own page. Query, globals and context are restored afterwards, also when the callback throws. Inside a regular page view it refuses and returns `''`.
- **Parameters:**
  - `WP_Post|int $post`: The post to render.
  - `callable $callback`: Receives the `WP_Post` inside the loop; may echo or return its HTML.
  - `array $args`: `'output' => 'js'` (default): placeholders only, so the response is cacheable and the browser decides per visitor. `'auto'`: each placement decides like on a page view (server-rendered ads are baked into the response).
- **Returns:** `string` - everything the callback echoed, followed by what it returned.
- **Example:**
```php
register_rest_route( 'mysite/v1', '/next-post/(?P<id>\d+)', [
    'methods'             => 'GET',
    'permission_callback' => '__return_true',
    'callback'            => function( $request ) {
        $html = adpresso_render_in_post_context( (int) $request['id'], function() {
            the_content();
            dynamic_sidebar( 'after-article' );
        } );
        return [ 'html' => $html ];
    },
] );
```
In the browser, insert the HTML with `AdPresso.content.insert( container, html, { view: 'new' } )` and turn on `adpresso_dynamic_content_support` for the pages that load it.
