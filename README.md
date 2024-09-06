# kube-scalerr
Kubernetes workload scaler

his is an open-source Kubernetes controller that automatically scales Deployments and StatefulSets based on custom annotations in the workloads. The controller periodically checks for annotations that specify scale-up and scale-down times, and adjusts replicas accordingly.

Features
Automatic Scaling: Supports scaling up and down based on annotations in Deployments and StatefulSets.
Custom Scaling Schedules: Use custom annotations like scaleup_time and scaledown_time to define when to scale resources.
Weekend Scaling Support: Optionally skip scaling on weekends using the weekend_scaleup annotation.
Concurrent Processing: Uses multi-threading to efficiently process workloads and scale them concurrently.
Configurable via Environment Variables: Control polling intervals, execution intervals, number of threads, and namespaces to exclude through environment variables.
Retry Logic: Retries scaling operations in case of API failures.
CSV-Based Tracking: Tracks scaling operations and stores relevant information in a CSV file for easy reference.
How It Works
The controller runs two primary loops:

Polling Loop: Fetches all Deployments and StatefulSets in the cluster and checks their annotations. If a workload has scaleup_time, scaleup_replicas, scaledown_time, or scaledown_replicas annotations, it logs the information and writes it to a CSV file for later use.

Scaling Loop: Every minute, the controller checks the CSV file for workloads scheduled to scale at the current time and performs scaling operations accordingly.

Custom Annotations
Workloads can be annotated with the following keys:

scaleup_time: The time in UTC when the resource should be scaled up. Format: HH-MM (e.g., 14-30 for 2:30 PM UTC).
scaleup_replicas: The number of replicas to scale up to at the specified scaleup_time.
scaledown_time: The time in UTC when the resource should be scaled down. Format: HH-MM.
scaledown_replicas: The number of replicas to scale down to at the specified scaledown_time.
weekend_scaleup: Set to on or off to enable or disable scaling on weekends (default: off).
fixed_scaleup: If defined, overrides scaleup_replicas with this fixed value.

## Environment Variables

| Variable           | Default | Description                                                                 |
| ------------------ | ------- | --------------------------------------------------------------------------- |
| `POLL_INTERVAL`     | 60      | The polling interval in minutes.                                            |
| `EXEC_INTERVAL`     | 1       | The execution interval in minutes.                                          |
| `POLL_THREADS`      | 4       | Number of threads for polling annotations.                                  |
| `EXEC_THREADS`      | 10      | Number of threads for executing scaling operations.                         |
| `EXCLUDE_NAMESPACES`| (empty) | Comma-separated list of namespaces to exclude from processing.              |



