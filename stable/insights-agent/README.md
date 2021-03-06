# Fairwinds Insights Agent

The Fairwinds Insights Agent is a collection of reporting tools, which send data back
to [Fairwinds Insights](https://insights.fairwinds.com).

## Installation
We recommend installing `insights-agent` in its own namespace.

```
helm repo add fairwinds-stable https://charts.fairwinds.com/stable
helm upgrade --install insights-agent fairwinds-stable/insights-agent \
  --namespace insights-agent \
  --set insights.organization=acme-co \
  --set insights.cluster=staging \
  --set insights.base64token="abcde=="
```

## CronJob schedules
Most Insights reports run as CronJobs, typically every hour.

In order to avoid every report consuming resources at once, you can stagger the reports
by setting the minute of the cron schedule to a random number. For example,
to run at a random minute every hour, your cron expression can be `rand * * * *`.

## Reports
There are several different report types which can be enabled and configured:
* `polaris`
* `goldilocks`
* `workloads`
* `kubehunter`
* `trivy`
* `kubesec`
* `releasewatcher`
* `rbacreporter`

See below for configuration details.

## Configuration
Parameter | Description | Default
--------- | ----------- | -------
`insights.organization` | The name of the organization to upload data to | ""
`insights.cluster` | The name of the cluster the data is coming from | ""
`insights.base64token` | Your cluster's base64-encoded auth token provided by Insights | ""
`insights.tokenSecretName` | If you don't provide a `base64token`, you can specify the name of a secret to pull the token from | ""
`insights.host` | The location of the Insights server | https://insights.fairwinds.com
`uploader.image.repository`  | The repository to pull the uploader script from | quay.io/fairwinds/insights-uploader
`uploader.image.tag` | The tag to use for the uploader script | 0.1
`cronjobs.backoffLimit` | Backoff limit to use for each report CronJob | 1
`cronjobs.failedJobsHistoryLimit` | Number of failed jobs to keep in history for each report | 2
`cronjobs.successfulJobHistoryLimit` | Number of successful jobs to keep in history for each report | 2
`cronjobs.nodeSelector` | Node selector to use for cronjobs | null
`{report}.enabled` | Enable the report type |
`{report}.schedule` | Cron expression for running the report |
`{report}.timeout` | Maximum time in seconds to wait for the report |
`{report}.resources` | CPU/memory requests and limits for the report |
`{report}.image.repository` | Repository to use for the report image |
`{report}.image.tag` | Image tag to use for the report |
`kubebench.mode` | Changes the way this plugin is deployed, either `cronjob` where it will run a single pod on the `schedule` that will pull the data necessary from a single node and report that back to Insights. `daemonset` which deploys a daemonset to the cluster which gathers data then a cronjob will gather data from each of those pods. `daemonsetMaster`  is the same as `daemonset` except the daemonset can also run on masters. | `cronjob`
`kubebench.hourInterval` | If running in `daemonset` or `daemonsetMaster` this configuration changes how often the daemonset pods will rescan the node they are running on | 2
`kubebench.aggregator` | contains `resources` and `image.repository` and `image.tag`, this controls the pod scheduled via a CronJob that aggregates from the daemonset in `daemonset` or `daemonsetMaster` deployment modes. |
`trivy.privateImages.dockerConfigSecret` | Name of a secret containing a docker `config.json` | ""
`trivy.maxConcurrentScans` | Maximum number of scans to run concurrently | 5
`trivy.namespaceBlacklist` | Specifies which namespaces to not scan, takes an array of namespaces for example: `--set trivy.namespaceBlacklist="{kube-system,default}"` | nil
`goldilocks.controller.exclude-namespaces` | Namespaces to exclude from the goldilocks report | `kube-system`
