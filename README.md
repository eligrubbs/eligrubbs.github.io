# eligrubbs.com

Source for [www.eligrubbs.com](https://www.eligrubbs.com), served by GitHub Pages.

This is a plain static site with no build step or dependencies.

## Run locally

```sh
python3 -m http.server 8000
```

Then open <http://localhost:8000>.

## Structure

- `index.html` is the landing page: intro, links, and projects.
- `404.html` is served by GitHub Pages for missing URLs.
- `assets/css/style.css` holds every style. It is organized as an atomic design system in cascade layers: tokens, reset, base, atoms, molecules, organisms, layout. To change colors, type, or spacing, edit the custom properties in the `tokens` layer.
- `CNAME` sets the custom domain. Don't delete it.

## Adding a project

Copy an existing `<li class="project">` block in `index.html` and edit the link, language tag, and description.
