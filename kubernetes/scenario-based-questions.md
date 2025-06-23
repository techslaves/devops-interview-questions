# Kubernetes Scenario-Based Questions

Welcome to the **Kubernetes Scenario-Based Questions** repository!  
This collection is designed to help engineers, SREs, DevOps professionals, and Kubernetes enthusiasts prepare for **real-world use cases**, **interview questions**, and **on-the-job troubleshooting scenarios**.

---

## 📌 What's Inside

- 🔧 **Realistic Kubernetes Scenarios** — Focused on production-grade challenges.
- 🧠 **Interview-Oriented Questions** — Frequently asked in technical discussions and whiteboard rounds.
- 🚨 **Debugging & Troubleshooting** — Designed to simulate operational incidents.
- ✅ **Best Practice Solutions** — Alongside answers or hints where appropriate.

---

## 🔥 Questions

<details>
<summary>There is a guest API which is abused leading to higher system resource utilization. How can you handle this scenario to reduce impact on system ?</summary>

These steps reduce system strain while you assess and mitigate.

- Rate Limit the API
   Use API gateway (e.g., Kong, NGINX Ingress, Envoy) to throttle requests per IP or per user.

   ```
    # Kong Rate Limiting Plugin (DB-less mode)
    plugins:
      - name: rate-limiting
        config:
          minute: 60
          policy: local
    ```

- Block Known Abusers
   Use WAF (Web Application Firewall) to block IPs or IP ranges causing abuse.
   Apply firewall rules or security groups to isolate traffic temporarily.

- Introduce Authentication / CAPTCHA
  Make the API authenticated or introduce hCaptcha/Google reCAPTCHA to deter bots.

</b></details>
