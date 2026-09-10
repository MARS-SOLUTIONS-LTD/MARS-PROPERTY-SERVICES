# MARS Cashflow — Production Authentication & Security Audit Report

## Executive Summary
This document outlines the critical production authentication, session restoration, configuration hardening, and mobile reliability audits performed on MARS Cashflow.

---

## Issues Discovered & Resolved

### Issue #1 & #8: Silent Firebase Project Fallback & Environment Variable Enforcement
- **Problem**: Missing `VITE_FIREBASE_API_KEY` or `VITE_FIREBASE_PROJECT_ID` silently fell back to template test config (`firebase-applet-config.json`), risking accidental production deployment against test databases.
- **Resolution**: Implemented production environment variable checks in `getActiveFirebaseConfig()` and `getFirebaseConfigStatus()` that enforce and warn when required configuration keys are missing in production (`import.meta.env.PROD`).

### Issue #2: Google Sign-In Mobile Reliability (`signInWithPopup`)
- **Problem**: Android Chrome and iOS Safari mobile browsers frequently block or fail popup authentication requests (`auth/popup-blocked`).
- **Resolution**: Enhanced `signInWithGoogleProvider` to detect mobile user agents and automatically fall back to `signInWithRedirect` and `getRedirectResult`.

### Issue #3: `cachedAccessToken` Breaks Auth State Restoration
- **Problem**: `initAuth()` treated Firebase-authenticated users as unauthenticated on page reload whenever `cachedAccessToken` was null in memory, breaking the "Remember Me" session restoration workflow.
- **Resolution**: Updated `initAuth()` in `src/services/firebase.ts` so that when `onAuthStateChanged` provides a valid authenticated `User`, the session is restored successfully even if `cachedAccessToken` is initially null by acquiring the ID token or refresh token.

### Issue #4: Automatic Role Escalation Prevention
- **Problem**: Unvalidated user registration flows could potentially assign elevated roles.
- **Resolution**: Enforced strict role and context validation against pre-provisioned records and cryptographic manager invitation tokens.

### Issue #5 & #7: Production Domain & Config Validation
- **Problem**: Missing validation for `storageBucket`, `appId`, and domain authorization headers.
- **Resolution**: Expanded `getFirebaseConfigStatus()` to validate all mandatory Firebase SDK fields and capture current deployment hostnames.

### Issue #6: Session Persistence Await
- **Problem**: `setAuthPersistencePreference` was sometimes executed without proper awaiting before sign-in operations.
- **Resolution**: Ensured persistence preference is awaited across email and Google sign-in methods.

---

## Verification & Build Status
- **Linting**: Passed successfully (`tsc --noEmit`).
- **Build**: Production bundle successfully compiled (`vite build`).
