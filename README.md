# fssf-website
New website for F# Software Foundation

Website is generated by [Fornax](https://github.com/ionide/Fornax).

Modifications are done by pull request, which are built automatically and the preview content is published to an [Azure Storage Account](https://fssfpreview.z22.web.core.windows.net) with any pull requests going to a subdirectory for the pull request number to help contributors test their changes.

## How to contribute

To contribute, first check the [projects](https://github.com/fsharp/fssf-website/projects/1) for static content to migrate. Move the note to "In progress", convert it to an issue, and assign it to yourself so others are aware you are working on it.

1. Fork and clone this repository
2. Install the [.NET Core 3.1 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.1).
3. Run `dotnet tool restore` - this will download the static site generator `fornax`.
4. Edit the website content locally. If using an editor such as VS Code with Ionide or Rider, you'll have syntax highlighting and typechecking as you author the content.
5. To build the site locally, use `dotnet fornax build`.
6. To run the site on a local server, use `dotnet fornax watch` and then browse to http://localhost:8080 to see the content.
7. When your works is ready to publish, commit the changes and create a pull request from your fork.

Once your work is merged, please close the issue. If the work is too large and you'd like to split it up, add an additional note card with the balance of work.

### No, I mean how to contribute with *Fornax*?

Fornax is a static site generator, so it generates (checks notes) _strings_. There are `Loaders` that can read data from various sources, put that into `SiteContents` and then `Generators` that output strings from the `SiteContents` record.

Fornax includes a full Domain Specific Language (DSL) for generating HTML markup in a type safe, composable way, but ultimately, a `Generator` will call `HtmlElement.ToString` and convert it all to a string. Anything else that can create a string can be used as well, including reading a static file of HTML markup created with some other editor. If you have an existing string of HTML content, use the `!!` operator to convert it to an `HtmlElement` so it can be composed with other markup using the DSL.

For a practical example, suppose you have a layout that defines the site's overall navigation elements, headers, and footers that you want to remain consistent throughout the site. Within those navigation elements, you want each page to have different content. To achieve this, you'll need a generator for the layout that each page can reference so they \render their own content then inject it into the layout's generator that will generate the frame around that content. The result is that each page has its own internal content that is wrapped by a standard layout.

In fact, in `[index.fsx](generators/index.fsx)`, you will see it does exactly this:

1. It loads `[layout.fsx](generators/layout.fsx)`, which loads each of the loader modules. It's only loaded once and then cached for each page that calls it.
1. The loader modules that `layout.fsx` loads each add their content to `SiteContent`. For instance `[globalloader.fsx](loaders/globalloader.fsx)` adds a `SiteInfo` record with the title and description to the `SiteContent` that is available further in the pipeline.
1. The `generate'` function builds all the markup for this page and the result is an `HtmlElement` which is passed to `Layout.layout`. It has access to data loaded into `SiteContent by the loaders.
1. In `[layout.fsx](generators/layout.fsx)`, the `layout` function wraps the content for that page with the headers, navigation elements, and footers, then returns an `HtmlElement`.
1. The `generate` function at the bottom passes the `SiteContents` to `generate'` to build this `HtmlElement` (this page's content wrapped in the navigation layout) which is then passed to `Layout.render` which calls `HtmlElement.ToString` to generate the HTML content for the page.
1. In `[config.fsx](config.fsx)`, the `config` is defined with all the `Generators[]` that are used for the site. It includes `index.fsx` to generate `index.html`.

From a the top down, the pipeline for Fornax looks like this:

1. Read `[config.fsx](config.fsx)` to find generator scripts.
1. Load each generator script file from the config.
1. Each generator script references `[layout.fsx](generators/layout.fsx)`.
1. The `layout.fsx` module loads the various loaders to populate `SiteContent`.
1. The `generate` function is called on each generator script.
1. In each script's `generate` function
    a. The `generate'` function generates content.
    a. The content is passed to `layout` function in `[layout.fsx](generators/layout.fsx)`
    a. The content wrapped in a layout is passed to `Layout.render` which finally converts it all to a string of HTML.
1. The string of HTML is written to the file specified in `[config.fsx](config.fsx)` for that generator.

### Markdown to Lower Complexity for Content Contributions
In some cases, it's necessary to use Fornax's F# DSL to generate content, such as data driven content or the overall layout to wrap content. This gives a great deal of flexibility, but also adds complexity since contributors must understand F#, the Fornax HTML DSL, and the website's CSS styles. Whenever the content is very static and follows a standard format, simple markdown can be used for the content.

The `[postloader.fsx](loaders/postloader.fsx)` provides an example as a way to load content from markdown files. Some minimal metadata can be provided at the top of each markdown file to use in content generation before rendering HTML from the markdown. If people will frequently be contributing static content, please consider supporting loading this content from markdown files.

### Contributions are Welcome

Thank you for your contributions, and please reach out if you have any questions or suggestions!
