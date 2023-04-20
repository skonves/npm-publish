# NPM Publish

This is a composite action that publishes an NPM package when a PR is merged that updates the version. This action is deisgned to be used with [skonves/npm-version](https://github.com/skonves/npm-version).

## Usage

The following workflow will be triggered when a PR is merged:

```yaml
# .github/workflows/publish.yml
name: publish

on:
  pull_request:
    types:
      - closed

jobs:
  compare:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    outputs:
      base_version: ${{ steps.base.outputs.version }}
      current_version: ${{ steps.current.outputs.version }}
    steps:
      - uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.base.sha }}
      - id: base
        run: echo "version=$(jq -r .version < package.json)" >> "$GITHUB_OUTPUT"
      - uses: actions/checkout@v3
      - id: current
        run: echo "version=$(jq -r .version < package.json)" >> "$GITHUB_OUTPUT"
  publish:
    needs: compare
    if: needs.compare.outputs.base_version != needs.compare.outputs.current_version
    runs-on: ubuntu-latest
    steps:
      - uses: skonves/npm-publish@main
        with:
          token: ${{ secrets.NPM_TOKEN }}
```

To support [NPM Provenance](https://github.blog/changelog/2023-04-19-npm-provenance-public-beta/), use the `@provenance` tag and supply the following permissions:

```yaml
  publish:
    # ...
    permissions:
      contents: write
      discussions: write
      id-token: write
      pull-requests: write
    steps:
      - uses: skonves/npm-publish@provenance
        # ...
```

This action requires that a [GitHub secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets) named `NPM_TOKEN` is configured that contains an [NPM Automation token](https://github.blog/changelog/2020-10-02-npm-automation-tokens/).

## More info

The following article on Medium describes the design philosophy behind this action: [Publishing NPM packages without a local environment](https://medium.com/@stevekonves/publishing-npm-packages-without-a-local-environment-b392f40d1817).
