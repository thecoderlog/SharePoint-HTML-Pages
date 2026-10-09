# SharePoint HTML pages: the two demo files

From the video "SharePoint HTML pages: your HTML in a sealed box" on The Coderlog.

SharePoint shows an HTML page inside a sandboxed iframe, under a strict Content Security Policy. Your page can draw anything, but it can't reach out: every piece of data has to be handed in by SharePoint. These two files show both sides of that.

## fetch-test.html: the glass

A heading and one `fetch()` to a public API (jsonplaceholder).

- Open it from disk in your browser and the names show up.
- Open it in SharePoint and the names never come. The viewer shows a banner, "Some content didn't load", with the details "Unhandled promise rejection: Failed to fetch". It never says why.
- The reason is in DevTools › Console: `Refused to connect … because it violates the following Content Security Policy directive: "connect-src 'none'"`.

The fetch has no `.catch` on purpose. Catch it and the banner goes away, but the data still never comes.

## team-dashboard.html: the slot

A table fed by a SharePoint list through the live data manifest (the `ka-livedata-manifest` block at the top). There is no `fetch()` anywhere: when someone opens the page, SharePoint reads the list on the server, as that person, and puts the rows in `window.__LD_RESULTS__` before the script runs. The Refresh button asks SharePoint for fresh rows with `parent.postMessage({ type: 'ka-html-viewer-refresh' }, '*')` (at most once every 5 seconds).

To try it:

1. In a site, create a list called **Team** with two single line text columns, **City** and **Role**, and add a few people (the name goes in **Title**).
2. Get the list ID: open the list's settings. The address ends in `List=%7B…%7D`, and the ID is the GUID between `%7B` and `%7D`.
3. In `team-dashboard.html`, replace `YOUR-TENANT`, `YOUR-SITE` and `YOUR-LIST-ID` (three places: `spItemUrl`, `webUrl`, `listId`).
4. Upload the file and open it. Change a city in the list, reload the page, and the new city shows.

Someone who can't open the list gets an error entry instead of rows; the page shows a message. Every value is written with `textContent`, never as HTML.

## Where to open them

Upload a file to a document library and open it: it renders in SharePoint's HTML viewer (the address ends in `?web=1`). The HTML page type in Site Pages is rolling out from mid-November to late December 2026 (MC1479517); Microsoft describes it the same way, a sandboxed iframe under a content security policy.

The rules the viewer enforces are served by your own tenant at `https://<your-tenant>.sharepoint.com/_html`.

## Read more

- Microsoft Support, Create, upload, edit, and publish HTML pages in SharePoint: https://support.microsoft.com/en-us/sharepoint/create-edit-publish-html-pages
- Microsoft's templates and agent skill: https://github.com/SharePoint/sharepoint-html
