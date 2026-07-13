# ⚙️ SFDX CLI Setup

This repository implements a simple GitHub composite action for installing the Salesforce CLI and related plugins. The CLI installation is done via [npm](https://www.npmjs.com/package/@salesforce/cli).

## Usage

In a GitHub workflow, the use of the action after the initial checkout step and selecting the Node.js version to be used could look like this:

```yaml
jobs:
  validation:
    name: Validation
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v7

      - name: Select Node Version
        uses: svierk/get-node-version@main

      - name: Install Dependencies
        run: npm ci

      - name: Install SF CLI
        uses: svierk/sfdx-cli-setup@main
        with:
          version: 2.32.8
          plugins: "['sfdx-git-delta', '@salesforce/plugin-packaging']"
```

Two optional parameters for the sfdx-cli-setup action can be used to set a specific CLI version if needed, in this example 2.32.8, as well as the CLI plugins to be installed, in this case sfdx-git-delta and @salesforce/plugin-packaging.
The Node.js version in this example worflow is selected by using the action [get-node-version](https://github.com/svierk/get-node-version) that automatically pulls the version to be used from the _package.json_ file of the SFDX project.

## Inputs

| Name      | Required | Default | Description                                                                                          |
| --------- | -------- | ------- | -------------------------------------------------------------------------------------------------- |
| `plugins` | no       |         | SF CLI plugins to install as a JSON array, e.g. `"['@salesforce/plugin-packaging','sfdx-git-delta']"`. |
| `version` | no       |         | SF CLI version to install, e.g. `2.32.8`. Installs the latest version if omitted.                   |
| `step-summary` | no  | `true`  | Write a result section to the GitHub Actions [job summary](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#adding-a-job-summary). Set to `false` to avoid collisions with a custom workflow summary. |

## Overriding the version via repository variable

Installing the latest SF CLI version is the recommended default, since Salesforce ships fixes and features at a high cadence. However, sometimes you temporarily need to pin the pipeline to a fixed, known-good CLI version - ideally without touching the workflow file at all. For this, wire the `version` input to a [repository variable](https://docs.github.com/en/actions/learn-github-actions/variables) (organization and environment variables work the same way):

```yaml
- name: Install SF CLI
  uses: svierk/sfdx-cli-setup@main
  with:
    version: ${{ vars.SF_CLI_VERSION_OVERRIDE }}
```

This wiring is safe to keep in place permanently: as long as the variable is unset or blank, the expression evaluates to an empty string and the action installs the latest version as usual. Only when the variable is populated does it take precedence - so a temporary pin becomes a pure configuration change, and removing the variable's value restores the default behaviour. The job summary always shows the CLI version that was actually installed.

## Releases

Latest release notes can be found on the [release page](https://github.com/svierk/sfdx-cli-setup/releases).

## License

The scripts and documentation in this project are released under the [MIT License](https://github.com/svierk/sfdx-cli-setup/blob/main/LICENSE).