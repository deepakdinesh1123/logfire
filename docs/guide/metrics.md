!!! warning "🚧 Work in Progress 🚧"
    Metrics are a work in progress, [contact us](../help.md) if you have any questions.

**Pydantic Logfire** can be used to collect metrics from your application and send them to a metrics backend.

Let's see how to create, and use metrics in your application.

```py
import logfire

# Create a counter metric
counter = logfire.metric_counter('exceptions')

try:
    # Simulate an exception
    raise Exception('oops')
except Exception:
    # Increment the counter
    counter.add(1)
```

## Metric Types

Metrics are a great way to record number values where you want to see an aggregation of the data (e.g. over time),
rather than the individual values.

### Counter

The Counter metric is particularly useful when you want to measure the frequency or occurrence of a certain
event or state in your application.

You can use this metric for counting things like:

* The number of exceptions caught.
* The number of requests received.
* The number of items processed.

To create a counter metric, use the [`logfire.metric_counter`][logfire.Logfire.metric_counter] function:

```py
import logfire

counter = logfire.metric_counter(
    'exceptions',
    unit='1',  # (1)!
    description='Number of exceptions caught'
)

try:
    raise Exception('oops')
except Exception:
    counter.add(1)
```

1. The `unit` parameter is optional, but it's a good practice to specify it.
    It should be a string that represents the unit of the counter.
    If the metric is _unitless_, you can use `'1'`.

You can read more about the Counter metric in the [OpenTelemetry documentation][counter-metric].

### Histogram

The Histogram metric is particularly useful when you want to measure the distribution of a set of values.

You can use this metric for measuring things like:

* The duration of a request.
* The size of a file.
* The number of items in a list.

To create a histogram metric, use the [`logfire.metric_histogram`][logfire.Logfire.metric_histogram] function:

```py
import logfire

histogram = logfire.metric_histogram(
    'request_duration',
    unit='ms',  # (1)!
    description='Duration of requests'
)

for duration in [10, 20, 30, 40, 50]:
    histogram.record(duration)
```

1. The `unit` parameter is optional, but it's a good practice to specify it.
    It should be a string that represents the unit of the histogram.

You can read more about the Histogram metric in the [OpenTelemetry documentation][histogram-metric].

### Up-Down Counter

The "Up-Down Counter" is a type of counter metric that allows both incrementing (up) and decrementing (down) operations.
Unlike a regular counter that only allows increments, an up-down counter can be increased or decreased based on
the events or states you want to track.

You can use this metric for measuring things like:

* The number of active connections.
* The number of items in a queue.
* The number of users online.

To create an up-down counter metric, use the [`logfire.metric_up_down_counter`][logfire.Logfire.metric_up_down_counter] function:

```py
import logfire

active_users = logfire.metric_up_down_counter(
    'active_users',
    unit='1',  # (1)!
    description='Number of active users'
)

def user_logged_in():
    active_users.add(1)

def user_logged_out():
    active_users.add(-1)
```

1. The `unit` parameter is optional, but it's a good practice to specify it.
    It should be a string that represents the unit of the up-down counter.
    If the metric is _unitless_, you can use `'1'`.

You can read more about the Up-Down Counter metric in the [OpenTelemetry documentation][up-down-counter-metric].

### Callback Metrics

Callback metrics, or observable metrics, are a way to create metrics that are automatically updated based on a time interval.

#### Counter Callback

To create a counter callback metric, use the [`logfire.metric_counter_callback`][logfire.Logfire.metric_counter_callback] function:

```py
import logfire
from opentelemetry.metrics import CallbackOptions, Observable


def cpu_time_callback(options: CallbackOptions) -> Iterable[Observation]:
    observations = []
    with open("/proc/stat") as procstat:
        procstat.readline()  # skip the first line
        for line in procstat:
            if not line.startswith("cpu"):
                break
            cpu, user_time, nice_time, system_time = line.split()
            observations.append(
                Observation(int(user_time) // 100, {"cpu": cpu, "state": "user"})
            )
            observations.append(
                Observation(int(nice_time) // 100, {"cpu": cpu, "state": "nice"})
            )
            observations.append(
                Observation(int(system_time) // 100, {"cpu": cpu, "state": "system"})
            )
    return observations

logfire.metric_counter_callback(
    'system.cpu.time',
    unit='s',
    callbacks=[cpu_time_callback],
    description='CPU time',
)
```

