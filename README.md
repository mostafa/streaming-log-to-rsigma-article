# Streaming Logs to RSigma for Real-Time Detection

Companion repository for the blog post [Streaming Logs to RSigma for Real-Time Detection](https://mostafa.dev/streaming-logs-to-rsigma-for-real-time-detection-72084b8041ad).

This repo contains the Sigma detection rules and sample events used throughout the article. Together they demonstrate how [RSigma](https://github.com/timescale/rsigma) correlates individual Okta detections into a single critical alert, reproducing the attack chain from Okta's [August 2023 cross-tenant impersonation advisory](https://sec.okta.com/articles/2023/08/cross-tenant-impersonation-prevention-and-detection).

## What's Inside

```
rules/
  okta_user_session_start_via_anonymised_proxy.yml    # SigmaHQ – session via proxy
  okta_mfa_reset_or_deactivated.yml                   # SigmaHQ – MFA deactivated
  okta_admin_role_assigned_to_user_or_group.yml       # SigmaHQ – admin role assigned
  okta_identity_provider_created.yml                  # SigmaHQ – rogue IdP created
  okta_cross_tenant_impersonation_correlation.yml     # Custom – temporal_ordered correlation
events/
  okta_audit.ndjson                                   # Sample Okta System Log events (NDJSON)
```

The four detection rules are from [SigmaHQ](https://github.com/SigmaHQ/sigma/tree/master/rules/identity/okta) and use the native [Okta System Log](https://developer.okta.com/docs/api/openapi/okta-management/management/tag/SystemLog/) field names (camelCase), so no processing pipeline is needed. The correlation rule is custom.

## Quick Start

Install [RSigma](https://github.com/timescale/rsigma), then run:

```bash
rsigma eval -r rules/ < events/okta_audit.ndjson
```

You should see four individual detections (one per attack-chain step) and one `critical` correlation alert tying them together by actor within a 30-minute window.

## The Attack Chain

| Step | Okta Event | Sigma Rule | Level |
|------|-----------|------------|-------|
| 1 | `user.session.start` from proxy | `okta_user_session_start_via_anonymised_proxy` | high |
| 2 | `user.mfa.factor.deactivate` | `okta_mfa_reset_or_deactivated` | medium |
| 3 | `user.account.privilege.grant` | `okta_admin_role_assigned_to_user_or_group` | medium |
| 4 | `system.idp.lifecycle.create` | `okta_identity_provider_created` | medium |
| **Correlation** | All four from the same `actor.alternateId` within 30 min | `okta_cross_tenant_impersonation_correlation` | **critical** |

## License

The SigmaHQ detection rules are licensed under the [DRL](https://github.com/SigmaHQ/sigma/blob/master/LICENSE.Detection.Rules.md). Everything else in this repository is MIT.
