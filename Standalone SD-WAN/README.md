# SD-WAN Underlay Only Configuration

This configuration is designed for a branch location that seeks to implement WAN edge intelligence for controlling how outbound traffic exits the local area network (LAN) when destined for the public internet. The setup focuses on using SD-WAN underlay features to manage a primary and backup WAN link scenario. 

In this example:
- The primary WAN link is used exclusively unless it fails to meet the defined SLA.
- Traffic is rerouted to the backup WAN connection when the SLA threshold is not met.

For detailed information on SD-WAN rules and configurations, refer to the **SD-WAN Rules** chapter in the FortiGate Administrator Guide.

## Assumptions

The following parameters are specific to this demonstration and should be customized to match your environment:

1. **WAN Interfaces**:
   - `port1` and `port2` are used for WAN1 and WAN2, respectively. Replace these with the ports that correspond to your WAN connections.

2. **LAN Subnet**:
   - This branch uses the subnet `10.1.0.0/24`. Update the `"Branch-NET"` object to reflect your local LAN subnet.

3. **Performance SLA**:
   - A health-check server is configured to monitor SLA compliance. Adjust the settings to reflect your specific traffic requirements and performance goals. For more details, see the **Performance SLA** chapter in the FortiGate Administrator Guide.

## Topology

### Underlay
This configuration applies SD-WAN underlay intelligence at the branch level, focusing on WAN link health and traffic routing without an overlay or additional SD-WAN extensions.

## Key Features

- **Primary and Backup WAN Links**:
  - The configuration prioritizes the primary link for all traffic.
  - If the primary link fails to meet the SLA criteria, traffic is seamlessly redirected to the backup link.

- **Dynamic Path Monitoring**:
  - A health-check server continuously evaluates link performance based on predefined SLA metrics.

- **Customizable Rules**:
  - FortiGate provides flexibility to define additional rules and conditions for managing traffic flow.

## How to Use

1. **Prepare Your Environment**:
   - Identify the interfaces for WAN1 and WAN2 and update the configuration file accordingly.
   - Set the correct LAN subnet for your branch.

2. **Define Performance SLA**:
   - Adjust the health-check server and SLA parameters to match your performance requirements.

3. **Apply the Configuration**:
   - Install the updated configuration to your FortiGate device.

4. **Monitor and Optimize**:
   - Use FortiGate's monitoring tools to verify traffic flow and ensure SLA compliance.

For further customization and advanced setups, consult the official Fortinet documentation.
