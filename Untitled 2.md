HEALTH ARCHITECTURE BRIEF (SUMMARY)

Mission
  To provide patients with secure, real-time access to their Personal Health
  Information (PHI) through a suite of modern web and mobile applications.

Technology Strategy
  We are moving from legacy monolithic systems to a fully cloud-native,
  microservices-based architecture. All new applications will be containerized
  (using Docker and Kubernetes) and deployed on a major cloud provider.

Primary Components
  The architecture relies heavily on APIs to communicate between services and
  with client applications. Data will be stored in a mix of managed SQL and
  NoSQL databases.

Key Risk
  The primary concern is protecting Personal Health Information (PHI) from
  unauthorized access, both from external attackers and internal misuse.
  A data breach would be catastrophic for our business.

Goal
  To enable developer agility while ensuring security is built-in and
  consistent across dozens of small, independent development teams.
