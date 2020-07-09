---
template: blog-post
title: Integrating Azure monitor metrics to Prometheus time-series Database with
  Azure exporter
slug: azure-exporter
date: 2020-07-09 15:38
description: Integrating Azure monitor metrics to Prometheus time-series
  Database with Azure exporter
---
Usecase is to get the Azure resource metrics and to save it in local DB. In this case, as this is time-series data, we can expect large chunks of data per minute. For Timesereis storage the first option that came into mind is [Prometheus](http://prometheus.io/).

Below are the steps we have to do to get Azure resource metrics to our Prometheus database.

Installation of Prometheus.

Setting up Azure metrics exporter.

Configuring .yaml files for Prometheus/Azure exporter.

In a nutshell, we have to generate an endpoint where it lists all the resource metrics like in below format. Then we have to configure the endpoint in the Prometheus configuration yaml file.

    metricname metricsvalue timestamp

> **Note**: In LINUX based system we are doing this process.

**Installation of Prometheus:**

    wget https://github.com/prometheus/prometheus/releases/download/v2.13.0/prometheus-2.13.0.linux-amd64.tar.gz
    extract tar using tar

**Installation of Azure metrics exporter:**
Azure exporter requires GO Lang to be installed. So we need to install GO in the system.
Installation of GO.

    curl -O https://storage.googleapis.com/golang/go1.12.9.linux-amd64.tar.gz   
            sha256sum go1.12.9.linux-amd64.tar.gz
    		
    tar go1.12.9.linux-amd64.tar.gz
    
    Move to local directory: sudo mv go /usr/local
    
    Set below paths in the end of the profile 
    export GOPATH=$HOME/go
    
     export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin
    Reload path: source ~/.profile

**Installing Azure metric exporter plugin:**

We are using the exporter from [https://github.com/RobustPerception/azure_metrics_exporter](https://github.com/RobustPerception/azure_metrics_exporter)

`go get -u github.com/RobustPerception/azure_metrics_exporter`

In the bin folder of azure go/bin: create below file azure.yml.

    active_directory_authority_url: "https://login.microsoftonline.com/"
    resource_manager_url: "https://management.azure.com/"
    credentials:
        subscription_id: ""
        client_id: ""
        client_secret: ""
        tenant_id: ""
    targets:
    resource_groups:
      - resource_group: "group-name"
        resource_types:
          - "Microsoft.Compute/virtualMachines"
        metrics:
          - name: "CPU Credits consumed"
          - name: "Percentage CPU"
          - name: "Network In Total"
          - name: "Network Out Total"
          - name: "Disk Read Bytes"
          - name: "Disk Write Bytes"
          - name: "Disk Read Operations/Sec"
          - name: "Disk Write Operations/Sec"
          - name: "CPU Credits Remaining"
**Note**: We have to create an application in Azure portal before doing this. After creating the application, we will get a client secret and client id. We can configure YAML as to get a given resource group of resources or given resource type metrics to fetch.

Start the Exporter service after the above configuration, `go/bin# ./azure_metrics_exporter`

It will enable metrics endpoint at 9276 port. To check, navigate to `http://localhost:9276/metrics`

You will get a list of metrics with values and timestamp in the above URL response.

Now the above endpoint to be sourced to Prometheus DB. This can be done in Prometheus configuration file. Create a new yaml file in the Prometheus installation directory as shown below.

prometheus-azure-metric.yml

     global:
      scrape_interval:     1m 
    scrape_configs:
    
      - job_name: 'azure'
        scrape_interval: 1m
        static_configs:
          - targets: ['localhost:9276']
We have given scrape interval as 1m, as we are getting metrics in 1 minute scrape interval.

Start the Prometheus server with new configuration file created above.

`/home/user/prometheus-2.13.0.linux-amd64# ./prometheus --config.file=prometheus-azure-metric.yml --web.listen-address=:9011`

Now the proemtheous running at port number 9011. Navigate to `http://localhost:9011`  and check the metrics.

As we discussed earlier all Prometheus needed is one endpoint where all the metrics will be available. That endpoint is localhost:9276 which we created using Azure exporter.

By default, Prometheus retains data till 15 days., which means it can hold history data of the last 15 days only. To make that customized we have to provide an extra flag while starting Prometheus.

`--storage.tsdb.retention=365d` -- you can provide any number of days.

Further extensions, we can also integrate the Prometheus with Grafana. Grafana is a metric dashboard interface. We can add Prometheus as a source to Grafana and see the metrics in Grafana.