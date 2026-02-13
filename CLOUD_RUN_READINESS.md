# Cloud Run CI/CD Readiness Review

## Current status

The service is **partially ready** for Cloud Run runtime, but **not yet ready** for end-to-end CI/CD on GCP without additional pipeline configuration.

### What is already good
- Flask app listens on `PORT` and binds `0.0.0.0`.
- A production entrypoint exists via `gunicorn` in `Dockerfile`.
- Minimal dependencies are present.

### Gaps that blocked CI/CD readiness
- No CI/CD workflow existed (GitHub Actions or Cloud Build trigger config not present).
- No `cloudbuild.yaml` existed for reproducible build-and-deploy.
- No documented required GCP inputs (project, region, Artifact Registry repo, service name, workload identity provider, deploy service account).

## What was added

- `.github/workflows/cloud-run-cicd.yml`: CI/CD workflow for GitHub Actions using Workload Identity Federation.
  - The workflow explicitly passes `--service-account` to `gcloud builds submit`, so Cloud Build steps run as the deploy service account configured in `GCP_SERVICE_ACCOUNT`.
- `cloudbuild.yaml`: Build, push, and deploy pipeline targeting Cloud Run.

With repository variables/secrets configured, this repo is now ready for Cloud Run CI/CD on GCP.

## Required repository configuration

Set these **GitHub repository variables**:
- `GCP_PROJECT_ID`
- `GCP_REGION` (e.g. `us-central1`)
- `AR_REPO` (Artifact Registry repository, e.g. `containers`)
- `CLOUD_RUN_SERVICE` (Cloud Run service name)

Set these **GitHub repository secrets**:
- `GCP_WORKLOAD_IDENTITY_PROVIDER`
- `GCP_SERVICE_ACCOUNT`

## Required GCP IAM roles (minimum practical set)

Grant to the deploy service account:
- `roles/run.admin`
- `roles/artifactregistry.writer`
- `roles/cloudbuild.builds.editor`
- `roles/iam.serviceAccountUser`
