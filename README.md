# AWS VPC Peering: Private Connectivity Between Two VPCs

<p align="center">
  <strong>A hands-on AWS networking project</strong><br />
  Connecting two isolated Amazon Virtual Private Clouds (VPCs) through a VPC Peering Connection and validating private IP communication.
</p>

<p align="center">
  <img alt="AWS" src="https://img.shields.io/badge/Cloud-AWS-orange?logo=amazonaws&logoColor=white" />
  <img alt="Amazon VPC" src="https://img.shields.io/badge/Networking-Amazon%20VPC-232F3E?logo=amazonaws&logoColor=white" />
  <img alt="EC2" src="https://img.shields.io/badge/Compute-EC2-FF9900?logo=amazonec2&logoColor=white" />
  <img alt="Project type" src="https://img.shields.io/badge/Type-Hands--on%20Lab-blue" />
</p>

## Project Overview

This project demonstrates how to connect two separate Amazon Virtual Private Clouds (VPCs) using **VPC Peering**. It covers the foundational network configuration required for private communication between resources in different VPCs, including subnets, route tables, security groups, EC2 instances, and connectivity validation.

The repository includes screenshots of the configuration steps, SSH access through a bastion host, connectivity checks, and resource-cleanup evidence.

> **Key learning:** A VPC peering connection does not automatically enable traffic. The peering connection must be active, routes must exist on both sides, and security controls must allow the intended traffic.

## Objectives

- Create two independent VPCs with non-overlapping IPv4 CIDR ranges.
- Configure public and private subnets.
- Configure an Internet Gateway and route tables where required.
- Launch EC2 instances for connectivity testing.
- Create and accept a VPC Peering Connection between the VPCs.
- Add routes to direct traffic through the peering connection.
- Configure security groups for controlled access.
- Connect to instances using SSH through a bastion host.
- Verify private connectivity and document cleanup.

## Architecture

The lab uses two VPCs connected by a VPC Peering Connection. A bastion host provides an SSH entry point to the environment, while private EC2 instances are used for internal access and connectivity testing.

```text
                         Administrator
                              |
                         SSH (allowed IP)
                              |
                       +----------------+
                       |  Bastion Host  |
                       |  Public Subnet |
                       +----------------+
                              |
                       Private SSH access
                              |
                  +-----------------------+
                  |         VPC A         |
                  | Public + Private      |
                  | Subnets / EC2         |
                  +-----------------------+
                              |
                     VPC Peering Connection
                       (private IP routing)
                              |
                  +-----------------------+
                  |         VPC B         |
                  | Public + Private      |
                  | Subnets / EC2         |
                  +-----------------------+
```

**Note:** This is a conceptual diagram, not a CIDR-accurate network diagram. Use the actual CIDR ranges, subnet IDs, route tables, and security-group rules from your AWS lab when reproducing the setup.

## AWS Services and Concepts

| Service / concept | Purpose in this project |
|---|---|
| Amazon VPC | Provides isolated virtual networks |
| Subnets | Separate public-facing and private resources |
| Internet Gateway | Provides internet routing for the public subnet when routes and public IP addressing allow it |
| Route tables | Direct traffic to local networks, the Internet Gateway, or the peering connection |
| VPC Peering | Provides private network connectivity between the two VPCs |
| Amazon EC2 | Hosts the bastion and test instances |
| Security Groups | Control permitted inbound and outbound instance traffic |
| SSH | Provides administrative access to Linux instances |
| ICMP / ping | Helps test IP reachability when permitted by security rules |

## Repository Structure

```text
aws-vpc-peering-project-main/
├── README.md
├── Parthiban_EWB_VPC_Peering.pdf
├── screenshots/
│   ├── 1.VPCs-a&b.png
│   ├── 2.internet_gateway.png
│   ├── 3.subnets.png
│   ├── 4.Route_table_config/
│   ├── 5.Security_groups/
│   ├── 6.ec2-instances.png
│   ├── 7.peer-a-b.png
│   ├── 8.rt-a-peer-update.png
│   ├── 9.rt-b-peer-update.png
│   ├── 10.ssh_bastion_server.png
│   ├── 11.establish_bastion_to_ec2-a.png
│   ├── 12.ping_a.png
│   └── 13.ping_b.png
└── Cleanup_evidence/
    ├── clean_ig.png
    ├── clean_instance.png
    └── clean_peering.png
```

