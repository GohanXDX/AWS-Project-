# AWS VPC Peering Lab — End-to-End Project

**Author:** Akshat Yadav  
**Project Type:** AWS Cloud Networking  
**AWS Region:** Asia Pacific (Sydney) — `ap-southeast-2`  
**Main Technology:** Amazon VPC, VPC Peering, EC2, Route Tables, Security Groups, SSH, ICMP

## 1. Project Overview

This project demonstrates how to establish private network communication between two separate Amazon Virtual Private Clouds (VPCs) using AWS VPC Peering.

The objective was to create two isolated networks, deploy EC2 instances in both networks, configure routing and security rules, and verify communication using private IP addresses. The project also involved accessing private instances through a bastion host, troubleshooting connectivity issues, and testing a controlled network failure followed by recovery.

This is a cloud networking project rather than a web application. It does not require a public website or a traditional live deployment URL.

## 2. Problem Statement

Cloud resources deployed in separate VPCs are isolated by default. Even when a peering connection is created, the resources cannot communicate until the correct routes and security rules are configured.

The problem addressed by this project was:

**How can two separate AWS VPCs communicate securely through private IP addresses, and how can connectivity problems be identified and resolved?**

The project demonstrates the configuration, validation, troubleshooting, and recovery process.

## 3. Project Objectives

- Create two VPCs with non-overlapping CIDR blocks.
- Configure public and private subnets.
- Deploy a bastion host and private EC2 instances.
- Establish an active VPC Peering connection.
- Configure route tables for communication in both directions.
- Configure security groups for SSH and ICMP traffic.
- Test connectivity using private IP addresses.
- Simulate an ICMP connectivity failure and restore connectivity.
- Document the architecture, configuration, results, and lessons learned.

## 4. AWS Architecture

The project consists of two VPCs connected through a VPC Peering connection.

- **VPC A:** `10.10.0.0/16`
- **VPC B:** `10.20.0.0/16`
- **Peering connection:** `pcx-0ff23feec843a2429`

App-A is deployed in VPC A, while App-B is deployed in VPC B. Both application instances use private IP addresses. A bastion host in VPC A provides SSH access to the private instances.

### Architecture Flow

```text
Local Computer
      |
      | SSH
      v
Bastion Host
Public IP: 3.107.177.160
Private IP: 10.10.1.209
      |
      +----------------------+
      |                      |
      v                      v
    App-A                  App-B
  10.10.2.10              10.20.2.10
      |                      |
      +---- VPC Peering ------+
          Active Connection
```

## 5. AWS Resources Used

| Resource | Name / ID | Configuration |
|---|---|---|
| VPC A | `ewb-vpc-a` | `10.10.0.0/16` |
| VPC B | `ewb-vpc-b` | `10.20.0.0/16` |
| VPC A ID | `vpc-0c672984ed110d8b2` | Sydney region |
| VPC B ID | `vpc-0c7ab19ace7fc8db3` | Sydney region |
| VPC Peering | `pcx-0ff23feec843a2429` | Active |
| VPC A public subnet | `subnet-040b447146f6f509b` | `10.10.1.0/24` |
| App-A subnet | `subnet-00c11c79142d937e2` | `10.10.2.0/24` |
| VPC B additional subnet | `subnet-0405abcae8dc97d87` | `10.20.1.0/24` |
| App-B subnet | `subnet-03c48fc89472d55de` | `10.20.2.0/24` |
| App-A private IP | EC2 instance | `10.10.2.10` |
| App-B private IP | EC2 instance | `10.20.2.10` |
| Bastion public IP | EC2 instance | `3.107.177.160` |
| Bastion private IP | EC2 instance | `10.10.1.209` |

**Note:** App-B is in subnet `10.20.2.0/24`. The separate `10.20.1.0/24` subnet is not the App-B subnet.

## 6. Technologies and Components Explained

### Amazon VPC

A Virtual Private Cloud provides an isolated virtual network in AWS. Each VPC has its own IP address range and networking configuration.

### Subnets

Subnets divide the VPC address space into smaller networks. The project uses a public subnet for the bastion and private subnets for the application instances.

### Amazon EC2

EC2 provides virtual machines. The project uses one bastion host and two private application instances to test connectivity.

### VPC Peering

VPC Peering creates a private network connection between two VPCs. It allows resources to communicate using private IP addresses when routes and security rules are correctly configured.

### Route Tables

Route tables determine where traffic is sent based on its destination IP range. Routes to the peer VPC must point to the peering connection.

### Security Groups

Security groups control inbound and outbound traffic for EC2 network interfaces. The project requires appropriate SSH access and ICMP rules for ping tests.

### Bastion Host

A bastion host is an entry point used to access private instances. App-A and App-B do not require public IP addresses for administrative access through the bastion.

## 7. Implementation Process

### Step 1: Create VPC A

Created `ewb-vpc-a` with CIDR `10.10.0.0/16`.

### Step 2: Create VPC B

Created `ewb-vpc-b` with CIDR `10.20.0.0/16`.

The CIDR ranges do not overlap, which is necessary for VPC Peering.

### Step 3: Create Subnets

