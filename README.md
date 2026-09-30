# Microsoft Entra ID Conditional Access – Country Blocking Lab

## Overview

This project documents a hands-on Microsoft Entra ID Conditional Access lab focused on geographic access control.

The goal was to configure a Conditional Access policy that blocks access from a selected country, safely test the policy using a VPN, validate the blocked authentication in Microsoft Entra sign-in logs, and correlate the event using Microsoft Defender XDR Advanced Hunting.

## Lab Objectives

- Configure a Named Location in Microsoft Entra ID
- Create a Conditional Access policy
- Test the policy in Report-only mode before enforcement
- Use a VPN to simulate authentication from a selected geographic location
- Enforce the Conditional Access policy
- Validate the blocked authentication in Entra sign-in logs
- Investigate the event using Defender XDR Advanced Hunting
- Correlate identity events using KQL

## Technologies Used

- Microsoft Entra ID
- Microsoft Entra Conditional Access
- Microsoft Defender XDR
- Advanced Hunting
- Kusto Query Language (KQL)
- Proton VPN
- Microsoft 365

## Investigation Workflow

### 1. Geographic Location Simulation

A VPN connection was used in the lab to simulate authentication originating from Mexico.

This allowed the geographic Conditional Access rule to be tested without requiring a physical connection from that location.

### 2. Conditional Access Configuration

A Named Location was created for the selected country.

A Conditional Access policy named:

`Block-Selected-Countries-Lab`

was configured to evaluate the selected user and Microsoft cloud resources.

The grant control was configured to:

**Block access**

The policy was initially placed in **Report-only** mode so its expected behavior could be evaluated without immediately enforcing the restriction.

### 3. Policy Enforcement

After validating the policy configuration, the Conditional Access policy was changed from **Report-only** to **On**.

A new authentication attempt was then performed while connected through the VPN.

Microsoft Entra successfully prevented access to the protected resource.

### 4. Sign-In Log Validation

The blocked authentication was reviewed in Microsoft Entra sign-in logs.

The event showed:

- Location: Mexico
- Application: One Outlook Web
- Conditional Access result: Failure
- Error code: `53003`
- Access control: Block

Error `53003` confirmed that access was blocked because the authentication request did not satisfy the Conditional Access policy.

## Defender XDR Advanced Hunting

The identity activity was then investigated using Microsoft Defender XDR Advanced Hunting.

The `EntraIdSignInEvents` table was queried to locate the corresponding authentication activity.

Example KQL:

```kusto
EntraIdSignInEvents
| where Timestamp > ago(24h)
| where ErrorCode == 53003
| project
    Timestamp,
    AccountUpn,
    Application,
    IPAddress,
    Country,
    City,
    ErrorCode,
    CorrelationId
| order by Timestamp desc
```

### Hunting Results

The Advanced Hunting query returned the blocked authentication events associated with the Conditional Access test.

The results confirmed:

- Authentication activity originated from the simulated Mexico location
- The affected application was One Outlook Web
- The events contained error code `53003`
- The events could be correlated using the `CorrelationId`
- The identity activity was visible in the `EntraIdSignInEvents` table

This demonstrated that the Conditional Access enforcement observed in Microsoft Entra could also be investigated through Microsoft Defender XDR Advanced Hunting.

## Key Findings

The lab demonstrated that successful authentication does not automatically guarantee access to a cloud resource. Microsoft Entra Conditional Access can evaluate additional signals, such as geographic location, before authorizing access.

The workflow demonstrated how a SOC analyst can move from:

**Policy Configuration → Authentication Test → Access Block → Sign-In Log Validation → Defender XDR Hunting**

## Security Takeaways

- Conditional Access adds an additional access-control layer beyond credentials.
- Report-only mode allows a policy to be evaluated before enforcement.
- Named Locations can be used as a condition in access-control policies.
- Entra sign-in logs provide evidence explaining why access was denied.
- Error code `53003` can indicate that access was blocked by a Conditional Access policy.
- Defender XDR Advanced Hunting provides another way to investigate and correlate Entra identity events using KQL.

## Result

**Successful security-control validation.**

The Conditional Access policy matched the simulated geographic location and prevented access to the protected Microsoft 365 resource.

The blocked authentication was validated in Microsoft Entra sign-in logs and subsequently identified through Microsoft Defender XDR Advanced Hunting.

## Skills Demonstrated

`Microsoft Entra ID` • `Conditional Access` • `Identity Security` • `Defender XDR` • `Advanced Hunting` • `KQL` • `Sign-In Log Analysis` • `Security Control Validation`

## Lab Environment

This project was completed in an authorized personal Microsoft 365 cybersecurity lab environment for educational and portfolio purposes.

Sensitive account information, tenant information, and IP addresses in the supporting evidence were sanitized before publication.
