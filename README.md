# The Quiet Margin — Supabase edition

## Status (read first)
- DONE: database schema, Row Level Security, storage rules, auth design, `src/api.js` data layer, build config.
- NOT DONE: `index.html` is still the original single-file prototype UI, which uses browser localStorage. Its screens must be rewired to call `src/api.js` (replace the `S`/`save()` calls). Until that is done, this project is NOT a multi-user site. It has not been deployed or tested against a live Supabase project.
- Not built yet: moderator UI (use the Supabase dashboard / SQL), document preview, drafts, named collections, password recovery.

## Setup
1. Create a project at supabase.com.
2. Authentication > Providers: keep Email on, turn **Confirm email OFF**. Authentication > Sign In: turn **Allow anonymous sign-ins ON**. Set minimum password length 8+. Enable a CAPTCHA (Turnstile/hCaptcha) to limit anonymous-account abuse.
3. SQL Editor: paste and run `supabase/schema.sql`.
4. Copy `.env.example` to `.env`; fill URL and anon key (Settings > API). Never use the `service_role` key here.
5. `npm install`, then `npm run dev`; `npm run build` makes `dist/`.
6. Deploy `dist/` (Vercel, Netlify or Cloudflare Pages); set the same three VITE_ variables in the host's settings. Add the site URL under Auth > URL Configuration.
7. First moderator: sign up, then in SQL Editor run `insert into moderators values ('<your profiles.id>');`.

## How the three modes are enforced (in Postgres, not in buttons)
- Member = password account (hidden login id derived from pen name). Temporary poet = anonymous Supabase session; profile expires after 12 h (edit `create_profile`); expired sessions cannot post, edit or like, but their posts remain. Visitor = reads with no login; an anonymous identity appears only when they ask or answer in Questions.
- No delete policies exist on posts, comments or attachments; edits copy the old version to `post_revisions`; accounts cannot be deleted while they own work.
- Limits: 20 posts/h and 60 comments/h per user, uploads 5 MB (images, PDF, text only).

## Known limits
- No email means no password recovery: a forgotten password is unrecoverable. Recovery codes could be added.
- Uploaded files are publicly readable by URL. Backups need Supabase Pro (daily backups / PITR). Orphaned uploads from abandoned posts are not cleaned up.

## Test before launch
Per mode, on phone and desktop: sign up, sign out, sign in on a second device; temp post then set `session_expires_at` to the past and confirm post stays but edit/like fail; visitor blocked from posts/likes/comments outside Questions (try the REST API directly).
