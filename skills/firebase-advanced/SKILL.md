---
name: firebase-advanced
description: >-
  Top-class Firebase security, architecture, and deployment skill.
  Covers Firestore Security Rules (testing + writing), Firebase Auth
  (server-side verification, custom claims, MFA), Cloud Functions security,
  Storage Rules, App Check, Remote Config, Hosting headers, and Firebase
  CLI deploy workflow. Use when building, auditing, or deploying any Firebase
  project — web or mobile.
  Trigger phrases: "firebase setup", "firestore rules", "firebase auth",
  "firebase security", "cloud functions", "firebase deploy", "app check",
  "firebase project", "firebase advanced".
---

# FIREBASE ADVANCED — TOP-CLASS SKILL

## SECTION 1: PRE-FLIGHT (Before ANY Firebase Work)

Always verify active project first:
```bash
firebase use          # Which project is active?
firebase projects:list  # List all projects
firebase use <project-id>  # Switch project
```
**NEVER deploy without confirming active project.**

---

## SECTION 2: FIRESTORE SECURITY RULES (Most Critical)

### Writing Rules — Mandatory Patterns

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // ✅ Helper functions — always define these
    function isAuthenticated() {
      return request.auth != null;
    }
    function isOwner(userId) {
      return request.auth.uid == userId;
    }
    function hasRole(role) {
      return request.auth.token.role == role;
    }
    function isAdmin() {
      return request.auth.token.admin == true;
    }
    function validData(required, optional) {
      return request.resource.data.keys().hasAll(required)
          && request.resource.data.keys().hasOnly(required.concat(optional));
    }

    // ✅ User documents — owner-only access
    match /users/{userId} {
      allow read: if isAuthenticated() && isOwner(userId);
      allow create: if isAuthenticated() && isOwner(userId)
                    && validData(['email', 'createdAt'], ['displayName', 'photoURL']);
      allow update: if isAuthenticated() && isOwner(userId)
                    && !request.resource.data.diff(resource.data).affectedKeys()
                        .hasAny(['email', 'createdAt', 'role']); // protect immutable fields
      allow delete: if false; // soft delete only via Cloud Function
    }

    // ❌ NEVER DO THIS:
    // match /{document=**} { allow read, write: if true; }
    // match /users/{userId} { allow read, write: if request.auth != null; }
  }
}
```

### Testing Rules — Mandatory Before Deploy

```bash
# Install Firebase Emulator Suite
firebase emulators:start --only firestore

# Run rules tests
npm install --save-dev @firebase/rules-unit-testing
```

```javascript
// rules.test.js — write tests for EVERY rule
const { initializeTestEnvironment, assertFails, assertSucceeds } = 
  require('@firebase/rules-unit-testing');

let testEnv;
beforeAll(async () => {
  testEnv = await initializeTestEnvironment({
    projectId: 'test-project',
    firestore: { rules: fs.readFileSync('firestore.rules', 'utf8') }
  });
});

test('user cannot read another user document (IDOR check)', async () => {
  const userA = testEnv.authenticatedContext('user-a');
  await assertFails(
    userA.firestore().collection('users').doc('user-b').get()
  );
});

test('unauthenticated user cannot read any document', async () => {
  const unauthed = testEnv.unauthenticatedContext();
  await assertFails(
    unauthed.firestore().collection('users').doc('user-a').get()
  );
});
```

### Security Checklist — Rules
- [ ] No `allow read, write: if true` anywhere in production
- [ ] Every collection has explicit auth check
- [ ] IDOR prevented: ownership validated with `request.auth.uid`
- [ ] Immutable fields protected (email, createdAt, role)
- [ ] Data validation with `request.resource.data` checks
- [ ] `rules_version = '2'` at top
- [ ] Rules tested with emulator before every deploy

---

## SECTION 3: FIREBASE AUTH — SERVER-SIDE (Critical)

### Always Verify Token Server-Side

```javascript
// Cloud Function or server — NEVER trust client-side auth state
const admin = require('firebase-admin');

async function verifyUser(idToken) {
  try {
    const decodedToken = await admin.auth().verifyIdToken(idToken);
    return decodedToken; // uid, email, custom claims
  } catch (error) {
    throw new Error('Unauthorized'); // Token invalid/expired
  }
}
```

### Custom Claims — Role-Based Access

```javascript
// Set custom claims (server-side ONLY — via Admin SDK)
await admin.auth().setCustomUserClaims(uid, { 
  role: 'admin',   // or 'teacher', 'student', etc.
  admin: true 
});

// NEVER set claims from client-side
// NEVER trust client-supplied role claims without server verification
```

### Auth Security Checklist
- [ ] All sensitive operations verify ID token server-side (Admin SDK)
- [ ] Custom claims set only via Admin SDK (server/Cloud Function)
- [ ] Token refresh handled — expired tokens rejected
- [ ] Email verification enforced for sensitive actions
- [ ] Password reset: single-use + time-limited
- [ ] Rate limiting on auth endpoints (Firebase has built-in, verify it's active)
- [ ] MFA enabled for admin accounts

---

## SECTION 4: CLOUD FUNCTIONS SECURITY

### Mandatory Pattern for Every HTTP Function

```javascript
const functions = require('firebase-functions');
const admin = require('firebase-admin');

