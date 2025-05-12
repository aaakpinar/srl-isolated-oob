# Isolated OOB Management Network Lab

## Overview

This lab demonstrates a secure out-of-band (OOB) management network architecture using Nokia SR Linux switches. The design implements multiple layers of security to isolate BMC/management traffic while allowing access to a management server.

## Architecture

```
                              Management Server
                              az1-mgmt
                              10.1.0.2/30 (eth1)
                                    |
                                    |
                              10.1.0.1/30 (e1-3)
                               SPINE (az1-spine)
                             RouterID: 10.0.0.1
                          10.0.1.0/31 (e1-1) | (e1-2) 10.0.2.0/31
                          /                             \
                         /                               \
                10.0.1.1/31 (e1-1)                 10.0.2.1/31 (e1-1)
                 LEAF1 (az1-leaf1)                  LEAF2 (az1-leaf2)
              RouterID: 10.0.0.2                  RouterID: 10.0.0.3
              IRB: 10.1.1.1/24                    IRB: 10.1.2.1/24
                    |                                   |
         +----------+----------+             +----------+----------+
         |                     |             |                     |
     (e1-2)                (e1-3)       (e1-2)                (e1-3)
    az1-srv1              az1-srv2      az1-srv3             az1-srv4
  10.1.1.11/24           10.1.1.12/24  10.1.2.11/24        10.1.2.12/24
   (eth1)                 (eth1)        (eth1)               (eth1)
```

## Network Design Overview

### 1. Network Instances (VRFs)
- **Loop-free VRF Design**: MAC-VRFs (L2) local to node and IP-VRF for inter-node traffic. 

### 2. Defense in Depth
- **Layer 2 Isolation**: Split-horizon groups prevent direct BMC server communication in MAC-VRF
- **Layer 3 Filtering**: ACLs on IRBs to control traffic flow to/from management server only 

### 3. Routing
- **OSPF Backbone**: Dynamic routing between spine and leaf switches

## Configuration Components

### Network Instances
- **oob-vrf**: IP routing for management network (OSPF)
- **bmc-net**: A local MAC-VRF for BMC interface access 

### OSPF Configuration
- Area: 0.0.0.0 (backbone)
- Authentication: MD5 via keychain
- Interface Types:
  - Point-to-point: Spine-leaf links
  - Broadcast: Spine's management server interface and IRB in Leaf and  (passive)

### ACL Rules
Applied on leaf switches' IRB interfaces:
- Allow all traffic to management server
- Deny all other traffic by default

### Split-Horizon Groups
- Group name: "OOB"
- Applied to: Server-facing interfaces on leaf
- Purpose: Prevent inter-server communication at Layer 2

## Low-level Topology Details


## Deployment Steps

1. **Start the lab**:
   ```bash
   containerlab deploy -t isolated-oob.clab.yml
   ```

2. **Verify OSPF adjacencies**:
   ```bash
   # On spine
   show network-instance oob-vrf protocols ospf neighbor
   ```

3. **Check routing tables**:
   ```bash
   # On all routers
   show network-instance oob-vrf route-table
   ```

4. **Test connectivity**:
   ```bash
   # From servers to management server
   ping 10.1.0.2
   
   ```

## Security Verification

### Test Traffic Isolation
```bash
# From srv1, try to reach srv2 (same leaf)
ping 10.1.1.12  # Should fail (via SHG)

# From srv1, try to reach srv3 (different leaf)
ping 10.1.2.11  # Should fail (via ACLs)
```

## Operational Commands

### OSPF Operations
```bash
# Show OSPF database
show network-instance oob-vrf protocols ospf database
```

### ACL Monitoring
```bash
# View ACL statistics
show acl acl-filter protect-mgmt type ipv4

```

### Troubleshooting
```bash
# Check interface status
show interface ethernet-1/1

# Verify MAC learning
show network-instance bmc-net bridge-table mac-table all

```

## References

- [Nokia SR Linux Documentation](https://documentation.nokia.com/srlinux/)
- [Containerlab Documentation](https://containerlab.dev/)