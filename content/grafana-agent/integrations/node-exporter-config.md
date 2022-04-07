---
title: "Node Exporter"
---



grafana-agent 内置了 [`node_exporter`](https://github.com/prometheus/node_exporter), 可以通过在配置文件中 `integrations` 部分定义 `node_exporter_config` 来开启该功能。 

# 配置并启用node_exporter
下面是开启了node_exporter的配置文件示例，生成的配置文件保存为 ./grafana-agent-cfg.yaml:
```yaml
cat <<EOF > ./grafana-agent-cfg.yaml
# grafana-agent 本身的配置
server:
  log_level: info
  http_listen_port: 12345

# grafana-agent 抓取 metrics 的相关配置（类似于prometheus的scrape_configs）
metrics:
  global:
    scrape_interval: 15s
    scrape_timeout: 10s
    remote_write:
      - url: https://n9e-server:19000/prometheus/v1/write
        basic_auth:
          username: <string>
          password: <string>

integrations:
  node_exporter:
    enabled: true
EOF
```
注意： remote_write 可以配置在 global 部分，也可以针对每个 integration 单独配置不同的remote_write 地址。 

重启grafana-agent后，通过以下两个命令，验证 node_exporter 工作是否符合预期。

```curl http://localhost:12345/integrations/node_exporter/metrics``` ，预期输出如下内容：

```plain
node_boot_time_seconds 1.643256088e+09
node_context_switches_total 1.5136425575e+10
node_cooling_device_cur_state{name="0",type="Processor"} 0
node_cooling_device_cur_state{name="1",type="Processor"} 0
node_cooling_device_cur_state{name="2",type="Processor"} 0
node_cooling_device_cur_state{name="3",type="Processor"} 0
node_cooling_device_max_state{name="0",type="Processor"} 0
node_cooling_device_max_state{name="1",type="Processor"} 0
node_cooling_device_max_state{name="2",type="Processor"} 0
node_cooling_device_max_state{name="3",type="Processor"} 0
node_cpu_seconds_total{cpu="0",mode="idle"} 1.66906519e+06
node_cpu_seconds_total{cpu="0",mode="iowait"} 5031.48
node_cpu_seconds_total{cpu="0",mode="irq"} 0
node_cpu_seconds_total{cpu="0",mode="nice"} 82.84
node_cpu_seconds_total{cpu="0",mode="softirq"} 2332.39
```

`curl http://localhost:12345/agent/api/v1/targets  | jq`，预期输出如下内容：

```json
{
  "status": "success",
  "data": [
    {
      "instance": "b81030837ec7f1d162489cb4009325c9",
      "target_group": "integrations/node_exporter",
      "endpoint": "http://127.0.0.1:12345/integrations/node_exporter/metrics",
      "state": "up",
      "labels": {
        "agent_hostname": "tt-fc-dev01.nj",
        "instance": "tt-fc-dev01.nj:12345",
        "job": "integrations/node_exporter"
      },
      "discovered_labels": {
        "__address__": "127.0.0.1:12345",
        "__metrics_path__": "/integrations/node_exporter/metrics",
        "__scheme__": "http",
        "__scrape_interval__": "15s",
        "__scrape_timeout__": "10s",
        "agent_hostname": "tt-fc-dev01.nj",
        "job": "integrations/node_exporter"
      },
      "last_scrape": "2022-02-16T18:53:08.79288957+08:00",
      "scrape_duration_ms": 20,
      "scrape_error": ""
    },
    {
      "instance": "b81030837ec7f1d162489cb4009325c9",
      "target_group": "local_scrape",
      "endpoint": "http://127.0.0.1:12345/metrics",
      "state": "up",
      "labels": {
        "cluster": "txnjdev01",
        "instance": "127.0.0.1:12345",
        "job": "local_scrape"
      },
      "discovered_labels": {
        "__address__": "127.0.0.1:12345",
        "__metrics_path__": "/metrics",
        "__scheme__": "http",
        "__scrape_interval__": "15s",
        "__scrape_timeout__": "10s",
        "cluster": "txnjdev01",
        "job": "local_scrape"
      },
      "last_scrape": "2022-02-16T18:53:22.336820442+08:00",
      "scrape_duration_ms": 4,
      "scrape_error": ""
    }
  ]
}
````

可以看到，上面的返回结果的 targets 列表中，已经新增了一个instance，其 job 为 `integrations/node_exporter`，这说明 node_exporter 已经在正常工作了。


注意：如果 grafana-agent 是运行在容器中时，那么要做以下修改调整：

1. 确保在运行容器时，将宿主机的相关目录映射到容器中，如下所示，即 `-v "/:/host/root"`、 `-v "/sys:/host/sys"`、`-v "/proc:/host/proc"`.
```
docker run \
  --net="host" \
  --pid="host" \
  --cap-add=SYS_TIME \
  -d \
  -v "/:/host/root:ro" \
  -v "/sys:/host/sys:ro" \
  -v "/proc:/host/proc:ro" \
  -v /tmp/grafana-agent:/etc/agent/data \
  -v /tmp/grafana-agent-config.yaml:/etc/agent/agent.yaml \
  grafana/agent \
  --config.file=/etc/agent/agent.yaml \
  --metrics.wal-directory=/etc/agent/data
```

2. 其中，配置文件 `/tmp/grafana-agent-config.yaml` 中 node_exporter 部分要指定 rootfs/sysfs/procfs 在容器中的路径，您可以运行以下命令生成该测试配置文件（当然，您需要把 remote_write 替换为适合您的地址）。

```yaml
cat <<EOF > /tmp/grafana-agent-config.yaml
server:
  log_level: info
  http_listen_port: 12345

metrics:
  global:
    scrape_interval: 15s
    scrape_timeout: 10s
    remote_write:
      - url: https://n9e-server:19000/prometheus/v1/write
        basic_auth:
          username: <string>
          password: <string>

integrations:
  node_exporter:
    enabled: true
    rootfs_path: /host/root
    sysfs_path: /host/sys
    procfs_path: /host/proc
EOF
```


注意：如果 grafana-agent 是运行在 K8s 环境中，那么调整步骤如下：

1. 推荐将 grafana-agent 的配置文件存储在configmap中, manifest文件如下：
```yaml
cat <<EOF |
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-agent
  namespace: ${NAMESPACE}
data:
  agent.yaml: |
    server:
      log_level: info
      http_listen_port: 12345

    metrics:
      global:
        scrape_interval: 15s
        remote_write:
          - url: 'https://n9e-server:19000/prometheus/v1/write'
            basic_auth:
              username: ${FC_USERNAME}
              password: ${FC_PASSWORD}

    integrations:
      agent:
        enabled: true
      node_exporter:
        enabled: true 
EOF

envsubst | kubectl apply -f -
kubectl describe configmap grafana-agent
```

2. 生成grafana-agent的pod manifest文件如下，并创建相应Pod实例：
```yaml
cat << EOF |
apiVersion: v1
kind: Pod
metadata:
  name: grafana-agent
  namespace: ${NAMESPACE}
spec:
  containers:
  - image: grafana/agent
    name: grafana-agent
    args:
    - --config.file=/fcetc/agent.yaml
    - --metrics.wal-directory=/etc/agent/data
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
      privileged: true
      runAsUser: 0
    volumeMounts:
    - name: rootfs
      mountPath: /host/root
      readOnly: true
    - name: sysfs
      mountPath: /host/sys
      readOnly: true
    - name: procfs
      mountPath: /host/proc
      readOnly: true
    - name: fccfg
      mountPath: /fcetc
  hostPID: true
  hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet
  volumes:
  - name: rootfs
    hostPath:
      path: /
  - name: sysfs
    hostPath:
      path: /sys
  - name: procfs
    hostPath:
      path: /proc
  - name: fccfg
    configMap:
      name: grafana-agent
EOF

envsubst |kubectl apply -f -
kubectl logs grafana-agent #查看 grafana-agent 的日志
```

# node_exporter采集的关键指标解析
```yaml
# SYSTEM
# CPU context switch 次数
node_context_switches_total: context_switches

# Interrupts 次数
node_intr_total: Interrupts

# 运行的进程数
node_procs_running: Processes in runnable state

# 熵池大小
node_entropy_available_bits: Entropy available to random number generators

node_time_seconds: System time in seconds since epoch (1970)
node_boot_time_seconds: Node boot time, in unixtime

# CPU
node_cpu_seconds_total: Seconds the CPUs spent in each mode
node_load1: cpu load 1m
node_load5: cpu load 5m
node_load15: cpu load 15m

# MEM
# 内核态
# 用户追踪已从交换区获取但尚未修改的页面的内存
node_memory_SwapCached_bytes: Memory that keeps track of pages that have been fetched from swap but not yet been modified

# 内核用于缓存数据结构供自己使用的内存
node_memory_Slab_bytes: Memory used by the kernel to cache data structures for its own use

# slab中可回收的部分
node_memory_SReclaimable_bytes: SReclaimable - Part of Slab, that might be reclaimed, such as caches

# slab中不可回收的部分
node_memory_SUnreclaim_bytes: Part of Slab, that cannot be reclaimed on memory pressure

# Vmalloc内存区的大小
node_memory_VmallocTotal_bytes: Total size of vmalloc memory area

# vmalloc已分配的内存，虚拟地址空间上的连续的内存
node_memory_VmallocUsed_bytes: Amount of vmalloc area which is used

# vmalloc区可用的连续最大快的大小，通过此指标可以知道vmalloc可分配连续内存的最大值
node_memory_VmallocChunk_bytes: Largest contigious block of vmalloc area which is free

# 内存的硬件故障删除掉的内存页的总大小
node_memory_HardwareCorrupted_bytes: Amount of RAM that the kernel identified as corrupted / not working

# 用于在虚拟和物理内存地址之间映射的内存
node_memory_PageTables_bytes: Memory used to map between virtual and physical memory addresses (gauge)

# 内核栈内存，常驻内存，不可回收
node_memory_KernelStack_bytes: Kernel memory stack. This is not reclaimable

# 用来访问高端内存，复制高端内存的临时buffer，称为“bounce buffering”，会降低I/O 性能
node_memory_Bounce_bytes: Memory used for block device bounce buffers

#用户态
# 单个巨页大小
node_memory_Hugepagesize_bytes: Huge Page size

# 系统分配的常驻巨页数
node_memory_HugePages_Total: Total size of the pool of huge pages

# 系统空闲的巨页数
node_memory_HugePages_Free: Huge pages in the pool that are not yet allocated

# 进程已申请但未使用的巨页数
node_memory_HugePages_Rsvd: Huge pages for which a commitment to allocate from the pool has been made, but no allocation

# 超过系统设定的常驻HugePages数量的个数
node_memory_HugePages_Surp: Huge pages in the pool above the value in /proc/sys/vm/nr_hugepages

# 透明巨页 Transparent HugePages (THP)
node_memory_AnonHugePages_bytes: Memory in anonymous huge pages

# inactivelist中的File-backed内存
node_memory_Inactive_file_bytes: File-backed memory on inactive LRU list

# inactivelist中的Anonymous内存
node_memory_Inactive_anon_bytes: Anonymous and swap cache on inactive LRU list, including tmpfs (shmem)

# activelist中的File-backed内存
node_memory_Active_file_bytes: File-backed memory on active LRU list

# activelist中的Anonymous内存
node_memory_Active_anon_bytes: Anonymous and swap cache on active least-recently-used (LRU) list, including tmpfs

# 禁止换出的页，对应 Unevictable 链表
node_memory_Unevictable_bytes: Amount of unevictable memory that can't be swapped out for a variety of reasons

# 共享内存
node_memory_Shmem_bytes: Used shared memory (shared between several processes, thus including RAM disks)

# 匿名页内存大小
node_memory_AnonPages_bytes: Memory in user pages not backed by files

# 被关联的内存页大小
node_memory_Mapped_bytes: Used memory in mapped pages files which have been mmaped, such as libraries

# file-backed内存页缓存大小
node_memory_Cached_bytes: Parked file data (file content) cache

# 系统中有多少匿名页曾经被swap-out、现在又被swap-in并且swap-in之后页面中的内容一直没发生变化
node_memory_SwapCached_bytes: Memory that keeps track of pages that have been fetched from swap but not yet been modified

# 被mlock()系统调用锁定的内存大小
node_memory_Mlocked_bytes: Size of pages locked to memory using the mlock() system call

# 块设备(block device)所占用的缓存页
node_memory_Buffers_bytes: Block device (e.g. harddisk) cache

node_memory_SwapTotal_bytes: Memory information field SwapTotal_bytes
node_memory_SwapFree_bytes: Memory information field SwapFree_bytes

# DISK

node_filesystem_files_free: Filesystem space available to non-root users in byte
node_filesystem_free_bytes: Filesystem free space in bytes
node_filesystem_size_bytes: Filesystem size in bytes

node_filesystem_files_free: Filesystem total free file nodes
node_filesystem_files: Filesystem total free file nodes

node_filefd_maximum: Max open files
node_filefd_allocated: Open files

node_filesystem_readonly: Filesystem read-only status
node_filesystem_device_error: Whether an error occurred while getting statistics for the given device

node_disk_reads_completed_total: The total number of reads completed successfully
node_disk_writes_completed_total: The total number of writes completed successfully
node_disk_reads_merged_total: The number of reads merged
node_disk_writes_merged_total: The number of writes merged

node_disk_read_bytes_total: The total number of bytes read successfully
node_disk_written_bytes_total: The total number of bytes written successfully

node_disk_io_time_seconds_total: Total seconds spent doing I/Os

node_disk_read_time_seconds_total: The total number of seconds spent by all reads
node_disk_write_time_seconds_total: The total number of seconds spent by all writes

node_disk_io_time_weighted_seconds_total: The weighted of seconds spent doing I/Os

# NET
node_network_receive_bytes_total: Network device statistic receive_bytes (counter)
node_network_transmit_bytes_total: Network device statistic transmit_bytes (counter)

node_network_receive_packets_total: Network device statistic receive_bytes
node_network_transmit_packets_total: Network device statistic transmit_bytes

node_network_receive_errs_total: Network device statistic receive_errs
node_network_transmit_errs_total: Network device statistic transmit_errs

node_network_receive_drop_total: Network device statistic receive_drop
node_network_transmit_drop_total: Network device statistic transmit_drop

node_nf_conntrack_entries: Number of currently allocated flow entries for connection tracking

node_sockstat_TCP_alloc: Number of TCP sockets in state alloc
node_sockstat_TCP_inuse: Number of TCP sockets in state inuse
node_sockstat_TCP_orphan: Number of TCP sockets in state orphan
node_sockstat_TCP_tw: Number of TCP sockets in state tw
node_netstat_Tcp_CurrEstab: Statistic TcpCurrEstab

node_sockstat_sockets_used: Number of IPv4 sockets in use

```

# node_expoter integration 完整的配置项说明
```yaml
  # Enables the node_exporter integration, allowing the Agent to automatically
  # collect system metrics from the host UNIX system.
  [enabled: <boolean> | default = false]

  # Sets an explicit value for the instance label when the integration is
  # self-scraped. Overrides inferred values.
  #
  # The default value for this integration is inferred from the agent hostname
  # and HTTP listen port, delimited by a colon.
  [instance: <string>]

  # Automatically collect metrics from this integration. If disabled,
  # the node_exporter integration will be run but not scraped and thus not remote-written. Metrics for the
  # integration will be exposed at /integrations/node_exporter/metrics and can
  # be scraped by an external process.
  [scrape_integration: <boolean> | default = <integrations_config.scrape_integrations>]

  # How often should the metrics be collected? Defaults to
  # prometheus.global.scrape_interval.
  [scrape_interval: <duration> | default = <global_config.scrape_interval>]

  # The timtout before considering the scrape a failure. Defaults to
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

  # Monitor the exporter itself and include those metrics in the results.
  [include_exporter_metrics: <boolean> | default = false]

  # Optionally defines the the list of enabled-by-default collectors.
  # Anything not provided in the list below will be disabled by default,
  # but requires at least one element to be treated as defined.
  #
  # This is useful if you have a very explicit set of collectors you wish
  # to run.
  set_collectors:
    - [<string>]

  # Additional collectors to enable on top of the default set of enabled
  # collectors or on top of the list provided by set_collectors.
  #
  # This is useful if you have a few collectors you wish to run that are
  # not enabled by default, but do not want to explicitly provide an entire
  # list through set_collectors.
  enable_collectors:
    - [<string>]

  # Additional collectors to disable on top of the default set of disabled
  # collectors. Takes precedence over enable_collectors.
  #
  # This is useful if you have a few collectors you do not want to run that
  # are enabled by default, but do not want to explicitly provide an entire
  # list through set_collectors.
  disable_collectors:
    - [<string>]

  # procfs mountpoint.
  [procfs_path: <string> | default = "/proc"]

  # sysfs mountpoint.
  [sysfs_path: <string> | default = "/sys"]

  # rootfs mountpoint. If running in docker, the root filesystem of the host
  # machine should be mounted and this value should be changed to the mount
  # directory.
  [rootfs_path: <string> | default = "/"]

  # Expose expensive bcache priority stats.
  [enable_bcache_priority_stats: <boolean>]

  # Regexp of `bugs` field in cpu info to filter.
  [cpu_bugs_include: <string>]

  # Enable the node_cpu_guest_seconds_total metric.
  [enable_cpu_guest_seconds_metric: <boolean> | default = true]

  # Enable the cpu_info metric for the cpu collector.
  [enable_cpu_info_metric: <boolean> | default = true]

  # Regexp of `flags` field in cpu info to filter.
  [cpu_flags_include: <string>]

  # Regexmp of devices to ignore for diskstats.
  [diskstats_ignored_devices: <string> | default = "^(ram|loop|fd|(h|s|v|xv)d[a-z]|nvme\\d+n\\d+p)\\d+$"]

  # Regexp of ethtool devices to exclude (mutually exclusive with ethtool_device_include)
  [ethtool_device_exclude: <string>]

  # Regexp of ethtool devices to include (mutually exclusive with ethtool_device_exclude)
  [ethtool_device_include: <string>]

  # Regexp of ethtool stats to include.
  [ethtool_metrics_include: <string> | default = ".*"]

  # Regexp of mount points to ignore for filesystem collector.
  [filesystem_mount_points_exclude: <string> | default = "^/(dev|proc|sys|var/lib/docker/.+)($|/)"]

  # Regexp of filesystem types to ignore for filesystem collector.
  [filesystem_fs_types_exclude: <string> | default = "^(autofs|binfmt_misc|bpf|cgroup2?|configfs|debugfs|devpts|devtmpfs|fusectl|hugetlbfs|iso9660|mqueue|nsfs|overlay|proc|procfs|pstore|rpc_pipefs|securityfs|selinuxfs|squashfs|sysfs|tracefs)$"]

  # How long to wait for a mount to respond before marking it as stale.
  [filesystem_mount_timeout: <duration> | default = "5s"]

  # Array of IPVS backend stats labels.
  #
  # The default is [local_address, local_port, remote_address, remote_port, proto, local_mark].
  ipvs_backend_labels:
    [- <string>]

  # NTP server to use for ntp collector
  [ntp_server: <string> | default = "127.0.0.1"]

  # NTP protocol version
  [ntp_protocol_version: <int> | default = 4]

  # Certify that the server address is not a public ntp server.
  [ntp_server_is_local: <boolean> | default = false]

  # IP TTL to use wile sending NTP query.
  [ntp_ip_ttl: <int> | default = 1]

  # Max accumulated distance to the root.
  [ntp_max_distance: <duration> | default = "3466080us"]

  # Offset between local clock and local ntpd time to tolerate.
  [ntp_local_offset_tolerance: <duration> | default = "1ms"]

  # Regexp of net devices to ignore for netclass collector.
  [netclass_ignored_devices: <string> | default = "^$"]

  # Ignore net devices with invalid speed values. This will default to true in
  # node_exporter 2.0.
  [netclass_ignore_invalid_speed_device: <boolean> | default = false]

  # Enable collecting address-info for every device.
  [netdev_address_info: <boolean>]

  # Regexp of net devices to exclude (mutually exclusive with include)
  [netdev_device_exclude: <string> | default = ""]

  # Regexp of net devices to include (mutually exclusive with exclude)
  [netdev_device_include: <string> | default = ""]

  # Regexp of fields to return for netstat collector.
  [netstat_fields: <string> | default = "^(.*_(InErrors|InErrs)|Ip_Forwarding|Ip(6|Ext)_(InOctets|OutOctets)|Icmp6?_(InMsgs|OutMsgs)|TcpExt_(Listen.*|Syncookies.*|TCPSynRetrans|TCPTimeouts)|Tcp_(ActiveOpens|InSegs|OutSegs|OutRsts|PassiveOpens|RetransSegs|CurrEstab)|Udp6?_(InDatagrams|OutDatagrams|NoPorts|RcvbufErrors|SndbufErrors))$"]

  # List of CPUs from which perf metrics should be collected.
  [perf_cpus: <string> | default = ""]

  # Array of perf tracepoints that should be collected.
  perf_tracepoint:
    [- <string>]

  # Regexp of power supplies to ignore for the powersupplyclass collector.
  [powersupply_ignored_supplies: <string> | default = "^$"]

  # Path to runit service directory.
  [runit_service_dir: <string> | default = "/etc/service"]

  # XML RPC endpoint for the supervisord collector.
  #
  # Setting SUPERVISORD_URL in the environment will override the default value.
  # An explicit value in the YAML config takes precedence over the environment
  # variable.
  [supervisord_url: <string> | default = "http://localhost:9001/RPC2"]

  # Regexp of systemd units to include. Units must both match include and not
  # match exclude to be collected.
  [systemd_unit_include: <string> | default = ".+"]

  # Regexp of systemd units to exclude. Units must both match include and not
  # match exclude to be collected.
  [systemd_unit_exclude: <string> | default = ".+\\.(automount|device|mount|scope|slice)"]

  # Enables service unit tasks metrics unit_tasks_current and unit_tasks_max
  [systemd_enable_task_metrics: <boolean> | default = false]

  # Enables service unit metric service_restart_total
  [systemd_enable_restarts_metrics: <boolean> | default = false]

  # Enables service unit metric unit_start_time_seconds
  [systemd_enable_start_time_metrics: <boolean> | default = false]

  # Regexp of tapestats devices to ignore.
  [tapestats_ignored_devices: <string> | default = "^$"]

  # Directory to read *.prom files from for the textfile collector.
  [textfile_directory: <string> | default = ""]

  # Regexp of fields to return for the vmstat collector.
  [vmstat_fields: <string> | default = "^(oom_kill|pgpg|pswp|pg.*fault).*"]
```


# node_exporter 自定义 collectors 


您可以在 integrations node_export 配置中，通过设置和修改 `set_collectors` `enable_collectors` `disable_collectors`，以控制哪些 collector 生效。


```Golang
const (
  CollectorARP          = "arp"
	CollectorBCache       = "bcache"
	CollectorBTRFS        = "btrfs"
	CollectorBonding      = "bonding"
	CollectorBootTime     = "boottime"
	CollectorBuddyInfo    = "buddyinfo"
	CollectorCPU          = "cpu"
	CollectorCPUFreq      = "cpufreq"
	CollectorConntrack    = "conntrack"
	CollectorDMI          = "dmi"
	CollectorDRBD         = "drbd"
	CollectorDRM          = "drm"
	CollectorDevstat      = "devstat"
	CollectorDiskstats    = "diskstats"
	CollectorEDAC         = "edac"
	CollectorEntropy      = "entropy"
	CollectorEthtool      = "ethtool"
	CollectorExec         = "exec"
	CollectorFibrechannel = "fibrechannel"
	CollectorFileFD       = "filefd"
	CollectorFilesystem   = "filesystem"
	CollectorHWMon        = "hwmon"
	CollectorIPVS         = "ipvs"
	CollectorInfiniband   = "infiniband"
	CollectorInterrupts   = "interrupts"
	CollectorKSMD         = "ksmd"
	CollectorLnstat       = "lnstat"
	CollectorLoadAvg      = "loadavg"
	CollectorLogind       = "logind"
	CollectorMDADM        = "mdadm"
	CollectorMeminfo      = "meminfo"
	CollectorMeminfoNuma  = "meminfo_numa"
	CollectorMountstats   = "mountstats"
	CollectorNFS          = "nfs"
	CollectorNFSD         = "nfsd"
	CollectorNTP          = "ntp"
	CollectorNVME         = "nvme"
	CollectorNetclass     = "netclass"
	CollectorNetdev       = "netdev"
	CollectorNetstat      = "netstat"
	CollectorNetworkRoute = "network_route"
	CollectorOS           = "os"
	CollectorPerf         = "perf"
	CollectorPowersuppply = "powersupplyclass"
	CollectorPressure     = "pressure"
	CollectorProcesses    = "processes"
	CollectorQDisc        = "qdisc"
	CollectorRAPL         = "rapl"
	CollectorRunit        = "runit"
	CollectorSchedstat    = "schedstat"
	CollectorSockstat     = "sockstat"
	CollectorSoftnet      = "softnet"
	CollectorStat         = "stat"
	CollectorSupervisord  = "supervisord"
	CollectorSystemd      = "systemd"
	CollectorTCPStat      = "tcpstat"
	CollectorTapestats    = "tapestats"
	CollectorTextfile     = "textfile"
	CollectorThermal      = "thermal"
	CollectorThermalzone  = "thermal_zone"
	CollectorTime         = "time"
	CollectorTimex        = "timex"
	CollectorUDPQueues    = "udp_queues"
	CollectorUname        = "uname"
	CollectorVMStat       = "vmstat"
	CollectorWiFi         = "wifi"
	CollectorXFS          = "xfs"
	CollectorZFS          = "zfs"
	CollectorZoneinfo     = "zoneinfo"
)
```
