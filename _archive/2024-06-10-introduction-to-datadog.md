---
title: "Datadog – The Heart of Modern Observability"
date: 2024-06-10 00:00:00 +0100
tags: [Datadog, Infrastructure, Monitoring]
---

## What is Datadog?
Datadog is a powerful Application Performance Monitoring (APM) platform that keeps an eye on your applications’ availability, reliability, and scalability. Think of Datadog as your all-seeing guardian across cloud, on-prem, and hybrid environments — a platform that not only tells you that something went wrong, but helps you discover where and why it happened. By bringing together metrics, logs, traces, and real-time visualizations, Datadog helps teams understand their systems end-to-end, reduce downtime, and improve user experience.

## The 4 Golden Signals of SRE
Site Reliability Engineers (SREs) rely on four key indicators to measure the health of any system. Datadog automatically tracks each of these signals, giving teams real-time visibility into how their applications perform under pressure.

- Latency – the time it takes to service a request. High latency often points to performance bottlenecks or overloaded resources.
- Traffic – the volume of demand placed on your system, such as requests per second or active sessions.
- Saturation - how much of your system’s capacity is currently in use. It reveals how close you are to hitting critical limits.
- Errors - the rate at which requests fail. A rising error rate is often the first sign of instability.

By monitoring these four signals together, Datadog helps you detect anomalies faster, prioritize what matters, and maintain reliability even as your system scales.

## Sending Data to Datadog
Datadog offers hundreds of built-in integrations, making it simple to collect and centralize data from virtually any part of your technology stack. Metrics, logs, traces, and events can be sent to Datadog through several methods:

- Datadog Agent - a lightweight service installed on your hosts that collects metrics and events locally before forwarding them securely to Datadog.
- Datadog API – for developers who prefer direct, programmatic data submission and custom instrumentation.
- Integrations - pre-built connectors for popular services, frameworks, and platforms such as AWS, Kubernetes, Docker, Redis, and PostgreSQL.

Whichever method you choose, Datadog unifies all your telemetry data — metrics, logs, traces, and events — in one place, enabling consistent observability across every environment.

## Datadog Basic Terms
Before diving deeper into dashboards and monitors, it’s helpful to understand a few key concepts that form the foundation of Datadog.

- **Host** - Datadog monitors. A host could be a server, container, or virtual instance running in the cloud or on-premises.

- **Datadog Agent** - lightweight software installed on your hosts. It collects metrics and events locally, then sends them securely to Datadog. The Agent acts as a bridge between your infrastructure and the Datadog platform.

- **Tags** - metadata that adds context to your data. Tags make it easy to filter, group, and compare metrics.

- **Retention period** - the amount of time your logs and metrics remain searchable within Datadog. Longer retention means a deeper window for troubleshooting and trend analysis.

Understanding these terms will make it easier to navigate Datadog’s dashboards, integrations, and alerts later on.

## Datadog Agent
You can install the Agent directly from the Datadog interface under **Integrations → Agent**.

After installation, it begins collecting data immediately — no complex configuration required. The data it sends can then be visualized, filtered, aggregated, or used to trigger alerts.

The Agent consists of two primary components:

- **Collector** – gathers system metrics (CPU, memory, disk, network, etc.) at regular intervals, typically every 15 seconds.
- **Forwarder** – transmits the collected data to Datadog over HTTPS.

The Agent collects data in fixed intervals, aggregates it into summary points, and sends these aggregates to Datadog — each becoming a single data point on a graph.

The main config file, `datadog.yaml`, defines global settings such as API keys and collection options. Integration-specific configurations are stored in the `conf.d` directory. After making configuration changes, you can restart the Agent using the Datadog Agent Manager on your host to apply updates.

## Host Map
The Host Map provides a real-time visual overview of all the hosts in your environment, helping you instantly understand system health and resource utilization at a glance. You can access it from **Infrastructure → Hosts → Host Map**.

Each tile in the map represents a single host, and the tile’s color reflects CPU utilization by default. Clicking on a tile reveals detailed system metrics — such as operating system, network activity, memory usage, and filesystem performance.

The Apps section within the Host Map displays blue tiles representing running integrations (for example: `agent`, `ntp` or `system`). As you enable more integrations, additional tiles automatically appear, allowing you to explore related metrics with just one click.

## Integrations
Datadog provides hundreds of built-in integrations, allowing you to connect seamlessly with the technologies your infrastructure relies on — from cloud providers and databases to web servers, containers, and more.

You can explore and manage integrations by navigating to **Integrations → Integrations** within the Datadog interface. Integrations can be filtered by status — such as installed, available, or broken.

Each integration page includes detailed setup instructions, configuration options, and a list of available metrics. Once an integration is installed, Datadog automatically begins collecting relevant data and often adds pre-configured dashboards to help you visualize that data instantly.

## Metrics
Metrics are the core building blocks of observability. They quantify what’s happening in your systems and applications over time. 

Each metric submitted to Datadog must have a defined type, which determines how data collected during a given interval is aggregated and displayed. Datadog supports four main metric types:

- Gauge - captures a snapshot of a value at a specific moment in time, such as CPU usage, available memory, or disk space.
- Rate - measures how often something happens per second during a specific interval, e.g. requests per second.
- Count - represents the total number of occurrences within a defined time interval, such as the number of completed jobs or transactions.
- Distribution - provides statistical insight into a set of values collected across distributed systems during an interval, such as latency percentiles.

Metrics can be submitted to Datadog through multiple methods, including:
- the Datadog Agent
- DogStatsD (bundled with the Agent)
- the Datadog HTTP API

## Dashboards
Dashboards in Datadog provide real-time visual insights into system health, performance, and trends. All dashboard data can be filtered by time or by tags for precise analysis.

Datadog offers two main types of dashboards:

- Timeboards – ideal for real-time monitoring and analysis. They display metrics over time and are commonly used for troubleshooting performance issues or tracking key indicators.
- Screenboards – flexible, visually rich dashboards designed for sharing summaries, overviews, or business-level metrics. They combine graphs, text, and images to tell the story behind your data.

After installing the Datadog Agent, six system dashboards are automatically created to give you instant visibility into your infrastructure. Additional dashboards appear as new integrations are enabled, offering out-of-the-box views for databases, containers, cloud services, and more.

## Monitors (Alerts) and Notifications
Monitors in Datadog allow you to detect and respond to issues in real time — before they affect users or service reliability. You can create a new monitor by navigating to **Monitors → New Monitor** within the Datadog interface.

Datadog supports multiple monitor types, each designed for different use cases — from metric thresholds and log patterns to integration-specific alerts. Hovering over each monitor option displays a short description, helping you choose the most relevant one for your scenario.

Once a monitor is configured, Datadog continuously evaluates your data against defined conditions. When thresholds are breached or anomalies are detected, the system triggers an alert and sends notifications to your chosen channels, e.g. email or Slack.

## Summary
Datadog brings clarity and control to modern infrastructure monitoring. By unifying metrics, logs, traces, and events from every part of your stack, it delivers complete visibility into how your systems perform and why issues occur.

Whether you’re managing a single application or a complex distributed ecosystem, Datadog serves as the heart of observability — helping you keep systems stable, users happy, and innovation moving forward.
