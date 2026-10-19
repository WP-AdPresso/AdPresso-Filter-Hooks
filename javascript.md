# AdPresso JavaScript API & Events

The browser-side API, events and filters of the AdPresso plugin.

Every JS-mode placeholder carries its own config, so markup loaded after page load renders without anything from the page's `window.adpressoData`. Insert it, then let AdPresso scan it.

**The `view` option.** `'continue'` (default): the content belongs to what the visitor already sees - a stream, the next page of a list. "Display only once" keeps blocking ads already shown. `'new'`: the content counts like a new page view - the next article. "Display only once" starts fresh. Impressions count per physical copy either way; a group refresh rotation in the same slot counts once.

### `AdPresso.content.insert( target, content, options )`
Inserts HTML (or nodes) into `target`, runs the ad scripts in it (`innerHTML` never does) and scans it. Prefer this over inserting with jQuery `.html()`, which runs the scripts itself - they would run twice.
- `Element target`
- `string|Node[] content`
- `object options`: `view` (`'continue'`|`'new'`), `position` (`'append'` (default), `'prepend'`, `'before'`, `'after'`).
- **Returns:** the inserted top-level nodes.

### `AdPresso.content.scan( root, options )`
Renders the ads in content your code already inserted. Safe to call again for the same root.
- `ParentNode root`
- `object options`: `view`.
- **Returns:** the placement controllers created.

### `AdPresso.content.release( root )`
Stops the placements in content that is about to be removed (refresh timers, listeners). Call it before removing the content, e.g. when an infinite scroll recycles old articles.

### `AdPresso.content.getConfig( publicId )` / `AdPresso.content.getConfigs()`
A placement's config by its public ID, or every config the page knows - from `window.adpressoData` and from the placeholders read so far. Read placement configs through these instead of `adpressoData.placementConfig`, which no longer lists placements whose placeholder carries the config.

### `window.adpressoContentQueue`
For code that runs before AdPresso has loaded: push `{ root, view }` objects; they are scanned once it has.
```js
( window.adpressoContentQueue = window.adpressoContentQueue || [] ).push( { root: container, view: 'new' } );
```

### `adpresso:contentLoaded`
Dispatch it on `document` instead of calling `scan()`. `detail`: `{ root, view }`.
```js
document.dispatchEvent( new CustomEvent( 'adpresso:contentLoaded', { detail: { root: container, view: 'continue' } } ) );
```

### `adpresso:beforeContentScan`
Fires before content is scanned. Change `detail.view` to override the view, e.g. for an integration you don't control. `detail`: `{ root, view }`.
```js
document.addEventListener( 'adpresso:beforeContentScan', ( event ) => {
    if ( event.detail.root.closest && event.detail.root.closest( '.next-article' ) ) {
        event.detail.view = 'new';
    }
} );
```

### `adpresso:serverAdsFound`
Fires per scanned root before any placement in it selects an ad. `detail`: `{ root, viewId, adIds }` - the IDs of the server-rendered ads in it.

### `adpresso:contentScanned`
Fires after a root was scanned. `detail`: `{ root, view, viewId, controllers }`.

### `adpresso:contentReleased`
Fires after `release()`. `detail`: `{ root, controllers }`.

### `adpresso:placementDestroyed`
Fires when a placement controller is destroyed. `detail`: `{ placementId, controller }`.

### `adpresso:claimSelectedAds`
Fires synchronously right after a placement copy selected its ads, before they render. Selection waits for conditions and consent, so copies of one placement select at the same time - claim here, not on `adRendered`, to keep two copies from showing the same ad. Set `detail.proceed = false` to refuse: the copy selects again, and a pick that is refused every time falls back. `detail`: `{ placementId, controller, itemConfig, ads, proceed }`.

### `adpresso:adRendered`
Fires per ad rendered in the browser. `detail`: `{ adElement, placementId, groupId, adId, rid, instance, viewId }` - `instance` is the placement copy (its controller), `viewId` is `'page'` or the view of content loaded with `view: 'new'`.

### `adpresso:trackEvent`
Fires for every impression and click AdPresso's tracking records, before it is sent - forward it to your own analytics. Only ads with tracking on; impressions count once per placement copy (a refresh rotation in the same slot counts once). Requires the Tracking module.
`detail`: `{ type, adId, groupId, placementId, rid?, targetUrl? }` - `type` is `'impression'`, `'click'` or (Pro Active View) `'impression_av'`; `rid` and `targetUrl` come with clicks.
```js
AdPresso.events.on( 'adpresso:trackEvent', ( event ) => {
    if ( event.detail.type === 'click' ) {
        window.dataLayer?.push( { event: 'ad_click', ad_id: event.detail.adId } );
    }
} );
```

