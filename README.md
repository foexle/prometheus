# Contents

- Introduction
  - [Overview](#overview)
  - [Pre-requisites](#pre-requisites)
  - [Installation & Configuration](#installation--configuration)
    - [Add Datasources & Dashboards](#add-datasources-and-dashboards)
    - [Alerting](#alerting)
    - [Test Alerts](#test-alerts)
  - [Security Considerations](#security-considerations)
  	- [Production Security](#production-security)
  - [Troubleshooting](#troubleshooting)
  - [Thanks](#thanks)

# Overview
This is a simple Prometheus stack to collect metrics from different nodes. The default dashboard in Grafana is configured to prepare and show graphs based on Telegraf's [Prometheus output plugin](https://github.com/influxdata/telegraf/tree/master/plugins/outputs/prometheus_client)

The stack is prepared to run only on a single machine if you would like to distribute for HA you should use `docker stack` or Kubernets instead.

# Pre-requisites
Before you start it's advisable to install a Telegraf agent first on any host/node otherwise you don't get any metrics. 
Follow the [official instructions](https://docs.influxdata.com/telegraf/v1.10/introduction/installation/#)

Next, you have to install `docker-compose` just follow the [offcial instructions](https://docs.docker.com/compose/install/) again.


# Installation & Configuration
Clone the project locally to your Docker host. 

The 'kafka' branch contains and older version which is based on the orginal repository + kafka exporter.


Once configurations are done let's start it up. Run the following command:

    $ docker-compose up -d


That's it the `docker-compose up -d' command deploys the entire Grafana and Prometheus stack automagically to your machine. 

The Grafana Dashboard is now accessible via: `http://<Host IP Address>:3000` for example http://127.0.0.1:3000

	username - admin
	password - foobar (Password is stored in the `/grafana/config.monitoring` env file)

In order to check the status of the newly created stack:


## Add Datasources and Dashboards
Grafana version 5.0.0 has introduced the concept of provisioning. This allows us to automate the process of adding Datasources & Dashboards. The `/grafana/provisioning/` directory contains the `datasources` and `dashboards` directories. These directories contain YAML files which allow us to specify which datasource or dashboards should be installed. 

If you would like to automate the installation of additional dashboards just copy the Dashboard `JSON` file to `/grafana/provisioning/dashboards` and it will be provisioned next time you stop and start Grafana.

## Add Grafana Plugins
The `/grafana/` directory contains a `Dockerfile` to build a new Grafana container with plugins included. You can add additional plugins with a comma separated list
	ARG GF_INSTALL_PLUGINS="grafana-piechart-panel"



## Alerting
Alerting has been added to the stack with Slack integration. 2 Alerts have been added and are managed

Alerts              - `prometheus/alert.rules`
Slack configuration - `alertmanager/config.yml`

The Slack configuration requires to build a custom integration.
* Open your slack team in your browser `https://<your-slack-team>.slack.com/apps`
* Click build in the upper right corner
* Choose Incoming Web Hooks link under Send Messages
* Click on the "incoming webhook integration" link
* Select which channel
* Click on Add Incoming WebHooks integration
* Copy the Webhook URL into the `alertmanager/config.yml` URL section
* Fill in Slack username and channel

View Prometheus alerts `http://<Host IP Address>:9090/alerts`
View Alert Manager `http://<Host IP Address>:9093`

### Test Alerts
A quick test for your alerts is to stop a service. Stop the node_exporter container and you should notice shortly the alert arrive in Slack. Also check the alerts in both the Alert Manager and Prometheus Alerts just to understand how they flow through the system.

High load test alert - `docker run --rm -it busybox sh -c "while true; do :; done"`

Let this run for a few minutes and you will notice the load alert appear. Then Ctrl+C to stop this container.

# Security Considerations
This project is intended to be a quick-start to get up and running with Docker and Prometheus. Security has not been implemented in this project. It is the users responsability to implement Firewall/IpTables and SSL.

Since this is a template to get started Prometheus and Alerting services are exposing their ports to allow for easy troubleshooting and understanding of how the stack works.

## Production Security:
Here are just a couple security considerations for this stack to help you get started.
* Remove the published ports from Prometheus and Alerting servicesi and only allow Grafana to be accessed
* Enable SSL for Grafana with a Proxy such as [jwilder/nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy/) or [Traefik](https://traefik.io/) with Let's Encrypt
* Add user authentication via a Reverse Proxy [jwilder/nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy/) or [Traefik](https://traefik.io/) for services cAdvisor, Prometheus, & Alerting as they don't support user authenticaiton
* Terminate all services/containers via HTTPS/SSL/TLS

# Troubleshooting
It appears some people have reported no data appearing in Grafana. If this is happening to you be sure to check the time range being queried within Grafana to ensure it is using Today's date with current time.



# Thanks
Spezial thanks to the [initiator]((https://github.com/vegasbrianc/) of this repo.

*Have an intersting Project which use this Repo? Submit yours to the list*