## Implementation Walkthrough

### 1. Create two VPCs

Create VPC A and VPC B in the required AWS Region. Assign **non-overlapping CIDR ranges**, for example:

| Example network | Example CIDR |
|---|---|
| VPC A | `10.10.0.0/16` |
| VPC B | `10.20.0.0/16` |

These are example ranges, not a claim about the exact CIDRs used in the screenshots. Record your actual CIDRs before configuring routes.

Evidence: [VPCs A and B](screenshots/1.VPCs-a%26b.png)

### 2. Create subnets and configure internet access

Create the public and private subnets required by the design. Attach an Internet Gateway to the VPC that hosts the bastion host, and configure the public route table with a route to the Internet Gateway.

A subnet is public when its associated route table directs internet-bound traffic to an Internet Gateway. An instance also needs appropriate public addressing and security-group rules for direct inbound internet access.

Evidence:
- [Internet Gateway](screenshots/2.internet_gateway.png)
- [Subnets](screenshots/3.subnets.png)

### 3. Configure route tables and security groups

Associate the appropriate route tables with each subnet. Review the security groups to allow only the traffic needed by the lab.

For a typical two-VPC setup, the route table in **each VPC** needs a route to the other VPC's CIDR through the VPC Peering Connection. The exact destination CIDR and target depend on your network.

| Route table | Destination | Target |
|---|---|---|
| VPC A route table | VPC B CIDR | VPC Peering Connection |
| VPC B route table | VPC A CIDR | VPC Peering Connection |

Do not copy the example CIDRs blindly. Use the real CIDR ranges for your VPCs.

Evidence:
- [Route-table configuration](screenshots/4.Route_table_config/)
- [Security groups](screenshots/5.Security_groups/)

### 4. Launch EC2 instances

Launch the bastion and test instances in the intended subnets. Use suitable key-pair access and security groups. Keep application or test instances private when they do not need direct internet access.

Evidence: [EC2 instances](screenshots/6.ec2-instances.png)

### 5. Create the VPC Peering Connection

Create a peering connection between VPC A and VPC B, then accept it if the workflow requires acceptance by the peer owner. Confirm that its state is **Active** before testing connectivity.

Evidence: [VPC Peering Connection](screenshots/7.peer-a-b.png)

### 6. Add peering routes on both sides

Update the route table associated with the source subnets in VPC A to route traffic for VPC B through the peering connection. Add the corresponding reverse route in VPC B.

A peering connection is not transitive. It does not automatically provide a path to a third VPC or serve as a general-purpose gateway.

Evidence:
- [VPC A peering route](screenshots/8.rt-a-peer-update.png)
- [VPC B peering route](screenshots/9.rt-b-peer-update.png)

### 7. Connect through the bastion host

Use the bastion host as a controlled SSH entry point. Restrict inbound SSH (TCP 22) on the bastion to your trusted public IP address. For private instances, allow SSH from the bastion's security group where supported by your configuration.

Example command from your local terminal:

```bash
ssh -i /path/to/your-key.pem ec2-user@<BASTION_PUBLIC_IP>
```

From the bastion host, connect to a private EC2 instance using its private IP address:

```bash
ssh -i /path/to/your-key.pem ec2-user@<PRIVATE_EC2_PRIVATE_IP>
```

The username depends on the AMI. For example, Amazon Linux commonly uses `ec2-user`, while Ubuntu commonly uses `ubuntu`. Ensure the private key is available securely to the SSH client and has appropriate file permissions. Do not commit private keys to GitHub.

