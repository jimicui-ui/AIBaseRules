# Compute, Storage, and Databases

## Compute Platforms

- GKE:
  - Choose Autopilot for operational simplicity.
  - Choose Standard when node-level control is required.
  - Use Node Auto Provisioning (NAP) where dynamic node sizing helps.
  - Enforce Binary Authorization in production.
  - Prefer Dataplane V2 and gVisor for stronger network and workload isolation.
- Cloud Run:
  - Use controlled VPC egress for private resources.
  - Lock down invocation with IAM `run.invoker` and service-to-service auth.
- Cloud Functions Gen 2:
  - Use Eventarc for event routing and consistent trigger patterns.

## Storage and Data Services

- Cloud Storage (GCS):
  - Enable uniform bucket-level access.
  - Set retention and lifecycle policies for compliance and cost.
- Cloud SQL:
  - Use HA configurations for production.
  - Require private IP.
  - Access with Cloud SQL Auth Proxy and prefer IAM DB authentication where supported.
- Spanner:
  - Model schemas carefully, including interleaved tables where fit.
  - Plan for global consistency and multi-region latency goals.
- Bigtable:
  - Design row keys to avoid hotspotting and preserve read/write efficiency.
- BigQuery:
  - Use partitioning and clustering for performance/cost.
  - Apply row-level security for controlled access.
- Pub/Sub:
  - Configure dead letter topics and retry policies for resilient async workflows.

## Supply Chain and Build

- Cloud Build:
  - Use private pools for stricter network and compliance requirements.
- Artifact Registry:
  - Apply cleanup policies to control storage growth.
  - Integrate vulnerability scanning and deployment controls.
