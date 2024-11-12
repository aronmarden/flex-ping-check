# New Relic Flex Integration: Ping Check

This repository contains two files that work with the [New Relic Flex integration](https://docs.newrelic.com/docs/infrastructure/host-integrations/host-integrations-list/flex-integration-tool-build-your-own-integration/): `pingCheck.yml` and `ips.json`. These files need to be placed inside the integrations folder on any server that has the [New Relic Infrastructure agent installed.](https://docs.newrelic.com/docs/infrastructure/introduction-infra-monitoring/)

## Installation

### For Linux:

Ensure the [New Relic Infrastructure agent is installed.](https://docs.newrelic.com/docs/infrastructure/introduction-infra-monitoring/)

Once the agent is installed, place the files in the following directory:

```
/etc/newrelic-infra/integrations.d
```

Further instructions can be found [here](https://docs.newrelic.com/docs/infrastructure/host-integrations/host-integrations-list/flex-integration-tool-build-your-own-integration/#installation).

### For Windows:

This flex config is only suitable for Linxu at this time. It is not hard to retrofit this for Windows.

## Usage

1. Populate the `ips.json` file with as many IP addresses as you like. The Flex agent will automatically iterate through each one.

2. You can view the results in New Relic using the following NRQL query:

```
SELECT * FROM pingCheck
```

You can change the `pingCheck` name by modifying line 8 in the `pingCheck.yml` file:

```
        - event_type: pingCheck
```

3. Create alerts based on the `packetLoss` attribute if it increases above 0. Alerting documentation can be found [here](https://docs.newrelic.com/docs/alerts/overview/).

## Example `ips.json` Structure

```json
[
  {
    "name": "CCTV1",
    "addr": "127.0.0.1"
  },
  {
    "name": "CCTV2",
    "addr": "8.8.8.8"
  }
]
```

## Example `pingCheck.yml` Structure

```yaml
integrations:
  - name: nri-flex
    interval: 60s
    config:
      name: pingCheck
      lookup_file: /etc/newrelic-infra/integrations.d/ips.json
      apis:
        - event_type: pingCheck
          custom_attributes:
            deviceName: ${lf:name}
            deviceAddr: ${lf:addr}
          commands:
            - run: ping -c 1 ${lf:addr} || true
              split_output: statistics ---
              regex_matches:
                - expression: ([0-9]+\.?[0-9]+)\/([0-9]+\.?[0-9]+)\/([0-9]+\.?[0-9]+)\/([0-9]+\.?[0-9]+)
                  keys: [min, avg, max, stddev]
                  ### there are two different variants for the packet statistics returned, below allows support for both
                - expression: (\d+) packets transmitted, (\d+) packets received, (\S+)% packet loss
                  keys: [packetsTransmitted, packetsReceived, packetLoss]
                - expression: (\d+) packets transmitted, (\d+) received, (\d+)% packet loss, time (\d+)
                  keys:
                    [packetsTransmitted, packetsReceived, packetLoss, timeMs]
```

## Example output from FLEX

```json
{
  "name": "com.newrelic.nri-flex",
  "protocol_version": "3",
  "integration_version": "1.5.3",
  "data": [
    {
      "metrics": [
        {
          "avg": 0.276,
          "deviceAddr": "127.0.0.1",
          "deviceName": "CCTV1",
          "event_type": "pingCheck",
          "flex.commandTimeMs": 588,
          "integration_name": "com.newrelic.nri-flex",
          "integration_version": "1.5.3",
          "max": 0.276,
          "min": 0.276,
          "packetLoss": 0,
          "packetsReceived": 1,
          "packetsTransmitted": 1,
          "stddev": 0
        },
        {
          "avg": 12.855,
          "deviceAddr": "8.8.8.8",
          "deviceName": "CCTV2",
          "event_type": "pingCheck",
          "flex.commandTimeMs": 601,
          "integration_name": "com.newrelic.nri-flex",
          "integration_version": "1.5.3",
          "max": 12.855,
          "min": 12.855,
          "packetLoss": 0,
          "packetsReceived": 1,
          "packetsTransmitted": 1,
          "stddev": 0
        },
        {
          "event_type": "flexStatusSample",
          "flex.Hostname": "xxxxxxxxxxxxx",
          "flex.IntegrationVersion": "1.5.3",
          "flex.counter.ConfigsProcessed": 2,
          "flex.counter.EventCount": 2,
          "flex.counter.EventDropCount": 0,
          "flex.counter.pingCheck": 2,
          "flex.time.elapsedMs": 608,
          "flex.time.endMs": 1731041423950,
          "flex.time.startMs": 1731041423342
        }
      ],
      "inventory": {},
      "events": []
    }
  ]
}
```

## Example Alert Condition in Terraform

```sql
resource "newrelic_nrql_alert_condition" "cctv_ping_check_condition" {
  account_id = <Your Account ID>
  policy_id = <Your Policy ID>
  type = "static"
  name = "CCTV Ping Check Condition"
  enabled = true
  violation_time_limit_seconds = 259200

  nrql {
    query = "FROM pingCheck SELECT latest(packetLoss) FACET deviceName, deviceAddr "
  }

  critical {
    operator = "above"
    threshold = 0
    threshold_duration = 60
    threshold_occurrences = "all"
  }
  fill_option = "none"
  aggregation_window = 60
  aggregation_method = "event_flow"
  aggregation_delay = 120
}


```

## Example Alert Condition in GraphQL

```sql
mutation {
  alertsNrqlConditionStaticCreate(
    accountId: <Your Account ID>
    policyId: <Your Policy ID>
    condition: {
      enabled: true
      name: "CCTV Ping Check Condition"
      description: null
      titleTemplate: null
      nrql: {
        query: "FROM pingCheck SELECT latest(packetLoss) FACET deviceName, deviceAddr "
      }
      expiration: null
      runbookUrl: null
      signal: {
        aggregationWindow: 60
        fillOption: NONE
        aggregationDelay: 120
        aggregationMethod: EVENT_FLOW
        aggregationTimer: null
        fillValue: null
        slideBy: null
        evaluationDelay: null
      }
      terms: [
        {
          operator: ABOVE
          threshold: 0
          priority: CRITICAL
          thresholdDuration: 60
          thresholdOccurrences: ALL
        }
      ]
      violationTimeLimitSeconds: 259200
    }
  ) {
    id
  }
}
```
