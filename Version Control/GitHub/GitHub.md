# GitHub Comprehensive, Structured, and Progressive Learning Roadmap

## From Version Control Hosting Foundations to Advanced Automation, Security, AI-Native Development, and Production Platform Engineering

GitHub is best learned as more than "a place to store code." The progression should cover **account setup → repositories → issues → pull requests → code review → GitHub Actions → CI/CD → packages → security → Copilot → Projects → discussions → wikis → pages → API → enterprise governance → production engineering**.

---

# I. GitHub Foundations

- **1. What GitHub Is**
  - GitHub
  - GitHub history
  - Microsoft acquisition
  - GitHub platform
  - Git hosting
  - Social coding
  - Open source
  - Collaboration
  - GitHub vs GitLab
  - GitHub vs Bitbucket
  - GitHub vs Gitea
  - GitHub use cases
    - Open source
    - Enterprise development
    - DevOps
    - CI/CD
    - Documentation
    - Project management
    - AI-assisted development
  - GitHub in modern software
  - GitHub Universe
  - GitHub Galaxy

- **2. GitHub Account Setup**
  - Account creation
  - Username
  - Email verification
  - Two-factor authentication
  - 2FA
  - SSH keys
  - GPG keys
  - Personal access tokens
  - Fine-grained tokens
  - Profile setup
  - Profile README
  - GitHub Sponsors
  - GitHub Education
  - GitHub Student Developer Pack
  - Account security
  - Account best practices

- **3. GitHub Interface**
  - Dashboard
  - Home page
  - Repository page
  - Code tab
  - Issues tab
  - Pull requests tab
  - Actions tab
  - Projects tab
  - Wiki tab
  - Security tab
  - Insights tab
  - Settings tab
  - Notifications
  - Search
  - Command palette
  - Keyboard shortcuts
  - GitHub Mobile
  - GitHub Desktop
  - GitHub CLI
  - `gh` command
  - Interface best practices

- **4. GitHub CLI**
  - GitHub CLI
  - `gh` command
  - Installation
  - Authentication
  - `gh auth login`
  - `gh auth status`
  - `gh repo`
  - `gh issue`
  - `gh pr`
  - `gh run`
  - `gh workflow`
  - `gh project`
  - `gh api`
  - `gh release`
  - `gh gist`
  - `gh copilot`
  - `gh extension`
  - `gh alias`
  - `gh config`
  - CLI best practices

---

# II. Repositories

- **5. Repository Fundamentals**
  - Repositories
  - Repository creation
  - Repository visibility
    - Public
    - Private
    - Internal
  - Repository templates
  - Repository initialization
  - README
  - LICENSE
  - `.gitignore`
  - Repository settings
  - Repository topics
  - Repository description
  - Repository website
  - Repository social preview
  - Repository best practices

- **6. Repository Structure**
  - Source code
  - Documentation
  - Tests
  - Configuration
  - Scripts
  - Assets
  - `.github/` directory
    - `workflows/`
    - `ISSUE_TEMPLATE/`
    - `PULL_REQUEST_TEMPLATE.md`
    - `CODEOWNERS`
    - `dependabot.yml`
    - `FUNDING.yml`
    - `SECURITY.md`
    - `CONTRIBUTING.md`
    - `CODE_OF_CONDUCT.md`
  - README structure
    - Project name
    - Badges
    - Description
    - Features
    - Quick start
    - Installation
    - Usage
    - Documentation
    - Contributing
    - License
  - Repository structure best practices
  - Essential files
    - README.md
    - LICENSE
    - CONTRIBUTING.md
    - CODE_OF_CONDUCT.md
    - SECURITY.md
    - `.gitignore`
    - `CODEOWNERS`
    - `dependabot.yml`

- **7. Repository Management**
  - Branch management
  - Default branch
  - Branch protection rules
  - Protected branches
  - Required reviews
  - Required status checks
  - Required signatures
  - Linear history
  - Merge strategies
    - Merge commit
    - Squash merge
    - Rebase merge
  - Repository rulesets
  - Repository insights
  - Repository traffic
  - Repository contributors
  - Repository best practices

