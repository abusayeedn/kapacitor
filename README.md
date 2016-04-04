# Kapacitor [![Circle CI](https://circleci.com/gh/influxdata/kapacitor/tree/master.svg?style=svg&circle-token=78c97422cf89526309e502a290c230e8a463229f)](https://circleci.com/gh/influxdata/kapacitor/tree/master)
Open source framework for processing, monitoring, and alerting on time series data

# Installation

Kapacitor has two binaries:

* kapacitor – a CLI program for calling the Kapacitor API.
* kapacitord – the Kapacitor server daemon.

You can either download the binaries directly from the [downloads](https://influxdata.com/downloads/#kapacitor) page or go get them:

```sh
go get github.com/influxdata/kapacitor/cmd/kapacitor
go get github.com/influxdata/kapacitor/cmd/kapacitord
```

# Configuration
An example configuration file can be found [here](https://github.com/influxdata/kapacitor/blob/master/etc/kapacitor/kapacitor.conf)

Kapacitor can also provide an example config for you using this command:

```sh
kapacitord config
```


# Getting Started

This README gives you a high level overview of what Kapacitor is and what its like to use it. As well as some details of how it works.
To get started using Kapacitor see [this guide](https://docs.influxdata.com/kapacitor/v0.11/introduction/getting_started/).

# Basic Example

Kapacitor use a DSL named [TICKscript](https://docs.influxdata.com/kapacitor/v0.11/tick/) to define tasks.

A simple TICKscript that alerts on high cpu usage looks like this:

```javascript
stream
    |from()
        .measurement('cpu_usage_idle')
        .groupBy('host')
    |window()
        .period(1m)
        .every(1m)
    |mean('value')
    |eval(lambda: 100.0 - "mean")
        .as('used')
    |alert()
        .message('{{ .Level}}: {{ .Name }}/{{ index .Tags "host" }} has high cpu usage: {{ index .Fields "used" }}')
        .warn(lambda: "used" > 70.0)
        .crit(lambda: "used" > 85.0)

        // Send alert to hander of choice.

        // Slack
        .slack()
        .channel('#alerts')

        // VictorOps
        .victorOps()
        .routingKey('team_rocket')

        // PagerDuty
        .pagerDuty()
```

Place the above script into a file `cpu_alert.tick` then run these commands to start the task:

```sh
# Define the task (assumes cpu data is in db 'telegraf')
kapacitor define \
    -name cpu_alert \
    -type stream \
    -dbrp telegraf.default \
    -tick ./cpu_alert.tick
# Start the task
kapacitor enable cpu_alert
```

For more complete examples see the [documentation](https://docs.influxdata.com/kapacitor/v0.11/examples/)
