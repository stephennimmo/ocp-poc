# ocp-poc

OpenShift 4.20 proof of concept deployment using Red Hat Advanced Cluster Management (ACM) to provision and manage a bare metal cluster from a Single Node OpenShift (SNO) hub.

https://stephennimmo.github.io/ocp-poc

## Documentation

The full documentation is built with [Material for MkDocs](https://squidfunk.github.io/mkdocs-material) and published via GitHub Pages.

### Local Development

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install mkdocs-material mkdocs-macros-plugin mkdocs-open-in-new-tab
mkdocs serve --livereload
```

Then open [http://localhost:8000](http://localhost:8000).