exports.sensitiveAction = functions.https.onCall(async (data, context) => {
  // 1. Auth check FIRST — always
  if (!context.auth) {
    throw new functions.https.HttpsError('unauthenticated', 'Login required');
  }
  
  // 2. Role/permission check
  if (!context.auth.token.admin) {
    throw new functions.https.HttpsError('permission-denied', 'Admin only');
  }
  
  // 3. Input validation
  const { userId, action } = data;
  if (!userId || typeof userId !== 'string' || userId.length > 128) {
    throw new functions.https.HttpsError('invalid-argument', 'Invalid userId');
  }
  
  // 4. Rate limiting (use Firestore or Redis counter)
  await checkRateLimit(context.auth.uid);
  
  // 5. Never log sensitive data
  functions.logger.info('Action performed', { uid: context.auth.uid, action });
  // NOT: functions.logger.info('User data:', { password, token, email });
  
  // 6. Execute with minimal permissions
  return await performAction(userId, action);
});
```

### Cloud Functions Checklist
- [ ] Every callable function checks `context.auth` first
- [ ] HTTP functions validate Bearer token via Admin SDK
- [ ] CORS configured explicitly — not `origin: '*'` for auth endpoints
- [ ] No secrets in function code — use `functions.config()` or Secret Manager
- [ ] Minimum IAM permissions for service account
- [ ] Functions timeout and memory configured appropriately
- [ ] Error messages don't expose internal implementation

---

## SECTION 5: FIREBASE STORAGE RULES

```javascript
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    
    // User uploads — owner only
    match /users/{userId}/{allPaths=**} {
      allow read: if request.auth != null && request.auth.uid == userId;
      allow write: if request.auth != null 
                   && request.auth.uid == userId
                   && request.resource.size < 5 * 1024 * 1024  // 5MB limit
                   && request.resource.contentType.matches('image/.*'); // type check
    }
    
    // Public assets — read only
    match /public/{allPaths=**} {
      allow read: if true;
      allow write: if false; // only via Admin SDK
    }
    
    // ❌ NEVER:
    // match /{allPaths=**} { allow read, write: if request.auth != null; }
  }
}
```

---

## SECTION 6: APP CHECK (Anti-Abuse)

```javascript
// Initialize App Check (prevents unauthorized app access)
import { initializeAppCheck, ReCaptchaV3Provider } from 'firebase/app-check';

const appCheck = initializeAppCheck(app, {
  provider: new ReCaptchaV3Provider('YOUR_RECAPTCHA_SITE_KEY'),
  isTokenAutoRefreshEnabled: true
});

// Enforce App Check on Cloud Functions:
exports.protectedFunction = functions
  .runWith({ enforceAppCheck: true })  // ← reject without valid App Check token
  .https.onCall(async (data, context) => {
    if (context.app == undefined) {
      throw new functions.https.HttpsError('failed-precondition', 'App Check failed');
    }
    // ... rest of function
  });
```

---

## SECTION 7: HOSTING SECURITY HEADERS

```json
// firebase.json
{
  "hosting": {
    "headers": [
      {
        "source": "**",
        "headers": [
          { "key": "X-Frame-Options", "value": "DENY" },
          { "key": "X-Content-Type-Options", "value": "nosniff" },
          { "key": "X-XSS-Protection", "value": "1; mode=block" },
          { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
          { "key": "Permissions-Policy", "value": "camera=(), microphone=(), geolocation=()" },
          {
            "key": "Content-Security-Policy",
            "value": "default-src 'self'; script-src 'self' 'unsafe-inline' https://www.gstatic.com; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self' https://*.googleapis.com https://*.firebaseio.com"
          },
          { "key": "Strict-Transport-Security", "value": "max-age=31536000; includeSubDomains" }
        ]
      }
    ]
  }
}
```

---

## SECTION 8: DEPLOY PROTOCOL (Firebase Safety)

```bash
# MANDATORY SEQUENCE — Never skip steps
# Step 1: Confirm project
firebase use
# → Must show correct project

# Step 2: Build
npm run build
# → Must exit 0

# Step 3: Test rules
npm run test:rules
# → Must exit 0

# Step 4: Git commit first
git add -A
git commit -m "feat: [description]"
git push origin <branch>

# Step 5: Deploy (specific targets only)
firebase deploy --only hosting        # hosting only
firebase deploy --only firestore:rules  # rules only
firebase deploy --only functions      # functions only
firebase deploy                        # FULL deploy — use carefully

# Step 6: Verify
firebase hosting:channel:deploy preview  # preview before production
```

---

## SECTION 9: RED TEAM CHECKLIST (Firebase)

Run these checks before every production deploy:
- [ ] `allow read, write: if true` scan: `grep -r "if true" *.rules`
- [ ] IDOR test: Can authenticated user A read user B's data?
- [ ] Unauthenticated access test: can non-logged-in user access data?
- [ ] Custom claims bypass: can client set their own admin claim?
- [ ] Storage upload: can user upload PHP/executable files?
- [ ] Cloud Function: can unauthenticated user call sensitive functions?
- [ ] Secret scan: `git log -p | grep -i "apikey\|secret\|password\|token"`
- [ ] Environment variables: no secrets in `firebase.json` or `.firebaserc`
- [ ] App Check: enforced on all sensitive functions?