VPC A contains:
- Public subnet: `10.10.1.0/24`
- App-A private subnet: `10.10.2.0/24`

VPC B contains:
- Additional subnet: `10.20.1.0/24`
- App-B private subnet: `10.20.2.0/24`

### Step 4: Configure the Bastion Host

The bastion host was deployed in VPC A's public subnet. It has public IP `3.107.177.160` and private IP `10.10.1.209`.

The public route table includes a default route to the Internet Gateway. SSH access to the bastion must be allowed only from the intended administrator source.

### Step 5: Deploy App-A and App-B

App-A was deployed in VPC A with private IP `10.10.2.10`.

App-B was deployed in VPC B with private IP `10.20.2.10`.

Neither application instance has a public IPv4 address.

### Step 6: Configure Security Groups

Security-group rules were configured for the required SSH and ICMP traffic.

For secure configuration:
- Restrict SSH access to trusted sources.
- Allow SSH to private instances from the bastion as required.
- Permit ICMP traffic between the relevant private addresses for ping testing.
- Avoid unnecessarily broad inbound rules.

### Step 7: Create the VPC Peering Connection

Created `ewb-peer-connection` between VPC A and VPC B.

Peering connection ID:

`pcx-0ff23feec843a2429`

The connection was verified as Active.

### Step 8: Configure Routes in VPC A

The required route for reaching VPC B is:

- Destination: `10.20.0.0/16`
- Target: `pcx-0ff23feec843a2429`

This route must exist in the route table associated with the subnet containing App-A.

### Step 9: Configure Routes in VPC B

The required route for reaching VPC A is:

- Destination: `10.10.0.0/16`
- Target: `pcx-0ff23feec843a2429`

This route was configured in VPC B's private route table, associated with App-B's subnet.

### Step 10: Verify the Configuration

The peering status, route destinations, route targets, subnet associations, and security-group rules were checked before testing connectivity.

## 8. Route Table Configuration

The important route tables recorded in the project are:

| Route Table | ID | Required Peering Route |
|---|---|---|
| VPC A public route table | `rtb-07e2002cf4da77cac` | `10.20.0.0/16` → peering |
| App-A subnet's associated route table | `rtb-0cd6c2887c65c0203` | `10.20.0.0/16` → peering |
| VPC B private route table | `rtb-05c5e0312358ec8d0` | `10.10.0.0/16` → peering |

### Important Route Table Finding

App-A's subnet was associated with route table `rtb-0cd6c2887c65c0203`. It was not associated with the custom route table named `ewb-rt-private` (`rtb-06e48f998b729e729`).

Initially, App-A's actual associated route table contained only the local VPC route. The route to VPC B was added to the table actually associated with App-A's subnet.

This was the key routing issue discovered during troubleshooting.

## 9. Connecting to Private Instances

The bastion was accessed from Windows using SSH. The local key file was stored at:

`C:\Users\aksha\Downloads\ewb-lab.pem`

Connect to the bastion:

```bash
ssh -o IdentitiesOnly=yes -i "C:\Users\aksha\Downloads\ewb-lab.pem" ec2-user@3.107.177.160
```

Connect to App-A through the bastion:

```bash
ssh -o IdentitiesOnly=yes -o "ProxyCommand=ssh -o IdentitiesOnly=yes -i C:\Users\aksha\Downloads\ewb-lab.pem -W %h:%p ec2-user@3.107.177.160" -i "C:\Users\aksha\Downloads\ewb-lab.pem" ec2-user@10.10.2.10
```

Connect to App-B through the bastion:

```bash
ssh -o IdentitiesOnly=yes -o "ProxyCommand=ssh -o IdentitiesOnly=yes -i C:\Users\aksha\Downloads\ewb-lab.pem -W %h:%p ec2-user@3.107.177.160" -i "C:\Users\aksha\Downloads\ewb-lab.pem" ec2-user@10.20.2.10
```

These commands assume the key remains at the same local path and the instances use the `ec2-user` account.

**Security warning:** Never upload the PEM private key or any AWS credentials to GitHub.

## 10. Connectivity Testing

### App-A to App-B

Run from App-A:

```bash
ping -c 4 10.20.2.10
```

**Observed result:** 4 replies, 0% packet loss, average round-trip time approximately `0.215 ms`.

### App-B to App-A

Run from App-B:

```bash
ping -c 4 10.10.2.10
```

**Observed result:** 4 replies, 0% packet loss, average round-trip time approximately `0.178 ms`.

### SSH Access

SSH access through the bastion to both App-A and App-B was successful.

### Controlled Failure and Recovery

A controlled ICMP failure/recovery test was reported as completed.

The intended test was to temporarily block the relevant inbound ICMP Echo Request rule on App-B from App-A's private address (`10.10.2.10/32`), observe the failed ping, restore the rule, and verify that ping succeeded again.

Capture the actual screenshots of:
1. The modified ICMP rule.
2. The failed ping.
3. The restored rule.
4. The successful recovery ping.

## 11. Problems Faced and Solutions

### Problem 1: Cross-VPC Ping Was Not Working

