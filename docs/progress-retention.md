# Progress backup and 180-day retention

I2DL stores study progress in the browser first and, after Google sign-in, in
the Firestore document `users/{uid}`. The Firebase Authentication account is
not deleted by this policy.

## User-facing behavior

- A successful sign-in or cloud sync writes `lastActiveAt` with a Firestore
  server timestamp.
- Cloud progress and notes expire after 180 days without a successful sign-in
  or sync.
- Backup downloads contain reviewed questions, the wrong book, notes, spaced
  repetition state, and activity data.
- New versioned backups and the original unversioned JSON exports are both
  accepted on import.
- Legacy cloud documents receive a new 180-day grace period before they become
  eligible for deletion.

## Free cleanup job

`.github/workflows/cleanup-progress.yml` runs on the first day of each month.
It uses normal Firestore reads/deletes and caps each run at 10,000 changes,
leaving headroom below the Spark plan's daily quota. The job deletes only the
progress document; it never calls the Firebase Authentication API.

The scheduled job is dry-run by default. It also records a small monthly commit
on the isolated `maintenance-heartbeat` branch. This keeps repository activity
separate from `main` and prevents an inactive public repository's scheduled
workflow from being silently disabled after 60 days.

## One-time GitHub and Google Cloud setup

Use Google Workload Identity Federation rather than storing a service-account
JSON key:

1. Create a Google Cloud service account for project `i2dl-c79f8`.
2. Grant it Firestore read, write, and delete access. `Cloud Datastore User`
   (`roles/datastore.user`) is the simple built-in role; a custom least-
   privilege role is preferable if one is already used in the project.
3. Enable the IAM Service Account Credentials API, then create a Workload
   Identity Pool/provider restricted to the GitHub repository
   `c0nsTantin77/i2dl-exam-qa` and its default branch. Grant that external
   principal `Workload Identity User` (`roles/iam.workloadIdentityUser`) on the
   service account.
4. Add these GitHub **repository variables**:
   - `GCP_WORKLOAD_IDENTITY_PROVIDER`: full provider resource name.
   - `GCP_SERVICE_ACCOUNT`: service-account email.
   - `PROGRESS_CLEANUP_DRY_RUN`: keep this as `true` during verification.

No service-account key or Firebase secret is stored in GitHub.

If the deployed Firestore security rules use an explicit field allow-list, add
`lastActiveAt` and `retentionPolicyVersion`. For stronger enforcement, require
client writes to satisfy `request.resource.data.lastActiveAt == request.time`;
Firestore resolves the client SDK's server-timestamp sentinel to that server
request time. Review the deployed rules before changing them rather than
blindly replacing them from this repository.

## Safe activation order

1. Merge and deploy the client change so active users begin receiving
   `lastActiveAt`.
2. Run **Clean up inactive progress** manually with `mode=migrate` and
   `dry_run=true`; review the `legacyCandidates` count.
3. Run the same migration with `dry_run=false`. Every legacy document receives
   a fresh server timestamp, so it has the full 180-day grace period.
4. Run `mode=cleanup`, `dry_run=true` and confirm that the initial expired count
   is zero or otherwise expected.
5. Set `PROGRESS_CLEANUP_DRY_RUN=false` to enable monthly deletion.

If migration reports `changeLimitReached=true`, rerun it until
`legacyCandidates` is zero. If it reports `scanLimitReached=true`, temporarily
raise `CLEANUP_MAX_SCANNED` while staying within the Firestore free read quota.

## Local dry-run

A short-lived Google OAuth access token is required:

```bash
FIREBASE_PROJECT_ID=i2dl-c79f8 \
FIRESTORE_ACCESS_TOKEN="$(gcloud auth print-access-token)" \
CLEANUP_MODE=cleanup \
CLEANUP_DRY_RUN=true \
node tools/cleanup-stale-progress.mjs
```

Any non-dry run also requires
`CLEANUP_CONFIRM_PROJECT=i2dl-c79f8`. This second value is an intentional guard
against modifying the wrong Firebase project.
