# Firebase Firestore Security Rules Fix

## Problem
You're getting this error:
```
Missing or insufficient permissions: The following request was denied by Firestore Security Rules
```

## Solution

### Step 1: Update Firestore Security Rules

1. Go to **Firebase Console** → **Firestore Database** → **Rules** tab
2. Replace all content with the rules from `firestore.rules` file in the project root
3. Click **Publish**

### Step 2: Required Fields in Firestore Documents

When creating a new request document, ensure it includes:

```javascript
{
  userId: "user-id-here",  // This is CRITICAL for the security rules
  title: "Help needed",
  description: "Description of help",
  location: {
    lat: 34.0522,
    lng: -118.2437
  },
  status: "open",
  createdAt: Timestamp.now(),
  updatedAt: Timestamp.now()
}
```

### Step 3: Update Your Collection Hook

The `useCollection` hook now needs to ensure documents have proper `userId` field.

### Important Security Rules Breakdown

```
// Authenticated users can READ all open requests
allow read: if request.auth != null;

// Only authenticated users can CREATE if userId matches their uid
allow create: if request.auth != null && request.resource.data.userId == request.auth.uid;

// Only document owner can UPDATE
allow update: if request.auth != null && resource.data.userId == request.auth.uid;

// Only document owner can DELETE
allow delete: if request.auth != null && resource.data.userId == request.auth.uid;
```

## Alternative: Use Supabase Instead

If you want to avoid Firestore security rule complexity, switch to Supabase:

```typescript
import { supabase } from '@/lib/supabase'

// Get requests
const { data, error } = await supabase
  .from('requests')
  .select('*')
  .eq('status', 'open')

// Create request
const { data, error } = await supabase
  .from('requests')
  .insert([{ userId, title, description, location, status: 'open' }])
```

## Verification Checklist

- [ ] Updated Firestore Security Rules
- [ ] Published the rules (not just in test mode)
- [ ] Ensure all request documents have `userId` field
- [ ] User is authenticated before accessing requests
- [ ] Test in browser console to verify permissions work
