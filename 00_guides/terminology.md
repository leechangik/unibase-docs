# UniBase Platform Terminology

## Purpose

This document serves as the canonical reference for terminology used throughout the UniBase platform. Consistent use of these terms ensures clear communication across teams and documentation.

## Table of Contents

- [Platform Core Concepts](#platform-core-concepts)
- [Technical Terminology](#technical-terminology)
- [Business Terminology](#business-terminology)
- [Game-Specific Terminology](#game-specific-terminology)

## Platform Core Concepts

| Term | Definition |
|------|------------|
| **UniBase** | The overarching platform that enables modular application development and deployment with integrated AI capabilities. |
| **Module** | A self-contained functional unit within the UniBase platform that can be enabled or disabled per application instance. |
| **App Instance** | A deployed application built on the UniBase platform, configured with specific modules and settings. |
| **Core Services** | Fundamental services (authentication, user management, etc.) that are available to all modules. |
| **Service Locator** | The dependency injection system that provides module access to core services and other modules. |

## Technical Terminology

| Term | Definition |
|------|------------|
| **AppRegistry** | The central registry where modules are registered and configured. |
| **ApiClient** | The HTTP client used for making API calls to the backend services. |
| **ApiResponse** | A standardized response format for all API calls, containing success status, data, and error information. |
| **BuildConfig** | Environment-specific configuration for the build process. |
| **Module Activation** | The process of enabling a module within an app instance. |
| **Feature Flag** | A toggle that enables or disables specific features within modules. |

## Business Terminology

| Term | Definition |
|------|------------|
| **Platform Owner** | The organization or team responsible for maintaining the UniBase platform. |
| **App Creator** | An entity that creates applications using the UniBase platform. |
| **End User** | The ultimate consumer of applications built on the UniBase platform. |
| **Tenant** | A logical isolation of data and configuration for a specific customer or organization. |
| **Marketplace** | A directory of available modules that can be integrated into UniBase applications. |

## Game-Specific Terminology

| Term | Definition |
|------|------------|
| **Farm** | A virtual space in the Idle Farm game where users grow crops and raise animals. |
| **Crop Cycle** | The time period from planting to harvesting a crop. |
| **Yield** | The amount of resources produced by a farm activity. |
| **Upgrade Path** | The progression of improvements available for farm elements. |
| **Resource** | Any in-game item that can be collected, spent, or traded. |

## Usage Guidelines

1. Always use the exact term as defined in this document.
2. When introducing new terms, submit them for addition to this document.
3. Terms should be used consistently across code, documentation, and user interfaces.
4. Technical discussions should reference these standardized terms to avoid confusion.

## Term Addition Process

To propose a new term for inclusion in this terminology guide:

1. Submit a pull request with the term and proposed definition
2. Include context and usage examples
3. Terms will be reviewed by the platform architecture team
4. Approved terms will be added in the next documentation update 