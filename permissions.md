# Permissions & Data Access Audit - Noctis Tales (Storytell App)

**Audit Date:** 2026-10-12
**App Type:** React Native (Expo Router, NativeWind, Zustand, Supabase, RevenueCat, Gemini API)
**Target Audience:** Children's story app (COPPA and Apple Kids Category applicable)

---

## 1. Summary Table

| Permission                                | Platform    | Status                 | Feature                      | When Requested                 | If Denied                             |
| ----------------------------------------- | ----------- | ---------------------- | ---------------------------- | ------------------------------ | ------------------------------------- |
| None (no runtime permissions)             | iOS/Android | N/A                    | N/A                          | N/A                            | N/A                                   |
| usesAppleSignIn                           | iOS         | KEPT                   | Sign in with Apple           | On sign-in button press        | User can use email/password instead   |
| ITSAppUsesNonExemptEncryption: false      | iOS         | KEPT                   | No custom encryption         | N/A                            | N/A                                   |
| expo-audio                                | iOS/Android | LIBRARY-ADDED (NEEDED) | TTS audio playback           | On story playback (Ultra plan) | Story narration unavailable           |
| expo-file-system                          | iOS/Android | LIBRARY-ADDED (NEEDED) | Cache TTS audio              | During TTS generation          | TTS fails, fallback to text-only      |
| expo-linking                              | iOS/Android | LIBRARY-ADDED (NEEDED) | Deep linking & external URLs | On auth callback, legal links  | Auth may fail, legal pages won't open |
| expo-web-browser                          | iOS/Android | LIBRARY-ADDED (NEEDED) | In-app browser for URLs      | On subscription management     | Management opens in system browser    |
| @react-native-async-storage/async-storage | iOS/Android | LIBRARY-ADDED (NEEDED) | Session persistence          | On app launch/auth             | Session not persisted                 |
| expo-crypto                               | iOS/Android | LIBRARY-ADDED (NEEDED) | Nonce for Apple auth         | On Apple sign-in               | Apple sign-in fails                   |
| expo-haptics                              | iOS/Android | REMOVED                | (Unused)                     | N/A                            | N/A                                   |
| @react-native-google-signin/google-signin | iOS/Android | REMOVED                | (Planned, not implemented)   | N/A                            | N/A                                   |

**Key Finding:** The app currently declares **ZERO runtime permissions** (camera, microphone, location, contacts, etc.). All capabilities are either via library APIs that don't require user permission prompts, or authentication providers that handle their own consent flows.

---

## 2. Detailed Entries for Kept Permissions/Capabilities

### 2.1 Apple Sign-in (iOS)

**Config Key:** `ios.usesAppleSignIn: true`
**Declared:** app.json:13
**Used:** lib/appleAuth.ts:2, 67-73; app/screens/auth/signUpScreen.tsx:398-425; app/screens/auth/signInScreen.tsx:260-287

**User-Facing Description:** (None required - Apple provides standard UI)

**Data Accessed:**

- Email (optional, Apple provides on first authorization only)
- Full name (optional, Apple provides on first authorization only)

**Data Leaves Device:** Yes - sent to Supabase Auth for authentication

**Storage:** Supabase auth.users table (email, full_name fields) + AsyncStorage for session persistence

**Retention:** As long as account exists (user-controlled deletion)

**Behavior If Denied:** User can sign up/sign in with email and password instead

**Permission Request Flow:**

