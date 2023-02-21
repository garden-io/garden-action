# GitHub Actions for Garden

This action can be used to run Garden against a given environment.

## Inputs

## `command`

**Optional** The Garden command to execute, including all options. For example `deploy`, `test`, `run workflow` etc. For full documentation see [documentation](https://docs.garden.io/reference/commands).

## `kubeconfig`

**Optional** Authentication to a Kubernetes Cluster can be done in multiple ways. This option allows to specify a base64 encoded kubeconfig, as a secret for GitHub actions. To use this option, base64 encode the relevant kubeconfig with the context referenced in your Garden project:

```

cat kubeconfig.yaml | base64

```

Encoding is necessary to deal with newlines and special characters. This action will decode the kubeconfig for usage in the action.

## `kubeconfig-location`

**Optional** Specify a location the GitHub action should be saved to in the container while running the action. This is only necessary if you have configured the `kubeconfig` parameter in your project.garden.yaml provider configuration. Please note that the home directory in the GitHub action context is `/github/home`.
Defaults to `/github/home/.kube/config`

## `garden-version`

**Optional** Garden version. Default is latest

## `garden-auth-token`

**Optional** A token to authenticate to Garden Cloud.

## `garden-workdir`

**Optional** A path to a garden project in a repository. Only necessary if there are multiple garden projects in a repository

## `github-token`

**Optional** This token will be used to authenticate to GitHub API for fetching the latest Garden release. Default is ${{ github.token }}


## Outputs


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
        uses: aws-actions/configure-aws-credentials@67fbcbb121271f7775d2e7715933280b06314838 # 1.7.0
        with:
          aws-region: eu-central-1
          role-to-assume: ${{ secrets.AWS_ROLE_EKS_DEV }}
          role-session-name: GitHubActionsDev
          role-duration-seconds: 3600
      - uses: actions/checkout@2541b1294d2704b0964813337f33b291d3f8596b # 3.0.2
      - name: Deploy preview env with Garden
        uses: ./.github/actions/garden
        with:
          command: deploy --env preview
          kubeconfig: ${{ secrets.KUBECONFIG }}
          garden-auth-token: ${{ secrets.GARDEN_AUTH_TOKEN }}
  garden-ci:
    runs-on: ubuntu-latest
    steps:
      - name: AWS auth
        uses: aws-actions/configure-aws-credentials@67fbcbb121271f7775d2e7715933280b06314838 # 1.7.0
        with:
          aws-region: eu-central-1
          role-to-assume: ${{ secrets.AWS_ROLE_EKS_DEV }}
          role-session-name: GitHubActionsDev
          role-duration-seconds: 3600
      - uses: actions/checkout@2541b1294d2704b0964813337f33b291d3f8596b # 3.0.2
      - name: Run tests in ci environment with Garden
        uses: ./.github/actions/garden
        id: garden
        with:
          command: >
            test --env ci
            --var postgres-database=postgres
            --var postgres-password=${{ secrets.PG_PASSWORD }}
          kubeconfig: ${{ secrets.KUBECONFIG }}
          garden-auth-token: ${{ secrets.GARDEN_AUTH_TOKEN }}
```
