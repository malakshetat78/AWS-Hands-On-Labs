# AWS Hands-On Labs ☁️

Hands-on AWS lab documentation focused on cloud infrastructure, identity and access management, security, networking, and DevOps practices.

Each lab includes its objective, implementation workflow, screenshots, validation evidence, and security takeaways.

## Labs

| # | Lab | Topics |
| --- | --- | --- |
| 01 | [IAM Administrator & Billing Access](01-IAM-Administrator-Billing-Access/README.md) | IAM users, `AdministratorAccess`, account-level billing access, verification |
| 02 | [AWS IAM Users, Groups, and Access Control](Lab-02-AWS-IAM/README.md) | IAM groups, least privilege, EC2 access testing, SCP troubleshooting |

## Documentation Format

Each lab is organized into an individual folder with a `README.md` and supporting screenshots. The documentation focuses on **what was configured, why it matters, and how the result was verified**.

## Security Notes

- Credentials, access keys, and tokens must never be committed.
- Sensitive information in screenshots should be redacted before publication.
- Administrative access should be restricted and protected by MFA.
- IAM Administrator access is not equivalent to root-user access.

> These labs document controlled learning activities. Follow least-privilege and cost-monitoring practices in real AWS environments.
