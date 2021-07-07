# Internet Speed Monitoring Stack

A simple and modern tool for monitoring internet speed history and uptime of specified URLs.

This is all implemented with a simple docker-compose stack with [Prometheus](http://prometheus.io/) + [Grafana](https://grafana.com/) with [blackbox-exporter](https://github.com/prometheus/blackbox_exporter) and [speedtest-exporter](https://github.com/MiguelNdeCarvalho/speedtest-exporter) to collect and graph home Internet reliability and throughput.

![dashboard-1](https://user-images.githubusercontent.com/20902250/124679414-db054500-deb3-11eb-8757-c578d59366d6.png)
![dashboard-2](https://user-images.githubusercontent.com/20902250/124681501-2f122880-deb8-11eb-86a3-8a990d035215.png)

## Pre-requisites

Make sure [Docker](https://docs.docker.com/engine/install/) and [Docker Compose](https://docs.docker.com/compose/install/) are installed on your Docker host machine.

## Configuration

- To change default grafana credentials, change the `GF_SECURITY_ADMIN_USER` and `GF_SECURITY_ADMIN_PASSWORD` variables in [`./grafana/auth.env`](./grafana/auth.env) file.
- To change what hosts you ping you change the `targets` section in [`./prometheus/blackbox.yaml`](./prometheus/blackbox.yaml) file.
- To change how often speedtest is ran, edit the `scrape_interval` under `speedtest` category in [`./prometheus/prometheus.yml`](./prometheus/prometheus.yml), default duration is 30 minutes, which might be too much if you have limit on downloads.

## Run the docker containers

After you've configured everything, you can simply use docker-compose to spin up all of the needed services:

```sh
docker-compose up -d
```

It will take a while until grafana loads, so be patient, after it does, it will be accessible via: `http://<Host IP Address>:3000` (default credentials are username: admin, password: admin).

If all works you should see 2 dashboards after clicking on Search button in the left navbar:

<img src=https://user-images.githubusercontent.com/20902250/124680714-7e575980-deb6-11eb-83ff-929151c0b59b.png height="120">

## Interesting urls

- <http://localhost:9798/metrics> speedtest exporter endpoint. Does take about 30 seconds to show its result as it runs an actual speedtest when requested.
- <http://localhost:9115> blackbox exporter endpoint. Lets you see what have failed/succeeded.
- <http://localhost:9090/targets> shows status of monitored targets as seen from prometheus - in this case which hosts being pinged and speedtest. note: speedtest will take a while before it shows as UP as it takes about 30s to respond.
- <http://localhost:9090/graph?g0.expr=probe_http_status_code&g0.tab=1> shows prometheus value for `probe_http_status_code` for each host. You can edit/play with additional values. Useful to check everything is okay in prometheus (in case Grafana is not showing the data you expect).

## Resetting stored data

All of the data from grafana and prometheus are stored using docker volumes, you will need to remove those in order to wipe the data:

1. Stop the application: `docker-compose down`
2. See all of the volumes `docker volume ls`
3. Remove unused ones `docker volume prune` (NOTE: this will remove all unused volumes, not just those from this project, if you have other volumes which you want to preserve use `docker volume rm [volume id 1] [volume id 2] ...` instead!)

## Editing the gauges

Everyone has a bit different internet speed requirements and different standards of what's considered as fast, for that reason, you can simply click on the name of any of the gauges, and click edit

<img src="https://user-images.githubusercontent.com/20902250/124659575-a59d2f00-de94-11eb-950e-b7dcb5a1d56a.png" width="400">

An edit menu will show where in the left panel you can configure the minimum and maximum values shown on the gauge:

<img src="https://user-images.githubusercontent.com/20902250/124659786-dc734500-de94-11eb-9e02-e3f73e8b8c33.png" width="400">

You can also edit the color thresholds depending on the speeds to your liking (i.e. you can set after which point should the color become red/orange/...). Note that with the download/upload gauges you will need to enter the speed in bits per second, not megabits!

<img src="https://user-images.githubusercontent.com/20902250/124659934-0fb5d400-de95-11eb-9cef-28c2c8033bf8.png" width="400">

After your editing is done and you're happy with your layout, make sure you save the layout and copy the JSON:

<img src="https://user-images.githubusercontent.com/20902250/124680973-10f7f880-deb7-11eb-87c1-c4007baad07d.png" width="300">

<img src="https://user-images.githubusercontent.com/20902250/124681039-3422a800-deb7-11eb-9da2-4a1947f556a4.png" width="600">

Simply click on Copy JSON to clipboard and paste it to [`./grafana/provisioning/dashboards/internet-connection.json`](./grafana/provisioning/dashboards/internet-connection.json). Alternatively, if you're editing the Blackbox probe dashboard, store them in [`probe.json`](./grafana/provisioning/dashboards/probe.json)

## Thanks and a disclaimer

- Thanks to [`@maxandersen`](https://github.com/maxandersen) for making the [original project](https://github.com/maxandersen/internet-monitoring) this fork is based on.
- Thanks to [`@geerlingguy`](https://github.com/geerlingguy) for perfecting the original project with a [custom fork](https://github.com/geerlingguy/internet-pi/tree/master/internet-monitoring).
- Thanks to [`@goddenrich`](https://github.com/goddenrich) for perfecting the original project with a [custom fork](https://github.com/goddenrich/local-network-monitoring).

This setup is not secured in any way, so please only use on non-public networks, or find a way to secure it on your own.
