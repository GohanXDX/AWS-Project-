# AWS VPC Peering Lab

**Author:** Akshat Yadav\
**AWS Region:** Asia Pacific (Sydney) --- `ap-southeast-2`\
**Project Type:** AWS cloud networking lab\
**Status:** Connectivity and controlled failure/recovery tests
completed; verify cleanup status before submission.

## Project Overview

This project demonstrates private network connectivity between two
Amazon Virtual Private Clouds (VPCs) using **VPC Peering**. The lab
includes subnet planning, route-table configuration, security-group
rules, SSH access through a bastion host, bidirectional ICMP testing,
and a controlled connectivity failure followed by recovery.

This is a networking lab, not a web application. A public website/live
demo URL is therefore not applicable.

## Objectives

-   Create two separate VPCs with non-overlapping CIDR ranges.
-   Configure subnets and route tables.
-   Establish and verify an active VPC peering connection.
-   Configure routes in both VPCs for private cross-VPC communication.
-   Access private EC2 instances through a bastion host.
-   Test connectivity in both directions.
-   Demonstrate a controlled ICMP failure and restore connectivity.
-   Document the setup and clean up lab resources after preserving
    evidence.

## Architecture and Resources

**Region:** `ap-southeast-2` (Sydney)

  ----------------------------------------------------------------------------
  Resource                Name / ID                    CIDR / Address
  ----------------------- ---------------------------- -----------------------
  VPC A                   `ewb-vpc-a` ---              `10.10.0.0/16`
                          `vpc-0c672984ed110d8b2`      

  VPC B                   `ewb-vpc-b` ---              `10.20.0.0/16`
                          `vpc-0c7ab19ace7fc8db3`      

  VPC Peering             `ewb-peer-connection` ---    Active
                          `pcx-0ff23feec843a2429`      

  VPC A public subnet     `ewb-public-subnet-1` ---    `10.10.1.0/24`
                          `subnet-040b447146f6f509b`   

  VPC A App-A subnet      `ewb-private-subnet-1` ---   `10.10.2.0/24`
                          `subnet-00c11c79142d937e2`   

  VPC B additional subnet `ewb-public-subnet-1` ---    `10.20.1.0/24`
                          `subnet-0405abcae8dc97d87`   

  VPC B App-B subnet      `ewb-private-subnet-1` ---   `10.20.2.0/24`
                          `subnet-03c48fc89472d55de`   

  App-A private IP        EC2 instance in VPC A        `10.10.2.10`

  App-B private IP        EC2 instance in VPC B        `10.20.2.10`

  Bastion host            EC2 instance in VPC A public Private IP
                          subnet                       `10.10.1.209`
  ----------------------------------------------------------------------------

> **Subnet note:** The App-B instance is in `10.20.2.0/24`. The
> `10.20.1.0/24` subnet is a separate subnet in VPC B. Subnet names are
> labels; confirm the CIDR and route-table association when reviewing
> the AWS console.

## Routing Configuration

VPC peering requires routes on both sides. The following routes were
configured for the lab:

-   **App-A subnet's associated route table** (`rtb-0cd6c2887c65c0203`):
    `10.20.0.0/16` → VPC peering connection `pcx-0ff23feec843a2429`.
-   **VPC A public route table** (`rtb-07e2002cf4da77cac`):
    `10.20.0.0/16` → VPC peering connection `pcx-0ff23feec843a2429`.
-   **VPC B private route table** (`rtb-05c5e0312358ec8d0`):
    `10.10.0.0/16` → VPC peering connection `pcx-0ff23feec843a2429`.

The VPC-local routes remain in place. The App-A subnet's actual
route-table association was important during troubleshooting: the route
to VPC B had to be added to the route table actually associated with the
subnet.

## Security and Access

-   App-A and App-B use private IP addresses and are not accessed
    directly from the public internet.
-   SSH access to the private instances is performed through the bastion
    host.
-   Security-group rules must allow SSH from the intended source and
    ICMP Echo Request traffic from the peer VPC where required by the
    test.
-   Network ACLs and security groups are separate controls; both should
    be checked if connectivity fails.
-   **Never commit `.pem` private keys, passwords, access keys, tokens,
    or other credentials to GitHub.**

## Validation and Results

