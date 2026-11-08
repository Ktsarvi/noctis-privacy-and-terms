# Privacy Policy — Noctis AI

**Effective Date:** November 8, 2026  
**Last Updated:** November 8, 2026  
**App Name:** Noctis AI (also referred to as Noctis / Noctis Tales)  
**Developer:** [FLAG: Legal Entity Name, e.g., Noctis AI Inc. / Noctis AI LLC]  
**Contact Email:** [privacy@noctis.ai]

---

## 1. Introduction

At Noctis AI, we believe family privacy is paramount. We create personalized bedtime stories for children using artificial intelligence. Because our service is designed for families and involves information regarding children, this Privacy Policy explains in clear, plain language:
- What data we collect and why;
- How third-party services assist in operating the app;
- How we protect children's data under COPPA and GDPR-K; and
- How you can inspect, export, or permanently delete your data at any time.

---

## 2. Who May Use the App & Parental Consent

- **Adults Only:** Accounts may only be created by individuals who are at least 18 years of age (or the legal age of majority in their jurisdiction). Noctis AI is designed for parents, legal guardians, and adult caregivers.
- **Parental Consent:** When you provide details about a child (such as their first name, age, or preferred language) to generate personalized stories, you confirm that you are the child’s parent or legal guardian and explicitly consent to our collection and processing of that information as described in this policy.

---

## 3. Information We Collect

We only collect data strictly necessary to authenticate your account, generate your personalized stories, and manage your subscription.

### A. Account & Login Data
- **Email Address & One-Time Passcodes (OTP):** When you sign in with email, we send a 6-digit OTP code to verify your identity. We do not use or store permanent account passwords. The OTP code is verified by Supabase Auth and discarded immediately.
- **Sign in with Apple:** If you choose Apple authentication, we receive an Apple user identifier, an identity token, and (only during your very first sign-in) the email address and full name you authorize Apple to share.
- **Sign in with Google:** If you choose Google authentication, we receive Google OAuth identity and access tokens to authenticate your account.
- **Session Tokens:** Secure access and refresh tokens are stored locally on your device in secure storage (AsyncStorage / SecureStore) to maintain your login session.

### B. Child & Story Preferences
To personalize bedtime stories for your child, you provide:
- **Child's Name:** Stored in your profile and used in story text and illustrations.
- **Child's Age:** (Ages 1 to 12) Used to adjust vocabulary, complexity, and themes suitable for their age group.
- **Preferred Language:** Stored in your profile to generate stories and narration in your selected language.
- **Story Customization:** Themes, morals/lessons, and settings you choose when creating a story.
- **Favorites:** Stories you flag as favorites in your library.

> *Note on Child Data:* We do **not** collect child email addresses, physical addresses, biometric identifiers, GPS locations, or school details.

### C. Generated Story Content & Media Storage
- **Story Text & Titles:** Generated stories and their titles are stored in our secure database linked to your account.
- **Illustration Images:** Cover images generated for your stories are stored in our secure cloud storage bucket (`story-images`).
- **Narration Audio (TTS):** Read-aloud text-to-speech narration files are stored in our secure cloud storage bucket (`story-audio`) so you can listen anytime.
- **Generation Timestamps & Types:** We record timestamps and generation types (`story`, `image`, `audio`) in a generation log to enforce daily and plan quotas.

### D. Subscriptions & Payments
All payments and subscription billings are processed directly through the **Apple App Store** or **Google Play Store**. 
- Noctis AI **never** receives, collects, or stores your credit card numbers, bank account numbers, or billing addresses.
- Through our purchase partner, RevenueCat, we receive purchase metadata (plan type, active/cancelled status, store receipt ID, and expiration date) to activate your features.

### E. Technical & Device Information Collected by SDKs
The RevenueCat SDK collects limited technical metadata to validate purchases and maintain app compatibility:
- Device model, operating system (iOS/Android), OS version, app version, and device locale/language.
- IP address (used to infer store region and prevent fraud).
- Advertising Identifiers (IDFA on iOS or GAID on Android): Only collected if you explicitly grant permission in your device settings. We do not display third-party advertisements.

> **[FLAG for Developer Review]:** If you do not run advertising campaigns, disable IDFA collection in your RevenueCat SDK configuration to avoid needing Apple's App Tracking Transparency (ATT) prompt.

---

## 4. How We Use Your Information

