---
weight: 70
title: "对接Grafana"
---

夜莺无需对接 Grafana，夜莺会把监控数据转存到后端时序库，比如 Prometheus、VictoriaMetrics、M3DB 等，把这些时序库配置为 Grafana 的数据源即可。