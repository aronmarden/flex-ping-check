# New Relic Flex Integration: Ping Check

This repository contains two files that work with the New Relic Flex integration: `pingCheck.yml` and `ips.json`. These files need to be placed inside the integrations folder on any server that has the New Relic Infrastructure agent installed.

## Installation

### For Linux:

Place the files in the following directory:

```
/etc/newrelic-infra/integrations.d
```

### For Windows:

Place the files in the following directory:

```
C:\Program Files\New Relic\newrelic-infra\integrations.d\
```

Further instructions can be found [here](https://docs.newrelic.com/docs/infrastructure/host-integrations/host-integrations-list/flex-integration-tool-build-your-own-integration/#installation).

## Usage

1. Populate the `ips.json` file with as many IP addresses as you like. The Flex agent will automatically iterate through each one.

2. You can view the results in New Relic using the following NRQL query:

```
SELECT * FROM pingCheck
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
    config:
      name: pingCheck
      lookup_file: ips.json
      apis:
        - event_type: pingCheck
          commands:
            - run: ping -c 1 ${lf:addr} || true
              split_output: statistics ---
              regex_matches:
                - expression: ([0-9]+\.?[0-9]+)\/([0-9]+\.?[0-9]+)\/([0-9]+\.?[0-9]+)\/([0-9]+\.?[0-9]+)
                  keys: [min, avg, max, stddev]
                - expression: (\d+) packets transmitted, (\d+) packets received, (\S+)% packet loss
                  keys: [packetsTransmitted, packetsReceived, packetLoss]
                - expression: (\d+) packets transmitted, (\d+) received, (\d+)% packet loss, time (\d+)
                  keys:
                    [packetsTransmitted, packetsReceived, packetLoss, timeMs]
```

## Example output

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
