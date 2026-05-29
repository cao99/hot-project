# cc-switch - Cloud Configuration Switcher

Cloud Configuration Switcher (cc-switch) is a powerful command-line tool for managing and switching between different cloud service configurations across multiple providers.

## Overview

cc-switch allows you to:
- Manage configurations for AWS, Azure, GCP, and other cloud providers
- Switch between different environments (dev, staging, prod)
- Manage multiple accounts and regions
- Securely store credentials
- Automate configuration switching in CI/CD pipelines

## Quick Start

```bash
# Install cc-switch
npm install -g cc-switch

# Initialize configuration
cc-switch init

# Add a cloud provider configuration
cc-switch add aws --profile dev

# Switch to a configuration
cc-switch use aws-dev

# List all configurations
cc-switch list
```