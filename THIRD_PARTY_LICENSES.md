# Third-party licenses

ovtools bundles or links against the libraries below. Each ships with its own license terms; this file lists them so attribution stays honest. ovtools itself is licensed under Apache 2.0 (see `LICENSE`).

This file covers direct dependencies and bundled assets. Transitive Go modules pulled in by client-go (about 30 of them) are tracked in `go.sum` and inherit Apache 2.0 / BSD-style licenses common to the Kubernetes ecosystem. Run `go-licenses report ./cmd/ovtools` if you need the exhaustive list for a compliance audit.

## Bundled JavaScript

`internal/server/static/js/chart.umd.min.js` is Chart.js v4.4.1, MIT License. https://github.com/chartjs/Chart.js/blob/master/LICENSE.md

`internal/server/static/js/cytoscape.min.js` is Cytoscape.js, MIT License. https://github.com/cytoscape/cytoscape.js/blob/master/LICENSE

`internal/server/static/js/cytoscape-dagre.min.js` is cytoscape-dagre, MIT License. https://github.com/cytoscape/cytoscape.js-dagre/blob/master/LICENSE

`internal/server/static/js/dagre.min.js` is dagre, MIT License. https://github.com/dagrejs/dagre/blob/master/LICENSE

`internal/server/static/js/htmx.min.js` is htmx, BSD Zero Clause License. https://github.com/bigskysoftware/htmx/blob/master/LICENSE

`internal/server/static/js/idiomorph-ext.min.js` is Idiomorph, BSD Zero Clause License. https://github.com/bigskysoftware/idiomorph/blob/main/LICENSE

`internal/server/static/js/disk-tooltip.js`, `nic-tooltip.js`, and `vm-drawer.js` are ovtools' own code, Apache 2.0.

## Bundled fonts

`internal/server/static/fonts/RedHatDisplay-*.woff2` and `RedHatText-*.woff2` are Red Hat Display and Red Hat Text by Red Hat Inc., SIL Open Font License 1.1. The full license text ships alongside the fonts in `internal/server/static/fonts/OFL.txt`. https://github.com/RedHatOfficial/RedHatFont

## Direct Go dependencies

The `require` block in `go.mod` lists what ovtools imports directly:

- `k8s.io/api`, `k8s.io/apimachinery`, `k8s.io/client-go` (Apache 2.0). The Kubernetes client. https://github.com/kubernetes/client-go/blob/master/LICENSE
- `github.com/xuri/excelize/v2` (BSD 2-Clause). XLSX export. https://github.com/qax-os/excelize/blob/master/LICENSE

## Versions and updates

When you bump a vendored library, update the version in this file in the same commit. Dependabot opens grouped PRs for Go modules weekly and Actions monthly; this file should follow.
