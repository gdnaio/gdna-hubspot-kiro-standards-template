# Requirements Document: Monorepo Migration

## Introduction

This document specifies the requirements for migrating the a10dit event check-in system from a single npm-based project to a modern monorepo architecture using pnpm and Turborepo. The migration will restructure the codebase into separate packages (web, common, infra), deploy the new PWA frontend via AWS Amplify, and retire the legacy S3-based frontend deployment.

## Glossary

- **Monorepo**: A single repository containing multiple packages with shared dependencies
- **pnpm**: Fast, disk-efficient package manager with strict dependency resolution
- **Turborepo**: Build system for managing monorepo tasks and caching
- **PWA_Frontend**: The new Vite + React + TypeScript frontend application
- **Legacy_Frontend**: The old vanilla HTML/CSS/JS frontend deployed to S3
- **Amplify**: AWS service for hosting and deploying web applications with CI/CD
- **CDK**: AWS Cloud Development Kit for infrastructure as code
- **Common_Package**: Shared package containing types and validators used by both frontend and backend
- **Web_Package**: The frontend application package (migrated from pwa-frontend)
- **Infra_Package**: The infrastructure package containing CDK code
- **Workspace**: A package in the pnpm monorepo structure

## Requirements

### Requirement 1: Monorepo Foundation

**User Story:** As a developer, I want to establish a monorepo foundation with pnpm and Turborepo, so that I have efficient dependency management and build orchestration across packages.

#### Acceptance Criteria

1. WHEN the project is initialized, THE System SHALL use pnpm as the package manager with a committed pnpm-lock.yaml file
2. WHEN the project is initialized, THE System SHALL include a pnpm-workspace.yaml file defining the packages directory
3. WHEN the project is initialized, THE System SHALL include a turbo.json file defining the build pipeline with proper task dependencies
4. WHEN the project is initialized, THE System SHALL include a tsconfig.base.json file that all packages extend
5. WHEN the project is initialized, THE System SHALL include a .npmrc file with strict dependency isolation settings (shamefully-hoist=false)
6. WHEN a developer runs `pnpm install`, THE System SHALL install dependencies using pnpm's hard-link strategy
7. WHEN a developer runs `pnpm turbo build`, THE System SHALL build packages in dependency order (common → web, infra)

### Requirement 2: Package Structure

**User Story:** As a developer, I want the codebase organized into separate packages (web, common, infra), so that concerns are properly separated and code can be shared efficiently.

#### Acceptance Criteria

1. THE System SHALL have a packages/web directory containing the migrated PWA frontend application
2. THE System SHALL have a packages/common directory containing shared types and validators
3. THE System SHALL have a packages/infra directory containing AWS CDK infrastructure code
4. WHEN packages/web references shared code, THE System SHALL import from "common" using workspace dependencies
5. WHEN packages/infra references shared code, THE System SHALL import from "common" using workspace dependencies
6. THE packages/common package SHALL have no dependencies on web or infra packages
7. WHEN any package is built, THE System SHALL output to a dist/ directory within that package

### Requirement 3: Shared Types and Validators

**User Story:** As a developer, I want shared types and validators in packages/common, so that frontend and backend stay in sync and validation logic is not duplicated.

#### Acceptance Criteria

1. THE Common_Package SHALL contain TypeScript interfaces for all domain entities (Attendee, Event, PresenterQueue)
2. THE Common_Package SHALL contain Zod schemas for all API request/response validation
3. WHEN the frontend validates form input, THE System SHALL use Zod schemas from Common_Package
4. WHEN the backend validates API requests, THE System SHALL use Zod schemas from Common_Package
5. THE Common_Package SHALL export all types and validators through a single index.ts entry point
6. WHEN Common_Package is modified, THE System SHALL rebuild dependent packages (web, infra) automatically via Turborepo

### Requirement 4: Frontend Migration

**User Story:** As a developer, I want the pwa-frontend migrated to packages/web with proper configuration, so that it builds and runs within the monorepo structure.

#### Acceptance Criteria

1. THE Web_Package SHALL contain all source code from the pwa-frontend directory
2. THE Web_Package SHALL have a package.json with workspace dependency on "common": "workspace:*"
3. THE Web_Package SHALL have a tsconfig.json extending ../../tsconfig.base.json
4. WHEN a developer runs `pnpm turbo dev --filter=web`, THE System SHALL start the Vite development server
5. WHEN a developer runs `pnpm turbo build --filter=web`, THE System SHALL build the production bundle to packages/web/dist
6. THE Web_Package SHALL import shared types and validators using `import { Type } from 'common'` syntax
7. WHEN the Web_Package is built, THE System SHALL include all necessary assets and dependencies in the output

### Requirement 5: Infrastructure as Code

**User Story:** As a developer, I want AWS CDK infrastructure code in packages/infra, so that infrastructure is code-managed, version controlled, and follows gdna standards.

#### Acceptance Criteria

