---
name: test-data-selection
description: Select and validate safe synthetic CIAM test identities for automated testing. Use when a test requires a customer identity matching a scenario, account status, MFA state, or other test-data constraint.
argument-hint: "[scenario] [optional constraints]"
---

# Test Data Selection

## Purpose

Select a suitable synthetic CIAM test identity for an automated test.

## Procedure

1. Read the project's CLAUDE.md before selecting test data.
2. Use only synthetic test data.
3. Never request or use production customer data.
4. Identify the exact test requirements:
   - scenario
   - account status
   - MFA state
   - any other explicitly required attributes
5. Use the approved test-data capability to retrieve matching identities.
6. Validate that the returned identity actually satisfies the requested conditions.
7. Return only the minimum information required by the test.
8. Prefer synthetic identities using `.example.test` domains.
9. Do not modify application source code.
10. Do not modify the test-data source.

## Output

Return:

- customerId
- scenario
- status
- MFA state

Do not return unnecessary personal information.

## Safety

If no matching test identity exists, report that clearly.

Do not invent a customer ID.

If the requested test data could involve production data, stop and request clarification.