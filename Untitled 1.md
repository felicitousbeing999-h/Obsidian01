## **Summary**

Monitoring your Google Cloud network is essential for keeping your applications healthy and performing at their best.

![](https://storage.googleapis.com/cloud-training/cls-html5-courses/T-STACKD2-B/v1.2/EN/content/assets/VPC.png)

**_________**

**Master VPC**

Remember that VPC Flow Logs are your best friend for seeing the network from the VM's perspective. To keep costs and noise down, the most critical thing is to enable logs at the subnet level and use sampling to capture exactly what you need.

![](https://storage.googleapis.com/cloud-training/cls-html5-courses/T-STACKD2-B/v1.2/EN/content/assets/Firewall%20-%2002-03-2026.png)

**_________**

**Firewall traffic**

For security, keep in mind that Firewall Rules Logging is disabled by default, so you’ll need to activate it rule-by-rule to verify if your traffic is actually being allowed or blocked as intended.

![](https://storage.googleapis.com/cloud-training/cls-html5-courses/T-STACKD2-B/v1.2/EN/content/assets/Health%20check.png)

**_________**

**Infrastructure health check**

The most important thing to look for in Load Balancer logs is the statusDetails field; it’s a lifesaver for determining if a 5XX error is coming from your backend code or a configuration issue in the load balancer itself.

![](https://storage.googleapis.com/cloud-training/cls-html5-courses/T-STACKD2-B/v1.2/EN/content/assets/NIC.png)

**_________**

**Deep-dive forensics**

When you need to get serious about security or PCI compliance, remember that Packet Mirroring lets you clone and forward actual traffic for inspection.

![](https://storage.googleapis.com/cloud-training/cls-html5-courses/T-STACKD2-B/v1.2/EN/content/assets/Forensics.png)

**_________**

**Intelligence at your fingertips**

Think of the Network Intelligence Center as your mission control. Use Connectivity Tests to proactively prevent outages before they happen, and lean on the Network Analyzer to automatically catch those sneaky misconfigurations or suboptimal routes that humans often miss.

## **Recap**

The core network observability tools in Google Cloud, including analyzing logs from VPCs, firewalls, load balancers, and Cloud NAT. We also covered how to enable Packet Mirroring for security forensics and how to leverage the Network Intelligence Center for centralized monitoring and rapid troubleshooting.