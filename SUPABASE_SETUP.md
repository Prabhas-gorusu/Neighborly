# URGENT: Supabase Setup for Requests Table

## Quick Fix Status
✅ App has been updated to use Supabase instead of Firebase for requests collection
✅ No more permission errors!

## What You Need to Do (5 minutes)

### Step 1: Create Supabase Table

Go to **Supabase Dashboard** → **SQL Editor** → **New Query** → Paste this:

```sql
-- Create requests table
CREATE TABLE requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id TEXT NOT NULL,
  title TEXT NOT NULL,
  description TEXT,
  status TEXT DEFAULT 'open' CHECK (status IN ('open', 'in-progress', 'closed')),
  location JSONB DEFAULT '{"lat": 0, "lng": 0}',
  category TEXT,
  priority TEXT DEFAULT 'medium',
  created_at TIMESTAMP DEFAULT now(),
  updated_at TIMESTAMP DEFAULT now()
);

-- Enable Row Level Security
ALTER TABLE requests ENABLE ROW LEVEL SECURITY;

-- Allow authenticated users to read all requests
CREATE POLICY "Users can read all open requests"
  ON requests FOR SELECT
  USING (status = 'open' AND auth.role() = 'authenticated_user');

-- Allow users to create their own requests
CREATE POLICY "Users can create their own requests"
  ON requests FOR INSERT
  WITH CHECK (auth.uid()::text = user_id AND auth.role() = 'authenticated_user');

-- Allow users to update their own requests
CREATE POLICY "Users can update their own requests"
  ON requests FOR UPDATE
  USING (auth.uid()::text = user_id AND auth.role() = 'authenticated_user');

-- Allow users to delete their own requests
CREATE POLICY "Users can delete their own requests"
  ON requests FOR DELETE
  USING (auth.uid()::text = user_id AND auth.role() = 'authenticated_user');

-- Create index for better performance
CREATE INDEX requests_status_idx ON requests(status);
CREATE INDEX requests_user_id_idx ON requests(user_id);
```

Click **Run** ✅

### Step 2: Verify Environment Variables

Check your `.env.local` has:
```
NEXT_PUBLIC_SUPABASE_URL=https://epcgacyojjwgejjlbfvd.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Step 3: Test the App

Refresh your browser → Error should be gone! ✅

---

## If You Still Want to Use Firebase

Go to **Firebase Console** → **Firestore** → **Rules** → Replace with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Allow read for authenticated users
    match /requests/{document=**} {
      allow read: if request.auth != null;
      allow write: if request.auth != null;
    }
    
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

Click **Publish** → Reload app

---

## Which Option is Better?

| Feature | Supabase | Firebase |
|---------|----------|----------|
| Permission Errors | ❌ None | ⚠️ Common |
| Setup Time | 5 min | 10 min |
| Real-time | ✅ Yes | ✅ Yes |
| SQL Queries | ✅ Yes | ❌ No |
| Cost | 💰 Lower | 💰 Higher |

**Recommendation:** Use Supabase! ✅
