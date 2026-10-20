# Implementing Access Controls in Active Directory

A hands-on lab on identification, authentication, and authorization in a Windows Server environment — managing Active Directory users, groups, and computer objects, then editing the domain's Default Domain Policy to enforce a stronger password policy.

## Scenario
Access control in Active Directory depends on more than individual account permissions — it requires an organizational structure (OUs and groups) and a consistently enforced password policy applied domain-wide. This lab walked through both.

## What I did

### 1. Inspected the existing Administrator account
- Ran `whoami /user` in an elevated PowerShell session to view the current session's Security Identifier (SID)
- Ran `get-aduser -identity administrator -properties *` to pull the full AD object and its attributes, distinct from just the session's identity

### 2. Built an organizational structure
- Created a new Organizational Unit, **NetworkAdmin**, under the domain in Active Directory Users and Computers
- Created a new user inside it: **IT_Head** (First name: IT, Last name: Head), with "User must change password at next logon" unchecked and "Password never expires" checked for the test account
- Created a global security group, **TechSupport**, inside the same OU, and added **IT_Head** to it — role-based access via group membership rather than individual grants
- Created a computer object, **Computer01**, inside a separate Finance OU

### 3. Verified objects via exported reports
- Ran `get-adcomputer -filter * | out-file C:\useraccounts.txt` and opened the resulting file to confirm **Computer01** actually existed in the domain's computer list

### 4. Edited the Default Domain Policy's password settings
Via Group Policy Management → Default Domain Policy → Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy:

| Policy | Setting |
|---|---|
| Minimum Password Length | 14 characters |
| Maximum Password Age | 90 days |
| Minimum Password Age | 1 day |
| Enforce Password History | 20 passwords remembered |
| Store Passwords Using Reversible Encryption | Disabled |

### 5. Documented the applied policy
- Ran `gpresult /H C:\passwords-policy-settings.html` to generate a browsable report of the actually-applied Group Policy settings, and reviewed it to confirm the changes took effect

## Key takeaways
- `whoami /user` reports on the current session; `get-aduser` queries the full directory object — related but distinct sources of identity information
- Group membership (not individual permission grants) is the real access-control lever in AD — add/remove one group membership instead of tracking individually-granted permissions per account
- Minimum password age (1 day) specifically exists to stop users from immediately cycling a forced password change back to their old password
- The Default Domain Policy is domain-wide by nature — keep it focused on password management rather than layering in unrelated settings, and apply more granular policies elsewhere
- Verifying a change via an independently exported report (a text file, an HTML `gpresult` export) is more reliable than trusting a console view that disappears once closed

## Tools
Active Directory Users and Computers, Group Policy Management, Windows PowerShell, Windows Server (AD Domain Controller)

---
*Completed in an isolated training lab environment against a provisioned AD domain.*