We use your data strictly to:
1. Generate personalized story text, illustrations, and audio narrations tailored to your child's age and language;
2. Maintain your personal library across your devices;
3. Enforce fair-use generation quotas according to your subscription plan (Free vs. Ultra);
4. Securely authenticate your logins without passwords; and
5. Provide customer support and troubleshoot technical issues.

**We do NOT sell, rent, or trade your personal information or your child's information to advertisers, data brokers, or third parties.**

---

## 5. Third-Party Sharing & Data Processors

To deliver Noctis AI, we share data only with the following service providers who are bound by strict confidentiality and security standards:

| Third Party | Data Received | Purpose |
|---|---|---|
| **Supabase** | User ID, email, profile details (child name, age, language), story text, generated images & audio, session tokens. | Primary database, authentication service, and cloud file storage. Protected with Row Level Security (RLS). |
| **Google Gemini API** | Child's name, child's age, story theme/moral/setting/language, image prompts, story text for audio synthesis. | AI generation of story text, cover illustrations, and speech narration. |
| **RevenueCat** | User ID, subscription status, purchase history, store receipt tokens, device metadata (OS, model, locale, IP). | In-app purchase validation, subscription status sync, and receipt processing. |
| **Apple** | Apple ID credentials (during sign-in), payment receipt tokens. | Sign in with Apple authentication and App Store in-app billing. |
| **Google** | Google OAuth credentials (during sign-in), payment receipt tokens. | Sign in with Google authentication and Google Play Store in-app billing. |

> **[FLAG for Developer Review — Google Gemini API Terms]:** Verify whether your Google Gemini API key is linked to a Google Cloud paid/enterprise tier with zero-data-retention terms enabled. Under Google Cloud Vertex AI / paid API terms, customer prompts are not used to train Google's models. Please confirm this in your production deployment.

---

## 6. Children's Privacy (COPPA & GDPR-K Compliance)

Noctis AI is designed to comply fully with the **Children’s Online Privacy Protection Act (COPPA)** in the US and the **General Data Protection Regulation (GDPR / GDPR-K)** in Europe and the UK:

1. **Parental Operation:** Only parents and legal guardians may create accounts.
2. **Minimal Data:** We only collect a first name, age, and preferred language necessary to tailor stories.
3. **Parental Controls:** Parents have the absolute right to:
   - View all saved stories and profile data;
   - Update child details at any time in the app Settings;
   - Delete individual stories from their library; or
   - Permanently wipe all child details and their entire account in one tap.

If you believe we collected information from a child without verified parental consent, please contact us at `[privacy@noctis.ai]`, and we will immediately purge the data.

---

## 7. Account Deletion & Data Retention

### A. Retention Period
Your profile, stories, illustration images, and audio files are retained only for as long as your account exists, so that you can access your family library.

### B. Instant Self-Serve Account Deletion
You do not need to email support to delete your data. You can delete everything immediately within the app:
1. Open the Noctis AI app.
2. Go to the **Settings** tab.
3. Tap **"Delete Account"** and confirm.

### C. What Happens Upon Deletion (Full Automated Cascade)
Our backend runs an automated cascade function that immediately and irreversibly deletes:
- Your user account and email address;
- Your profile record and child data;
- All saved stories and story text;
- All illustration images in the `story-images` cloud storage bucket;
- All narration audio files in the `story-audio` cloud storage bucket;
- All generation logs and subscription database records.

*Note on Subscriptions:* Deleting your account does **not** automatically cancel an active Apple or Google Play subscription. You must cancel the subscription in your device's store settings.

---

## 8. Data Security

We implement rigorous technical safeguards:
- **Encryption:** All network traffic between the app, servers, and AI APIs is encrypted with HTTPS/TLS.
- **Row Level Security (RLS):** Supabase database tables enforce strict RLS rules ensuring each user can access only their own records.
- **Storage Isolation:** Cloud storage paths for images and audio are tied to authenticated user IDs.

---

## 9. Your Legal Privacy Rights

Depending on your country or state of residence (e.g., EU/UK GDPR, California CCPA/CPRA), you have rights to:
- Access the personal data we hold about you;
- Correct inaccurate personal data;
- Delete your personal data ("Right to be Forgotten");
- Withdraw consent at any time;
- Request data portability.

To exercise these rights, simply use the in-app controls or email us at `[privacy@noctis.ai]`.

---

## 10. Contact Us

If you have questions about this Privacy Policy, please contact:

**Noctis AI Privacy Officer**  
Email: `[privacy@noctis.ai]`  
Address: `[FLAG: Insert Company Postal Address, City, Country]`  
Website: `https://ktsarvi.github.io/noctis-privacy-and-terms/`

