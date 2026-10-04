# Incident Report: Failed AWS Console Authentication

## Incident Summary

A controlled failed AWS Management Console authentication attempt was performed against the temporary IAM user `soc-lab-user`. AWS CloudTrail recorded the event, a CloudWatch metric filter detected it, and a CloudWatch alarm sent an email notification through Amazon SNS.

## Incident Classification

- Incident ID: AWS-SOC-LAB-001
- Severity: Informational
- Status: Contained and closed
- Event type: Failed console authentication
- Affected identity: `soc-lab-user`
- Environment: AWS security monitoring lab

## Timeline

| Time (UTC) | Event |
|---|---|
| 01:37:45 | Failed console authentication occurred |
| 01:40:11 | Event became available in CloudWatch Logs |
| 01:41:22 | CloudWatch alarm entered the ALARM state |
| 01:41:22 | SNS notification action executed successfully |
| 01:46:22 | Alarm returned to OK |
| After investigation | Console access for the test user was disabled |

## Investigation Findings

- CloudTrail event: `ConsoleLogin`
- Event source: `signin.amazonaws.com`
- Identity type: `IAMUser`
- Username: `soc-lab-user`
- Result: `Failed authentication`
- MFA used: `No`
- Source IP address: Redacted
- Recorded event region: `us-east-2`
- Monitoring region: `us-east-1`

The authentication attempt failed, so no AWS console session was created and no unauthorized access occurred.

## Detection Logic

```text
{ ($.eventName = "ConsoleLogin") && ($.errorMessage = "Failed authentication") }
```

The metric filter incremented `FailedConsoleLoginCount`, causing the `SOC-Failed-Console-Login` alarm to enter the ALARM state when the value reached one.

## Containment

Console access for `soc-lab-user` was disabled after the event was investigated. The account had no permissions, groups, access keys, or permissions boundary.

## Recommendations

- Require MFA for active IAM users.
- Use least-privilege permissions.
- Monitor failed and successful console authentication events.
- Investigate unfamiliar source IP addresses and user agents.
- Disable inactive identities.
- Avoid using the root user for regular administrative work.

## Conclusion

The test demonstrated an end-to-end AWS security-monitoring workflow using CloudTrail, CloudWatch Logs, metric filters, alarms, and SNS. The event was detected and an alert was delivered approximately four minutes after the simulated login attempt.
