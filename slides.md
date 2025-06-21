---
# You can also start simply with 'default'
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
hideInToc: true
title: Observability Training
author: Mischa Taylor
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# open graph
# seoMeta:
#  ogImage: https://cover.sli.dev
themeConfig:
  paginationX: r
  paginationY: t
  paginationPagesDisabled: [1]
---

# Observability

##### Mischa Taylor | 📧 <taylor@linux.com>

<div @click="$slidev.nav.next" class="mt-12 py-1" hover:bg="white op-10">
  Press Space for next page <carbon:arrow-right />
</div>

<div class="abs-br m-6 text-xl">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="slidev-icon-btn">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" class="slidev-icon-btn">
    <carbon:logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
hideInToc: true
routeAlias: toc
---

# Table of Contents

<Toc columns="2"/>

---
layout: section
---

# Prometheus

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

```bash
cat >prometheus.yml <<EOF
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
EOF
```

```bash
docker container run -it --rm \
  --mount type=bind,source="$(pwd)/prometheus.yml",target=/prometheus/prometheus.yml,readonly \
  --entrypoint promtool \
  docker.io/boxcutter/prometheus check config prometheus.yml
```

```bash
docker container run -it --rm \
  -p 9090:9090 \
  --mount type=bind,source="$(pwd)/prometheus.yml",target=/etc/prometheus/prometheus.yml,readonly \
  docker.io/boxcutter/prometheus
```

---
hideInToc: true
---

Visit http://localhost:9090/

Metrics are served up via http://localhost:9090/metrics

Check health of targets with http://localhost:9090/targets

To use the expression browser http://localhost:9090/query

Try the following query. This expression tells you the CPU usage in cores of Prometheus itself, averaged over a 1-minute window:

`rate(process_cpu_seconds_total{job="prometheus"}[1m])`

This expression reports the resident memory usage of Prometheus itself in mebibytes (MiB):

`process_resident_memory_bytes{job="prometheus"} / 1024 / 1024`

---
hideInToc: true
---

This expression show the number of samples per second that Prometheus is storing, as averaged over a 1-minute window:

`rate(prometheus_tsdb_head_samples_appended_total{job="prometheus"}[1m])`

This is a synthetic up metric that Prometheus records for every target. In this case it is scoped to the prometheus job:

`up{job="prometheus"}`

---
layout: section
---

# Node Exporter

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

# Exercise 02 - Your first target node_exporter

```bash
cat >prometheus.yml <<EOF
global:
  scrape_interval: 5s
  evaluation_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']

  - job_name: 'dynamic'
    file_sd_configs:
    - files:
      - /etc/prometheus/targets.yml
      - /etc/prometheus/targets.json
      refresh_interval: 30s
EOF
```

---
hideInToc: true
---

```bash
cat >targets.yml <<EOF
- targets: [ 'node-exporter:9100' ]
  labels:
    job: 'node'
EOF
```

---
hideInToc: true
---

```bash
docker volume create prometheus-data

docker network create monitoring
```

```bash
docker container run -it --rm \
  -d \
  --name prometheus \
  -p 9090:9090 \
  --network monitoring \
  --mount type=bind,source="$(pwd)/prometheus.yml",target=/etc/prometheus/prometheus.yml,readonly \
  --mount type=bind,source="$(pwd)/targets.yml",target=/etc/prometheus/targets.yml,readonly \
  --mount type=volume,source=prometheus-data,target=/prometheus,volume-driver=local \
  docker.io/boxcutter/prometheus
```

```bash
docker container run -it --rm \
  --name node-exporter \
  -p 9100:9100 \
  --network monitoring \
  docker.io/boxcutter/node-exporter
```

---
hideInToc: true
---

```bash
docker container stop prometheus
```

---
layout: section
---

# Grafana

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

---
hideInToc: true
---

---
hideInToc: true
---

---
hideInToc: true
---


---
layout: section
---

# SNMP Exporter

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

