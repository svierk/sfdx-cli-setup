# ⚙️ SFDX CLI Setup

This repository implements a simple GitHub composite action for installing the Salesforce CLI and related plugins. The CLI installation is done via [npm](https://www.npmjs.com/package/@salesforce/cli).

## Usage

In a GitHub workflow, the use of the action after the initial checkout step and selecting the Node.js version to be used could look like this:

```
jobs:
  validation:
    name: Validation
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

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


## Releases

Latest release notes can be found on the [release page](https://github.com/svierk/sfdx-cli-setup/releases).

## License

The scripts and documentation in this project are released under the [MIT License](https://github.com/svierk/sfdx-cli-setup/blob/main/LICENSE).