- **8. Repository Templates**
  - Template repositories
  - Creating templates
  - Using templates
  - Template best practices
  - Repository templates vs forks
  - Repository templates vs generated projects

- **9. Repository Migration**
  - Importing repositories
  - GitHub Importer
  - Migrating from GitLab
  - Migrating from Bitbucket
  - Migrating from SVN
  - Migrating from Mercurial
  - Repository migration best practices

---

# III. Issues and Project Management

- **10. Issues Fundamentals**
  - Issues
  - Issue creation
  - Issue title
  - Issue description
  - Issue assignees
  - Issue labels
  - Issue milestones
  - Issue projects
  - Issue comments
  - Issue reactions
  - Issue pinning
  - Issue closing
  - Issue reopening
  - Issue best practices

- **11. Issue Labels**
  - Labels
  - Default labels
  - Custom labels
  - Label colors
  - Label descriptions
  - Label usage
  - Label best practices
  - Common labels
    - `bug`
    - `enhancement`
    - `feature`
    - `documentation`
    - `good first issue`
    - `help wanted`
    - `priority:high`
    - `priority:low`
    - `duplicate`
    - `invalid`
    - `wontfix`

- **12. Issue Milestones**
  - Milestones
  - Milestone creation
  - Milestone due dates
  - Milestone progress
  - Milestone closing
  - Milestone best practices

- **13. Issue Templates**
  - Issue templates
  - Template configuration
  - Bug report template
  - Feature request template
  - Custom templates
  - Template best practices
  - Template YAML
  - Template markdown

- **14. Issue Automation**
  - Issue automation
  - GitHub Actions for issues
  - Issue assignment
  - Issue labeling
  - Issue closing
  - Issue comments
  - Issue best practices

- **15. GitHub Projects**
  - GitHub Projects
  - Projects v2
  - Project creation
  - Project views
    - Board view
    - Table view
    - Timeline view
    - Roadmap view
  - Project fields
    - Status
    - Priority
    - Sprint
    - Assignee
    - Labels
    - Milestones
  - Project automation
  - Project workflows
  - Project insights
  - Project best practices
  - Project templates
  - Project linking to issues and PRs

- **16. GitHub Discussions**
  - Discussions
  - Discussion categories
  - Discussion creation
  - Discussion replies
  - Discussion answers
  - Discussion polls
  - Discussion announcements
  - Discussion best practices
  - Discussions vs issues
  - Discussions vs comments

- **17. GitHub Wikis**
  - Wikis
  - Wiki creation
  - Wiki pages
  - Wiki sidebar
  - Wiki footer
  - Wiki editing
  - Wiki best practices
  - Wiki vs README
  - Wiki vs documentation

---

# IV. Pull Requests and Code Review

- **18. Pull Requests Fundamentals**
  - Pull requests
  - PR creation
  - PR title
  - PR description
  - PR assignees
  - PR reviewers
  - PR labels
  - PR milestones
  - PR projects
  - PR linked issues
  - PR comments
  - PR reviews
  - PR draft status
  - PR ready for review
  - PR merge
  - PR closing
  - PR best practices

- **19. Pull Request Templates**
  - PR templates
  - Template creation
  - Template configuration
  - Template content
  - Template best practices
  - Template YAML
  - Template markdown

- **20. Code Review**
  - Code review
  - Review requests
  - Review comments
  - Review suggestions
  - Review approvals
  - Review changes requested
  - Review dismissal
  - Review re-request
  - Review best practices
  - Inline comments
  - File comments
  - Line comments
  - Suggestion commits
  - Multi-line comments
  - Review checklist
  - Review etiquette
  - Review automation
  - Auto code review
  - Copilot code review
  - Agentic code review

- **21. Code Owners**
  - CODEOWNERS
  - Code owner syntax
  - Code owner patterns
  - Code owner assignments
  - Code owner reviews
  - Code owner best practices

- **22. Pull Request Automation**
  - PR automation
  - Auto-assign
  - Auto-label
  - Auto-merge
  - Merge queue
  - PR best practices

