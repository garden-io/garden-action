# Garden Github Action

This action installs garden and can optionally be used to run any [Garden](https://garden.io) command, for example `deploy`, `test` or `run workflow`.

Garden combines rapid development, testing, and DevOps automation in one tool. 

This action will perform the following steps:

1. Download Garden from the GitHub release artifacts for the given version (default latest) at [garden-io/garden](https://github.com/garden-io/garden)
2. Verify the SHA256 checksum
3. Export garden to the `PATH`, so it can be used from any scripts in the following steps of the GitHub Action job.
4. If the `command` option is provided, it will run the given garden command.

   If the `command` option is *not* provided it will only prepare garden, which means it will also export the `KUBECONFIG` and `GARDEN_AUTH_TOKEN` environment variables if the `kubeconfig` and `garden-auth-token` are configured. This is helpful when calling `garden` in scripts from one of the following steps.

**Note:** At the moment this action only works with Linux-based GitHub Action runners.
If you are using macOS or Windows runners and need this action, please open a GitHub issue – in case there is demand, we will rewrite this action to make it platform-independent. (We also accept Pull requests for rewriting this Action in Typescript)

## Inputs

## `command`

**Optional** The Garden command to execute, including all options. For example `deploy`, `test`, `run workflow` etc.

If not provided, the garden-action will only install garden and export the `KUBECONFIG` and `GARDEN_AUTH_TOKEN` environment variables for use in scripts in later steps.

For the full documentation please refer to the [Garden CLI documentation](https://docs.garden.io/reference/commands).

## `kubeconfig`

**Optional** Authentication to a Kubernetes Cluster can be done in multiple ways. This option allows to specify a base64 encoded kubeconfig, as a secret for GitHub actions. To use this option, base64 encode the relevant kubeconfig with the context referenced in your Garden project:

```

cat kubeconfig.yaml | base64

```

Encoding is necessary to deal with newlines and special characters. This action will decode the kubeconfig for usage in the action.

The secret will be [masked to prevent accidental exposure in logs](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#masking-a-value-in-log)

**If no command has been supplied, the action will expose this value to the the following steps in the GitHub Action job by exporting a `KUBECONFIG` environment variable.**

## `kubeconfig-location`

**Optional** Specify a location the GitHub action should be saved to in the container while running the action. This is only necessary if you have configured the `kubeconfig` parameter in your project.garden.yaml provider configuration.

Defaults to `${{ runner.temp }}/garden/kubeconfig`

## `garden-version`

**Optional** Garden version. Default is latest

## `garden-auth-token`

**Optional** A token to authenticate to Garden Cloud.

The secret will be [masked to prevent accidental exposure in logs](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#masking-a-value-in-log).

**If no command has been supplied, the action will expose this value to the the following steps in the GitHub Action job by exporting a `GARDEN_AUTH_TOKEN` environment variable.**

## `garden-workdir`

**Optional** A path to a garden project in a repository.

Only necessary if there are multiple garden projects in a repository or if the `project.garden.yml` is in a subdirectory.

## `github-token`

**Optional** This token will be used to authenticate to GitHub API for fetching the latest Garden release. Defaults to `${{ github.token }}`.

The secret will be [masked to prevent accidental exposure in logs](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#masking-a-value-in-log).


## Outputs

The garden-action does not export any outputs.

## Example usage

This example uses the `aws-actions/configure-aws-credentials` action beforehand to authenticate to AWS. This might look different with other cloud providers.
It deploys a preview environment for other team members/teams to explore and tests the latest pushed code in a separate ci environment. In the ci environment, some additional variables are used.

```
name: garden
on:
  push:
    branches:
      - main
jobs:
  garden-preview:
    runs-on: ubuntu-latest
    steps:
      - name: AWS auth
        uses: aws-actions/configure-aws-credentials@v1.7.0
        with:
          aws-region: eu-central-1
          role-to-assume: ${{ secrets.AWS_ROLE_EKS_DEV }}
          role-session-name: GitHubActionsDev
          role-duration-seconds: 3600
      - uses: actions/checkout@v3.0.2
      - name: Deploy preview env with Garden
        uses: garden-io/garden-action@v1.1
        with:
          command: deploy --env preview
          kubeconfig: ${{ secrets.KUBECONFIG }}
          garden-auth-token: ${{ secrets.GARDEN_AUTH_TOKEN }}
  garden-ci:
    runs-on: ubuntu-latest
    steps:
      - name: AWS auth
        uses: aws-actions/configure-aws-credentials@v1.7.0
        with:
          aws-region: eu-central-1
          role-to-assume: ${{ secrets.AWS_ROLE_EKS_DEV }}
          role-session-name: GitHubActionsDev
          role-duration-seconds: 3600
      - uses: actions/checkout@v3.0.2
      - name: Run tests in ci environment with Garden
        uses: garden-io/garden-action@v1.1
        with:
          command: >
            test --env ci
            --var postgres-database=postgres
            --var postgres-password=${{ secrets.PG_PASSWORD }}
          kubeconfig: ${{ secrets.KUBECONFIG }}
          garden-auth-token: ${{ secrets.GARDEN_AUTH_TOKEN }}
```