### `adpresso:privacyStateChange`
Fires when the visitor's consent state changes (CMP answer, cookie, TCF update, timeout). Requires the Privacy module.
`detail`: `{ status, renderNPA, source, reason, tcfData? }` - `status` is `'unknown'`, `'accepted'`, `'partial'`, `'rejected'` or `'not_needed'`; `renderNPA` says whether non-personalized ads may show; `source` names where the state came from.
```js
AdPresso.events.on( 'adpresso:privacyStateChange', ( event ) => {
    if ( event.detail.status === 'accepted' ) {
        // load something that needs consent
    }
} );
```
The current state at any time: `AdPresso.privacy.getCurrentConsentDetails()`.

**Built-in integrations.** Jetpack Infinite Scroll (`is.post-load`), Ajax Load More (`window.almComplete`; the Single Posts add-on loads as `'new'`), BuddyPress/BuddyBoss activity streams and PeepSo streams are scanned automatically as `'continue'`. A callback a theme assigns to `window.almComplete` keeps working.

**Deprecated.** Matching placeholders loaded via AJAX by `data-{prefix}-pl` (the placement ID) instead of their inline config is still read for cached markup, but will be removed.

## Statistics page (`wp.hooks`)

JS filters of the admin statistics app (`AdPresso > Statistics`), registered with `wp.hooks.addFilter()`. Register them in a script that the app's script (`adpresso-analytics-app`) depends on, so they exist before it mounts. Stats are counted per day in the site's timezone; for your own date math use `window.adpressoAnalyticsData.timezone` (`wp_timezone_string()`: an IANA name or an offset like `'+05:45'`), never the browser's timezone.

### `adpresso.analytics.filterBarComponents`
Adds React components to the filter bar. Runs once, when the app mounts. Each component gets the props `compareMode`, `onCompareModeChange( mode )` and `currentRange` (a preset key or `{ key: 'custom', start, end }`).
- `array components` (default: `[]`)
- **Returns:** the array of components.
```js
wp.hooks.addFilter( 'adpresso.analytics.filterBarComponents', 'my-plugin', ( components ) => [
    ...components,
    ( { currentRange } ) => wp.element.createElement( 'span', { className: 'my-range-note' }, typeof currentRange === 'string' ? currentRange : 'custom' ),
] );
```

### `adpresso.analytics.fetchChartData`
Replaces the chart's data request. Runs every time the date range, custom dates, grouping, entity filters or compare mode change. Return `defaultLogic` to keep the built-in request, or a function that fetches and stores the data through the payload's setters. Setters of a request that was superseded by a newer one are ignored.
- `function defaultLogic`: the built-in request.
- `object payload`: `dateRange`, `customStartDate`, `customEndDate` (`Y-m-d` or `null`), `compareMode` (`null`, `'previous_period'`, `'yoy'`), `filters` (`{ ads, groups, placements }`, ID arrays), `grouping` (`'day'`|`'week'`|`'month'`|`'year'`), `viewMode` (always `'date'`), `sorting` (`{ orderBy: 'date', order: 'asc' }`), `setChartData( rows )`, `setCompareData( rows|null )`, `setIsChartLoading( bool )`, `apiNonce`, `restUrl`.
- **Returns:** the function to run.
```js
wp.hooks.addFilter( 'adpresso.analytics.fetchChartData', 'my-plugin', ( defaultLogic, payload ) => {
    if ( payload.grouping !== 'week' ) {
        return defaultLogic;
    }
    return () => {
        payload.setIsChartLoading( true );
        wp.apiFetch( { path: '/my-plugin/v1/weekly-stats?range=' + payload.dateRange } )
            .then( ( rows ) => { payload.setChartData( rows ); payload.setCompareData( null ); } )
            .finally( () => payload.setIsChartLoading( false ) );
    };
} );
```

### `adpresso.analytics.fetchTableData`
Replaces the table's data request. Runs every time the date range, custom dates, view (tab), sorting, grouping, entity filters or compare mode change. Same contract as `adpresso.analytics.fetchChartData`. Compare rows are matched to the table rows by their offset in the range (`'date'` view) or by entity ID (`ad_id`, `group_id`, `placement_id`).
- `function defaultLogic`: the built-in request.
- `object payload`: `dateRange`, `customStartDate`, `customEndDate`, `compareMode`, `viewMode` (`'date'`|`'ads'`|`'groups'`|`'placements'`), `filters`, `grouping`, `sorting` (`{ orderBy, order }`), `setTableData( rows )`, `setCompareData( rows|null )`, `setIsTableLoading( bool )`, `apiNonce`, `restUrl`.
- **Returns:** the function to run.
```js
wp.hooks.addFilter( 'adpresso.analytics.fetchTableData', 'my-plugin', ( defaultLogic, payload ) => () => {
    defaultLogic();
    console.log( 'Statistics table reloaded for', payload.viewMode, payload.dateRange );
} );
```