- **23. Merge Strategies**
  - Merge commit
  - Squash merge
  - Rebase merge
  - Merge queue
  - Branch protection
  - Required checks
  - Required reviews
  - Merge best practices
  - Merge conflicts
  - Conflict resolution

---

# V. GitHub Actions

- **24. GitHub Actions Fundamentals**
  - GitHub Actions
  - Workflows
  - Jobs
  - Steps
  - Actions
  - Runners
  - Events
  - Triggers
  - Workflow syntax
  - YAML
  - `.github/workflows/`
  - Actions best practices

- **25. Workflow Syntax**
  - `name`
  - `on`
  - `jobs`
  - `steps`
  - `uses`
  - `run`
  - `with`
  - `env`
  - `secrets`
  - `if`
  - `needs`
  - `strategy`
  - `matrix`
  - `services`
  - `container`
  - `timeout-minutes`
  - `continue-on-error`
  - `outputs`
  - `permissions`
  - Workflow syntax best practices

- **26. Events and Triggers**
  - `push`
  - `pull_request`
  - `pull_request_target`
  - `schedule`
  - `workflow_dispatch`
  - `repository_dispatch`
  - `release`
  - `issues`
  - `issue_comment`
  - `pull_request_review`
  - `pull_request_review_comment`
  - `discussion`
  - `discussion_comment`
  - `create`
  - `delete`
  - `fork`
  - `watch`
  - `star`
  - `page_build`
  - `deployment`
  - `deployment_status`
  - `check_run`
  - `check_suite`
  - `status`
  - Event filters
  - Branch filters
  - Path filters
  - Tag filters
  - Event best practices

- **27. Jobs and Steps**
  - Jobs
  - Steps
  - Job dependencies
  - Job outputs
  - Job conditionals
  - Job matrix
  - Job strategy
  - Job containers
  - Job services
  - Job runners
  - Job best practices

- **28. Runners**
  - GitHub-hosted runners
    - `ubuntu-latest`
    - `windows-latest`
    - `macos-latest`
    - `ubuntu-22.04`
    - `ubuntu-24.04`
    - `windows-2022`
    - `windows-2025`
    - `macos-13`
    - `macos-14`
    - `macos-15`
  - Self-hosted runners
  - Runner groups
  - Runner scaling
  - Runner security
  - Runner best practices

- **29. Actions Marketplace**
  - Actions Marketplace
  - Official actions
    - `actions/checkout`
    - `actions/setup-node`
    - `actions/setup-python`
    - `actions/setup-java`
    - `actions/setup-dotnet`
    - `actions/cache`
    - `actions/upload-artifact`
    - `actions/download-artifact`
    - `actions/github-script`
    - `actions/create-release`
    - `actions/upload-release-asset`
  - Community actions
  - Action versions
  - Action security
  - Action best practices

- **30. CI/CD Workflows**
  - CI pipeline
  - CD pipeline
  - Build
  - Test
  - Lint
  - Security scan
  - Deploy
  - Release
  - CI/CD best practices
  - Example CI workflow
  - Example CD workflow
  - Example release workflow
  - Matrix builds
  - Caching
  - Artifacts

- **31. Secrets and Variables**
  - Secrets
  - Repository secrets
  - Environment secrets
  - Organization secrets
  - Environment variables
  - Configuration variables
  - Secret management
  - Secret best practices
  - Secret rotation
  - Secret scanning

- **32. Environments**
  - Environments
  - Environment protection rules
  - Required reviewers
  - Wait timer
  - Deployment branches
  - Environment secrets
  - Environment variables
  - Environment best practices

- **33. Reusable Workflows**
  - Reusable workflows
  - Workflow calls
  - `workflow_call`
  - Inputs
  - Secrets
  - Outputs
  - Reusable workflow best practices

- **34. Composite Actions**
  - Composite actions
  - Action metadata
  - `action.yml`
  - Composite action steps
  - Composite action best practices

- **35. Custom Actions**
  - JavaScript actions
  - Docker actions
  - Composite actions
  - Action metadata
  - Action inputs
  - Action outputs
  - Action best practices

