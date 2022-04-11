---
title: "Process Exporter"
---



grafana-agent内置集成了[`process-exporter`](https://github.com/ncabatoff/process-exporter)，基于`/proc`的文件分析结果，来收集Linux系统进程相关的指标（注意，非Linux系统开启该exporter不起作用）。

如果grafana-agent运行在container中，那么在容器的启动命令中，要做以下调整，即将宿主机的/proc目录映射到容器中相应的位置。


```bash
docker run \
  -v "/proc:/proc:ro" \
  -v /tmp/agent:/etc/agent \
  -v /path/to/config.yaml:/etc/agent-config/agent.yaml \
  grafana/agent:v0.23.0 \
  --config.file=/etc/agent-config/agent.yaml
```

注意，将`/path/to/config.yaml`替换成您自己相应的配置文件。

如果grafana-agent运行在Kubernetes中，那么同样的需要在manifest文件中，做如下调整，即将宿主机的/proc目录映射到容器中相应的位置。


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: grafana-agent
spec:
  containers:
  - image: grafana/agent:v0.23.0
    name: agent
    args:
    - --config.file=/etc/agent-config/agent.yaml
    volumeMounts:
    - name: procfs
      mountPath: /proc
      readOnly: true
  volumes:
  - name: procfs
    hostPath:
      path: /proc
```

# 配置并启用process_exporter

如下的配置，将会开启process_exporter，并追踪系统中的所有进程。
```yaml
process_exporter:
  enabled: true
  process_names:
  - name: "{{.Comm}}"
    cmdline:
    - '.+'
```

# 采集的指标列表
```yaml
# Context switches
# 上下文切换数量
# Counter
namedprocess_namegroup_context_switches_total

# Cpu user/system usage in seconds
# CPU 时间（秒）
# Counter
namedprocess_namegroup_cpu_seconds_total

# Major page faults
# 主要页缺失次数
# Counter
namedprocess_namegroup_major_page_faults_total

# Minor page faults
# 次要页缺失次数
# Counter
namedprocess_namegroup_minor_page_faults_total

# number of bytes of memory in use
# 内存占用（byte）
# Gauge
namedprocess_namegroup_memory_bytes

# number of processes in this group
# 同名进程数量
# Gauge
namedprocess_namegroup_num_procs

# Number of processes in states Running, Sleeping, Waiting, Zombie, or Other
# 同名进程状态分布
# Gauge
namedprocess_namegroup_states

# Number of threads
# 线程数量
# Gauge
namedprocess_namegroup_num_threads

# start time in seconds since 1970/01/01 of oldest process in group
# 启动时间戳
# Gauge
namedprocess_namegroup_oldest_start_time_seconds

# number of open file descriptors for this group
# 打开文件描述符数量
# Gauge
namedprocess_namegroup_open_filedesc

# the worst (closest to 1) ratio between open fds and max fds among all procs in this group
# 打开文件数 / 允许打开文件数
# Gauge
namedprocess_namegroup_worst_fd_ratio

# number of bytes read by this group
# 读数据量（byte）
# Counter
namedprocess_namegroup_read_bytes_total

# number of bytes written by this group
# 写数据量（byte）
# Counter
namedprocess_namegroup_write_bytes_total

# Number of threads in this group waiting on each wchan
# 内核wchan等待线程数量
# Gauge
namedprocess_namegroup_threads_wchan
```

# process_exporter的详细配置项说明

```yaml
  # Enables the process_exporter integration, allowing the Agent to automatically
  # collect system metrics from the host UNIX system.
  [enabled: <boolean> | default = false]

  # Sets an explicit value for the instance label when the integration is
  # self-scraped. Overrides inferred values.
  #
  # The default value for this integration is inferred from the agent hostname
  # and HTTP listen port, delimited by a colon.
  [instance: <string>]

  # Automatically collect metrics from this integration. If disabled,
  # the process_exporter integration will be run but not scraped and thus not
  # remote-written. Metrics for the integration will be exposed at
  # /integrations/process_exporter/metrics and can be scraped by an external
  # process.
  [scrape_integration: <boolean> | default = <integrations_config.scrape_integrations>]

  # How often should the metrics be collected? Defaults to
  # prometheus.global.scrape_interval.
  [scrape_interval: <duration> | default = <global_config.scrape_interval>]

  # The timeout before considering the scrape a failure. Defaults to
  # prometheus.global.scrape_timeout.
  [scrape_timeout: <duration> | default = <global_config.scrape_timeout>]

  # Allows for relabeling labels on the target.
  relabel_configs:
    [- <relabel_config> ... ]

  # Relabel metrics coming from the integration, allowing to drop series
  # from the integration that you don't care about.
  metric_relabel_configs:
    [ - <relabel_config> ... ]

  # How frequent to truncate the WAL for this integration.
  [wal_truncate_frequency: <duration> | default = "60m"]

  # procfs mountpoint.
  [procfs_path: <string> | default = "/proc"]

  # If a proc is tracked, track with it any children that aren't a part of their
  # own group.
  [track_children: <boolean> | default = true]

  # Report on per-threadname metrics as well.
  [track_threads: <boolean> | default = true]

  # Gather metrics from smaps file, which contains proportional resident memory
  # size.
  [gather_smaps: <boolean> | default = true]

  # Recheck process names on each scrape.
  [recheck_on_scrape: <boolean> | default = false]

  # A collection of matching rules to use for deciding which processes to
  # monitor. Each config can match multiple processes to be tracked as a single
  # process "group."
  process_names:
    [- <process_matcher_config>]
```

> **process_matcher_config**

```yaml
# The name to use for identifying the process group name in the metric. By
# default, it uses the base path of the executable.
#
# The following template variables are available:
#
# - {{.Comm}}:      Basename of the original executable from /proc/<pid>/stat
# - {{.ExeBase}}:   Basename of the executable from argv[0]
# - {{.ExeFull}}:   Fully qualified path of the executable
# - {{.Username}}:  Username of the effective user
# - {{.Matches}}:   Map containing all regex capture groups resulting from
#                   matching a process with the cmdline rule group.
# - {{.PID}}:       PID of the process. Note that the PID is copied from the
#                   first executable found.
# - {{.StartTime}}: The start time of the process. This is useful when combined
#                   with PID as PIDS get reused over time.
[name: <string> | default = "{{.ExeBase}}"]

# A list of strings that match the base executable name for a process, truncated
# at 15 characters. It is derived from reading the second field of
# /proc/<pid>/stat minus the parens.
#
# If any of the strings match, the process will be tracked.
comm:
  [- <string>]

# A list of strings that match argv[0] for a process. If there are no slashes,
# only the basename of argv[0] needs to match. Otherwise the name must be an
# exact match. For example, "postgres" may match any postgres binary but
# "/usr/local/bin/postgres" can only match a postgres at that path exactly.
#
# If any of the strings match, the process will be tracked.
exe:
  [- <string>]

# A list of regular expressions applied to the argv of the process. Each
# regex here must match the corresponding argv for the process to be tracked.
# The first element that is matched is argv[1].
#
# Regex Captures are added to the .Matches map for use in the name.
cmdline:
  [- <string>]
```
