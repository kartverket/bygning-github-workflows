# Bygning K8S Deployment Feedback

Custom GitHub Action for å verifisere at en K8S deployment har blitt deployet korrekt til et gitt miljø. 
Actionen sjekker at alle pods i en deployment har blitt deployet med riktig image tag og at alle pods er i "Ready" state.

I praksis er dette en wrapper rundt [kartverket/matrikkel-actions/k8s-deployment-feedback](https://github.com/kartverket/matrikkel-actions/tree/main/k8s-deployment-feedback)
med verdier spesifikke for Team Bygning.

## For å ta det i bruk

For å ta workflowen i bruk må følgende være på plass.

### Service account
En service account definert i [gcp-service-accounts](https://github.com/kartverket/gcp-service-accounts/blob/main/resources.yaml) 
med minimum følgende rettigheter. I tillegg må repoet du skal deploye fra (applikasjonens repo) ligge under `repositories`.
Dette gjør at service accounten kan bruke Workload Identity Federation til å autentisere seg mot GCP og K8S.

Eksempel på service account:

```yaml
matbrukeradmin:
  gcp_projects:
    sandbox: mat-brukeradmin-sandbox-8b4d
    dev: mat-brukeradmin-dev-8cf7
    prod: mat-brukeradmin-prod-482b
  repositories:
    - kartverket/matrikkel-brukerrettighet
  extra_sa_roles:
    - roles/iam.serviceAccountTokenCreator
  extra_kubernetes_sa_roles:
    - roles/container.clusterViewer
```

Service accounten er definert i denne workflowen og trenger derfor ikke defineres i hvert enkelt applikasjonsrepo (se `inputs.config`).

### RBAC regler

For at workflowen skal kunne bruke `kubectl` til å verifisere at applikasjonen har startet opp kreves det at deploy kontoen har rettigheter til dette i det aktuelle namespacet, definert gjennom noen RBAC-regler.

Dette må defineres i [bygning-apps](https://github.com/kartverket/bygning-apps) hvor man kan bruke funksjonen 
[K8sFeedbackRbac](https://github.com/kartverket/bygning-apps/blob/main/lib/bygning-libsonnet/v1/k8s.libsonnet#L146C3-L146C18) 
og sørge for at ressursene funksjonen definerer er satt opp i namespacet.

## Bruk

```yaml
  deploy-dev:
    name: "Deploy to dev"
    runs-on: ubuntu-26.04
    if: github.ref_name == 'main'
    needs:
      - build
    permissions:
      actions: read
      contents: read
      id-token: write
    env:
      version: ${{ needs.build.outputs.image_tag }}
    steps:
      # ... deploy applikasjon
      - name: "Verify deployment"
        uses: kartverket/bygning-github-workflows/bygning-k8s-deployment-feedback@main
        with:
          cluster: atkv3-dev
          apps: |
            mat-brukeradmin-stage:matrikkel-brukerrettighet:${{ env.version }}
            mat-brukeradmin-prodtest:matrikkel-brukerrettighet:${{ env.version }}
```