- **36. Caching**
  - Dependency caching
  - `actions/cache`
  - Cache keys
  - Cache restore keys
  - Cache scopes
  - Caching best practices

- **37. Artifacts**
  - Build artifacts
  - `actions/upload-artifact`
  - `actions/download-artifact`
  - Artifact retention
  - Artifact best practices

- **38. Debugging Workflows**
  - Workflow logs
  - Step logs
  - Debug logging
  - `ACTIONS_STEP_DEBUG`
  - `ACTIONS_RUNNER_DEBUG`
  - Workflow debugging best practices

- **39. Security Hardening**
  - Action pinning
  - SHA pinning
  - Permissions
  - Least privilege
  - Secret scanning
  - Dependency scanning
  - Security hardening best practices

---

# VI. GitHub Packages

- **40. GitHub Packages Fundamentals**
  - GitHub Packages
  - Package registries
    - npm
    - Docker
    - RubyGems
    - Apache Maven
    - Gradle
    - NuGet
  - Package hosting
  - Package management
  - Package best practices

- **41. Container Registry**
  - GitHub Container Registry
  - GHCR
  - `ghcr.io`
  - Docker images
  - OCI images
  - Image publishing
  - Image pulling
  - Image tagging
  - Image linking
  - Container registry best practices
  - Authentication
  - Personal access tokens
  - GitHub Actions tokens
  - `GITHUB_TOKEN`

- **42. Publishing Packages**
  - Publishing npm packages
  - Publishing Docker images
  - Publishing Maven packages
  - Publishing NuGet packages
  - Publishing RubyGems
  - Publishing best practices
  - Workflow integration
  - Package permissions

- **43. Package Management**
  - Package versions
  - Package deletion
  - Package access control
  - Package visibility
  - Package linking
  - Package best practices

---

# VII. GitHub Security

- **44. Security Fundamentals**
  - Security
  - Threat modeling
  - Attack surface
  - Defense in depth
  - Least privilege
  - Secure defaults
  - Security best practices

- **45. GitHub Advanced Security**
  - GitHub Advanced Security
  - GHAS
  - GitHub Secret Protection
  - GitHub Code Security
  - Security overview
  - Security best practices

- **46. Secret Scanning**
  - Secret scanning
  - Secret detection
  - Secret alerts
  - Secret revocation
  - Push protection
  - Secret scanning best practices
  - AI-detected secrets
  - Custom patterns
  - Secret scanning partners

- **47. Dependabot**
  - Dependabot
  - Dependabot alerts
  - Dependabot security updates
  - Dependabot version updates
  - `dependabot.yml`
  - Dependency graph
  - Dependency review
  - Dependabot best practices

- **48. Code Scanning**
  - Code scanning
  - CodeQL
  - CodeQL analysis
  - Code scanning alerts
  - Code scanning workflows
  - Code scanning best practices
  - SARIF
  - Code scanning tools

- **49. Copilot Autofix**
  - Copilot Autofix
  - AI-powered fixes
  - Vulnerability remediation
  - Autofix best practices

- **50. Security Advisories**
  - Security advisories
  - Vulnerability disclosure
  - Advisory creation
  - Advisory management
  - Advisory best practices

- **51. Security Policies**
  - Security policies
  - `SECURITY.md`
  - Security best practices
  - Security auditing
  - Security compliance

- **52. Security Automation**
  - Security automation
  - GitHub Actions for security
  - CodeQL actions
  - Dependabot automation
  - Secret scanning automation
  - Security automation best practices

---

# VIII. GitHub Copilot and AI

- **53. GitHub Copilot Fundamentals**
  - GitHub Copilot
  - AI pair programmer
  - Copilot features
  - Copilot chat
  - Copilot completions
  - Copilot for individuals
  - Copilot for business
  - Copilot for enterprise
  - Copilot best practices

- **54. Copilot Chat**
  - Copilot Chat
  - Chat interface
  - Code explanation
  - Code generation
  - Code refactoring
  - Code debugging
  - Code documentation
  - Chat best practices

- **55. Copilot Agent Mode**
  - Agent mode
  - Autonomous tasks
  - Multi-step tasks
  - Code changes
  - Test running
  - Error fixing
  - Agent mode best practices

