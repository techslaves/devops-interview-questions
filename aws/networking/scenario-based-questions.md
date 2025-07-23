# AWS networking Scenario-Based Questions

Welcome to the **AWS networking Scenario-Based Questions** repository!  
This collection is designed to help engineers, Cloud Enginners, SREs, DevOps professionals, and Kubernetes enthusiasts prepare for **real-world use cases**, **interview questions**, and **on-the-job troubleshooting scenarios**.

---

## 🔥 Questions

<details>
<summary>You allowed inbound traffic on port 80 using a NACL, but your application isn’t responding. Why might that be ?</summary>
NACLs are stateless. You must also allow the outbound traffic on ephemeral ports for the response to be sent back.
</b></details>
