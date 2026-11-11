# Bygning approval notifier

Custom GitHub action for å sende slack-melding til bygning-github kanalen når en ny versjon av en applikasjon er deployet til dev, test eller prod.

I praksis er dette en wrapper rundt heimdall sin [slack-approval-notifier](https://github.com/kartverket/matrikkel-actions/tree/main/slack-approval-notifier)
med default verdier for Team Bygning.

## For å ta det i bruk

Repoet som actionen skal brukes fra må være satt opp med korrekt Octo STS konfigurasjon i [bygning-apps](https://github.com/kartverket/bygning-apps/tree/main/.github/chainguard) repoet.

`SLACK_BOT_TOKEN` må være tilgjengelig som en Action secret på det aktuelle repoet som skal bruke actionen.

## Bruk

```yaml

# Varsler til Slack at prodsetting vil skje
jobs:
  deploy-prod-notifier:
    name: Slack Notify; Deploy to Production
    runs-on: ubuntu-26.04
    outputs:
      messageId: ${{ steps.slack_notify.outputs.messageId }}
    permissions:
      actions: read
      contents: read
      id-token: write
    steps:
      - name: Notify slack (Setup)
        uses: kartverket/matrikkel-actions/bygning-approval-notifier/setup@main
        with:
          identity: 'matrikkel-brukeradmin'
          environment: "produksjon"
          appDescriptor: atkv3-prod:mat-brukeradmin-prod:matrikkel-brukeradmin:2026.11.06-09.22-9da3d8d
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

# Oppdaterer Slack-melding med status på utrulling
  deploy-prod:
    name: Deploy to production
    runs-on: ubuntu-26.04
    environment:
      name: production
    permissions:
      actions: read
      contents: read
      id-token: write
    steps:
      - name: Notify slack (Update)
        uses: kartverket/matrikkel-actions/bygning-approval-notifier/update@main
        with:
          identity: 'matrikkel-brukeradmin'
          environment: "produksjon"
          appDescriptor: atkv3-prod:mat-brukeradmin-prod:matrikkel-brukeradmin:2026.11.06-09.22-9da3d8d
          messageId: ${{ needs.deploy-prod-notifier.outputs.messageId }}
          status: ${{ job.status }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```
