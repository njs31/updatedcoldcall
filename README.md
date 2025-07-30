# Cold Reach Out - Google Sign-In Setup

This application now includes real Google Sign-In authentication using Supabase. Follow these steps to set it up:

## Prerequisites

1. A Supabase account (free at https://supabase.com)
2. A Google Cloud Console account for OAuth setup

## Setup Instructions

### 1. Supabase Setup

1. **Create a new Supabase project:**

   - Go to https://supabase.com
   - Click "New Project"
   - Choose your organization
   - Enter project name and database password
   - Choose a region close to your users
   - Click "Create new project"

2. **Get your Supabase credentials:**

   - In your Supabase dashboard, go to Settings → API
   - Copy the "Project URL" and "anon public" key

3. **Create the database table:**
   - Go to SQL Editor in your Supabase dashboard
   - Run this SQL to create the outreach history table:

```sql
CREATE TABLE outreach_history (
  id BIGSERIAL PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  user_email TEXT NOT NULL,
  contact_type TEXT NOT NULL,
  contact_info TEXT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE outreach_history ENABLE ROW LEVEL SECURITY;

-- Create policy to allow users to see only their own history
CREATE POLICY "Users can view own history" ON outreach_history
  FOR SELECT USING (auth.uid() = user_id);

-- Create policy to allow users to insert their own history
CREATE POLICY "Users can insert own history" ON outreach_history
  FOR INSERT WITH CHECK (auth.uid() = user_id);
```

### 2. Google OAuth Setup

1. **Create a Google Cloud project:**

   - Go to https://console.cloud.google.com
   - Create a new project or select existing one

2. **Enable Google+ API:**

   - Go to APIs & Services → Library
   - Search for "Google+ API" and enable it

3. **Create OAuth 2.0 credentials:**
   - Go to APIs & Services → Credentials
   - Click "Create Credentials" → "OAuth 2.0 Client IDs"
   - Choose "Web application"
   - Add authorized redirect URIs:
     - `https://[YOUR_PROJECT_REF].supabase.co/auth/v1/callback`
     - `http://localhost:3000/auth/callback` (for local development)
   - Copy the Client ID and Client Secret

### 3. Configure Supabase Authentication

1. **Enable Google provider:**

   - In Supabase dashboard, go to Authentication → Providers
   - Find Google and click "Enable"
   - Enter your Google Client ID and Client Secret
   - Save the configuration

2. **Configure redirect URLs:**
   - In Authentication → URL Configuration
   - Set Site URL to your domain (or `http://localhost:3000` for development)
   - Add redirect URLs if needed

### 4. Update the Application

1. **Replace the placeholder credentials in `index.html`:**

   - Find these lines in the JavaScript section:

   ```javascript
   const SUPABASE_URL = "YOUR_SUPABASE_URL";
   const SUPABASE_ANON_KEY = "YOUR_SUPABASE_ANON_KEY";
   ```

   - Replace with your actual Supabase URL and anon key

2. **Test the application:**
   - Open the HTML file in a browser
   - Click "Sign In" to test Google authentication
   - Try making some outreach calls/emails to test history tracking

## Features

- **Real Google Sign-In:** Users can sign in with their Google accounts
- **Persistent History:** Outreach history is saved to Supabase database
- **User Isolation:** Each user only sees their own history
- **Offline Support:** History is also cached locally for immediate access
- **Automatic Sync:** Data syncs between local storage and Supabase

## Troubleshooting

1. **"Sign in not working":**

   - Check browser console for errors
   - Verify Supabase credentials are correct
   - Ensure Google OAuth is properly configured in Supabase

2. **"History not loading":**

   - Check if the database table was created correctly
   - Verify RLS policies are in place
   - Check browser console for Supabase errors

3. **"Redirect errors":**
   - Ensure redirect URLs are correctly configured in both Google Console and Supabase
   - Check that your domain is properly set in Supabase

## Security Notes

- The anon key is safe to use in client-side code
- Row Level Security (RLS) ensures users can only access their own data
- All authentication is handled securely by Supabase
- No sensitive credentials are stored in the client code
