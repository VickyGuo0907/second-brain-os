# tools

## Automatic publishing

The `Build and deploy website` workflow rebuilds the graph, reader, resources
page, and tree from source on every push to `main`. It checks that documentation
pages appear in the generated reader, graph, and tree before publishing.
Pull requests run the same build checks without deploying.

One-time setup: in GitHub **Settings → Pages → Build and deployment**, set
**Source** to **GitHub Actions**. Merge the workflow into `main`; that push
starts the first deployment. If it needs retrying, use **Actions → Build and
deploy website → Run workflow** on `main`.

After setup, edit the Markdown, review and commit those source changes, and
push. You no longer need to run the four generators or commit generated output
for publication. Existing generated files remain in Git for compatibility;
the workflow always rebuilds them and deploys the fresh artifact without
writing a generated commit back to the repository.

The artifact contains the three HTML entry pages, `assets/`, `docs/`, and
`resources/`. Keep all content in those folders public. Your separate private
Obsidian vault is not used by this workflow. External JavaScript CDNs and
external resource links still need a browser check; the build checks do not
test their availability.

For a local preview, install `python -m pip install -r tools/requirements-site.txt`,
then use the build commands below and `python3 -m http.server 4173`.

The site at [secondbrainos.dev](https://secondbrainos.dev/)
is generated from this repository, so it cannot drift from the guide.

## The home page: one graph of the whole site

```bash
pip install markdown
python3 tools/build_graph_site.py   # -> assets/site/graph-data.js, assets/site/pages-data.js
python3 -m http.server 4173         # then open http://localhost:4173
```

`index.html` draws every section, page, term, skill, command, subagent, script
and resource as a node in a 3D graph, and every real link between them as an
edge. Click a node to fly to it, see what it links to and what links to it, and
open a page in the reader without leaving the graph.

| File | What it is |
|---|---|
| `index.html` | The page shell. Hand-written |
| `assets/site/app.css` | Layout and type. Palette and fonts are the guide's |
| `assets/site/app.js` | The graph, the panels, the reader, search, routing |
| `assets/site/graph-data.js` | Generated: nodes and edges |
| `assets/site/pages-data.js` | Generated: rendered guide pages |
| `tools/build_graph_site.py` | Builds both data files from the markdown |
| `tools/glossary.json` | The only hand-kept data: the terms on the graph |

`build_graph_site.py` keeps the reading order declared in each section's
`README.md`, resolves every relative link to a page, section or toolkit node,
links commands to the skill they follow, pages to the resources whose URLs they
cite, and terms to the pages that mention them most. Add a term by adding a line
to `glossary.json` and rerunning.

The graph is drawn with [3d-force-graph](https://github.com/vasturiano/3d-force-graph)
and three.js, loaded from jsDelivr at pinned versions. Everything else is in the
repo. Old links such as `index.html#02-setup/claude-md` still open that page.

## The other pages

```bash
python3 tools/extract_site.py   # docs and resources -> site_data.json
python3 tools/build_site.py     # -> resources.html
python3 scripts/build_tree.py   # -> tree.html
```

`resources.html` and `tree.html` are still built the old way. Neither script
touches `index.html`, and neither does `scripts/build_tracks.py`, which now only
keeps the course and handbook READMEs in the declared order.
