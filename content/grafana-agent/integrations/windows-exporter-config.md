---
title: "Windows Exporter"
---



grafana-agent内置了[`windows_exporter`](https://github.com/grafana/windows_exporter)的实现，可以采集到windows平台的指标。

# 配置并启用windows_exporter
```
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
  windows_exporter:
    enabled: true
```

# 采集的关键指标列表
```yaml
windows_cpu_clock_interrupts_total: Total number of received and serviced clock tick interrupts(counter)
windows_cpu_core_frequency_mhz: Core frequency in megahertz(gauge)
windows_cpu_cstate_seconds_total: Time spent in low-power idle state(counter)
windows_cpu_dpcs_total: Total number of received and serviced deferred procedure calls (DPCs)(counter)
windows_cpu_idle_break_events_total: Total number of time processor was woken from idle(counter)
windows_cpu_interrupts_total: Total number of received and serviced hardware interrupts(counter)
windows_cpu_parking_status: Parking Status represents whether a processor is parked or not(gauge)
windows_cpu_processor_performance: Processor Performance is the average performance of the processor while it is executing instructions, as a percentage of the nominal performance of the processor. On some processors, Processor Performance may exceed 100%(gauge)
windows_cpu_time_total: Time that processor spent in different modes (idle, user, system, ...)(counter)
windows_cs_hostname: Labeled system hostname information as provided by ComputerSystem.DNSHostName and ComputerSystem.Domain(gauge)
windows_cs_logical_processors: ComputerSystem.NumberOfLogicalProcessors(gauge)
windows_cs_physical_memory_bytes: ComputerSystem.TotalPhysicalMemory(gauge)
windows_exporter_build_info: A metric with a constant '1' value labeled by version, revision, branch, and goversion from which windows_exporter was built.(gauge)
windows_exporter_collector_duration_seconds: Duration of a collection.(gauge)
windows_exporter_collector_success: Whether the collector was successful.(gauge)
windows_exporter_collector_timeout: Whether the collector timed out.(gauge)
windows_exporter_perflib_snapshot_duration_seconds: Duration of perflib snapshot capture(gauge)
windows_logical_disk_free_bytes: Free space in bytes (LogicalDisk.PercentFreeSpace)(gauge)
windows_logical_disk_idle_seconds_total: Seconds that the disk was idle (LogicalDisk.PercentIdleTime)(counter)
windows_logical_disk_read_bytes_total: The number of bytes transferred from the disk during read operations (LogicalDisk.DiskReadBytesPerSec)(counter)
windows_logical_disk_read_latency_seconds_total: Shows the average time, in seconds, of a read operation from the disk (LogicalDisk.AvgDiskSecPerRead)(counter)
windows_logical_disk_read_seconds_total: Seconds that the disk was busy servicing read requests (LogicalDisk.PercentDiskReadTime)(counter)
windows_logical_disk_read_write_latency_seconds_total: Shows the time, in seconds, of the average disk transfer (LogicalDisk.AvgDiskSecPerTransfer)(counter)
windows_logical_disk_reads_total: The number of read operations on the disk (LogicalDisk.DiskReadsPerSec)(counter)
windows_logical_disk_requests_queued: The number of requests queued to the disk (LogicalDisk.CurrentDiskQueueLength)(gauge)
windows_logical_disk_size_bytes: Total space in bytes (LogicalDisk.PercentFreeSpace_Base)(gauge)
windows_logical_disk_split_ios_total: The number of I/Os to the disk were split into multiple I/Os (LogicalDisk.SplitIOPerSec)(counter)
windows_logical_disk_write_bytes_total: The number of bytes transferred to the disk during write operations (LogicalDisk.DiskWriteBytesPerSec)(counter)
windows_logical_disk_write_latency_seconds_total: Shows the average time, in seconds, of a write operation to the disk (LogicalDisk.AvgDiskSecPerWrite)(counter)
windows_logical_disk_write_seconds_total: Seconds that the disk was busy servicing write requests (LogicalDisk.PercentDiskWriteTime)(counter)
windows_logical_disk_writes_total: The number of write operations on the disk (LogicalDisk.DiskWritesPerSec)(counter)
windows_net_bytes_received_total: (Network.BytesReceivedPerSec)(counter)
windows_net_bytes_sent_total: (Network.BytesSentPerSec)(counter)
windows_net_bytes_total: (Network.BytesTotalPerSec)(counter)
windows_net_current_bandwidth: (Network.CurrentBandwidth)(gauge)
windows_net_packets_outbound_discarded_total: (Network.PacketsOutboundDiscarded)(counter)
windows_net_packets_outbound_errors_total: (Network.PacketsOutboundErrors)(counter)
windows_net_packets_received_discarded_total: (Network.PacketsReceivedDiscarded)(counter)
windows_net_packets_received_errors_total: (Network.PacketsReceivedErrors)(counter)
windows_net_packets_received_total: (Network.PacketsReceivedPerSec)(counter)
windows_net_packets_received_unknown_total: (Network.PacketsReceivedUnknown)(counter)
windows_net_packets_sent_total: (Network.PacketsSentPerSec)(counter)
windows_net_packets_total: (Network.PacketsPerSec)(counter)
windows_os_info: OperatingSystem.Caption, OperatingSystem.Version(gauge)
windows_os_paging_free_bytes: OperatingSystem.FreeSpaceInPagingFiles(gauge)
windows_os_paging_limit_bytes: OperatingSystem.SizeStoredInPagingFiles(gauge)
windows_os_physical_memory_free_bytes: OperatingSystem.FreePhysicalMemory(gauge)
windows_os_process_memory_limix_bytes: OperatingSystem.MaxProcessMemorySize(gauge)
windows_os_processes: OperatingSystem.NumberOfProcesses(gauge)
windows_os_processes_limit: OperatingSystem.MaxNumberOfProcesses(gauge)
windows_os_time: OperatingSystem.LocalDateTime(gauge)
windows_os_timezone: OperatingSystem.LocalDateTime(gauge)
windows_os_users: OperatingSystem.NumberOfUsers(gauge)
windows_os_virtual_memory_bytes: OperatingSystem.TotalVirtualMemorySize(gauge)
windows_os_virtual_memory_free_bytes: OperatingSystem.FreeVirtualMemory(gauge)
windows_os_visible_memory_bytes: OperatingSystem.TotalVisibleMemorySize(gauge)
windows_service_info: A metric with a constant '1' value labeled with service information(gauge)
windows_service_start_mode: The start mode of the service (StartMode)(gauge)
windows_service_state: The state of the service (State)(gauge)
windows_service_status: The status of the service (Status)(gauge)
windows_system_context_switches_total: Total number of context switches (WMI source is PerfOS_System.ContextSwitchesPersec)(counter)
windows_system_exception_dispatches_total: Total number of exceptions dispatched (WMI source is PerfOS_System.ExceptionDispatchesPersec)(counter)
windows_system_processor_queue_length: Length of processor queue (WMI source is PerfOS_System.ProcessorQueueLength)(gauge)
windows_system_system_calls_total: Total number of system calls (WMI source is PerfOS_System.SystemCallsPersec)(counter)
windows_system_system_up_time: System boot time (WMI source is PerfOS_System.SystemUpTime)(gauge)
windows_system_threads: Current number of threads (WMI source is PerfOS_System.Threads)(gauge)
```

# 完整地配置项说明
```yaml
  # Enables the windows_exporter integration, allowing the Agent to automatically
  # collect system metrics from the local windows instance
  [enabled: <boolean> | default = false]

  # Sets an explicit value for the instance label when the integration is
  # self-scraped. Overrides inferred values.
  #
  # The default value for this integration is inferred from the agent hostname
  # and HTTP listen port, delimited by a colon.
  [instance: <string>]

  # Automatically collect metrics from this integration. If disabled,
  # the consul_exporter integration will be run but not scraped and thus not
  # remote-written. Metrics for the integration will be exposed at
  # /integrations/windows_exporter/metrics and can be scraped by an external
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

  # Monitor the exporter itself and include those metrics in the results.
  [include_exporter_metrics: <bool> | default = false]

  #
  # Exporter-specific configuration options
  #

  # List of collectors to enable. Any non-experimental collector from the
  # embeded version of windows_exporter can be enabeld here.
  [enabled_collectors: <string> | default = "cpu,cs,logical_disk,net,os,service,system,textfile"]

  # Settings for collectors which accept configuration. Settings specified here
  # are only used if the corresponding collector is enabled in
  # enabled_collectors.

  # Configuration for Exchange Mail Server
  exchange:
    # Comma-separated List of collectors to use. Defaults to all, if not specified.
    # Maps to collectors.exchange.enabled in windows_exporter
    [enabled_list: <string>]

  # Configuration for the IIS web server
  iis:
    # Regexp of sites to whitelist. Site name must both match whitelist and not match blacklist to be included.
    # Maps to collector.iis.site-whitelist in windows_exporter
    [site_whitelist: <string> | default = ".+"]

    # Regexp of sites to blacklist. Site name must both match whitelist and not match blacklist to be included.
    # Maps to collector.iis.site-blacklist in windows_exporter
    [site_blacklist: <string> | default = ""]

    # Regexp of apps to whitelist. App name must both match whitelist and not match blacklist to be included.
    # Maps to collector.iis.app-whitelist in windows_exporter
    [app_whitelist: <string> | default=".+"]

    # Regexp of apps to blacklist. App name must both match whitelist and not match blacklist to be included.
    # Maps to collector.iis.app-blacklist in windows_exporter
    [app_blacklist: <string> | default=".+"]

  # Configuration for reading metrics from a text files in a directory
  text_file:
    # Directory to read text files with metrics from.
    # Maps to collector.textfile.directory in windows_exporter
    [text_file_directory: <string> | default="C:\Program Files\windows_exporter\textfile_inputs"]

  # Configuration for SMTP metrics
  smtp:
    # Regexp of virtual servers to whitelist. Server name must both match whitelist and not match blacklist to be included.
    # Maps to collector.smtp.server-whitelist in windows_exporter
    [whitelist: <string> | default=".+"]

    # Regexp of virtual servers to blacklist. Server name must both match whitelist and not match blacklist to be included.
    # Maps to collector.smtp.server-blacklist in windows_exporter
    [blacklist: <string> | default=""]

  # Configuration for Windows Services
  service:
    # "WQL 'where' clause to use in WMI metrics query. Limits the response to the services you specify and reduces the size of the response.
    # Maps to collector.service.services-where in windows_exporter
    [where_clause: <string> | default=""]

  # Configuration for Windows Processes
  process:
    # Regexp of processes to include. Process name must both match whitelist and not match blacklist to be included.
    # Maps to collector.process.whitelist in windows_exporter
    [whitelist: <string> | default=".+"]

    # Regexp of processes to exclude. Process name must both match whitelist and not match blacklist to be included.
    # Maps to collector.process.blacklist in windows_exporter
    [blacklist: <string> | default=""]

  # Configuration for NICs
  network:
    # Regexp of NIC's to whitelist. NIC name must both match whitelist and not match blacklist to be included.
    # Maps to collector.net.nic-whitelist in windows_exporter
    [whitelist: <string> | default=".+"]

    # Regexp of NIC's to blacklist. NIC name must both match whitelist and not match blacklist to be included.
    # Maps to collector.net.nic-blacklist in windows_exporter
    [blacklist: <string> | default=""]

  # Configuration for Microsoft SQL Server
  mssql:
    # Comma-separated list of mssql WMI classes to use.
    # Maps to collectors.mssql.classes-enabled in windows_exporter
    [enabled_classes: <string> | default="accessmethods,availreplica,bufman,databases,dbreplica,genstats,locks,memmgr,sqlstats,sqlerrors,transactions"]

  # Configuration for Microsoft Queue
  msqm:
    # WQL 'where' clause to use in WMI metrics query. Limits the response to the msmqs you specify and reduces the size of the response.
    # Maps to collector.msmq.msmq-where in windows_exporter
    [where_clause: <string> | default=""]

  # Configuration for disk information
  logical_disk:
    # Regexp of volumes to whitelist. Volume name must both match whitelist and not match blacklist to be included.
    # Maps to collector.logical_disk.volume-whitelist in windows_exporter
    [whitelist: <string> | default=".+"]

    # Regexp of volumes to blacklist. Volume name must both match whitelist and not match blacklist to be included.
    # Maps to collector.logical_disk.volume-blacklist in windows_exporter
    [blacklist: <string> | default=".+"]
```
