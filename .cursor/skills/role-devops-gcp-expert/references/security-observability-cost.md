# Security, Observability, and Cost

## Security

- Security Command Center (SCC):
  - Enable at org scope.
  - Use Standard or Premium based on detection and compliance depth.
  - Route active findings to SIEM for centralized triage.
- Cloud Armor:
  - Attach WAF policies to external load balancers.
  - Start rate limiting and custom rules in preview mode before enforcement.
- Cloud KMS and HSM:
  - Enforce key rotation schedules.
  - Use Cloud HSM for stronger key custody requirements.
- Secret Manager:
  - Use versioned secrets.
  - Enable audit logging.
  - Never store plaintext secrets in env files or code.

## Observability

- Cloud Logging:
  - Configure sinks for long-term retention and SIEM exports.
  - Use log exclusion filters to control cost/noise.
- Cloud Monitoring:
  - Use MQL-based alerts for advanced conditions.
  - Implement SLO burn-rate alerting for user-impact focus.
- Cloud Trace:
  - Configure sampling to balance fidelity and cost.
- Cloud Profiler:
  - Profile production services for CPU and memory bottlenecks.
- Cloud Error Reporting:
  - Track and prioritize crash-class issues.

## Cost Optimization

- Committed Use Discounts:
  - Align commitments with stable baseline usage for GCE and Cloud SQL.
- Billing Export:
  - Export billing data to BigQuery for detailed analysis and chargeback.
- Budgets and Alerts:
  - Create budget alerts for proactive cost control.
- Recommender API:
  - Operationalize rightsizing and idle resource recommendations.
- Cost governance:
  - Enforce cost allocation labels across all resources.
