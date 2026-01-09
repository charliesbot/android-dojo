                 ..                     ..
                 .-.                   .-.
                  .-.     .......     --
                   .------------------.
                 .-----------------------.
               .---------------------------.
             .------------------------------.
            .------   --------------   ------.
           .----------------------------------.
           ------------------------------------
           ------------------------------------
           ------------------------------------

# Android Dojo

Personal Android development learning repository and reusable patterns.

## Quick Navigation

- [Architecture Overview](ARCHITECTURE.md) - Simplified multi-module architecture for personal projects
- [GitHub Actions](github-actions/) - Reusable CI/CD templates for Play Store deployment

## GitHub Actions Templates

The `github-actions/` folder contains reusable templates for automated Play Store deployments using GitHub Actions with Workload Identity Federation.

### Contents

| File                                                                                  | Description                                                 |
| ------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| [android-deployment-setup-guide.md](github-actions/android-deployment-setup-guide.md) | Step-by-step guide to set up automated deployments          |
| [google-cloud-setup.md](github-actions/google-cloud-setup.md)                         | Google Cloud project and Workload Identity Federation setup |
| [google-play-console-setup.md](github-actions/google-play-console-setup.md)           | Play Console permissions for service accounts               |
| [workflows/deploy-play-store.yml](github-actions/workflows/deploy-play-store.yml)     | GitHub Actions workflow template                            |
| [whatsnew/](github-actions/whatsnew/)                                                 | Release notes templates (en-US, es-ES)                      |

### Features

- Weekly automated deployments (configurable schedule)
- Manual deployment trigger with version suffix support
- Secure keyless authentication via Workload Identity Federation
- Multi-platform support (phone, Wear OS, TV, Automotive)
- Date-based version numbering (YYMMDDHH format)

### Quick Start

1. Set up Google Cloud project following `google-cloud-setup.md`
2. Configure Play Console permissions using `google-play-console-setup.md`
3. Copy `workflows/deploy-play-store.yml` to your project's `.github/workflows/`
4. Add required secrets to your GitHub repository
5. Customize placeholders in the workflow file

See the [full setup guide](github-actions/android-deployment-setup-guide.md) for detailed instructions.