The following results were reported during the lab:

  ------------------------------------------------------------------------
  Test                    Command / Method         Result
  ----------------------- ------------------------ -----------------------
  Bastion access          SSH to bastion public IP Successful

  Bastion → App-A         SSH through bastion to   Successful
                          `10.10.2.10`             

  Bastion → App-B         SSH through bastion to   Successful
                          `10.20.2.10`             

  App-A → App-B           `ping -c 4 10.20.2.10`   4 replies, 0% packet
                                                   loss

  App-B → App-A           `ping -c 4 10.10.2.10`   4 replies, 0% packet
                                                   loss

  Controlled failure      Temporarily remove/block Reported completed;
                          the relevant App-B       include failure
                          inbound ICMP rule        screenshot

  Recovery                Restore the ICMP rule    Reported completed;
                          and repeat the ping      include recovery
                                                   screenshot
  ------------------------------------------------------------------------

The exact screenshot evidence should be included in the final project
report. The ping tests prove private IP connectivity in both directions;
a self-ping only proves local interface reachability and does not, by
itself, prove VPC peering.

## Evidence Checklist

Add screenshots to the report or place them in an `evidence/` folder
before pushing to GitHub. Use clear captions showing the resource/test
and result.

-   [ ] Both VPCs and their CIDR blocks
-   [ ] Active VPC peering connection
-   [ ] Subnets and CIDR blocks
-   [ ] App-A subnet's associated route table and peering route
-   [ ] VPC B private route table and peering route
-   [ ] VPC A public route table and peering route
-   [ ] EC2 instance private IPs and bastion public IP
-   [ ] Relevant security-group inbound rules
-   [ ] App-A → App-B ping success
-   [ ] App-B → App-A ping success
-   [ ] Controlled ICMP failure
-   [ ] Recovery ping success
-   [ ] Cleanup evidence, after cleanup is complete

## Suggested Repository Structure

``` text
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
    ├── 09-recovery.png
    └── 10-cleanup.png
```

The filenames above are suggestions. Replace them with the actual
evidence files you capture. Do not add empty placeholder images.

## Troubleshooting Notes

When cross-VPC ping did not work initially, the route table actually
associated with the App-A subnet was checked. The route `10.20.0.0/16`
to the active peering connection was added to that associated route
table. For similar problems, verify:

1.  The peering connection is **Active**.
2.  Each VPC has a route to the other VPC's CIDR through the peering
    connection.
3.  Routes are configured in the route tables associated with the
    relevant subnets.
4.  Security groups allow the intended ICMP traffic in both directions.
5.  Network ACL rules permit the traffic and return traffic.
6.  The VPC CIDR ranges do not overlap.

## Cleanup

After saving all screenshots and the final report, remove only the
resources created for this lab if they are no longer needed. Before
deleting anything, confirm that the resource belongs to this project and
not another workload.

Suggested cleanup sequence:

1.  Preserve and verify the report and evidence.
2.  Terminate the lab EC2 instances.
3.  Check for and remove leftover EBS volumes and snapshots created by
    the lab.
4.  Release any Elastic IP allocated for the lab, if present.
5.  Delete the VPC peering connection and remove associated peering
    routes.
6.  After network interfaces are gone, delete lab security groups.
7.  Delete lab subnets and custom route tables.
8.  Detach and delete the lab internet gateway, if present.
9.  Delete the two lab VPCs.
10. Verify that no lab resources remain and review AWS billing.

Do **not** delete the default VPC or resources belonging to other
projects. Do not perform cleanup until all required evidence has been
saved.

## Learning Outcomes

-   Understanding VPC CIDR planning and subnetting.
-   Configuring VPC peering between separate VPCs.
-   Understanding route-table associations and bidirectional routing.
-   Applying security-group rules for SSH and ICMP.
-   Using a bastion host to access private EC2 instances.
-   Testing, troubleshooting, and recovering private network
    connectivity.
-   Documenting and cleaning up cloud resources responsibly.

## Project Report

The project report is maintained separately as
`Akshat_Yadav_EWB_VPC_Peering.pdf`. Add the PDF to the `report/` folder
before publishing the repository if you want it available alongside this
README.

------------------------------------------------------------------------

**Author:** Akshat Yadav\
**Project:** AWS VPC Peering Lab\
**Region:** `ap-southeast-2` (Sydney)
#   A W S - P r o j e c t -  
 