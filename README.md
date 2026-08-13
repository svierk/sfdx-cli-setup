# ⚙️ SFDX CLI Setup

This repository implements a simple GitHub composite action for installing the Salesforce CLI and related plugins. The CLI installation is done via [npm](https://www.npmjs.com/package/@salesforce/cli).

## Usage

In a GitHub workflow, the use of the action after the initial checkout step and selecting the Node.js version to be used could look like this:

```yaml
jobs:
  validation:
    name: Validation
    runs-on: ubuntu-latest
    permissions:
      contents: read # least-privilege GITHUB_TOKEN - this job only reads the repository
    steps:
      - name: Checkout
        uses: actions/checkout@v7.0.1
        with:
          persist-credentials: false # the GITHUB_TOKEN is not needed after the checkout - don't leave it in .git/config

      - name: Select Node Version
        uses: svierk/get-node-version@v1.5.1

      - name: Install Dependencies
        run: npm ci

      - name: Install SF CLI
        uses: svierk/sfdx-cli-setup@v1.1.2
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
  uses: svierk/sfdx-cli-setup@v1.1.2
  with:
    version: ${{ vars.SF_CLI_VERSION_OVERRIDE }}
```

This wiring is safe to keep in place permanently: as long as the variable is unset or blank, the expression evaluates to an empty string and the action installs the latest version as usual. Only when the variable is populated does it take precedence - so a temporary pin becomes a pure configuration change, and removing the variable's value restores the default behaviour. The job summary always shows the CLI version that was actually installed.

## Security & versioning

Every `uses:` reference in the snippets above is **pinned to an exact release version**, e.g. `svierk/sfdx-cli-setup@v1.1.2`. Do the same in your own pipelines:

- **Never reference a mutable ref** such as `@main` or `@v1`. It runs whatever code sits on that branch/tag at run time - with access to your org credentials - so a compromised or rewritten ref would run unnoticed.
- **Good - pin to an exact release tag** (`@v1.1.2`). Readable, concrete, and bumped through reviewed pull requests.
- **Strictest - pin to a full-length commit SHA** (`@a1b2c3d…`) with the version as a trailing comment. A SHA can never be re-pointed by the publisher; the cost is readability.
- **Enable [Dependabot](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/keeping-your-actions-up-to-date-with-dependabot) for `github-actions`** in the repository that consumes this action, so those pins are bumped for you instead of silently ageing.

This action itself needs **no secrets and no `GITHUB_TOKEN` permissions** - it only installs the Salesforce CLI and its plugins. Grant the calling job the least privilege it needs (`contents: read` is enough for the example above) and use `persist-credentials: false` on the checkout so the token is not written to `.git/config` where the CLI or a plugin could pick it up.

If you pass credentials to *other* steps of the same workflow, reference them in shell steps as **environment variables** (`"$SFDX_AUTH_URL"`), never by interpolating `${{ secrets.… }}` into the script itself - that would allow command injection and can leak values into the log. The steps of this action follow that rule for all of their inputs.

## Releases

Latest release notes can be found on the [release page](https://github.com/svierk/sfdx-cli-setup/releases).

## License

The scripts and documentation in this project are released under the [MIT License](https://github.com/svierk/sfdx-cli-setup/blob/main/LICENSE).