---
title: Peregrine Publish Pipeline
inclusion: always
---

# Peregrine Publish Pipeline

This project includes a CI/CD pipeline that packages build artifacts and ships them to Peregrine for hosting and deployment.

## peregrine.json — Fill It In

The file `peregrine.json` at the project root controls what gets packaged and where it goes. **When you have enough context to fill it in, do so immediately.** Do not leave `CHANGE_ME` placeholders.

### When to write peregrine.json

Fill in `peregrine.json` as soon as you know:
- The product name / identifier
- What type of project this is (`landing-page`, `demo`, `onboarding`, or `saas-app`)
- The URL slug for hosting

### Project type → artifact path conventions

| projectType | Default artifactPath | Zip produced | Peregrine deploys to |
|-------------|---------------------|-------------|---------------------|
| `landing-page` | `landing/` | `landing-dist.zip` | `{slug}.{domain}` |
| `demo` | `demo/` | `demo-bundle.zip` | `{slug}.demos.gdna.io` |
| `onboarding` | `onboarding/` | `onboarding-dist.zip` | `{slug}.onboard.gdna.io` |
| `saas-app` | `dist/` | `frontend-dist.zip` | Customer account |

## Rules for the agent

- **Write peregrine.json early.** Replace CHANGE_ME values as soon as you know the product.
- **Match artifactPath to where you put the build output.**
- **Use lowercase hyphenated values** for productId and slug.
- **Don't invent new projectTypes.** Use one of: `landing-page`, `demo`, `onboarding`, `saas-app`.
- **The slug becomes a subdomain.** Keep it short, memorable, and URL-safe.
