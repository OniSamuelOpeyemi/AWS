# Shell Script for AWS IAM Management (CloudOps Solutions)

## Project Scenario
CloudOps Solutions is a growing company that recently adopted AWS to manage its cloud infrastructure. As the company scales, they have decided to automate the process of managing AWS Identity and Access Management (IAM) resources. This includes the creation of users, user groups, and the assignment of permissions for new hires, especially for their DevOps team.

## Purpose
This script creates functions inside the [`aws-iam-manger.sh`](./aws-iam-manager.sh) script to fulfill the objectives below. **Ensure that you have already configured AWS CLI in your terminal and the configured AWS Account has the appropriate permissions to manage IAM resources.** 


## Objectives
1. Define IAM User Names Array to store the names of the five IAM users in an array for easy iteration during user creation.  
2. Create the IAM Users as you iterate through the array using AWS CLI commands.  
3. Define and call a function to create an IAM group named `admin` using AWS CLI commands.  
4. Attach an AWS-managed administrative policy (`AdministratorAccess`) to the `admin` group to grant administrative privileges.  
5. Iterate through the array of IAM user names and assign each user to the `admin` group using AWS CLI commands.

## Deliverables
1. Comprehensive documentation detailing the approach and decision-making (this file).  
2. Link to the extended script file (downloadable below).

---

## Files Provided
- `aws-iam-manager.sh` — The executable Bash script that implements objectives 1–5.
- This documentation file explaining the approach, assumptions, testing plan, and cleanup steps.

You can download the script directly:
- [Download aws-iam-manager.sh](https://github.com/OniSamuelOpeyemi/AWS/blob/patch1/IAM/aws-iam-manager.sh)

---

## Prerequisites
- AWS CLI installed and configured (`aws configure`).  
- AWS credentials that have the following IAM permissions at minimum:
  - `iam:CreateUser`
  - `iam:CreateGroup`
  - `iam:AttachGroupPolicy`
  - `iam:AddUserToGroup`
  - `iam:GetUser`
  - `iam:GetGroup`
  - `sts:GetCallerIdentity` (recommended to verify configuration)

- Optional: Set environment variable `DRY_RUN=true` to preview commands without making changes.

---

## How the Script Works (Step-by-step)
1. **Configuration**: The script defines a bash array `IAM_USER_NAMES` with five sample users: `samuel, grace, david, amara, chike`. Change these names to match your organization.
2. **Helper utilities**: `log`, `warn`, `err` provide consistent output. `run_cmd` executes commands or prints them if `DRY_RUN=true`.
3. **Pre-checks**: `check_aws_cli` ensures AWS CLI is available. The script attempts `aws sts get-caller-identity` to confirm credentials are configured (warns but does not exit so you can see more helpful messages).
4. **create_iam_users**: Loops through the array and uses `aws iam get-user` to check existence, and `aws iam create-user` to create missing users. Prints success/warning messages.
5. **create_admin_group**: Checks if the `admin` group exists using `aws iam get-group`. If missing, creates it and attaches the AWS-managed `AdministratorAccess` policy using `aws iam attach-group-policy`.
6. **add_users_to_admin_group**: For each user, ensures the user exists and then runs `aws iam add-user-to-group` to add the user into the `admin` group. The function handles failures gracefully (likely due to the user already being a member).
7. **Idempotency**: Each step checks for existing resources before attempting to create them. This makes the script safe to re-run without duplicating resources.
8. **Dry run**: Set `DRY_RUN=true` in the environment to print actions instead of running them:
   ```bash
   DRY_RUN=true ./aws-iam-manager.sh
   ```

---

## How to Run (example)
1. Make the script executable:
```bash
chmod +x aws-iam-manager.sh
```

2. Optional: Preview changes with `DRY_RUN`:
```bash
DRY_RUN=true ./aws-iam-manager.sh
```

3. Run for real (ensure `aws configure` has been done):
```bash
./aws-iam-manager.sh
```

---

## Expected Output (example)
```
==================================
 AWS IAM Management Script
==================================

[INFO] Starting IAM user creation process...
[INFO] Creating IAM user: Jackson ...
[INFO] Created user: Jackson
...
[INFO] Creating admin group and attaching policy...
[INFO] Group 'admin' created.
[INFO] AdministratorAccess policy attached (or already attached).
...
[INFO] Adding users to admin group...
[INFO] Added Jackson to admin.
...
==================================
 AWS IAM Management Completed
==================================
```

---

## Thought Process and Design Decisions
**Goals amd constraints**
- Keep the script **simple to read**, **idempotent**, and **safe** for repeated runs.  
- Minimize hard failures: prefer informative warnings where an operation might already have been performed.  
- Avoid creating unnecessary resources inadvertently (hence the `DRY_RUN` option).

**Why functions and arrays?**
- Arrays allow working with lists (users) concisely and make iteration straightforward.  
- Functions make the operations modular and testable, and they separate concerns (user creation vs. group management).

**Idempotency strategy**
- Use AWS `get-*` calls to determine whether a resource exists before creating it. This prevents duplicate-creation errors. When attaching policies, the attach operation is safe to run repeatedly (AWS will not duplicate attachments).

**Error handling**
- Use return codes and conditional messages instead of unconditional `set -e`. This gives more informative output when part of the script fails (e.g., one user fails while others succeed).  
- `DRY_RUN` mode prevents destructive actions during validation/testing.

**Permissions & safety**
- The script assumes a user with sufficient IAM privileges; running with least-privilege is recommended in production. Prefer testing in a sandbox account or an AWS Organization OU dedicated for development.

---
## Testing Plan
- **Dry-run** first to ensure commands look correct.  
- **Run in sandbox account** to verify behavior and inspect created resources in IAM console.  
- Use `aws iam list-users`, `aws iam list-groups`, and `aws iam get-group --group-name admin` to confirm results.

---

## Cleanup / Teardown (manual commands)
If you want to remove created users and group (be careful — ensure no attached access keys or resources depend on these users), perform the following in order:
```bash
# Remove users from group
for u in Jackson Mary John grace chike; do
  aws iam remove-user-from-group --user-name "$u" --group-name admin || true
done

# Detach policy
aws iam detach-group-policy --group-name admin --policy-arn arn:aws:iam::aws:policy/AdministratorAccess || true

# Delete group
aws iam delete-group --group-name admin || true

# Delete users
for u in Jackson Mary John grace chike; do
  # Remove access keys, login profile etc. if present before deleting
  aws iam delete-user --user-name "$u" || true
done
```

---

## Notes and Next Enhancements
- Create access keys or console login profiles for users when required (with care, and by storing secrets safely).  
- Add logging to a file and/or CloudWatch.  
- Accept user list from a CSV or external config for larger-scale onboarding.  
- Integrate with an identity provider (IdP) or AWS SSO for long-term enterprise management.

---

**Author:** Samuel Oni  
**Date:** October 2025
