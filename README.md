Copied from https://github.com/vegasbrianc/prometheus/

# Contents

- Introduction
  - [Pre-requisites](#pre-requisites)
  - [Installation & Configuration](#installation--configuration)
    - [Add Datasources & Dashboards](#add-datasources-and-dashboards)
    - [Install Dashboards the Old Way](#install-dashboards-the-old-way)
  	- [Alerting](#alerting)
  	- [Test Alerts](#test-alerts)
    - [Add additional Datasources](#add-additional-datasources)
  - [Troubleshooting](#troubleshooting)
  	- [Mac Users](#mac-users)
  - [Interesting Projects that use this Repo](#interesting-projects-that-use-this-repo)

# Pre-requisites
Before we get started installing the Prometheus stack. Ensure you install the latest version of docker and [docker swarm](https://docs.docker.com/engine/swarm/swarm-tutorial/) on your Docker host machine. Docker Swarm is installed automatically when using Docker for Mac or Docker for Windows.

# Installation & Configuration
Clone the project locally to your Docker host.

If you would like to change which targets should be monitored or make configuration changes edit the [/prometheus/prometheus.yml](prometheus/prometheus.yml) file. The targets section is where you define what should be monitored by Prometheus. The names defined in this file are actually sourced from the service name in the docker-compose file. If you wish to change names of the services you can add the "container_name" parameter in the `docker-compose.yml` file.

Once configurations are done let's start it up. From the /prometheus project directory run the following command:

    $ make deploy


That's it the `docker stack deploy' command deploys the entire Grafana and Prometheus stack automagically to the Docker Swarm. By default cAdvisor and node-exporter are set to Global deployment which means they will propogate to every docker host attached to the Swarm.

The Grafana Dashboard is now accessible via: `http://<Host IP Address>:3000` for example http://192.168.10.1:3000

	username - admin
	password - foobar (Password is stored in the `/grafana/config.monitoring` env file)

In order to check the status of the newly created stack:

    $ docker stack ps prom

View running services:

    $ docker service ls

View logs for a specific service

    $ docker service logs prom_<service_name>

## Add Datasources and Dashboards
Grafana version 5.0.0 has introduced the concept of provisioning. This allows us to automate the process of adding Datasources & Dashboards. The `/grafana/provisioning/` directory contains the `datasources` and `dashboards` directories. These directories contain YAML files which allow us to specify which datasource or dashboards should be installed. 

If you would like to automate the installation of additional dashboards just copy the Dashboard `JSON` file to `/grafana/provisioning/dashboards` and it will be provisioned next time you stop and start Grafana.

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

### Add Additional Datasources
Now we need to create the Prometheus Datasource in order to connect Grafana to Prometheus 
* Click the `Grafana` Menu at the top left corner (looks like a fireball)
* Click `Data Sources`
* Click the green button `Add Data Source`.

**Ensure the Datasource name `Prometheus`is using uppercase `P`**

<img src="https://raw.githubusercontent.com/vegasbrianc/prometheus/master/images/Add_Data_Source.png" width="400" heighth="400">

# Security Considerations
This project is intended to be a quick-start to get up and running with Docker and Prometheus. Security has not been implemented in this project. It is the users responsability to implement Firewall/IpTables and SSL.

Since this is a template to get started Prometheus and Alerting services are exposing their ports to allow for easy troubleshooting and understanding of how the stack works.

# Troubleshooting

It appears some people have reported no data appearing in Grafana. If this is happening to you be sure to check the time range being queried within Grafana to ensure it is using Today's date with current time.

## Mac Users

1. The node-exporter does not run the same as Mac and Linux. Node-Exporter is not designed to run on Mac and in fact cannot collect metrics from the Mac OS due to the differences between Mac and Linux OS's. I recommend you comment out the node-exporter section in the `docker-compose.yml` file and instead just use the cAdvisor.

2. If you find after you deploy your project that the prometheus and alertmanager services are in pending status due to "no suitable node" this is due to file system permissions. Be sure to Open Docker for Mac Preferences -> File Sharing Menu and add the following:

![Docker for Mac File Sharing Settings](https://github.com/vegasbrianc/prometheus/raw/master/images/mac-filesystem.png)

# Interesting Projects that use this Repo
Several projects utilize this Prometheus stack. Here's the list of projects:

* [Docker Pulls](https://github.com/vegasbrianc/docker-pulls) - Visualize Docker-Hub pull statistics with Prometheus
* [GitHub Monitoring](https://github.com/vegasbrianc/github-monitoring) - Monitor your GitHub projects with Prometheus
* [Traefik Reverse Proxy/Load Balancer Monitoring](https://github.com/vegasbrianc/docker-traefik-prometheus) - Monitor the popular Reverse Proxy/Load Balancer Traefik with Prometheus
* [internet monitoring](https://github.com/maxandersen/internet-monitoring) - Monitor your local network, internet connection and speed with Prometheus.
* [Dockerize Your Dev](https://github.com/RiFi2k/dockerize-your-dev) - Docker compose a VM to get LetsEncrypt / NGINX proxy auto provisioning, ELK logging, Prometheus / Grafana monitoring, Portainer GUI, and more...

*Have an intersting Project which use this Repo? Submit yours to the list*
