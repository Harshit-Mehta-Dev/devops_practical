# DevOps: Practice Questions and Solutions

This document focuses on practical DevOps scenarios involving CI/CD pipelines, Infrastructure as Code, and version control.

---

## Scenario 1: Fixing a Broken CI Pipeline
**Question:**
A developer merges code into the `main` branch, triggering an automated CI pipeline. The pipeline fails during the unit testing stage. As a DevOps engineer, what steps do you take to investigate and resolve the issue without halting other developers' work?

**Solution:**
When a pipeline breaks on the main branch, resolving it quickly is a priority.
1.  **Investigate the Logs:** Open the CI/CD tool (e.g., Jenkins, GitHub Actions, GitLab CI) and read the specific job logs that failed. Look for the exact error message or stack trace in the unit test output.
2.  **Identify the Culprit Commit:** Check the Git history to see which commit triggered the pipeline. 
3.  **Reproduce Locally:** Pull the `main` branch to your local machine and run the unit tests locally to confirm the failure.
4.  **Revert or Fix Forward:**
    *   **Revert:** If the fix is complex or unknown, immediately `git revert` the offending commit and push to `main`. This restores the pipeline to a green state so other developers are not blocked.
    *   **Fix Forward:** If the fix is a simple, obvious typo, create a quick hotfix branch, correct the test or the code, and merge it back to `main`.

---

## Scenario 2: Infrastructure Drift
**Question:**
You are managing your cloud infrastructure using Terraform. A developer manually logs into the AWS console and changes the security group rules for an EC2 instance to troubleshoot a problem, forgetting to change it back. What is this called, and how do you detect and fix it?

**Solution:**
This scenario is known as **Infrastructure Drift**—when the actual state of the infrastructure deviates from the declared state in your IaC templates.
1.  **Detection:** Run `terraform plan`. Terraform will compare the current actual state of the AWS resources against the desired state defined in your `.tf` files. The output will highlight that the security group rule differs.
2.  **Resolution:** You have two choices:
    *   **Enforce IaC:** Run `terraform apply`. This will overwrite the manual changes made by the developer in the console, bringing the infrastructure back in line with the code.
    *   **Incorporate Change:** If the manual change was actually necessary, update your `.tf` files to include the new security group rule, then run `terraform apply` to synchronize the state file.
3.  **Prevention:** Remove manual write access to the cloud console for developers and enforce that all infrastructure changes must go through the Terraform CI/CD pipeline via pull requests.

---

## Scenario 3: Zero-Downtime Deployments
**Question:**
Your team is deploying a new version of a critical web application. The deployment must happen with absolutely zero downtime for the end users. Describe a deployment strategy that achieves this.

**Solution:**
A **Blue/Green Deployment** strategy is ideal for zero-downtime releases.
1.  **Environment Setup:** Maintain two identical production environments: Blue (currently live and handling user traffic) and Green (currently idle).
2.  **Deploy to Idle:** Deploy the new version of the application (v2.0) to the Green environment.
3.  **Testing:** Run automated integration and smoke tests against the Green environment to ensure the new version functions correctly in a production-like setting.
4.  **Switch Traffic:** Once Green is verified, update the Load Balancer or DNS router to direct all user traffic from the Blue environment to the Green environment. The switch happens instantly.
5.  **Rollback:** If users report issues on Green, immediately switch the Load Balancer back to Blue. If Green is stable, Blue becomes the new idle environment for the next release.

---

## Scenario 4: Managing Secrets in CI/CD
**Question:**
Your CI/CD pipeline needs to connect to a production database to run schema migrations during deployment. How do you securely provide the database credentials to the pipeline without hardcoding them in your Git repository?

**Solution:**
Hardcoding secrets in source code is a major security vulnerability.
1.  **Use a Secrets Manager:** Store the database credentials in a secure vault like AWS Secrets Manager, HashiCorp Vault, or GitHub Repository Secrets.
2.  **Pipeline Integration:** Configure the CI/CD tool to fetch these secrets at runtime. For example, if using GitHub Actions, inject the secrets as environment variables into the specific job step:
    ```yaml
    env:
      DB_PASSWORD: ${{ secrets.PROD_DB_PASSWORD }}
    ```
3.  **Least Privilege:** Ensure the credentials provided to the pipeline only have the permissions necessary to perform the schema migrations, not full admin access.
4.  **Masking:** Ensure the CI/CD tool is configured to mask secret values in the console logs so they cannot be accidentally read by anyone viewing the pipeline output.
