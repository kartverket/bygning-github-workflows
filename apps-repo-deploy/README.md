# Apps repo deploy

Custom GitHub action for å oppdatere versjonsnummeret til en applikasjon i apps repoet for å deploy ny versjon

## For å ta det i bruk

Repoet som actionen skal brukes fra må være satt opp med korrekt Octo STS konfigurasjon i [bygning-apps](https://github.com/kartverket/bygning-apps/tree/main/.github/chainguard) repoet

## Bruk

```yaml
jobs:
  deploy-to-dev:
    name: Deploy til dev
    runs-on: ubuntu-26.04
    permissions:
      id-token: write # To get OctoSTS token for updating apps-repo
  steps:
    - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1  # v7.0.1
    - uses: kartverket/bygning-github-workflows/apps-repo-deploy/action.yml@main
      with:
        cluster: atkv3-dev
        namespace: mat-brukeradmin-prod
        application_name: matrikkel-brukeradmin
        identity: matrikkel-brukeradmin
        image_name: ghcr.io/kartverket/matrikkel-brukeradmin
        image_tag: 2026.11.06-09.22-9da3d8d
        image_digest: sha256:151e2605fe4719bf5cd73aca9cb38063aeb735c402377337fb6ad41ef11985cb
```