1. THE Infra_Package SHALL contain CDK stacks for all existing AWS resources (DynamoDB tables, Lambda functions, API Gateway)
2. THE Infra_Package SHALL contain a CDK stack for Amplify hosting configuration
3. THE Infra_Package SHALL tag all resources with gdna:project, gdna:environment, gdna:owner, gdna:cost-center tags
4. WHEN a developer runs `pnpm turbo cdk:synth --filter=infra`, THE System SHALL synthesize CloudFormation templates to cdk.out/
5. THE Infra_Package SHALL use L2/L3 CDK constructs instead of L1 CloudFormation primitives where available
6. THE Infra_Package SHALL define separate stacks for data layer, compute layer, and hosting layer
7. WHEN infrastructure is deployed, THE System SHALL maintain all existing resource configurations and connections

### Requirement 6: Amplify Deployment

**User Story:** As a developer, I want the new PWA frontend deployed via Amplify with proper CI/CD, so that I have automated deployments with preview environments.

#### Acceptance Criteria

1. THE System SHALL have an amplify.yml file at the project root configured for monorepo builds
2. WHEN Amplify builds the application, THE System SHALL install pnpm and run `pnpm install --frozen-lockfile`
3. WHEN Amplify builds the application, THE System SHALL run `pnpm turbo build --filter=common` before building web
4. WHEN Amplify builds the application, THE System SHALL run `pnpm turbo build --filter=web` and output artifacts from packages/web/dist
5. THE Amplify application SHALL be configured to deploy the main branch to production at a10dit.com
6. THE Amplify application SHALL have environment variables configured for VITE_API_URL, VITE_COGNITO_USER_POOL_ID, VITE_COGNITO_CLIENT_ID
7. WHEN a pull request is created, THE System SHALL create a preview deployment with a unique URL

### Requirement 7: Legacy Frontend Retirement

**User Story:** As an operator, I want the old S3 frontend deployment retired, so that we only maintain one frontend codebase and reduce operational complexity.

#### Acceptance Criteria

1. WHEN the migration is complete, THE Legacy_Frontend directory SHALL be moved to an archive/ directory
2. WHEN the migration is complete, THE S3 bucket hosting the Legacy_Frontend SHALL have static website hosting disabled
3. WHEN the migration is complete, THE CloudFront distribution pointing to the S3 bucket SHALL be deleted or reconfigured to point to Amplify
4. THE System SHALL maintain a migration log documenting what was archived and when
5. WHEN users access a10dit.com after migration, THE System SHALL serve the PWA_Frontend from Amplify

### Requirement 8: Build and Test Pipeline

**User Story:** As a developer, I want a working build and test pipeline using Turborepo, so that I can validate changes across all packages efficiently.

#### Acceptance Criteria

1. WHEN a developer runs `pnpm turbo build`, THE System SHALL build all packages in dependency order
2. WHEN a developer runs `pnpm turbo test`, THE System SHALL run tests for all packages that have tests
3. WHEN a developer runs `pnpm turbo lint`, THE System SHALL lint all packages
4. WHEN a developer runs `pnpm turbo typecheck`, THE System SHALL type-check all packages
5. THE turbo.json pipeline SHALL cache build outputs and skip rebuilds when inputs haven't changed
6. WHEN Common_Package changes, THE System SHALL invalidate caches for web and infra packages
7. THE System SHALL support running tasks for a single package using --filter flag (e.g., `pnpm turbo test --filter=web`)

### Requirement 9: Documentation and Developer Experience

**User Story:** As a developer, I want clear documentation on the new monorepo structure and setup process, so that I can onboard quickly and understand the architecture.

#### Acceptance Criteria

1. THE System SHALL have a README.md at the project root documenting the monorepo structure
2. THE README.md SHALL include setup instructions for installing pnpm and dependencies
3. THE README.md SHALL include common commands for building, testing, and deploying
4. THE README.md SHALL document the purpose of each package (web, common, infra)
5. WHEN a new developer clones the repository, THE README.md SHALL provide all information needed to run the application locally
6. THE System SHALL have a .env.example file documenting all required environment variables
7. WHEN a developer encounters an error, THE System SHALL provide clear error messages indicating which package failed

### Requirement 10: Zero-Downtime Migration

**User Story:** As an operator, I want zero downtime during the migration, so that users can continue using the application throughout the transition.

#### Acceptance Criteria

1. WHEN the Amplify deployment is being set up, THE Legacy_Frontend SHALL continue serving traffic
2. WHEN the Amplify deployment is ready, THE DNS cutover SHALL happen atomically
3. THE System SHALL maintain backward compatibility with all existing API endpoints during migration
4. WHEN the migration is complete, THE System SHALL verify that all existing functionality works in the new deployment
5. THE System SHALL have a rollback plan documented in case of deployment issues
6. WHEN monitoring the migration, THE System SHALL track error rates and response times to detect issues