Evidence:
- [SSH to bastion host](screenshots/10.ssh_bastion_server.png)
- [Bastion-to-EC2 SSH access](screenshots/11.establish_bastion_to_ec2-a.png)

### 8. Verify connectivity

Test the target instance's private IP from the appropriate source instance. For example:

```bash
ping <PEER_INSTANCE_PRIVATE_IP>
```

A successful ping indicates that ICMP packets are reaching the target and replies are returning. Ping may fail even when the peering route is configured correctly if ICMP is not allowed in security groups or network ACLs, or if the target does not respond to ICMP. Use the relevant application port or a TCP test as well when appropriate.

Evidence:
- [Connectivity test A](screenshots/12.ping_a.png)
- [Connectivity test B](screenshots/13.ping_b.png)

## Validation Checklist

- [ ] Both VPCs exist and have non-overlapping CIDR ranges.
- [ ] Subnets are associated with the intended route tables.
- [ ] The VPC Peering Connection is active.
- [ ] The route table in VPC A includes a route to VPC B through the peering connection.
- [ ] The route table in VPC B includes a route to VPC A through the peering connection.
- [ ] Security groups and network ACLs allow the intended test traffic.
- [ ] SSH access to the bastion is limited to a trusted source IP.
- [ ] Private IP connectivity is verified from the correct source.
- [ ] Lab resources have been cleaned up after testing.

## Troubleshooting

| Symptom | What to check |
|---|---|
| Peering connection is pending | Accept the request from the peer VPC owner/account if required |
| Instances cannot reach each other | Check peering state, both route tables, correct CIDRs, security groups, and network ACLs |
| SSH to the bastion fails | Check public IP/DNS, public-subnet route, instance state, key pair, username, and TCP 22 rules |
| SSH to a private instance fails | Check private IP, private route tables, bastion-to-instance security-group rules, and SSH credentials |
| Ping fails but routes look correct | Confirm ICMP is allowed by security groups and network ACLs, and that the target responds to ping |
| Internet access fails from a public subnet | Check Internet Gateway attachment, the subnet's route-table association, public IP addressing, and outbound rules |

## Cleanup and Cost Awareness

AWS resources can incur charges while they exist, even after connectivity testing is complete. Review the resources created for this lab and remove them when they are no longer needed.

A safe cleanup sequence is to terminate test EC2 instances, remove the VPC Peering Connection, delete resources that depend on the VPC, detach and delete the Internet Gateway, and then delete the VPCs and remaining lab resources. Adjust the order to account for dependencies.

The repository includes cleanup evidence:
- [EC2 instance cleanup](Cleanup_evidence/clean_instance.png)
- [VPC Peering cleanup](Cleanup_evidence/clean_peering.png)
- [Internet Gateway cleanup](Cleanup_evidence/clean_ig.png)

Always verify the correct AWS account and Region before deleting anything. Do not remove resources shared with other workloads.

## Learning Outcomes

By completing this project, you can practice:

- Designing isolated VPC networks and subnet layouts.
- Using VPC Peering for private IPv4 routing between VPCs.
- Updating route tables on both sides of a network connection.
- Applying security groups to control SSH and connectivity tests.
- Using a bastion host to access private EC2 instances.
- Validating network paths and troubleshooting common connectivity issues.
- Cleaning up cloud lab resources to reduce unnecessary costs.

## Project Evidence

Browse the [screenshots directory](screenshots/) to review the implementation evidence, and the [cleanup evidence directory](Cleanup_evidence/) to review resource-cleanup screenshots.

The repository also contains [a PDF reference](Parthiban_EWB_VPC_Peering.pdf). Review it alongside the screenshots when following the lab.

## Author

**Kunal Jadhav**

- GitHub: [@devkunaljadhav](https://github.com/devkunaljadhav)
- LinkedIn: [kunaljadhav](https://www.linkedin.com/in/devkunaljadhav/)

If you find this project useful, consider giving the repository a ⭐.

---

*This is a hands-on learning project. The CIDR ranges and SSH commands shown above are examples; use the values and AMI details from your own AWS environment.*