- **56. Copilot Coding Agent**
  - Coding agent
  - Issue assignment
  - Autonomous code changes
  - Pull request creation
  - Coding agent best practices

- **57. Copilot Code Review**
  - Copilot code review
  - AI-generated reviews
  - Review suggestions
  - Agentic code review
  - Code review best practices

- **58. Copilot CLI**
  - Copilot CLI
  - Command-line assistance
  - Copilot CLI best practices

- **59. Copilot in IDE**
  - VS Code
  - Visual Studio
  - JetBrains
  - Neovim
  - Copilot in IDE best practices

- **60. Copilot Extensions**
  - Copilot Extensions
  - Custom agents
  - MCP servers
  - Copilot Extensions best practices

- **61. Agent HQ**
  - Agent HQ
  - Agent orchestration
  - Agent collaboration
  - Agent governance
  - Agent best practices
  - Enterprise Agent Control Plane
  - Agent metrics

- **62. AI-Native Development**
  - AI-native development
  - AI-assisted coding
  - AI code review
  - AI testing
  - AI documentation
  - AI best practices

---

# IX. GitHub Pages and Documentation

- **63. GitHub Pages Fundamentals**
  - GitHub Pages
  - Static site hosting
  - Pages setup
  - Pages configuration
  - Custom domains
  - HTTPS
  - Jekyll
  - Static site generators
  - Pages best practices

- **64. Jekyll**
  - Jekyll
  - Jekyll themes
  - Jekyll configuration
  - Jekyll plugins
  - Jekyll best practices

- **65. Documentation**
  - Documentation
  - README
  - Wiki
  - GitHub Pages
  - Documentation best practices
  - Documentation tools
  - MkDocs
  - Docusaurus
  - VitePress

---

# X. GitHub API and Integrations

- **66. GitHub API Fundamentals**
  - GitHub API
  - REST API
  - GraphQL API
  - API versions
  - API authentication
  - API rate limits
  - API best practices

- **67. REST API**
  - REST API
  - Endpoints
  - Resources
  - HTTP methods
  - Status codes
  - Pagination
  - Rate limiting
  - Authentication
  - REST API best practices

- **68. GraphQL API**
  - GraphQL API
  - GraphQL queries
  - GraphQL mutations
  - GraphQL schema
  - GraphQL introspection
  - GraphQL best practices
  - GraphQL vs REST

- **69. GitHub Apps**
  - GitHub Apps
  - App creation
  - App permissions
  - App authentication
  - App webhooks
  - App best practices

- **70. OAuth Apps**
  - OAuth Apps
  - OAuth flow
  - OAuth scopes
  - OAuth best practices

- **71. Webhooks**
  - Webhooks
  - Webhook events
  - Webhook configuration
  - Webhook security
  - Webhook best practices

- **72. GitHub Marketplace**
  - GitHub Marketplace
  - Actions
  - Apps
  - Marketplace best practices

- **73. Integrations**
  - Slack
  - Microsoft Teams
  - Jira
  - Linear
  - Azure Boards
  - Notion
  - Integration best practices

---

# XI. GitHub Enterprise

- **74. GitHub Enterprise Fundamentals**
  - GitHub Enterprise
  - GitHub Enterprise Cloud
  - GHEC
  - GitHub Enterprise Server
  - GHES
  - Enterprise features
  - Enterprise best practices

- **75. Enterprise Governance**
  - Enterprise governance
  - Enterprise teams
  - Enterprise roles
  - Enterprise policies
  - Enterprise security
  - Enterprise compliance
  - Enterprise best practices

- **76. Enterprise Security**
  - Enterprise security
  - Enterprise Security Manager
  - ESM
  - Security overview
  - Security policies
  - Security compliance
  - Security best practices

- **77. Enterprise Management**
  - User management
  - Organization management
  - Repository management
  - Billing
  - Usage metrics
  - Enterprise best practices

- **78. Enterprise Compliance**
  - Compliance
  - Audit logs
  - Data residency
  - SOC 2
  - ISO 27001
  - GDPR
  - HIPAA
  - Compliance best practices

