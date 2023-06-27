# initialize-variables
An action to check if a "full build" should be run by inspecting commit message and manual input for push, pull request, and manual runs (`workflow_dispatch`).


## Usage

To provide a `workflow_dispatch` input (e.g. to do a manual full build from the UI), add the following to your `on` block:

```yaml
on:
  # Allow for manual runs to override whether or not a "full build" is run
  workflow_dispatch:
    ci-full:
      description: "Run Full CI Build"
      required: false
      type: boolean
      default: false
```


To utilize this action, add it in a block like so:

```yaml
initialize:
runs-on: ubuntu-22.04
outputs:
  FULL_BUILD: ${{ steps.initialize.outputs.FULL_BUILD || steps.initialize.outputs.FULL_BUILD || steps.initialize.outputs.FULL_BUILD }}

steps:
  - name: Checkout
    uses: actions/checkout@v3
    with:
      fetch-depth: 2

  - name: Initialize variables
    uses: actions-ext/general/initialize-variables@v1
    id: initialize
    with:
        full: ${{ github.event_name == 'workflow_dispatch' && github.event.inputs.ci-full }}
```


You can now condition steps like so:
```yaml
  do_some_heavy_build:
    needs: [initialize]
    runs-on: ubuntu-latest
    if: ${{ needs.initialize.outputs.FULL_BUILD == 'true' }}
```


## Options

This action will set `FULL_BUILD` output in 2 ways:
- provide `full: true` input to the action
- have `[ci-full]` in the commit message or PR title
