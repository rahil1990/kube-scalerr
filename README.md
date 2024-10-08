# kube-scalerr
Kubernetes workload scaling controller

This is an open-source Kubernetes controller that automatically scales Deployments and StatefulSets based on custom annotations in the workloads. The controller periodically checks for annotations that specify scale-up and scale-down times, and adjusts replicas accordingly.

## Poll and Exec Components

The controller consists of two key components: **Poll** and **Exec**, which work together to handle scaling operations.

- **Poll Component**: 
  - Runs every `POLL_INTERVAL` minutes, fetching all `Deployments` and `StatefulSets` in the cluster.
  - Checks each workload's annotations for `scaleup_time`, `scaledown_time`, and replica settings.
  - Records relevant workload information (namespace, name, scale-up/down times, replicas) into a CSV file (`/data/deployment_data.csv`).
  - Uses multi-threading, controlled by the `POLL_THREADS` variable, to fetch and process workloads concurrently.

- **Exec Component**: 
  - Runs every minute (configurable via `EXEC_INTERVAL`), reading the CSV file generated by the Poll component.
  - Compares the current time to the `scaleup_time` or `scaledown_time` annotations and scales the workload to the specified replica count.
  - Skips scaling if the `weekend_scaleup` annotation is set to `off` during weekends.
  - Uses multi-threading, controlled by the `EXEC_THREADS` variable, to scale workloads concurrently.
  - Implements retry logic to handle API errors during scaling operations.

## Features

| Feature                                  | Description                                                                                     |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Automatic Scaling**                    | Supports scaling up and down based on annotations in `Deployments` and `StatefulSets`.           |
| **Custom Scaling Schedules**             | Use annotations like `scaleup_time` and `scaledown_time` to define when to scale resources.      |
| **Weekend Scaling Support**              | Optionally skip scaling on weekends using the `weekend_scaleup` annotation.                     |
| **Concurrent Processing**                | Uses multi-threading to efficiently process and scale workloads concurrently.                   |
| **Configurable via Environment Variables**| Control polling intervals, execution intervals, thread counts, and excluded namespaces.          |
| **Retry Logic**                          | Automatically retries scaling operations if API calls fail.                                      |
| **CSV-Based Tracking**                   | Tracks scaling operations and stores relevant information in a CSV file for reference.           |


## How It Works

- **Polling Loop**: Periodically fetches all `Deployments` and `StatefulSets` in the cluster and checks their annotations.
- **Annotation Check**: Checks for annotations like `scaleup_time`, `scaleup_replicas`, `scaledown_time`, and `scaledown_replicas`.
- **CSV Logging**: Logs relevant information from workloads and stores it in a CSV file for later reference.
- **Scaling Loop**: Every minute, checks the CSV file for workloads scheduled to scale and performs the scaling operation.
- **Concurrent Scaling**: Utilizes multi-threading to scale multiple workloads concurrently based on the schedule.




## Custom Annotations

Workloads can be annotated with the following keys:

| Annotation            | Description                                                                                   |
| --------------------- | --------------------------------------------------------------------------------------------- |
| `scaleup_time`        | The time in UTC when the resource should be scaled up. Format: `HH-MM` (e.g., `14-30`).        |
| `scaleup_replicas`    | The number of replicas to scale up to at the specified `scaleup_time`.                         |
| `scaledown_time`      | The time in UTC when the resource should be scaled down. Format: `HH-MM`.                      |
| `scaledown_replicas`  | The number of replicas to scale down to at the specified `scaledown_time`.                     |
| `weekend_scaleup`     | Set to `on` or `off` to enable or disable scaling on weekends (default: `off`).                |
| `fixed_scaleup`       | If defined, overrides `scaleup_replicas` with this fixed value.                                |



## Environment Variables

| Variable           | Default | Description                                                                 |
| ------------------ | ------- | --------------------------------------------------------------------------- |
| `POLL_INTERVAL`     | 60      | The polling interval in minutes.                                            |
| `EXEC_INTERVAL`     | 1       | The execution interval in minutes.                                          |
| `POLL_THREADS`      | 4       | Number of threads for polling annotations.                                  |
| `EXEC_THREADS`      | 10      | Number of threads for executing scaling operations.                         |
| `EXCLUDE_NAMESPACES`| (empty) | Comma-separated list of namespaces to exclude from processing.              |




## License

This project is licensed under the MIT License. See the `LICENSE` file for more details.