- **79. Enterprise Migration**
  - Migration
  - GitHub Enterprise Importer
  - Migration from GitHub.com
  - Migration from other platforms
  - Migration best practices

---

# XII. GitHub Projects by Difficulty

## Beginner Projects

- **1. Personal Repository**
  - Repository creation
  - README
  - Commits
  - Branches
  - Push

- **2. Open Source Contribution**
  - Forking
  - Pull requests
  - Code review
  - Issues

- **3. Documentation Site**
  - GitHub Pages
  - Jekyll
  - Markdown
  - Custom domain

- **4. GitHub Profile**
  - Profile README
  - Pinned repositories
  - Profile best practices

- **5. GitHub Actions Workflow**
  - Basic CI
  - Testing
  - Build
  - Artifacts

---

## Intermediate Projects

- **6. CI/CD Pipeline**
  - GitHub Actions
  - Build
  - Test
  - Security scan
  - Deploy

- **7. Package Publishing**
  - GitHub Packages
  - npm
  - Docker
  - Versioning

- **8. Security Hardening**
  - Dependabot
  - Secret scanning
  - Code scanning
  - Copilot Autofix

- **9. Project Management**
  - GitHub Projects
  - Issues
  - Labels
  - Milestones
  - Automation

- **10. Copilot Integration**
  - Copilot Chat
  - Agent mode
  - Code review
  - Best practices

---

## Advanced Projects

- **11. Enterprise Governance**
  - Enterprise teams
  - Roles
  - Policies
  - Compliance
  - Audit logs

- **12. API Integration**
  - REST API
  - GraphQL API
  - Webhooks
  - GitHub Apps

- **13. Monorepo with Actions**
  - Monorepo
  - Matrix builds
  - Caching
  - Reusable workflows

- **14. Security Automation**
  - CodeQL
  - Dependabot
  - Secret scanning
  - Security workflows

- **15. AI-Native Development**
  - Agent HQ
  - Copilot agents
  - Custom agents
  - MCP servers

---

## Expert Projects

- **16. Enterprise Platform**
  - GitHub Enterprise
  - Governance
  - Security
  - Compliance
  - Migration

- **17. Custom GitHub App**
  - GitHub App
  - OAuth
  - Webhooks
  - API integration

- **18. Multi-Repository Automation**
  - GitHub Actions
  - Reusable workflows
  - Composite actions
  - Cross-repo automation

- **19. Security Operations**
  - Security overview
  - Alert management
  - Incident response
  - Compliance reporting

- **20. AI Agent Orchestration**
  - Agent HQ
  - Multi-agent workflows
  - Agent governance
  - Agent metrics

---

# XIII. Progressive GitHub Learning Sequence

## Level 1 — GitHub Fundamentals

- Master:
  - Account setup
  - Repositories
  - README
  - Issues
  - Pull requests
  - GitHub CLI
  - GitHub Desktop
  - GitHub Mobile

## Level 2 — Collaboration

- Master:
  - Branching
  - Merging
  - Code review
  - Code owners
  - Pull request templates
  - Issue templates
  - Labels
  - Milestones

## Level 3 — Project Management

- Master:
  - GitHub Projects
  - Project views
  - Project fields
  - Project automation
  - Discussions
  - Wikis

## Level 4 — GitHub Actions

- Master:
  - Workflows
  - Jobs
  - Steps
  - Actions
  - Triggers
  - Runners
  - Secrets
  - Environments
  - Caching
  - Artifacts
  - Reusable workflows
  - Composite actions
  - Custom actions

## Level 5 — CI/CD

- Master:
  - CI pipelines
  - CD pipelines
  - Build
  - Test
  - Lint
  - Security scan
  - Deploy
  - Release
  - Matrix builds
  - Debugging workflows

## Level 6 — Packages

- Master:
  - GitHub Packages
  - Container Registry
  - Publishing packages
  - Package management
  - Package access control

## Level 7 — Security

- Master:
  - GitHub Advanced Security
  - Secret scanning
  - Dependabot
  - Code scanning
  - CodeQL
  - Copilot Autofix
  - Security advisories
  - Security policies
  - Security automation

