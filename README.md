# Ghost / Algolia integration

This app enables [Ghost](https://ghost.org) sites owners to index their content through [Algolia](https://www.algolia.com).

*Disclaimer: Ghost apps are not mature as of July 2017 (https://github.com/TryGhost/Ghost/wiki/Apps-Getting-Started-for-Ghost-Devs). As a result, this app is merely using the "app" denomination to set its intention but does not leverage any part of the burgeoning App framework yet. Hooking into Ghost thus requires patching core for now, a workaround which will be removed as soon as the App framework is rolled out and makes this step unnecessary.*

# What it does

When you work on a story, and publish it, the content of that story is sent to Algolia's indexing engine. Any change you make to that story or its state afterwards (updating content, deleting the story or unpublishing it) is automatically synchronised with your index.

## Fragment indexing

Fragment indexing refers to breaking up an HTML document into smaller blocks (or fragments) before sending them to the indexing engine. Those fragments are generally composed of a heading (h1, h2, ...) and some text. You may read about the rationale behind fragment indexing on the [KirbyAlgolia project page](https://github.com/mlbrgl/kirby-algolia#kirby--algolia-integration).

Here is how the fragmenting engine handles the different types of fragments, in terms of when the indexing events are fired:

```
line
line
--> INDEXING (headless fragment)
# heading
line
--> INDEXING
## heading
--> INDEXING (content-less heading)
### subheading
line
line
--> INDEXING
# unlikely heading
--> INDEXING by code convenience but very little value
```

## Structure of a fragment

- `objectID`: automatically generated by Algolia (e.g. 565098020)
- `post_uuid`: automatically generated by Ghost (e.g. 8693c79d-7880-4e17-903d-7afd448e3517)
- `heading`: the heading of the fragment being indexed (e.g. My first paragraph)
- `id`: the ID of the fragment being indexed (e.g. my-new-blog-post#card-markdown--My-first-paragraph--1)
- `importance`: an integer reprensenting how deep in the article structure a fragment is located (e.g. 1). The deeper the less relevant.
- `post_title`: the title of the post being indexed (e.g. My new blog post)
- `content`: the content of the fragment being indexed (e.g. The content of the first paragraph)

# What it does not do

This app only deals with the indexing side of things. Adding the actual search widget is not part of the scope at this point. A good option to look into is [InstantSearch.js](https://community.algolia.com/instantsearch.js/v2/).

# Installation

1. Place the app code in the `[PATH_TO_GHOST_ROOT]/content/apps` folder so that the index.js file can be found at this location: `[PATH_TO_GHOST_ROOT]/content/apps/ghost-algolia/index.js`.

   You may use the following command from the ghost root:

   ```shell
   git clone git@github.com:mlbrgl/ghost-algolia.git content/apps/ghost-algolia
   ```

2. Install dependencies by running `yarn`(recommended) or `npm install` in the `ghost-algolia` folder.

3. Configure Algolia's index

Create a new API key on Algolia's dashboard. You want to make sure that the generated key has the following authorizations on your index:
- Search (search)
- Add records (addObject)
- Delete records (deleteObject)

Next add the following attributes as searcheable attributes, in the ranking tab under the "Basic settings" section:
- `post_title`
- `heading`
- `content`
- `post_uuid`

Ignore any warnings about the attributes not being found in a sample of your records, as you should not have any records at that stage yet.

Finally, add `importance` as a custom ranking attribute in the ranking tab under the "Ranking Formula & Custom Ranking" section. This will allow the tie-break algorithm to give preference to higher fragments in the document structure. In other words, h1 tags will rank higher than h2 tags if they otherwise have the same textual score.

4. Locate your ghost config file (config.production.json if ghost is running in production mode) and append the algolia object to it:

   ```json
   "algolia": {
       "active": true,
       "applicationID": "[YOUR_ALGOLIA_APP_ID]",
       "apiKey": "[YOUR_ALGOLIA_API_KEY]",
       "index": "[YOUR_ALGOLIA_INDEX]"
     }
   ```
5. Apply the `ghost_algolia_register_events.patch` patch found in the app download by running the following command from the ghost root:

   ```shell
   patch -p1 < ./content/apps/ghost-algolia/ghost_algolia_register_events.patch
   ```

6. Restart ghost


# Usage

Triggering indexing is transparent once the app is installed and happens on the following ghost panel operations:

- publishing a new post (add a new record)
- updating a published post (update an existing record)
- unpublishing a post (remove a record)
- deleting a post (remove a record)

# Compatibility
Tested against Ghost 1.x.x releases.

# Roadmap

- ~~Switching to [fragment indexing](https://github.com/mlbrgl/kirby-algolia#principle).~~
- Bulk indexing existing articles.
- Upgrade to App API when available, to remove core hacking and simplify the installation process.
