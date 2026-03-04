# FUTURE.md — domain_bridge

> Third-party ROS 2 package (Open Robotics, Apache 2.0) that bridges topics, services, and actions between different ROS domain IDs.
> Last updated: 2026-03-04

## Purpose
Enables ROS 2 communication across different `ROS_DOMAIN_ID` networks. Used in the AirLab multi-robot system to relay topics between isolated domain segments (e.g., on-robot domain ↔ basestation domain). This is an upstream third-party package from the ROS 2 ecosystem; modifications should be minimal. Version 0.5.0.

## Nodes

| Node | Executable | Purpose |
|------|-----------|---------|
| `domain_bridge` | `domain_bridge` | Bridges topics/services/actions per YAML config |

## Design Pattern

YAML-configured bridge. Each entity (topic/service/action) specifies source domain ID, destination domain ID, and optional QoS overrides. Runs as a standalone executable or as a ROS component.

```yaml
# Example config (examples/example_bridge_config.yaml):
topics:
  /chatter:
    type: std_msgs/msg/String
    from: 0
    to: 1
services:
  /add_two_ints:
    type: example_interfaces/srv/AddTwoInts
    from: 0
    to: 1
```

Bridge creates two `rclcpp::Context` objects (one per domain) and relays messages between them using `GenericSubscription` + `GenericPublisher` (type-erased).

## ROS Interfaces

### Publishers
Dynamically created per YAML config on the destination domain.

### Subscribers
Dynamically created per YAML config on the source domain.

### Services
Bridged bidirectionally per YAML config.

### Parameters
None (configured via YAML file argument, not ROS parameters).

## Launch Files

| File | Description |
|------|-------------|
| `launch/domain_bridge.launch.py` | Launch bridge with config file arg |

Launch args:
- `config` — path to YAML config file (required)
- `from_domain` — override from-domain ID for all entities
- `to_domain` — override to-domain ID for all entities

## Config Files

| File | Purpose |
|------|---------|
| `examples/example_bridge_config.yaml` | Reference YAML for bridge configuration |

Config schema:
```yaml
topics:
  <topic_name>:
    type: <package/msg/Type>
    from: <domain_id>
    to: <domain_id>
    qos: { reliability: reliable, durability: volatile, history: keep_last, depth: 10 }
```

## Key Dependencies

| Dependency | Usage |
|-----------|-------|
| `rclcpp`, `rclcpp_components` | Node and component infrastructure |
| `rcutils` | Utility functions |
| `zstd` (optional) | Message compression |

## Build Notes
- Standard `ament_cmake` C++ package. Can be installed from apt: `ros-<distro>-domain-bridge`.
- This is the upstream Open Robotics package — do not fork; use as-is or install from apt.
- If custom features are needed, create a wrapper package instead of modifying this repo.

## Known Issues

| Severity | Description |
|----------|-------------|
| Low | Third-party package — issues should be reported upstream at https://github.com/ros2/domain_bridge |
| Low | Version 0.5.0 may lag behind the apt-installable version for newer ROS distros |
