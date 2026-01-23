# AWS networking Questions

Welcome to the **AWS networking Questions** repository!  

1. What is a VPC?
   A logically isolated virtual network in AWS where you define IP range, subnets, routing, and security.

2. What is CIDR and why is it important in VPC?
   CIDR defines IP address ranges. It determines VPC size, subnet size, and future scalability.

3. Difference between Public and Private Subnet?

Public subnet: Route to Internet Gateway

Private subnet: No direct internet route

4. What makes a subnet public?
   A route to an Internet Gateway (IGW) in its route table.

5. What is an Internet Gateway?
   A horizontally scaled AWS-managed gateway enabling internet access for VPC resources.

6. What is a Route Table?
   Rules that decide where network traffic is directed.

7. How does routing work inside a VPC?

Local route handles VPC-internal traffic

Longest prefix match wins

8. What is a NAT Gateway?
   Allows outbound internet access for private subnet resources without allowing inbound traffic.

9. NAT Gateway vs NAT Instance?
   | NAT Gateway          | NAT Instance   |
   | -------------------- | -------------- |
   | Managed              | Self-managed   |
   | Scales automatically | Manual scaling |
   | No SSH               | SSH possible   |

10. Security Group vs NACL?
    | Security Group   | NACL         |
    | ---------------- | ------------ |
    | Stateful         | Stateless    |
    | Instance-level   | Subnet-level |
    | Allow rules only | Allow + Deny |

11. Why is SG stateful important?
    Return traffic is automatically allowed.

12. When would you use NACL deny rules?
    Blocking specific IP ranges at subnet level.

13. What is Route 53?
    AWS managed DNS service supporting public, private, and hybrid DNS.

14. What is a Hosted Zone?
    A container for DNS records.

15. Public vs Private Hosted Zone?

Public: Internet-facing

Private: VPC-internal DNS

16. What is VPC DNS Resolution?
    Allows instances to resolve DNS names using AmazonProvidedDNS.

19. What is VPC Peering?
    Private connectivity between two VPCs using AWS backbone.

20. Limitations of VPC Peering?

No transitive routing

Overlapping CIDR not allowed

21. How do you access AWS services privately?
    Using VPC Endpoints

22. Gateway vs Interface Endpoint?
    | Gateway           | Interface         |
    | ----------------- | ----------------- |
    | S3, DynamoDB      | Most AWS services |
    | Route-table based | ENI + Private IP  |

26. What is ENI?
    Elastic Network Interface – virtual NIC for EC2.

27. How does EKS networking work?
    Pods get VPC IPs via AWS VPC CNI.

28. What is MTU and why important in AWS?
    Maximum packet size (1500 default). Incorrect MTU causes packet drops.

29. How do you design multi-AZ high availability?

Subnets in multiple AZs

ALB + Auto Scaling

NAT per AZ


---