You can read more about the Counter metric in the [OpenTelemetry documentation][counter-callback-metric].

#### Gauge Callback

The gauge metric is particularly useful when you want to measure the current value of a certain state
or event in your application. Unlike the counter metric, the gauge metric does not accumulate values over time.

To create a gauge callback metric, use the [`logfire.metric_gauge_callback`][logfire.Logfire.metric_gauge_callback] function:

```py
import logfire


def get_temperature(room: str) -> float:
    ...


def temperature_callback(options: CallbackOptions) -> Iterable[Observation]:
    for room in ["kitchen", "living_room", "bedroom"]:
        temperature = get_temperature(room)
        yield Observation(temperature, {"room": room})


logfire.metric_gauge_callback(
    'temperature',
    unit='°C',
    callbacks=[temperature_callback],
    description='Temperature',
)
```

You can read more about the Gauge metric in the [OpenTelemetry documentation][gauge-callback-metric].

#### Up-Down Counter Callback

This is the callback version of the [up-down counter metric](#up-down-counter).

To create an up-down counter callback metric, use the
[`logfire.metric_up_down_counter_callback`][logfire.Logfire.metric_up_down_counter_callback] function:

```py
import logfire


def get_active_users() -> int:
    ...


def active_users_callback(options: CallbackOptions) -> Iterable[Observation]:
    active_users = get_active_users()
    yield Observation(active_users, {})


logfire.metric_up_down_counter_callback(
    'active_users',
    unit='1',
    callbacks=[active_users_callback],
    description='Number of active users',
)
```

You can read more about the Up-Down Counter metric in the [OpenTelemetry documentation][up-down-counter-callback-metric].


## System Metrics

By default, **Logfire** does not collect system metrics.

To enable metrics, you need to install the `logfire[system-metrics]` extra:

```bash
pip install 'logfire[system-metrics]'
```

### Available Metrics

Logfire collects the following system metrics:

* `system.cpu.time`: CPU time spent in different modes.
* `system.cpu.utilization`: CPU utilization in different modes.
* `system.memory.usage`: Memory usage.
* `system.memory.utilization`: Memory utilization in different modes.
* `system.swap.usage`: Swap usage.
* `system.swap.utilization`: Swap utilization
* `system.disk.io`: Disk I/O operations (read/write).
* `system.disk.operations`: Disk operations (read/write).
* `system.disk.time`: Disk time (read/write).
* `system.network.dropped.packets`: Dropped packets (transmit/receive).
* `system.network.packets`: Packets (transmit/receive).
* `system.network.errors`: Network errors (transmit/receive).
* `system.network.io`: Network I/O (transmit/receive).
* `system.network.connections`: Network connections (family/type).
* `system.thread_count`: Thread count.
* `process.runtime.memory`: Process memory usage.
* `process.runtime.cpu.time`: Process CPU time.
* `process.runtime.gc_count`: Process garbage collection count.

[counter-metric]: https://opentelemetry.io/docs/specs/otel/metrics/api/#counter
[histogram-metric]: https://opentelemetry.io/docs/specs/otel/metrics/api/#histogram
[up-down-counter-metric]: https://opentelemetry.io/docs/specs/otel/metrics/api/#updowncounter
[counter-callback-metric]: https://opentelemetry.io/docs/specs/otel/metrics/api/#asynchronous-counter
[gauge-callback-metric]: https://opentelemetry.io/docs/specs/otel/metrics/api/#asynchronous-gauge
[up-down-counter-callback-metric]: https://opentelemetry.io/docs/specs/otel/metrics/api/#asynchronous-updowncounter