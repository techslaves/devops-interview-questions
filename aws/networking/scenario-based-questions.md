# AWS networking Scenario-Based Questions

Welcome to the **AWS networking Scenario-Based Questions** repository!  
This collection is designed to help engineers, Cloud Enginners, SREs, DevOps professionals, and Kubernetes enthusiasts prepare for **real-world use cases**, **interview questions**, and **on-the-job troubleshooting scenarios**.

---

## 🔥 Questions

<details>
<summary>You allowed inbound traffic on port 80 using a NACL, but your application isn’t responding. Why might that be ?</summary>
NACLs are stateless. You must also allow the outbound traffic on ephemeral ports for the response to be sent back.
</b></details>

<details>
<summary>You associated a custom NACL with a private subnet and now can't SSH into your instance. What's the issue?</summary>
Possibly:

NACL is blocking SSH (port 22) on inbound or outbound.
NACL lacks rules to allow return traffic due to statelessness.
</b></details>

<details>
<summary>Can two different subnets with different NACLs communicate if their routes allow it?</summary>
Yes, if the NACLs on both subnets allow the required traffic inbound and outbound, and the route tables support it.
</b></details>

<details>
<summary>How can you detect traffic blocked by a NACL?</summary>
Use VPC Flow Logs, filter for REJECT status to see blocked traffic by NACL.
</b></details>

<details>
<summary>If two VPCs are peered, what NACL considerations must be made? </summary>
Ensure NACLs in both VPCs allow inbound/outbound traffic from the CIDR range of the peer VPC.
</b></details>
