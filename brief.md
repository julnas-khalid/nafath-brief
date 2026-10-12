Version: NAFATH-V2
Track: Software

## Overview

Keeping an emergency-response 5G service reliable requires coordinated work across software, artificial intelligence, mobile design, networking, cybersecurity, and data engineering. APIs must behave predictably when requests time out, preventing retries from accidentally duplicating operations. AI can forecast congestion, but its model must be tested on unseen conditions before operators trust its predictions. The responder app should remain useful without coverage and synchronize queued updates carefully when connectivity returns. Network slicing can tailor connectivity for mission-critical traffic, while incident response helps teams contain attacks and restore service. Regional daily usage reports then depend on a data model that supports accurate, consistent aggregation. These controls complement one another: a dedicated slice cannot eliminate every outage, and a predictive model cannot replace operational safeguards. The software question therefore focuses on safe and idempotent HTTP methods and retry behavior.

## Software

HTTP methods have two distinct properties. GET, HEAD, OPTIONS, and TRACE are safe because their requested effect is read-only; safe methods are also idempotent. PUT and DELETE are idempotent but not safe, while POST is generally neither. Idempotency means repeating an identical request has the same intended server effect as sending it once. After a timeout, clients may retry idempotent requests more safely; non-idempotent operations need safeguards such as idempotency keys or confirmation that the original was not applied.

Resource: RFC 9110 - HTTP Semantics - https://www.rfc-editor.org/rfc/rfc9110.html

## AI

Overfitting occurs when a congestion model learns noise or details specific to its training data and performs poorly on new network conditions. Cross-validation splits available data into training and validation folds, repeating the process so performance is assessed across different held-out subsets. A large gap between strong training results and weaker validation results can signal overfitting. For time-dependent traffic, splits should respect chronology to avoid future information leaking into training.

Resource: 3.1. Cross-validation: evaluating estimator performance - https://scikit-learn.org/stable/modules/cross_validation.html

## Mobile

The responder app should clearly show which information is current and which actions are waiting to sync. Cache essential, authorized operational information locally, store offline changes in a durable queue, and show delivery status rather than implying that an unsent message reached dispatch. When coverage returns, retry queued work with backoff, use unique operation IDs to prevent duplicates, and resolve conflicts using explicit rules and server acknowledgements. Sensitive local data should be encrypted and expire when no longer needed.

Resource: Build an offline-first app - Android Developers - https://developer.android.com/topic/architecture/data-layer/offline-first

## Networking

5G network slicing creates multiple logical networks on shared physical infrastructure, each configured for different service needs. A dedicated responder slice could prioritize dispatch messages, voice, location updates, and incident video with defined latency, reliability, and capacity targets. The operator should enforce access controls and test isolation, congestion behavior, coverage, and failover. A slice improves resource management but cannot guarantee service if shared infrastructure or power fails.

Resource: ETSI TS 123 501 - 5G; System architecture for the 5G System (5GS), clause 5.15 Network slicing - https://www.etsi.org/deliver/etsi_ts/123500_123599/123501/18.11.00_60/ts_123501v181100p.pdf

## Cybersecurity

Incident response is a continuous cycle: prepare capabilities and playbooks; detect and analyze suspicious activity; contain the threat; eradicate its cause; recover systems safely; and learn from the incident. During an attack, the operator should validate alerts, preserve evidence, isolate affected components without unnecessarily disabling emergency communications, remove malicious access, restore from trusted systems, and monitor for recurrence. Afterward, update controls and plans based on lessons learned. NIST’s current guidance integrates response across broader risk management.

Resource: NIST SP 800-61 Rev. 3 - Incident Response Recommendations and Considerations for Cybersecurity Risk Management - https://csrc.nist.gov/pubs/sp/800/61/r3/final

## Data

A star schema places a central fact table—such as usage events or data volume—beside dimensions for date and region. A snowflake schema normalizes one or more dimensions into related tables, reducing duplicated descriptive data but adding joins and complexity. For daily regional reports, a star schema is usually preferable because users can filter by region and date and aggregate measures simply. Define the fact table grain clearly, for example one record per region, date, and service metric.

Resource: Understand star schema and the importance for Power BI - Microsoft Learn - https://learn.microsoft.com/en-us/power-bi/guidance/star-schema

## Misconception in My Track

A common misconception is that “safe” and “idempotent” mean the same thing, so every idempotent request must be safe. They are different properties. Safe methods do not request a state change; idempotent methods may change state, but repeating the same request has the same intended effect as doing it once. DELETE and PUT are idempotent but are not safe. This distinction matters because automatic retries should not accidentally repeat actions with non-idempotent effects. The proof is in RFC 9110, Section 9.2.2, “Idempotent Methods”: “Of the request methods defined by this specification, PUT, DELETE, and safe request methods are idempotent.”

## Open Question

Which retry policy and idempotency-key design will prevent duplicate dispatch actions when acknowledgements are lost during a network outage?