**Problem:** The instances could not initially communicate as expected.

**Investigation:** Checked the peering status, route tables, and subnet-to-route-table associations.

**Root cause:** App-A's subnet was using route table `rtb-0cd6c2887c65c0203`, which initially did not contain the route to VPC B.

**Solution:** Added the route `10.20.0.0/16` pointing to the active peering connection in App-A's actual associated route table.

**Result:** Bidirectional ping tests later succeeded.

**Learning:** A route added to an unassociated route table does not fix traffic for a subnet using a different route table.

### Problem 2: Active Peering Did Not Automatically Enable Communication

**Problem:** An Active peering connection alone was not enough to guarantee connectivity.

**Solution:** Configured routes to the peer VPC CIDR on both sides and verified security-group rules.

**Learning:** Peering, route tables, subnet associations, and security rules must all be correct.

### Problem 3: Accessing Instances Without Public IPs

**Problem:** App-A and App-B had private IPs only.

**Solution:** Used the bastion host and SSH ProxyCommand to reach each private instance.

**Learning:** A bastion can provide an administrative access path without exposing each private instance directly to the internet.

### Problem 4: Distinguishing Routing and Security Issues

**Problem:** Ping can fail because of missing routes or because ICMP is blocked.

**Solution:** Checked routing first, then security groups and network ACLs, and tested one change at a time.

**Learning:** A controlled failure helps demonstrate how a security rule affects connectivity and how restoring the rule recovers communication.

## 12. Troubleshooting Checklist

When cross-VPC communication fails, verify:

1. The peering connection is Active.
2. Both VPC CIDR ranges are non-overlapping.
3. Each VPC has a route to the peer CIDR.
4. The route is in the table associated with the relevant subnet.
5. The route target is the correct peering connection.
6. Security groups permit the required protocol and source.
7. Network ACLs allow both request and return traffic.
8. The destination instance is running and the operating system permits the test traffic.

## 13. Evidence and Screenshots

Include actual screenshots of:

- Both VPCs with CIDRs.
- Active peering connection.
- Subnets and CIDRs.
- App-A subnet's actual route-table association.
- VPC A and VPC B route tables.
- EC2 private IPs and bastion public IP.
- Relevant security-group rules.
- SSH access to App-A and App-B.
- Successful App-A → App-B ping.
- Successful App-B → App-A ping.
- Controlled failure and recovery.
- Cleanup completion, once verified.

Use clear captions with the test and result. The screenshots must be added separately; this README does not embed them.

## 14. Suggested Repository Structure

```text
aws-vpc-peering-lab/
├── README.md
├── report/
│   └── Akshat_Yadav_EWB_VPC_Peering.pdf
├── architecture/
│   └── architecture-diagram.png
└── evidence/
    ├── 01-vpcs.png
    ├── 02-active-peering.png
    ├── 03-subnets.png
    ├── 04-route-tables.png
    ├── 05-security-groups.png
    ├── 06-app-a-to-app-b-ping.png
    ├── 07-app-b-to-app-a-ping.png
    ├── 08-controlled-failure.png
    └── 09-recovery.png
```

These filenames are examples. Replace them with the actual filenames you upload.

## 15. Cleanup

Save the final report and all screenshots before deleting resources.

Recommended cleanup sequence:

1. Terminate the lab EC2 instances.
2. Check for leftover EBS volumes and snapshots created by the lab.
3. Release any Elastic IP allocated for the lab, if present.
4. Delete the peering connection.
5. Remove routes pointing to the deleted peering connection.
6. Delete lab security groups after their network interfaces are gone.
7. Delete lab subnets and custom route tables.
8. Detach and delete the lab Internet Gateway, if present.
9. Delete VPC A and VPC B.
10. Verify that no lab resources remain and review AWS billing.

Delete only resources belonging to this lab. Do not delete the default VPC or resources from other projects. Cleanup should be marked complete only after verification.

## 16. Learning Outcomes

Through this project, I learned how to:

- Plan non-overlapping VPC CIDR ranges.
- Create subnets for public entry and private workloads.
- Establish VPC Peering between two VPCs.
- Configure routing in both directions.
- Verify subnet route-table associations.
- Configure security groups for SSH and ICMP.
- Access private EC2 instances through a bastion.
- Test private connectivity using ping.
- Diagnose routing problems and validate recovery.
- Document AWS resources and clean them up responsibly.

## 17. Conclusion

This project demonstrated that VPC Peering requires more than creating a connection. Correct route-table entries, subnet associations, security-group rules, and network ACLs are all important for successful communication.

The most important troubleshooting lesson was to verify the route table actually associated with the App-A subnet. Adding the missing route to that table resolved the recorded routing issue, after which bidirectional ping tests succeeded.

## Project Links

- **GitHub Repository:** Add the repository URL after creating it.
- **Live Project URL:** Not applicable; this is a cloud networking lab, not a public web application.
- **Project Report:** Add `Akshat_Yadav_EWB_VPC_Peering.pdf` to the `report/` folder if you want to publish it with the repository.

**Author:** Akshat Yadav  
**AWS Region:** `ap-southeast-2` — Sydney