## Level 8 — AI and Copilot

- Master:
  - GitHub Copilot
  - Copilot Chat
  - Agent mode
  - Coding agent
  - Code review
  - Copilot CLI
  - Copilot Extensions
  - Agent HQ
  - AI-native development

## Level 9 — API and Integrations

- Master:
  - GitHub API
  - REST API
  - GraphQL API
  - GitHub Apps
  - OAuth Apps
  - Webhooks
  - GitHub Marketplace
  - Integrations

## Level 10 — Enterprise

- Master:
  - GitHub Enterprise
  - Enterprise governance
  - Enterprise teams
  - Enterprise roles
  - Enterprise security
  - Enterprise compliance
  - Enterprise migration
  - Enterprise management

## Level 11 — Production Engineering

- Master:
  - Monorepo management
  - Multi-repository automation
  - Security operations
  - AI agent orchestration
  - Platform engineering
  - Enterprise architecture
  - Production best practices

---

# XIV. Final GitHub Competency Map

- **Foundations**

  - Account setup
  - Repositories
  - README
  - GitHub CLI
  - GitHub Desktop
  - GitHub Mobile
  - Interface

- **Collaboration**

  - Issues
  - Pull requests
  - Code review
  - Code owners
  - Templates
  - Labels
  - Milestones
  - Branching
  - Merging

- **Project Management**

  - GitHub Projects
  - Project views
  - Project fields
  - Project automation
  - Discussions
  - Wikis

- **Actions**

  - Workflows
  - Jobs
  - Steps
  - Actions
  - Triggers
  - Runners
  - Secrets
  - Environments
  - Caching
  - Artifacts
  - Reusable workflows
  - Composite actions
  - Custom actions

- **CI/CD**

  - CI pipelines
  - CD pipelines
  - Build
  - Test
  - Lint
  - Security scan
  - Deploy
  - Release
  - Matrix builds
  - Debugging workflows

- **Packages**

  - GitHub Packages
  - Container Registry
  - Publishing packages
  - Package management

- **Security**

  - GitHub Advanced Security
  - Secret scanning
  - Dependabot
  - Code scanning
  - CodeQL
  - Copilot Autofix
  - Security advisories
  - Security policies
  - Security automation

- **Copilot and AI**

  - GitHub Copilot
  - Copilot Chat
  - Agent mode
  - Coding agent
  - Code review
  - Copilot CLI
  - Copilot Extensions
  - Agent HQ
  - AI-native development

- **API**

  - GitHub API
  - REST API
  - GraphQL API
  - GitHub Apps
  - OAuth Apps
  - Webhooks
  - GitHub Marketplace

- **Enterprise**

  - GitHub Enterprise
  - Enterprise governance
  - Enterprise teams
  - Enterprise roles
  - Enterprise security
  - Enterprise compliance
  - Enterprise migration

- **Production**

  - Monorepo management
  - Multi-repository automation
  - Security operations
  - AI agent orchestration
  - Platform engineering
  - Enterprise architecture

---

## Recommended Overall Progression

**GitHub Fundamentals → Collaboration → Project Management → GitHub Actions → CI/CD → Packages → Security → Copilot and AI → API and Integrations → Enterprise → Production Engineering**

For maximum practical mastery, combine this GitHub roadmap with the Git, DSA, JavaScript, TypeScript, Node.js, REST API, SQL, Discrete Mathematics, React, Laravel, jQuery, Jupyter, Python, Java, C#, C++, C Language, Dart, Flutter, Kotlin, and R Language roadmaps above so the progression becomes:

**Discrete Mathematics → DSA Foundations → Git Fundamentals → GitHub Fundamentals → Collaboration → Pull Requests → Code Review → GitHub Actions → CI/CD → GitHub Packages → Security → GitHub Advanced Security → Dependabot → CodeQL → GitHub Copilot → Agent HQ → AI-Native Development → GitHub API → GitHub Apps → Enterprise Governance → Platform Engineering → Enterprise Architecture → Production GitHub Engineering → Open Source Contribution → DevOps Engineering → Platform Engineering.**