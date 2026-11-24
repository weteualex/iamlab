# Day 2 — Authentication in Depth

**Date:** 2026-11-24
**Time spent:** ~2 hours
**Topics:** Authentication factors, MFA strength, password storage, NIST password guidance, intro to SSO

---

## 🎯 Goals for Today
- [x] Understand the difference between authentication and authorization
- [x] Learn the authentication factor categories and what makes MFA "multi-factor"
- [x] Rank MFA methods by strength and understand phishing resistance
- [x] Understand how passwords should be stored (salting + hashing)
- [x] Learn current NIST SP 800-63B password guidance
- [x] Hands-on: set up an Okta developer org and enforce MFA on a test user

---

## 📘 Key Concepts

### Authentication vs. Authorization
| | Authentication (AuthN) | Authorization (AuthZ) |
|---|---|---|
| Question | *Who are you?* | *What are you allowed to do?* |
| Happens | First | After AuthN succeeds |
| Ticket example | "My password won't work" | "I can log in but can't open the finance share" |

### Authentication Factors
| Factor | Examples |
|---|---|
| Something you **know** | Password, PIN, security question |
| Something you **have** | Authenticator app, hardware key (YubiKey), smart card |
| Something you **are** | Fingerprint, face, other biometrics |

Location and behavior (typing patterns, for example) are usually signals for risk scoring, not true factors.

> **Key takeaway:** MFA means two or more *different categories*, not two steps. A password plus a security question is still single-factor, because both are "something you know."

### MFA Methods, Weakest to Strongest
1. **SMS codes** — vulnerable to SIM swapping and interception
2. **TOTP apps** (rotating 6-digit codes) — better, but still phishable
3. **Push notifications** — vulnerable to *MFA fatigue / prompt bombing*; number matching mitigates this
4. **FIDO2 / WebAuthn (passkeys, hardware keys)** — **phishing-resistant**, because the credential is cryptographically bound to the real site's domain

### Password Storage
- Systems should **never store plaintext passwords**.
- They store a **salted hash**, using slow one-way algorithms: **bcrypt, scrypt, Argon2**.
- **Salt** = random data added per user → identical passwords produce different hashes → defeats rainbow tables.
- At login, the system hashes the input and compares hashes. Nothing is ever "decrypted."

### NIST SP 800-63B Guidance
- ✅ Favor **length** over complexity rules
- ✅ Screen new passwords against **breached-password lists**
- ❌ **No forced periodic resets** unless there's evidence of compromise
- Why: forced rotation leads to predictable passwords like `Summer2026!`

### Preview: SSO
- **Identity Provider (IdP):** authenticates the user once (e.g., Okta, Microsoft Entra ID, Ping)
- **Service Provider (SP):** the app that trusts the IdP
- Protocols that carry the trust: **SAML, OAuth 2.0, OIDC** (covered in upcoming days)

---

## 🛠️ Hands-On Lab: Enforcing MFA in Okta

**Objective:** Create users in an Okta developer org, build an MFA policy, and verify that a user is forced to complete MFA before they can sign in.

**Result: ✅ Success.** I created new users and an MFA policy. My test user signed in, was forced to complete MFA, and then signed in successfully.

### Step 1: Created users
I created new users in the Okta admin console (**Directory → People**). A newly created user shows a status of **Password expired** until they sign in and set their own password, which is expected Okta behavior for a temporary password.

![Users in the Okta directory](images/01-users-list.png)

### Step 2: Created an MFA policy (authenticator enrollment)
I created the **kingfisher policy** ("MFA for all members") and assigned it to the **kingfisher** group.

| Authenticator | Setting |
|---|---|
| Okta Verify | **Required** |
| Password | **Required** |
| Passkey (FIDO2 WebAuthn) | Optional (prompted on every sign-in) |

Okta Verify and Password together make this true MFA: a password is *something you know*, and Okta Verify on a phone is *something you have*. The passkey option is the phishing-resistant method from today's lesson.

![Authenticator enrollment policy](images/02-authenticator-enrollment-policy.png)

### Step 3: Added a sign-on rule that requires MFA
I added a sign-on policy for the **kingfisher** group with an active rule, **kingfisher MFA**, that allows access only after MFA.

![Sign-on policy with MFA rule](images/03-sign-on-policy-mfa-rule.png)

### Step 4: Tested as the user
I signed in as the test user in a separate browser session.

**4a. Entered the username:**

![User sign-in](images/04-user-sign-in.png)

**4b. Was forced to complete MFA.** Okta prompted for a code from the Okta Verify app before granting access:

![MFA prompt from Okta Verify](images/05-mfa-okta-verify-prompt.png)

**4c. Signed in successfully** after entering the Okta Verify code, landing on the user's Okta dashboard:

![Successful sign-in to the Okta dashboard](images/06-successful-sign-in-dashboard.png)

### Authenticators in this lab
- **Used:** Password, Okta Verify
- **Available but optional:** Passkey (FIDO2 WebAuthn), the phishing-resistant option

> Email addresses in the screenshots are blurred for privacy.

---

## ✅ Quiz Result
**Score:** _ / 6

**Missed / review:**
- _fill in_

---

## 🔗 Connection to My Tech Support Work
Triage tickets as **AuthN vs. AuthZ**. "Can't log in" is usually an identity or credential problem. "Logged in but access denied" is usually a permissions problem (group membership or role).

---

## ⏭️ Next Up: Day 3
Authorization models: **RBAC, ABAC, and least privilege**
