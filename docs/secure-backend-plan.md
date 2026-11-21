# Way To Jannah — Secure Backend Plan

Status: design only. No database or production resources have been created or changed.

## Safety boundaries
- Keep the existing `main` branch and production deployment unchanged until preview testing passes.
- Never connect this project to the SmartReach Supabase project.
- Use a dedicated Supabase project for Way To Jannah only, after the owner selects the organization and approves any non-zero recurring or one-time cost.
- Never place a Supabase service-role/secret key in browser code or any `NEXT_PUBLIC_*` variable.

## Current frontend risks
The current single-file demo stores profiles, quiz results, claims, proof images and winner history in browser LocalStorage. Browser storage is per-device, editable by the visitor, and is not a shared or secure admin database. The admin password is embedded in client-side JavaScript. Quiz answers and scoring logic are also delivered to the browser, so scores cannot be trusted for a real prize contest.

## Recommended player experience
1. Let visitors read the rules and play using a random server-issued attempt token; collect only a display name and optional city for the public leaderboard.
2. Do not request phone, full address, pincode, UPI ID or proof image before a player is provisionally selected as a winner.
3. After an eligible result is selected, invite that person to submit the minimum claim details over HTTPS. Explain why each field is needed and how long it is retained.
4. Keep all claim details private to authorized admins. Public pages must never return phone, address, pincode, UPI ID, proof object paths or auth identifiers.
5. Display only a moderated display name/city, score and rank on the public leaderboard. Provide a clear correction/removal contact and contest rules.

## Proposed data model
- `contest_weeks`: week number, title, prize description/value, start/end timestamps, status and published rules.
- `question_bank`: week id, question text, options, answer key and explanation. Answer keys are server-only.
- `quiz_attempts`: opaque attempt id, week id, player auth/anonymous identity, started/submitted timestamps, status, score and elapsed time. Enforce one eligible attempt per identity/week server-side.
- `attempt_answers`: attempt id, question id, selected option and server-recorded answer time. Do not expose answer keys to public roles.
- `public_profiles`: owner id, display name, optional city, moderation state and consent timestamp. No contact details.
- `prize_claims`: winner/attempt id, claimant id, phone, delivery address, pincode, payout/delivery method, UPI only if needed, status and admin notes. Never public.
- `proof_uploads`: claim id, private storage object path, moderation status and timestamps. Store files in a private bucket; issue short-lived signed URLs only to authorized admins.
- `admin_audit_log`: admin id, action, target id and timestamp for sensitive actions.

## Access and scoring model
- Enable RLS on every exposed table.
- Public/anonymous users may read only published contest metadata and the approved public leaderboard projection.
- A player may read/update only their own limited profile and claim status; no player may read another player's private claim.
- No client role may read question answer keys, write scores, declare winners, or modify prize status.
- Submit answers and finalize attempts through server-side code that validates the attempt, timing, question order, one-attempt rule and score. The browser must not calculate an authoritative score.
- Admin actions must use verified server-side admin authorization and audit logging. Never rely on a password embedded in HTML or user-editable metadata.
- Apply rate limits, bot protection, input validation, file type/size limits and retention/deletion rules.

## Delivery sequence
1. Approve a separate backend project and its cost/organization.
2. Create the dedicated project only after any required cost approval; confirm it is healthy.
3. Apply and verify schema, RLS, storage policies and server-side functions.
4. Integrate the preview branch using publishable client keys only; keep secrets in server-side environment variables.
5. Test with two independent player accounts plus an admin account: cross-user reads, score tampering, duplicate attempts, claim privacy, proof access and admin actions.
6. Run security/performance advisors and fix findings.
7. Only after successful preview acceptance, merge to `main` and promote production.

## Launch blockers
Do not advertise real cash/gift prizes or accept real personal/payment details until server-side scoring, secure claims, admin access, storage privacy, contest rules and operational winner verification are implemented and tested.
