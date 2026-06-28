# Enterprise Zero-Trust & IAM Architecture Lab (AWS)

## Project Overview
This project demonstrates a production-ready Identity & Access Management (IAM) infrastructure built inside AWS. It implements core Zero Trust principles, including Least Privilege access, identity lifecycle governance, and mandatory multi-factor authentication (MFA) gating.

## 🏢 Identity Architecture Design (RBAC Matrix)

| User Name | Assigned Group | AWS Managed Policy Attached | Access Level Description | Zero Trust Control Applied |
| :--- | :--- | :--- | :--- | :--- |
| **Keshav_sharma** | `Marketing-Dept-Group` | `ReadOnlyAccess` | Can view files/dashboards; cannot delete or create things. | **Least Privilege**: Prevents accidental data deletion by a new hire. |
| **Aysuh_sharma** | `Marketing-Dept-Group` | `ReadOnlyAccess` | Same as Alice, but holds managerial oversight. | **Role-Based Isolation**: Group-assigned access matches his department. |
| **Lucky_sharma** | `Contractor-External-Group` | `SecurityAudit` | Can read configuration settings but cannot view actual company data. | **Data Minimization**: Third-party vendors cannot access private data. |
| **Kshitiz_Sharma** | `IT-Admins-Privileged-Group` | `AdministratorAccess` + Custom MFA Policy | Full control over the entire cloud infrastructure. | **Conditional Deny**: Admin powers are completely frozen unless active MFA code is typed. |

## 🔐 Custom MFA Enforcement Policy (JSON)
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "BlockEverythingExceptMFASetup",
            "Effect": "Deny",
            "NotAction": [
                "iam:CreateVirtualMFADevice",
                "iam:EnableMFADevice",
                "iam:ListMFADevices",
                "iam:ListVirtualMFADevices",
                "iam:ListUsers"
            ],
            "Resource": "*",
            "Condition": {
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresent": "false"
                }
            }
        }
    ]
}
```

## 🔄 Identity Lifecycle (Joiner-Mover-Leaver Workflow)
- **Joiner**: New accounts are funneled directly into department-specific groups with baseline `ReadOnlyAccess` permissions. No direct inline policies allowed.
- **Mover**: Upon inter-departmental transfers, cross-department roles are audited. New group privileges are granted, and old group memberships are instantly revoked to avoid **Privilege Creep**.
- **Leaver**: Offboarding actions immediately flip account status to disabled, revoke active console login sessions globally, and clear group assignments.

## 📸 Lab Evidence
![AWS IAM Configuration](./screenshots/iam-mfa-policy.png)
