# Internet Monitoring Docker Stack with Prometheus + Grafana

Stand-up a Docker [Prometheus](http://prometheus.io/) stack containing Prometheus, Grafana with [blackbox-exporter](https://github.com/prometheus/blackbox_exporter), and [speedtest-exporter](https://github.com/MiguelNdeCarvalho/speedtest-exporter) to collect and graph home Internet reliability and throughput.

## Pre-requisites

Make sure Docker and [Docker Compose](https://docs.docker.com/compose/install/) are installed on your Docker host machine.

## Configuration

To change what hosts you ping you change the `targets` section in [/prometheus/pinghosts.yaml](./prometheus/pinghosts.yaml) file.

For speedtest the only relevant configuration is how often you want the check to happen. It is at 30 minutes by default which might be too much if you have limit on downloads. This is changed by editing `scrape_interval` under `speedtest` in [/prometheus/prometheus.yml](./prometheus/prometheus.yml).

## Start the monitoring containers

Use `docker-compose` which handles starting all of the needed containers and specifies how they communicate.

```
docker-compose up -d
```

It will take a while until grafana loads, so be patient.

The Grafana Dashboard will now be accessible via: `http://<Host IP Address>:3000` for example <http://localhost:3000>

username - admin
password - admin (Password is stored in the `config.monitoring` env file)

The DataSource and Dashboard for Grafana are automatically provisioned.

If all works it should be available at <http://localhost:3000/d/o9mIe_Aik/internet-connection> - if no data shows up try change the timeduration to something smaller.

![image](https://user-images.githubusercontent.com/20902250/124660583-ecd7ef80-de95-11eb-89f4-cf4b87644f8b.png)

## Interesting urls

- <http://localhost:9090/targets> shows status of monitored targets as seen from prometheus - in this case which hosts being pinged and speedtest. note: speedtest will take a while before it shows as UP as it takes about 30s to respond.
- <http://localhost:9090/graph?g0.expr=probe_http_status_code&g0.tab=1> shows prometheus value for `probe_http_status_code` for each host. You can edit/play with additional values. Useful to check everything is okay in prometheus (in case Grafana is not showing the data you expect).
- <http://localhost:9115> blackbox exporter endpoint. Lets you see what have failed/succeeded.
- <http://localhost:9798/metrics> speedtest exporter endpoint. Does take about 30 seconds to show its result as it runs an actual speedtest when requested.

## Editing the gauges

Everyone has a bit different internet speed requirements and different standards of what's considered as fast, for that reason, you can simply click on the name of any of the gauges, and click edit 

<img src="https://user-images.githubusercontent.com/20902250/124659575-a59d2f00-de94-11eb-950e-b7dcb5a1d56a.png" width="400">

An edit menu will show where in the left pannel you can configure the minimum and maximum values shown on the gauge: 

<img src="https://user-images.githubusercontent.com/20902250/124659786-dc734500-de94-11eb-9e02-e3f73e8b8c33.png" width="400">

You can also edit the color thresholds depending on the speeds to your liking (i.e. you can set after which point should the color become red/orange/...). Note that with the download/upload gauges you will need to enter the speed in bytes per second, not megabites!

<img src="https://user-images.githubusercontent.com/20902250/124659934-0fb5d400-de95-11eb-9cef-28c2c8033bf8.png" width="400">

## Thanks and a disclaimer

- Thanks to @maxandersen for making the original project this fork is based on.
- Thanks to @vegasbrianc work on making a [super easy docker](https://github.com/vegasbrianc/github-monitoring) stack for running prometheus and grafana.
- Thanks to @geerlingguy for perfecting the original project.

This setup is not secured in any way, so please only use on non-public networks, or find a way to secure it on your own.