- Requested only when user taps "Sign in with Apple" button
- Apple's standard consent screen shown
- No custom in-app explanation shown (Apple's UI is sufficient per guidelines)

---

### 2.2 expo-audio (TTS Playback)

**Config Key:** Plugin in app.json:52
**Declared:** app.json:52
**Used:** hooks/useStoryTTS.ts:2-6, 92, 126

**User-Facing Description:** (None required - audio playback doesn't need permission)

**Data Accessed:**

- Audio files from device cache (FileSystem.cacheDirectory)
- No microphone access

**Data Leaves Device:** No - audio is generated server-side by Gemini, cached locally, played back

**Storage:** Temporary cache (FileSystem.cacheDirectory), deleted after playback (useStoryTTS.ts:57, 120, 134, 161, 178)

**Retention:** Until playback completes or app unmounts (immediate cleanup)

**Behavior If Denied:** Not applicable - no permission required for audio playback

**Note:** Uses `setAudioModeAsync({ playsInSilentMode: true })` to enable playback in silent mode (useStoryTTS.ts:92)

---

### 2.3 expo-file-system (Cache Storage)

**Config Key:** No explicit config (part of expo-file-system package)
**Declared:** package.json:31
**Used:** hooks/useStoryTTS.ts:8-12, 110-114

**User-Facing Description:** (None required - file system cache access doesn't need permission)

**Data Accessed:**

- Write to: FileSystem.cacheDirectory (temporary TTS audio files)
- Delete from: FileSystem.cacheDirectory (cleanup after playback)

**Data Leaves Device:** No - cache is local only

**Storage:** Device cache directory (app-specific sandbox)

**Retention:** Transient - files deleted immediately after use or on cleanup

**Behavior If Denied:** TTS feature would fail, but no permission prompt is shown (cache access is implicit)

---

### 2.4 expo-linking (Deep Links & External URLs)

**Config Key:** scheme: "noctis-ai" in app.json:8
**Declared:** app.json:8
**Used:**

- app/auth/callback.tsx:9, 48 (auth deep links)
- app/auth/confirmed.tsx:9, 48 (auth deep links)
- app/(tabs)/profile.tsx:129, 311, 320, 338, 473, 481 (external URLs)
- app/(tabs)/subscription.tsx:176, 186 (management URLs)
- app/screens/auth/signUpScreen.tsx:441, 448 (legal URLs)
- app/screens/auth/signInScreen.tsx (via imports)

**User-Facing Description:** (None required - deep linking is implicit)

**Data Accessed:**

- Incoming deep links (auth callbacks from Apple)
- Outgoing links (legal pages, app stores, email)

**Data Leaves Device:** No - links are opened in system browser or in-app browser

**Storage:** None

**Retention:** N/A

**Behavior If Denied:** Auth callback may fail; legal pages may not open; but no permission prompt

---

### 2.5 expo-web-browser (In-App Browser)

**Config Key:** Plugin in app.json:51
**Declared:** app.json:51
**Used:** app/(tabs)/profile.tsx:3

**User-Facing Description:** (None required - in-app browser doesn't need permission)

**Data Accessed:**

- Opens external URLs (subscription management, app stores) within app context

**Data Leaves Device:** No - browsing happens in-app

**Storage:** None

**Retention:** N/A

**Behavior If Denied:** URLs would open in system browser instead (fallback handled by OS)

---

### 2.6 @react-native-async-storage/async-storage (Session Persistence)

**Config Key:** No explicit config (standard React Native library)
**Declared:** package.json:21
**Used:**

- lib/supabaseClient.ts:1, 36 (Supabase session storage)
- lib/appleAuth.ts:1, 32, 40, 108, 114, 124 (pending Apple name sync)
- app/(tabs)/profile.tsx:1, 267, 269, 503, 504, 513, 515 (local data clear)

**User-Facing Description:** (None required - local storage is implicit)

**Data Accessed:**

- Supabase auth session tokens
- Pending Apple name (retry logic for failed name sync)
- App state data

**Data Leaves Device:** No - AsyncStorage is device-local only

**Storage:** Device local storage (encrypted on iOS keychain, Android Keystore on supported devices)

**Retention:** Until user signs out or clears app data

**Behavior If Denied:** Session not persisted; user must sign in again on app restart

**Privacy Note:** AsyncStorage on iOS uses Keychain (encrypted), on Android uses SharedPreferences (not encrypted by default but app-private)

---

### 2.7 expo-crypto (Nonce Generation)

**Config Key:** No explicit config (part of expo-crypto package)
**Declared:** package.json:29
**Used:** lib/appleAuth.ts:3, 60-64

**User-Facing Description:** (None required - crypto operations are implicit)

**Data Accessed:**

- Generates UUID and SHA-256 hash for Apple auth nonce

**Data Leaves Device:** No - nonce is used locally for auth flow

**Storage:** None (transient memory)

**Retention:** N/A

**Behavior If Denied:** Apple sign-in would fail (cannot generate secure nonce)

---

### 2.8 react-native-purchases (RevenueCat)

**Config Key:** No explicit config (library handles it)
**Declared:** package.json:48-49
**Used:** lib/revenueCat.ts (entire file)

**User-Facing Description:** (None required - purchase flow handled by App Store/Play Store)

**Data Accessed:**

- User's App Store/Play Store account (via native in-app purchase APIs)
- Supabase user ID (to sync purchases)

**Data Leaves Device:** Yes - purchase data sent to RevenueCat and synced to Supabase

**Storage:** RevenueCat servers + Supabase user_story_quota table

**Retention:** As long as account exists (user-controlled deletion)

**Behavior If Denied:** In-app purchases unavailable; free plan still works

**Privacy Note:** RevenueCat collects purchase data for subscription management. See RevenueCat privacy policy.

---

### 2.9 @supabase/supabase-js (Backend Communication)

**Config Key:** No explicit config (library handles it)
**Declared:** package.json:23
**Used:** lib/supabaseClient.ts (entire file); throughout app

**User-Facing Description:** (None required - network access is implicit)

**Data Accessed:**

- Auth tokens
- User data (stories, quota, profile)
- Edge function calls (Gemini API proxy)

**Data Leaves Device:** Yes - sent to Supabase backend

**Storage:** Supabase PostgreSQL database (auth.users, stories, user_story_quota, api_rate_limits)

**Retention:** As long as account exists (user-controlled deletion)

**Behavior If Denied:** App cannot function (requires network connectivity)

---

## 3. Removed Permissions and Dependencies

### 3.1 expo-haptics

**Reason:** Declared in package.json but never used in code. No haptic feedback implementation found.

**Removed From:** package.json:34

**Impact:** None - feature never existed

**Rebuild Required:** No (only package.json change)

---

### 3.2 @react-native-google-signin/google-signin

**Reason:** Declared in package.json but never implemented. Google Sign-in is planned (see GOOGLE_AUTH_SETUP.md) but not yet integrated. The AGENTS.md file contains Supabase documentation for Google auth, but the actual UI and integration code are missing.

**Removed From:** package.json:22

**Impact:** None - feature never existed

**Rebuild Required:** No (only package.json change)

**Note:** GOOGLE_AUTH_SETUP.md and AGENTS.md updated to mark Google auth as "PLANNED (NOT YET IMPLEMENTED)"

---

## 4. Planned Permissions (Not Yet Requested)

### 4.1 Google Sign-in

**Status:** Planned but not implemented

**When Added:** When Google auth integration is completed

**Required Changes:**

- Re-add @react-native-google-signin/google-signin to package.json
- Implement Google sign-in UI (similar to Apple button in signUpScreen.tsx:398-425)
- Configure Google Cloud Console OAuth clients
- Add Google provider to Supabase dashboard
- Update app.json with any required Google configuration

**Permission Required:** None (Google sign-in uses native consent flow, no runtime permission)

**Data Impact:** Would add Google OAuth token exchange and potential Google email/name collection (similar to Apple)

---

## 5. Data Collected Without Device Permissions

| Data Item                      | Purpose                | Storage Location                               | Shared With                  | Retention                                | Deletion Method                                                      |
| ------------------------------ | ---------------------- | ---------------------------------------------- | ---------------------------- | ---------------------------------------- | -------------------------------------------------------------------- |
| Email                          | Account authentication | Supabase auth.users                            | Supabase (auth provider)     | Until account deletion                   | Delete account function (supabase/functions/delete-account/index.ts) |
| Password (hashed)              | Account authentication | Supabase auth.users                            | None (never leaves Supabase) | Until account deletion                   | Delete account function                                              |
| Name (optional)                | User profile           | Supabase auth.users.user_metadata              | None                         | Until account deletion                   | Delete account function                                              |
| Child Name                     | Story personalization  | Supabase stories.child_name                    | Gemini API (for generation)  | Until story deletion or account deletion | Delete story or delete account                                       |
| Child Age                      | Story personalization  | Supabase stories.age                           | Gemini API (for generation)  | Until story deletion or account deletion | Delete story or delete account                                       |
| Child Gender                   | Story personalization  | Supabase stories.gender                        | Gemini API (for generation)  | Until story deletion or account deletion | Delete story or delete account                                       |
| Child Appearance               | Story personalization  | Supabase stories.appearance                    | Gemini API (for generation)  | Until story deletion or account deletion | Delete story or delete account                                       |
| Story Theme                    | Story generation       | Supabase stories.theme                         | Gemini API (for generation)  | Until story deletion or account deletion | Delete story or delete account                                       |
| Story Title                    | Story metadata         | Supabase stories.title                         | None                         | Until story deletion or account deletion | Delete story or delete account                                       |
| Story Length                   | Story metadata         | Supabase stories.story_length                  | None                         | Until story deletion or account deletion | Delete story or delete account                                       |
| Additional Details             | Story generation       | Supabase stories.additional_details            | Gemini API (for generation)  | Until story deletion or account deletion | Delete story or delete account                                       |
| Generated Story Text           | User content           | Supabase stories.story_text                    | None                         | Until story deletion or account deletion | Delete story or delete account                                       |
| Generated Cover Image (base64) | User content           | Supabase stories.cover_image_data_uri          | None                         | Until story deletion or account deletion | Delete story or delete account                                       |
| Generated TTS Audio (base64)   | Playback               | Device cache (transient)                       | None                         | Until playback completes                 | Automatic cleanup on playback complete/app unmount                   |
| Session Tokens                 | Auth persistence       | AsyncStorage (device)                          | Supabase (auth verification) | Until sign out or app data clear         | Sign out or clear app data                                           |
| Pending Apple Name             | Retry logic            | AsyncStorage (device)                          | None                         | Until successful sync or retry exhausted | Automatic cleanup on success                                         |
| Quota Data                     | Plan enforcement       | Supabase user_story_quota                      | None                         | Until account deletion                   | Delete account function                                              |
| Rate Limit Data                | Abuse prevention       | Supabase api_rate_limits                       | None                         | 24-hour rolling window                   | Automatic cleanup (old entries purged)                               |
| Purchase Data                  | Subscription           | RevenueCat servers + Supabase user_story_quota | Apple/Google, RevenueCat     | Until account deletion                   | Delete account function + revoke app subscription                    |
| Device Info (implicit)         | Auth session           | Supabase auth.sessions                         | None                         | Until session expires                    | Automatic expiration on sign out                                     |

**OPEN QUESTION - Story Data Retention:** What is the retention policy for story data? Should stories be automatically deleted after X days of inactivity, or retained indefinitely until user deletion? Current implementation: **Indefinite retention until user deletes story or account.**

---

## 6. Third-Party Processors

### 6.1 Supabase

**What They Receive:**

- Auth tokens (email/password or OAuth)
- User profile data (name, email)
- Story metadata (child details, themes, generated content)
- Quota and subscription status
- Rate limiting data

**Purpose:** Authentication, database, edge functions (Gemini proxy)

**Data Controller:** Noctis AI (you control the Supabase project)

**Retention:** As configured in your Supabase project (user-controlled)

**Data Location:** Supabase cloud (AWS us-east-1 by default, check your project settings)

**Subprocessors:** Supabase uses AWS infrastructure

---

### 6.2 Google Gemini API

**What They Receive:**

- Story prompts (child name, age, gender, appearance, theme, details)
- Text-to-speech text (for audio generation)
- Image prompts (for cover generation)

**Purpose:** AI content generation (stories, images, audio)

**Data Controller:** Google (you use their API)

**Retention:** Google's Gemini API data retention policy (check Google AI terms)

- Current implementation: Data sent via Supabase edge functions, not directly from app
- Edge functions act as proxy to hide API keys

**Data Location:** Google Cloud

**Subprocessors:** None disclosed by Google for Gemini API

**Privacy Note:** Gemini API may process prompts for model improvement. Check Google's data usage policy.

---

### 6.3 RevenueCat

**What They Receive:**

- Purchase transaction data (Apple/Google receipts)
- App User ID (linked to Supabase user ID)
- Subscription status and entitlements

**Purpose:** Subscription management and receipt validation

**Data Controller:** RevenueCat (you are the customer, they process on your behalf)

**Retention:** As long as subscription is active + required for compliance (check RevenueCat terms)

**Data Location:** RevenueCat servers

**Subprocessors:** Apple (App Store receipts), Google (Play Store receipts)

**Privacy Note:** RevenueCat has its own privacy policy. You must disclose their processing in your Privacy Policy.

---

### 6.4 Apple (via Sign in with Apple)

**What They Receive:**

- Authentication request (via native iOS API)
- Optional email and name (first authorization only)

**Purpose:** Authentication

**Data Controller:** Apple (they control Apple ID data)

**Retention:** Apple controls user data; app only receives what user authorizes

**Data Location:** Apple servers (user's Apple ID account)

**Subprocessors:** None

**Privacy Note:** Apple's Sign in with Apple privacy policy applies. User can revoke access in Apple ID settings.

---

### 6.5 Apple (via In-App Purchases)

**What They Receive:**

- Purchase transaction data
- App bundle ID and user account

**Purpose:** Process subscriptions

**Data Controller:** Apple (App Store)

**Retention:** Apple's transaction retention policies

**Data Location:** Apple App Store servers

**Subprocessors:** None

**Privacy Note:** App Store privacy policy applies. RevenueCat acts as intermediary.

---

### 6.6 Google (via Play Store - Future)

**What They Receive:**

- Purchase transaction data (if Android build added)
- App package name and user account

**Purpose:** Process subscriptions (when Android build is added)

**Data Controller:** Google (Play Store)

**Retention:** Google Play transaction retention policies

**Data Location:** Google Play servers

**Subprocessors:** None

**Status:** Not currently implemented (iOS-only per current config)

---

## 7. Children's Data Notes

### 7.1 Data Collected from Children

**Direct Collection:**

- Child name (for story personalization)
- Child age (for age-appropriate content)
- Child gender (for story pronouns)
- Child appearance (for story illustration)
- Additional details (for story customization)

**Indirect Collection:**

- None (no tracking, analytics, or behavioral data)

**Parental Control:**

- **CRITICAL GAP:** No explicit parental consent flow exists. The app assumes the adult creating the account is the parent/guardian of the child.
- Sign-up form asks for parent's name/email, not child's
- Story creation form collects child details, but no parental consent checkbox or verification
- **RECOMMENDATION:** Add explicit parental consent checkbox during story creation or on sign-up for COPPA compliance

### 7.2 Parental Consent Touchpoints

**Current State:**

- None explicitly implemented
- Privacy Policy page (legalConstants.ts:8-18) may contain consent language, but not verified in this audit

**Recommended Addition:**

- Checkbox: "I confirm I am the parent or legal guardian of the child named above and consent to their data being used for story generation."
- Link to Privacy Policy at consent point
- Clear explanation of what child data is collected and how it's used

### 7.3 Deletion of Children's Data

**Available Methods:**

1. **Delete individual story:** Via library screen (app/(tabs)/library.tsx - implementation not fully audited but deleteStoryById exists in storiesApi.ts:289-310)
2. **Delete entire account:** Via profile screen → Delete Account (app/(tabs)/profile.tsx - calls delete-account edge function)
3. **Contact developer:** privacy@noctis.ai (legalConstants.ts:25)

**What Gets Deleted:**

- Account deletion cascades to:
  - All stories (stories table)
  - Quota data (user_story_quota table)
  - Rate limit data (api_rate_limits table)
  - Auth session (auth.users table)
- Individual story deletion removes only that story's data

**Verification:** Delete-account edge function (supabase/functions/delete-account/index.ts:108-110) uses admin API with CASCADE, confirming all related data is deleted.

### 7.4 COPPA Considerations

**Current Status:**

- App is a children's app (story generation for kids)
- Collects child's name, age, gender, appearance
- No verifiable parental consent mechanism
- No age verification (child age is self-reported input field)
- Data is sent to third-party AI (Gemini) for processing

**Compliance Gaps:**

1. No parental consent checkbox
2. No age verification (could be used by adults for personal stories, but intended for children)
3. No clear notice to parents about third-party processing (Gemini)
4. No opt-out mechanism for data sharing with AI (required for story generation)

**Recommendations:**

1. Add explicit parental consent flow
2. Add age gate or verification
3. Disclose Gemini AI processing in Privacy Policy with parental opt-out if possible (may not be feasible if required for core feature)
4. Consider implementing a "child profile" system where parent can manage child's data separately from their own account

---

## 8. Privacy Policy Mapping

### 8.1 What We Collect

**Account Data:**

- Email address (required for authentication)
- Password (hashed, never stored in plaintext)
- Name (optional, from sign-up or Apple/Google auth)
- Source: signUpScreen.tsx:27-30, 109-117; appleAuth.ts:67-73

**Child Data:**

- Child's name (for story personalization)
- Child's age (for age-appropriate content)
- Child's gender (for story pronouns)
- Child's appearance (for story illustration)
- Additional story details
- Source: newStory.tsx:40-45; gemini.ts:5-14

**Generated Content:**

- Story text (AI-generated)
- Cover images (AI-generated)
- Audio narration (AI-generated, cached locally)
- Source: gemini.ts:68-93, 96-119, 122-148; useStoryTTS.ts:110-114

**Technical Data:**

- Session tokens (for authentication persistence)
- Quota and subscription data
- Rate limiting data
- Source: supabaseClient.ts:36; storiesApi.ts:38-86; supabase.sql:560-614

**Payment Data:**

- Purchase transactions (processed by Apple/Google via RevenueCat)
- Subscription status
- Source: revenueCat.ts:145-157; lib/planLimits.ts

---

### 8.2 Why We Collect It

**Account Data:**

- Email and password: To create and secure your account
- Name: To personalize your experience

**Child Data:**

- Child details: To generate personalized stories with the child as the main character
- Age: To ensure story content is age-appropriate
- Gender: To use correct pronouns in stories
- Appearance: To generate accurate character illustrations

**Generated Content:**

- Stories, images, audio: To provide the core service of personalized children's stories

**Technical Data:**

- Session tokens: To keep you signed in securely
- Quota data: To enforce plan limits and prevent abuse
- Rate limiting: To protect against API abuse

**Payment Data:**

- Purchase data: To process subscriptions and provide paid features

---

### 8.3 Who Gets It

**Shared With Third Parties:**

1. **Supabase:** Database and authentication provider (data controller: you)
2. **Google Gemini API:** AI content generation (data controller: Google)
3. **RevenueCat:** Subscription management (data controller: RevenueCat)
4. **Apple:** Sign in with Apple and App Store purchases (data controller: Apple)
5. **Google (Play Store):** Future Android purchases (data controller: Google)

**Not Shared With:**

- No analytics or advertising companies
- No data brokers
- No third parties for marketing purposes

---

### 8.4 How Long We Keep It

**Account Data:** Until you delete your account (delete-account edge function)

**Child Data & Stories:** Until you delete the story or your account (deleteStoryById or delete-account)

**Generated Content:** Until you delete the story or your account

**Technical Data:**

- Session tokens: Until sign out or app data cleared
- Rate limiting data: 24-hour rolling window (automatic cleanup)
- Quota data: Until account deletion

**Payment Data:** As required for subscription management (RevenueCat terms apply)

**OPEN QUESTION:** Should there be an automatic retention policy for stories (e.g., delete after 1 year of inactivity)? Current: Indefinite retention.

---

### 8.5 How to Delete It

**Delete Individual Story:**

- Navigate to Library tab
- Swipe or tap delete on story
- Alternatively, contact privacy@noctis.ai

**Delete All Data:**

- Navigate to Profile tab
- Tap "Delete Account"
- Confirm deletion
- This deletes all stories, child data, and account information permanently

**Request Deletion by Email:**

- Email privacy@noctis.ai
- Include account email
- Request account deletion

**Delete from Third Parties:**

- Apple: Manage in Apple ID settings
- Google: Manage in Google Account settings (if/when Google auth added)
- RevenueCat: Contact RevenueCat support or revoke subscription in App Store/Play Store

---

## 9. Terms of Service Mapping

### 9.1 Permission-Dependent Features

**Story Generation (No Permission Required):**

- Core feature available to all users
- Requires network connectivity
- No device permissions needed

**Text-to-Speech (No Permission Required):**

- Ultra plan feature
- Uses expo-audio (no runtime permission)
- Requires network connectivity for Gemini API

**Sign in with Apple (No Runtime Permission):**

- Uses Apple's native consent flow
- Available on iOS devices that support it
- Alternative: Email/password sign-in

**In-App Purchases (No Runtime Permission):**

- Handled by App Store/Play Store native flow
- RevenueCat SDK manages purchase validation
- No app-level permissions needed

---

### 9.2 AI-Generated Content

**Ownership:**

- Generated stories, images, and audio are owned by the user
- User grants Noctis AI license to store and deliver content
- No claim of ownership over AI-generated content by Noctis AI

**Warranty Disclaimer:**

- AI-generated content may not be accurate or appropriate
- No warranty on story quality, safety, or suitability
- Parents should review stories before sharing with children

**Content Moderation:**

- No content moderation filtering verified in this audit
- Gemini API may have its own safety filters
- **RECOMMENDATION:** Implement content filtering or review mechanism

**Liability:**

- No liability for AI-generated content errors or omissions
- User responsible for inputs provided to AI
- User responsible for sharing content with children

---

### 9.3 User Responsibility for Inputs

**User Representations:**

- User represents they have right to use child's data
- User represents child details are accurate
- User represents they are parent/guardian (no verification currently)

**Prohibited Uses:**

- Do not input inappropriate, harmful, or offensive content
- Do not use AI to generate content that violates laws
- Do not impersonate others

**Indemnification:**

- User indemnifies Noctis AI for violations arising from their inputs
- User is responsible for all content they input

---

### 9.4 Age Requirement

**Current Status:**

- No explicit age gate on sign-up
- Sign-up collects parent's name/email
- Story creation collects child's age (1-65 validation)
- **GAP:** No verification that account holder is adult

**Recommended Terms:**

- "You must be at least 18 years old to create an account"
- "By creating an account, you represent you are the parent or legal guardian"
- "If you are under 18, you must have parental consent"

---

### 9.5 Subscription and Account Terms

**Subscription Terms:**

- Weekly subscription for Ultra plan ($4.49/week per planLimits.ts)
- Auto-renewal via Apple/Play Store
- Cancel anytime in App Store/Play Store settings
- No refunds for partial periods (Apple/Play Store policy)

**Account Terms:**

- One account per person
- Do not share account credentials
- Account deletion is permanent
- You are responsible for account security

**Plan Limits:**

- Free plan: 1 short, 1 medium story (one-time, no reset)
- Ultra plan: 3 short, 3 medium, 2 long stories per 24 hours + TTS
- Quota resets every 24 hours for Ultra plan
- Free plan quota does not reset (one-time use)

---

## 10. Open Questions

### 10.1 Story Data Retention

**Question:** What is the retention policy for story data? Should stories be automatically deleted after X days of inactivity, or retained indefinitely until user deletion?

**Current Implementation:** Indefinite retention until user deletes story or account.

**Decision Needed:**

- Implement automatic retention policy (e.g., 1 year)?
- Keep indefinite retention?
- Add user setting for retention period?

**Impact:** Privacy Policy must disclose retention period.

---

### 10.2 Parental Consent Flow

**Question:** Should explicit parental consent be required before collecting child data? Currently, no consent checkbox or verification exists.

**Current Implementation:** Assumes adult signing up is parent/guardian. No consent flow.

**DECISION MADE (2026-10-12):** Added explicit Terms of Service and Privacy Policy agreement checkbox to sign-up screen (signUpScreen.tsx:38, 75-79, 440-464). This is **NOT** a parental consent checkbox for child data - it's for the adult's agreement to terms. A separate parental consent checkbox for child data collection is still needed for COPPA compliance.

**Remaining Decision Needed:**

- Add separate parental consent checkbox for child data collection (during story creation)?
- Add age gate or verification for account holder?
- What language to use for parental consent?

**Impact:** COPPA compliance, Privacy Policy disclosure, Terms of Service updates.

---

### 10.3 AI Content Moderation

**Question:** Is there content moderation for AI-generated stories? This audit found no moderation implementation.

**Current Implementation:** No content filtering verified. Relies on Gemini API's built-in safety filters.

**Decision Needed:**

- Implement client-side content filtering?
- Implement server-side content filtering?
- Add parental review before story display?
- What content to filter (violence, inappropriate themes, etc.)?

**Impact:** Child safety, Terms of Service liability, user trust.

---

### 10.4 Child Age Verification

**Question:** Should there be age verification for child input? Currently, age is a self-reported text field (1-65 validation).

**Current Implementation:** Self-reported age in newStory.tsx:329, validation 1-65.

**Decision Needed:**

- Keep self-reported age?
- Add age gate for the app user (parent)?
- Add verification that child is a minor?
- What to do if age indicates adult (e.g., user enters age 30 for "child")?

**Impact:** COPPA compliance, app positioning (children's app vs. general story app).

---

### 10.5 Data Sharing with AI Opt-Out

**Question:** Can users opt out of sharing child data with Gemini API? Currently, data sharing is required for core feature.

**Current Implementation:** Child data is sent to Gemini for every story generation (gemini-generate-story/index.ts:68-78).

**Decision Needed:**

- Is opt-out feasible? (Core feature requires AI)
- If not feasible, must be clearly disclosed in Privacy Policy
- Consider offering manual story writing alternative (no AI)?

**Impact:** Privacy Policy disclosure, potential alternative features.

---

### 10.6 Analytics and Crash Reporting

**Question:** Should analytics or crash reporting be added? Currently, none are implemented.

**Current Implementation:** No analytics, crash reporting, or tracking SDKs found.

**Decision Needed:**

- Add analytics for product improvement?
- Add crash reporting for stability?
- If added, which provider? (Must comply with children's app restrictions)
- How to disclose in Privacy Policy?

**Impact:** App improvement, privacy implications, COPPA compliance.

---

### 10.7 Android Build

**Question:** When will Android build be added? Currently, config is iOS-only (no Android native folder).

**Current Implementation:** iOS-only per app.json:12-16. Android config exists (app.json:18-26) but no native Android folder.

**Decision Needed:**

- Timeline for Android build?
- Will Google Sign-in be added with Android build?
- Additional permissions needed for Android?

**Impact:** Additional privacy considerations for Google Play Store, Android permissions, Google Play Services.

---

### 10.8 Legal Page Hosting

**Question:** Where are Privacy Policy and Terms of Service hosted? Currently, external GitHub Pages (legalConstants.ts:8-18).

**Current Implementation:** https://ktsarvi.github.io/noctis-privacy-and-terms/

**Decision Needed:**

- Keep external hosting?
- Host on own domain?
- Ensure legal pages are updated to reflect this audit findings?

**Impact:** App Store/Play Store review, user trust, legal compliance.

---

## Summary of Changes Made

### Removed Dependencies

1. **expo-haptics** - Unused, no haptic feedback implementation
2. **@react-native-google-signin/google-signin** - Planned but not implemented

### Updated Documentation

1. **GOOGLE_AUTH_SETUP.md** - Added status note: "PLANNED (NOT YET IMPLEMENTED)"

### Added Terms Agreement Checkbox (2026-10-12)

1. **signUpScreen.tsx** - Added explicit checkbox for Terms of Service and Privacy Policy agreement
   - State: `agreedToTerms` (line 38)
   - Validation: Blocks sign-up if not checked (lines 75-79)
   - UI: Checkbox with checkmark icon (lines 440-464)
   - Button state: Disabled and grayed out if not agreed (lines 387-407)
   - Apple Sign-in: Also requires agreement (lines 161-166, 441-453)
   - Accessibility: Proper accessibilityRole and accessibilityState

### No Runtime Permissions Removed

- App had zero runtime permissions declared
- No usage descriptions needed (no NS\*UsageDescription keys)
- All capabilities are library-based or authentication-provider-based

---

## Rebuild and Re-Test Steps

### Required Rebuild

**No rebuild required** - Only package.json changes (removed unused dependencies). These changes don't affect native code or permissions.

### Recommended Re-Test

1. **Test app launch** - Verify no errors from removed packages
2. **Test sign-up/sign-in** - Verify Apple auth still works
3. **Test story generation** - Verify TTS audio playback still works
4. **Test profile page** - Verify external links and subscription management still work
5. **Test account deletion** - Verify delete-account edge function still works

### No App Store/Play Store Update Required

- No permissions changed
- No native code changed
- Update only needed if next submission cycle

---

## Audit Conclusions

### Current State

The app is **permission-minimal** with zero runtime permissions declared. All functionality is achieved through:

- Library APIs that don't require user permission prompts
- Authentication providers (Apple) that handle their own consent flows
- Network-based services (Supabase, Gemini, RevenueCat)

### Compliance Gaps

1. **COPPA:** No explicit parental consent flow for child data collection
2. **Age Verification:** No age gate or verification for account holder
3. **Content Moderation:** No AI content filtering verified
4. **Data Retention:** No automatic retention policy for stories (indefinite)

### Recommendations Priority

1. **HIGH:** Add separate parental consent checkbox for child data collection (Terms agreement checkbox added 2026-10-12)
2. **HIGH:** Update Privacy Policy to reflect this audit findings
3. **HIGH:** Update Terms of Service with AI content disclaimers
4. **MEDIUM:** Add age gate for app users (verify adult)
5. **MEDIUM:** Define story data retention policy
6. **MEDIUM:** Implement content moderation for AI-generated stories
7. **LOW:** Consider analytics/crash reporting (with children's app restrictions)

### Open Questions for Decision

All open questions in Section 10 require product/business decisions before implementation.

---

**Audit Completed:** 2026-10-12
**Audited By:** Devin (automated code analysis)
**Next Review:** Before next App Store/Play Store submission
