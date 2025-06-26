# GitHub Actions for Maestro E2E Tests

This repository provides an example of how to use [Maestro](https://docs.maestro.dev/) for end-to-end (E2E) testing of mobile applications, integrated with GitHub Actions for automated CI/CD workflows.

## Overview

- **Maestro**: A simple and effective framework for mobile UI testing.
- **GitHub Actions**: Automate your Maestro E2E tests on every push, pull request, or on a schedule.
- **Example Apps**: Includes sample APK (Android) and .app (iOS) builds for demonstration.
- **Example Tests**: Includes sample tests to run on CI

## Requirements for local runs

- [Maestro CLI](https://docs.maestro.dev/getting-started/installing-maestro)
- Running Android emulator and/or iOS simulator


## How the Pipeline Works

This repository includes a comprehensive GitHub Actions pipeline for running Maestro E2E tests on both Android and iOS platforms, generating test reports, and (optionally) deploying them to AWS S3.

### Workflow Triggers
- **On Push**: Runs on every push to the `main` branch
- **On Schedule**: Runs by cron every night
- **Manual Dispatch**: Can be triggered manually with a platform choice (`android`, `ios`, or `all`).

### Jobs Overview

#### 1. `test-ios`
- **Runs on**: `macos-15` (required for iOS Simulator)
- **Matrix**: Runs tests on multiple iPhone device types (e.g., iPhone 16, iPhone 16 Pro Max).
- **Steps**:
  - Checks out the code.
  - Boots the required iOS Simulator.
  - Installs the provided `.app` bundle.
  - Sets up Java and installs Maestro.
  - Runs Maestro E2E tests, outputs JUnit XML reports.
  - Annotates reports with device info and uploads them as artifacts.

#### 2. `test-android`
- **Runs on**: `ubuntu-latest`
- **Matrix**: Runs tests across multiple Android API levels (26–34) and device architectures.
- **Steps**:
  - Checks out the code.
  - Enables KVM for hardware acceleration.
  - Sets up Java and installs Maestro.
  - Uses the `reactivecircus/android-emulator-runner` to launch emulators.
  - Installs the provided APK.
  - Runs Maestro E2E tests, outputs JUnit XML reports.
  - Annotates reports with API level info and uploads them as artifacts.

#### 3. `generate-report`
- **Runs on**: `ubuntu-latest`
- **Depends on**: Both `test-ios` and `test-android` jobs.
- **Steps**:
  - Downloads all test result artifacts.
  - Merges JUnit XML reports into a single file.
  - Generates an HTML report using `xunit-viewer`.
  - Uploads the HTML report as an artifact.

#### 4. `deploy-report` (Optional, commented out by default)
- **Purpose**: Deploys the generated HTML report to an AWS S3 bucket using a custom composite action (`.github/actions/deploy-report`).
- **Requires**: AWS credentials and S3 bucket configuration via repository secrets and workflow inputs.

### Custom Action: Deploy HTML Report to S3
- Located at `.github/actions/deploy-report`.
- Installs AWS CLI, configures credentials, and uploads HTML reports to the specified S3 bucket.
- Adds links to the uploaded reports in the GitHub Actions summary.

---

## Customization
- Update the workflow file at `.github/workflows/run-maestro-e2e-tests.yml` to adjust device matrices, test paths, or add deployment steps.
- You will probably need to update the app versions by downloading them as artifacts or generating them from the source code.
- To enable S3 deployment, uncomment the `deploy-report` job and provide the required secrets and inputs.

## Resources

- [Maestro Documentation](https://maestro.mobile.dev/getting-started/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
