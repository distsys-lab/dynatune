
# Dynatune

Dynatune is a fork of [this commit](https://github.com/etcd-io/etcd/tree/3eca40d) of etcd, a widely-used distributed key-value store that leverages the Raft consensus algorithm for reliable coordination and fault tolerance.

Dynatune introduces dynamic election parameter optimization based on real-time network measurements, significantly reducing leader recovery time and ensuring availability.

We are grateful to the etcd team for their valuable contributions to the open-source community, which made Dynatune possible.

---

## Features and Modifications

### Dynamic Election Parameter Optimization
Dynatune dynamically optimizes Raft's election timeout and heartbeat interval by measuring key network metrics such as round-trip time (RTT) and packet loss in real-time. These metrics are collected from heartbeat communication between nodes, enabling precise adjustments to these parameters for optimal performance and availability.

### UDP-based Heartbeat Communication
In the original etcd implementation, all Raft communications use TCP. Dynatune introduces a new port (**2381**) dedicated to UDP-based heartbeat communication. The official etcd ports are:
- **2379**: Client requests
- **2380**: Peer communication
- **2381**: UDP-based heartbeat communication (added by Dynatune)

UDP is used for heartbeat communication to ensure accurate network measurements, which are critical for optimizing election parameters.

### New Runtime Arguments
Dynatune extends etcd's runtime arguments to enable more precise control over network measurement and parameter optimization. The following arguments are supported:

| Argument                          | Description                                                                                                            | Default Value |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | ------------- |
| `--listen-peer-udp-url`           | Specifies the address for UDP-based heartbeat communication.                                                           | None          |
| `--max-election-metrics-capacity` | Sets the maximum size of the list for storing network metrics.                                                         | 1000          |
| `--min-election-metrics-capacity` | Sets the minimum size of the list for storing network metrics. Optimization will not start until this size is reached. | 10            |
| `--election-safety-factor`        | Specifies the safety factor for election timeout calculations.                                                         | 2             |
| `--heartbeat-reachability-goal`   | Sets the target reachability for heartbeats (range: 0-1).                                                              | 0.999         |

The `--election-timeout` and `--heartbeat-interval` parameters remain available and are not removed in Dynatune. Until Dynatune's optimization process begins, the values of these parameters will be used. If these parameters are not explicitly specified, their default values in etcd—1000 ms for `--election-timeout` and 100 ms for `--heartbeat-interval`—will be applied. Once the optimization process starts, these parameters are dynamically adjusted by Dynatune to their optimal values based on real-time network conditions.

---

## Prerequisites

Dynatune shares the same prerequisites as [etcd v3.5](https://etcd.io/docs/v3.5/). Please ensure the following requirements are met:

- **Operating System**: Linux or macOS (Ubuntu 20.04+ recommended)
- **Go**: Version 1.16 or later (1.19+ recommended)
- **Ports**: Ensure that the necessary ports are open and accessible:
  - **2379**: Client requests
  - **2380**: Peer communication
  - **2381**: UDP-based heartbeat communication (added by Dynatune)

For more details, refer to the [etcd documentation](https://etcd.io/docs/v3.5/).

---

## Quick Start

To try Dynatune, follow these steps:

1. **Clone the repository**:
    ```bash
    git clone https://github.com/distsys-lab/dynatune.git
    cd dynatune
    ```

2. **Build the project**:
    ```bash
    make build
    ```

3. **Run Dynatune (example)**:
    ```bash
    ./dynatune       --name node1       --data-dir /path/to/data-dir       --listen-peer-urls http://0.0.0.0:2380       --listen-client-urls http://0.0.0.0:2379       --advertise-client-urls http://<node1-ip>:2379       --initial-cluster node1=http://<node1-ip>:2380,node2=http://<node2-ip>:2380,node3=http://<node3-ip>:2380       --initial-cluster-state new       --initial-cluster-token dynatune-cluster       --election-timeout 1000       --heartbeat-interval 100       --listen-peer-udp-url 0.0.0.0:2381       --max-election-metrics-capacity 1000       --min-election-metrics-capacity 10       --election-safety-factor 2       --heartbeat-reachability-goal 0.99       --log-level debug
    ```

---

## Support and Questions

If you have any questions or encounter issues while using Dynatune, please do the following:
- If the README does not answer your question, [open an issue](https://github.com/distsys-lab/dynatune/issues) on the GitHub repository.
- Alternatively, you can contact the committers directly via email.

---

## License

Dynatune is licensed under the [Apache 2.0 License](LICENSE), the same as etcd. See the LICENSE file for more details.

---
