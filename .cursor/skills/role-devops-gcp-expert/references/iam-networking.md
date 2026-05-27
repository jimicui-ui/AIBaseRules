# IAM and Networking

## Resource Hierarchy and IAM

- Structure organizations into org -> folders -> projects by environment and business domain.
- Use least privilege IAM with predefined roles where possible; scope roles at the narrowest resource level.
- Enforce Organization Policy constraints:
  - Disable service account key creation.
  - Require Shielded VMs where applicable.
  - Restrict allowed regions and resource locations.

## Workload Identity

- Use Workload Identity Federation (WIF) for CI/CD with OIDC from GitHub Actions or GitLab.
- Avoid static service account keys in pipelines.
- Use Workload Identity for GKE to map Kubernetes service accounts to Google service accounts.
- Apply service account best practices:
  - One service account per workload boundary.
  - Minimal IAM role grants.
  - Short-lived credentials through federation.

## Shared VPC and Service Perimeters

- Use Shared VPC with centralized network host projects.
- Segment service projects by environment and sensitivity.
- Apply VPC Service Controls to sensitive services and datasets.

## Core Network Controls

- Enable Private Google Access for private subnets needing Google APIs.
- Use Cloud NAT for controlled outbound internet egress.
- Choose load balancers by traffic profile:
  - Global external HTTP(S) for internet apps.
  - Regional internal/external as needed for locality or private traffic.
  - Network load balancing for L4 use cases.
- Use Cloud CDN for edge caching and latency reduction.
- Configure Cloud DNS routing policies for geo/weighted/failover needs.
- Consider Traffic Director for advanced service routing and traffic policy.