```bash
% docker run -it --rm docker.io/boxcutter/snmp \
  snmpstatus -v3 -l authPriv \
    -u snmpreader \
    -a SHA -A superseekret \
    -x AES -X superseekret \
    10.137.56.1
[UDP: [10.137.56.1]:161->[0.0.0.0]:45090]=>[Peplink MAX BR2 Pro] Up: 1 day, 2:41:05.34
Interfaces: 9, Recv/Trans packets: 44205108/42666635 | IP: 0/0
2 interfaces are down!
```

---
hideInToc: true
---

```bash
% docker run -it --rm docker.io/boxcutter/snmp \
  snmpwalk -v3 -l authPriv \
    -u snmpreader \
    -a SHA -A superseekret \
    -x AES -X superseekret \
    10.137.56.1
iso.3.6.1.2.1.1.1.0 = STRING: "Peplink MAX BR2 Pro"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.23695
iso.3.6.1.2.1.1.3.0 = Timeticks: (28708) 0:04:47.08
iso.3.6.1.2.1.1.4.0 = STRING: "support@peplink.com"
iso.3.6.1.2.1.1.5.0 = STRING: "MAX-BR2-2013"
iso.3.6.1.2.1.1.6.0 = STRING: "Peplink"
iso.3.6.1.2.1.1.8.0 = Timeticks: (1) 0:00:00.01
...
```

---
hideInToc: true
---

```bash
git clone https://github.com/prometheus/snmp_exporter.git
cd snmp_exporter/generator
```

```bash
cp generator.yml generator.yml.orig
```

---
hideInToc: true
---

```bash
---
auths:
  public_v3:
    version: 3
    username: snmpreader
    password: superseekret
    auth_protocol: SHA
    priv_protocol: AES
    priv_password: superseekret

modules:
  # Default IF-MIB interfaces table with ifIndex.
  if_mib:
    walk: [sysUpTime, interfaces, ifXTable]
    lookups:
      - source_indexes: [ifIndex]
        lookup: "IF-MIB::ifAlias"
      - source_indexes: [ifIndex]
        # Disambiguate from PaloAlto PAN-COMMON-MIB::ifDescr.
        lookup: "IF-MIB::ifDescr"
      - source_indexes: [ifIndex]
        # Disambiguate from Netscaler NS-ROOT-MIB::ifName.
        lookup: "IF-MIB::ifName"
    overrides:
      ifAlias:
        ignore: true # Lookup metric
      ifDescr:
        ignore: true # Lookup metric
      ifName:
        ignore: true # Lookup metric
      ifType:
        type: EnumAsInfo
```

---
hideInToc: true
---

```bash
docker run -it --rm \
  --mount type=bind,source="$(pwd)",target=/code \
  --entrypoint /bin/bash \
  docker.io/boxcutter/snmp-generator
# cd /code
# generator -m /opt/snmp_exporter/mibs/mibs -m /opt/snmp_exporter/mibs/peplink/peplink generate
```

---
hideInToc: true
---

```bash
docker network create monitoring
```

```bash
docker run -it --rm \
  --name snmp-exporter \
  -p 9116:9116 \
  --network monitoring \
  --mount type=bind,source="$(pwd)/snmp.yml",target=/etc/snmp_exporter/snmp.yml \
  docker.io/boxcutter/snmp-exporter \
    --config.file=/etc/snmp_exporter/snmp.yml
```

```bash
Target: 10.137.56.1
Auth: public_v3
Module: if_mib
```

---
hideInToc: true
---

```bash
cat >prometheus.yml <<EOF
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'network'
    metrics_path: /snmp
    params:
      auth: [public_v3]
      module: [if_mib]
    static_configs:
      - targets: ['10.137.56.1']
        labels:
          job: 'peplink'
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: snmp-exporter:9116 # The SNMP exporter real host
EOF
```

---
hideInToc: true
---

```bash
docker container run -it --rm \
  --mount type=bind,source="$(pwd)/prometheus.yml",target=/prometheus/prometheus.yml,readonly \
  --entrypoint promtool \
  docker.io/boxcutter/prometheus check config prometheus.yml
```

---
hideInToc: true
---

```bash
docker container run -it --rm \
  --name prometheus \
  -p 9090:9090 \
  --network monitoring \
  --mount type=bind,source="$(pwd)/prometheus.yml",target=/etc/prometheus/prometheus.yml,readonly \
  docker.io/boxcutter/prometheus
```
