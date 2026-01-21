# Sample-cross-account-deploy

Sample-cross-account-deploy is a demo repository that contains examples and guidance for performing cross-AWS-account deployments. This repository is intended for experimentation and testing only — it demonstrates patterns and sample templates that show how to deploy resources from one AWS account into another (cross-account deployment), manage permissions, and automate the deployment workflow.

> NOTE: This is a demo repo for testing cross AWS account deployment. Do not use these templates in production without reviewing and adapting them for your security and compliance requirements.

## Contents

- README.md — this file
- (Placeholder) deployment/ — sample deployment templates (CloudFormation / CDK / Terraform) and scripts
- (Placeholder) scripts/ — helper scripts to assume roles, run deployments, or perform account setup
- (Placeholder) docs/ — additional documentation and diagrams

If you already have templates or scripts in this repo, replace the placeholders above with the actual files and paths.

## What this repo demonstrates

- How to structure a cross-account deployment demo
- Sample patterns for granting deployment roles across accounts (IAM role trust policies)
- Example CI/CD flow that assumes a role in a target account and deploys infrastructure there
- Tips for securely delegating deployment permissions and minimizing privileges

## Prerequisites

- An AWS account for the "management" or "pipeline" side (the account performing deployments).
- One or more AWS target accounts where resources will be deployed.
- AWS CLI configured with credentials for the management account (or CI credentials).
- Optional: an AWS CodePipeline/CodeBuild or other CI system configured to run deployment steps.

## Quickstart (high-level)

1. Review the sample templates/scripts in this repository (e.g., deployment/ and scripts/).
2. In the target account, create an IAM role for cross-account deployments with a trust policy that allows the management account to assume it.
3. Grant the role the minimal permissions required to create the target resources.
4. From the management account (or CI), use an assume-role call to obtain temporary credentials for the deployment role.
5. Run deployment commands (CloudFormation deploy, Terraform apply, CDK deploy) using the assumed-role credentials.
6. Verify resources in the target account.

Important: For a real environment, follow the principle of least privilege, enable logging (CloudTrail), and require MFA for sensitive operations.

## Example IAM trust policy (illustrative)
Replace ACCOUNT_ID with your management account ID and adjust principals and conditions to your security policy.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_ID:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {}
    }
  ]
}
```

## Security considerations

- Do not embed long-lived credentials in repository files.
- Use short-lived assumed-role credentials for deployments.
- Restrict IAM roles to the minimal set of actions and resources required.
- Review CloudTrail logs and AWS Config for cross-account activity.

## Contributing

This repository is a demo. If you want to add a sample template or script, please:
1. Add the file under an appropriate directory (e.g., `deployment/` or `scripts/`).
2. Update this README to document the added content and provide usage instructions.
3. Open a pull request with a description of what the sample demonstrates.

## License

Add a license file (e.g., `LICENSE`) if you want to open source these examples. For demo/internal use, document any internal policies that apply.

## Contact / Questions

If you want me to:
- tailor this README to actual templates in the repo, or
- push this README update to the repository,

tell me which you'd like and I will proceed.