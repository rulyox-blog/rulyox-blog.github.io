---
layout: post
title: Prometheus, Loki, Grafana를 이용한 모니터링 시스템 구축
thumbnail-img: /assets/img/posts/2021-10-24-Prometheus-Loki-Grafana를-이용한-모니터링-시스템-구축/thumb.png
share-img: /assets/img/posts/2021-10-24-Prometheus-Loki-Grafana를-이용한-모니터링-시스템-구축/thumb.png
tags: [Tutorials]
comments: true
---

모니터링 시스템이란, 어떤 서비스 혹은 시스템으로부터 시계열 데이터를 수집하는 시스템 입니다.

대표적인 시계열 데이터에는 CPU, 메모리 사용량 등에 대한 수치 데이터와 사용자의 활동을 기록하는 로그 데이터 등이 있습니다.

본 포스트는 임의의 시계열 데이터를 생성하는 프로그램(실제 상황에서는 모니터링 대상 시스템)을 만들고, 이를 모니터링하는 시스템을 구축하는 예시입니다. 예를 들기 위해 Prometheus와 Loki를 둘 다 이용했으며, 상황에 따라 둘 중 하나만 이용하는 방식도 가능합니다.

이용하는 모든 코드는 [GitHub](https://github.com/rulyox/logging-example)에 있습니다.

## 모니터링 시스템 구조

![diagram.png](/assets/img/posts/2021-10-24-Prometheus-Loki-Grafana를-이용한-모니터링-시스템-구축/diagram.png){: .mx-auto.d-block :}
<p align="center"><i>System Diagram</i></p>

- Collector : 모니터링 대상 시스템이 생성하는 데이터를 Prometheus와 Promtail이 원하는 형식으로 제공
- Prometheus : 시계열 수치(float) 데이터를 저장하는 DB
- Loki : 시계열 로그(string) 데이터를 저장하는 DB
- Promtail : Collector가 출력하는 로그 데이터를 Loki에게 전달
- Grafana : Prometheus와 Loki의 데이터를 시각화

위 구조 중 Collector만 직접 프로그래밍 하고, Prometheus, Loki, Promtail, Grafana는 완성된 프로그램을 이용하는 방식입니다.

Collector가 Prometheus에 데이터를 전달하기 위해서는 [Prometheus client libraries](https://prometheus.io/docs/instrumenting/clientlibs/)를 이용한 [Exporter](https://prometheus.io/docs/instrumenting/writing_exporters/) 방식의 HTTP 인터페이스 제공하며, Promtail을 위해서는 단순히 로그를 특정 파일에 저장하는 방식을 사용합니다.

각 모듈들은 밀접하게 관련이 있기 때문에, Docker Compose를 이용하여 환경을 구성하였습니다. 따라서 각 프로그램은 상대 프로그램을 Docker Network에서 제공하는 host name을 통해 호출합니다.

## Jobs & Metrics

Prometheus와 Loki가 수집하는 데이터는 크게 job으로 구분할 수 있습니다. job은 여러 대상(Exporter, 로그 파일 등)으로부터 수집된 데이터 중 관련있는 데이터끼리 묶을 수 있는 집합입니다. 본 포스트에서는 `logging`이라는 하나의 job만 사용합니다.

[Metric](https://prometheus.io/docs/concepts/metric_types/)은 Prometheus에 저장할 수 있는 데이터의 종류입니다. Prometheus가 가지는 각 metric은 Counter, Gauge, Histogram, Summary 중 하나의 타입을 가집니다. (실제로는 저장되는 값은 float이지만, 제공하는 기능을 분리하기 위해 4가지로 나눠 사용됩니다.)

예를 들어, `System` job에는 `CPU Usage`, `Memory Usage`, `Disk Usage`, `Network Speed` 등의 metric이 있을 수 있고, `Application` job에는 `Number of User Connections`, `Service Health` `Pending Tasks` 등의 metric이 있을 수 있습니다.

## Collector

직접 작성한 유일한 모듈인 Collector는 go로 프로그래밍되었으며, [GitHub](https://github.com/rulyox/logging-example/tree/master/collector)에서 전체 코드를 보실 수 있습니다.

Collector는 아래 2가지 요소를 가집니다.

- Prometheus를 위한 수치 데이터 제공 HTTP 인터페이스인 Exporter
- Loki(Promtail)를 위한 로그 데이터를 저장하는 파일

아래는 일부 중요한 코드에 대한 설명입니다.

~~~
func main() {
	log.InitLogPath()
	exporter.InitMetrics()

	go exporter.StartExporter(port)
	collect.StartCollecting()
}
~~~

`collector/cmd/collector/main.go`의 main 함수입니다.

`InitMetrics()`는 Prometheus metric을 생성합니다.

`StartExporter()`는 Prometheus가 이용할 HTTP 서버를 실행합니다.

`StartCollecting()`에서는 반복문을 통해 일정 시간마다 Prometheus metric을 업데이트하고 Loki를 위한 로그를 파일에 출력합니다.

~~~
func InitMetrics() {
	MyCounter = promauto.NewCounter(prometheus.CounterOpts{
		Name: "my_counter",
		Help: "My Counter",
	})
	MyGauge = promauto.NewGauge(prometheus.GaugeOpts{
		Name: "my_gauge",
		Help: "My Gauge",
	})
}
~~~

위 코드는 `collector/internal/exporter/metrics.go`의 일부입니다. `promauto`를 이용하여 `my_counter`와 `my_gauge` 2개의 metric을 쉽게 생성하였습니다.

~~~
func StartExporter(port int) {
	http.Handle("/metrics", promhttp.Handler())
	err := http.ListenAndServe(":"+strconv.Itoa(port), nil)
	if err != nil {
		panic(err)
	}
}
~~~

`collector/internal/exporter/exporter.go`의 코드입니다. HTTP 서버를 실행하고 `/metrics` 경로에 위에서 `promauto`로 생성한 2개의 metric의 값을 출력합니다. `/metrics` 경로는 Prometheus가 탐색할 기본 경로입니다.

~~~
var logDir = "logs"
var logFile = "collector.log"

func InitLogPath() {
	currentDir := utility.CurrentDir()

	// create logs path if not exists
	err := os.MkdirAll(path.Join(currentDir, logDir), os.ModePerm)
	if err != nil {
		panic("Log path creation failed")
	}
}

func Log(text string) {
	currentDir := utility.CurrentDir()

	// create file if not exist and append
	f, _ := os.OpenFile(path.Join(currentDir, logDir, logFile), os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
	defer f.Close()

	datetime := time.Now()
	formattedDatetime := datetime.Format("2006-01-02 15:04:05 ")

	f.WriteString(formattedDatetime + text + "\n")
}
~~~

`collector/internal/log/log.go`입니다. 파일 시스템의 `logs/collector.log`에 로그를 출력하는 함수인 `Log()`를 정의했습니다.

~~~
var cycleInterval = 10

func StartCollecting() {
	for range time.Tick(time.Second * time.Duration(cycleInterval)) {
		log.Log("Starting collect cycle")
		cycle()
	}
}

func cycle() {
	exporter.MyCounter.Add(1)
	log.Log("Counter increased")

	random := rand.Float64()
	exporter.MyGauge.Set(random)
	log.Log("Gauge updated " + fmt.Sprintf("%f", random))

	log.Log("Cycle Finished")
}
~~~

마지막으로, 실질적인 데이터를 수집하는 부분인 `collector/internal/collect/collect.go`입니다. 10초 마다 `cycle()` 함수를 실행합니다.

`cycle()` 함수는 `MyCounter`와 `MyGauge`의 값을 업데이트하며, `Log()`함수를 통해 로그를 출력합니다.

## Prometheus

아래는 Prometheus의 config입니다.

~~~
global:
  scrape_interval:     15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: logging
    static_configs:
      - targets: ['collector:8080']
~~~

핵심은 HTTP request를 이용해 `collector:8080`에서 metric을 15초 마다 pull하는 방식이라는 것입니다.

Prometheus는 기본적으로 pull 방식만을 지원하지만, 시스템 구성에 따라 push 방식이 필요하다면 [Prometheus Pushgateway](https://github.com/prometheus/pushgateway)를 참고하세요.

## Loki

Loki도 config가 존재하나, 여기에는 인스턴스를 생성하기 위한 기본적인 설정들만 존재하고, 실제 데이터는 Promtail을 통해 전달받기 때문에 생략하겠습니다.

## Promtail

아래는 Promtail의 config입니다.

~~~
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
- job_name: logging
  static_configs:
  - targets:
      - localhost
    labels:
      job: logging
      __path__: /logs/*.log
~~~

Promtail은 `/logs`에 존재하는 모든 `.log` 파일을 읽어서 이를 `loki:3100`에게 push하는 방식입니다.

## Grafana

Collector, Prometheus, Loki가 구동중이라면 데이터는 이미 수집되고 있습니다.

Grafana는 수집된 데이터를 쿼리하고 시각화해서 보여주는 프로그램입니다.

![screenshot-grafana-login.png](/assets/img/posts/2021-10-24-Prometheus-Loki-Grafana를-이용한-모니터링-시스템-구축/screenshot-grafana-login.png){: .mx-auto.d-block :}
<p align="center"><i>Grafana Login</i></p>

Grafana의 기본 username과 password는 `admin`입니다.

![screenshot-grafana-prometheus.png](/assets/img/posts/2021-10-24-Prometheus-Loki-Grafana를-이용한-모니터링-시스템-구축/screenshot-grafana-prometheus.png){: .mx-auto.d-block :}
<p align="center"><i>Grafana Prometheus</i></p>

![screenshot-grafana-loki.png](/assets/img/posts/2021-10-24-Prometheus-Loki-Grafana를-이용한-모니터링-시스템-구축/screenshot-grafana-loki.png){: .mx-auto.d-block :}
<p align="center"><i>Grafana Loki</i></p>

우선 설정에서 Prometheus와 Loki를 Data Source로 등록해야 Grafana를 통해 시각화할 수 있습니다.

URL은 각각 `http://prometheus:9090`과 `http://loki:3100`입니다.

![screenshot-grafana-counter.png](/assets/img/posts/2021-10-24-Prometheus-Loki-Grafana를-이용한-모니터링-시스템-구축/screenshot-grafana-counter.png){: .mx-auto.d-block :}
<p align="center"><i>Grafana Counter</i></p>

![screenshot-grafana-gauge.png](/assets/img/posts/2021-10-24-Prometheus-Loki-Grafana를-이용한-모니터링-시스템-구축/screenshot-grafana-gauge.png){: .mx-auto.d-block :}
<p align="center"><i>Grafana Gauge</i></p>

![screenshot-grafana-log.png](/assets/img/posts/2021-10-24-Prometheus-Loki-Grafana를-이용한-모니터링-시스템-구축/screenshot-grafana-log.png){: .mx-auto.d-block :}
<p align="center"><i>Grafana Log</i></p>

대시보드에서 패널 추가 버튼을 눌러 Prometheus와 Loki가 제공하는 데이터의 패널을 추가할 수 있습니다. Collector에서 만든 `my_counter`, `my_gauge`와 Promtail이 읽는 파일인 `/logs/collector.log`에 해당하는 패널을 각각 만들었습니다.

패널에서 데이터를 쿼리하기 위해 이용한 언어는 PromQL과 LogQL입니다.

![screenshot-grafana-dashboard.png](/assets/img/posts/2021-10-24-Prometheus-Loki-Grafana를-이용한-모니터링-시스템-구축/screenshot-grafana-dashboard.png){: .mx-auto.d-block :}
<p align="center"><i>Grafana Dashboard</i></p>

Grafana 대시보드를 꾸밀 수 있는 방법은 무궁무진합니다.

본인의 상황에 가장 적합한 모니터링 화면을 구상하는 것이 중요합니다.
