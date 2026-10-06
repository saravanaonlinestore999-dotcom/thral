<div align="center">
<img width="1200" height="475" alt="GHBanner" src="https://ai.google.dev/static/site-assets/images/share-ais-513315318.png" />
</div>

# Run and deploy your AI Studio app

This contains everything you need to run your app locally.

View your app in AI Studio: https://ai.studio/apps/87340bbd-6180-4c53-b4a4-b96f1f21bdf4

## Run Locally

**Prerequisites:**  Node.js


1. Install dependencies:
   `npm install`
2. Set the `GEMINI_API_KEY` in [.env.local](.env.local) to your Gemini API key
3. Run the app:
   `npm run dev`

## Private Cloud Storage

The app can sync the student roster, cumulative record, teacher register, photos, and school
logo through Supabase. Storage is scalable but not literally unlimited; available capacity and
any charges depend on the Supabase plan.

For privacy, the app source no longer contains the real student roster; a fresh installation
starts empty. Before switching devices or moving from the old local site, use **Export Roster
Backup** in the old app and keep the downloaded JSON private. After signing into the new site,
choose **Import Roster Backup** to load those student details and photos into the private cloud.

1. Create a Supabase project.
2. In the Supabase SQL Editor, run [`supabase/schema.sql`](./supabase/schema.sql). This creates
   the private state table and media bucket with per-account row-level security.
3. Enable Email/Password sign-in in Supabase Authentication, and set the app's Site URL and
   redirect URLs to the URL where this app will run.
4. Create the school-admin user privately in Supabase Authentication → Users, using a unique
   password (at least 12 characters; do not use `admin123`). Disable public sign-ups in
   Authentication settings. Add the account's email to the allowlist in the SQL Editor:

   ```sql
   insert into public.school_admin_allowlist (email)
   values (lower('school-admin@example.com'));
   ```

5. Copy the Project URL and publishable (or legacy anon) key from the Supabase API settings into
   `.env.local`:

   ```dotenv
   VITE_SUPABASE_URL="https://YOUR_PROJECT.supabase.co"
   VITE_SUPABASE_ANON_KEY="YOUR_SUPABASE_PUBLISHABLE_OR_ANON_KEY"
   ```

6. Restart the Vite server and sign in using the allowlisted email and the password created
   privately in Supabase. The public website does not offer account creation. On the first sync,
   the current browser's school data is uploaded. Sign into that same account on each device to
   use the shared copy.

The media bucket is private; the app uploads photos separately from the database and uses
temporary signed URLs to display them. Never put a Supabase `service_role` key in `.env.local` or
any browser code. Do not store student information unless your school is authorized to use the
selected cloud provider and has configured its retention and access policies appropriately.

## Public Website (GitHub Pages)

The repository includes a GitHub Actions workflow to publish the app at
`https://r52069716-tech.github.io/student/`. To activate it:

1. In the sanitized deployment repository's **Settings → Pages**, set the build and deployment source to **GitHub Actions**.
2. In **Settings → Secrets and variables → Actions → Variables** for that public repository, add:
   - `VITE_SUPABASE_URL` — the Supabase Project URL.
   - `VITE_SUPABASE_ANON_KEY` — the Supabase publishable/anon key. This is public client
     configuration; access is enforced by Supabase policies. Never use a service-role key.
   The deployment workflow also requires secure-auth mode; if either value is missing, it stops
   before deployment.
3. Add `https://r52069716-tech.github.io/student-cumulative-record-demo/` to the Supabase Auth Site URL and allowed redirect URLs.
4. Push the workflow and site source to the repository's `main` branch, or run **Deploy public
   website** from the Actions tab.

The website and frontend source are public, but student records and photos remain private behind
Supabase authentication and the school-admin email allowlist. Do not publish a student export,
uploaded photos, service-role key, or passwords to the repository.
