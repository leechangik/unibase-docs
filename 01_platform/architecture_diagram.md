# UniBase Architecture Diagram

## Purpose

This document provides detailed architectural diagrams of the UniBase platform, illustrating the infrastructure components, service interactions, and data flows. These diagrams serve as reference for both development and operational activities.

## Table of Contents

- [System Context Diagram](#system-context-diagram)
- [Container Diagram](#container-diagram)
- [Component Diagram](#component-diagram)
- [Deployment Architecture](#deployment-architecture)
- [Data Flow Diagrams](#data-flow-diagrams)

## System Context Diagram

The System Context diagram shows UniBase in relation to its users and external systems.

```
                      ┌───────────────────┐
                      │                   │
                      │    End Users      │
                      │                   │
                      └─────────┬─────────┘
                                │
                                ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│                 │    │                 │    │                 │
│  Third-Party    │◄───┤    UniBase      ├───►│   Analytics     │
│  Services       │    │    Platform     │    │   Systems       │
│                 │    │                 │    │                 │
└─────────────────┘    └────────┬────────┘    └─────────────────┘
                                │
                                ▼
                      ┌─────────────────┐
                      │                 │
                      │   Content       │
                      │   Providers     │
                      │                 │
                      └─────────────────┘
```

## Container Diagram

The Container diagram shows the high-level technical architecture of the UniBase platform.

```
┌─────────────────────────────────────────────────────────────┐
│                UniBase Platform                              │
│                                                             │
│  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐  │
│  │             │      │             │      │             │  │
│  │  Mobile     │      │  Web        │      │  Desktop    │  │
│  │  Client     │      │  Client     │      │  Client     │  │
│  │             │      │             │      │             │  │
│  └──────┬──────┘      └──────┬──────┘      └──────┬──────┘  │
│         │                    │                    │         │
│         └──────────┬─────────┴────────┬──────────┘         │
│                    │                  │                     │
│          ┌─────────▼──────────┐      ┌▼────────────────┐   │
│          │                    │      │                  │   │
│          │   API Gateway      │      │  CDN             │   │
│          │                    │      │                  │   │
│          └─────────┬──────────┘      └──────────────────┘   │
│                    │                                        │
│  ┌─────────────────┼─────────────────────────────────────┐  │
│  │                 │       Service Layer                 │  │
│  │  ┌──────────────▼─────────┐   ┌────────────────────┐  │  │
│  │  │                        │   │                    │  │  │
│  │  │   Core Services        │   │  Module Services   │  │  │
│  │  │                        │   │                    │  │  │
│  │  └──────────────┬─────────┘   └──────────┬─────────┘  │  │
│  │                 │                        │            │  │
│  └─────────────────┼────────────────────────┼────────────┘  │
│                    │                        │               │
│  ┌─────────────────┼────────────────────────┼────────────┐  │
│  │                 │      Data Layer        │            │  │
│  │  ┌──────────────▼──────┐   ┌─────────────▼──────────┐ │  │
│  │  │                     │   │                        │ │  │
│  │  │   Database Cluster  │   │  Object Storage       │ │  │
│  │  │                     │   │                        │ │  │
│  │  └─────────────────────┘   └────────────────────────┘ │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Component Diagram

The Component diagram focuses on the internal structure of the Flutter client.

```
┌─────────────────────────────────────────────────────────────┐
│                     Flutter Client                           │
│                                                             │
│  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐  │
│  │             │      │             │      │             │  │
│  │  UI Layer   │──┬──►│  App Core   │◄─┬───│  Modules    │  │
│  │             │  │   │             │  │   │             │  │
│  └─────────────┘  │   └─────────────┘  │   └─────────────┘  │
│                   │                    │                     │
│                   │   ┌─────────────┐  │                     │
│                   └──►│             │  │                     │
│                       │  Services   │◄─┘                     │
│                       │             │                        │
│                       └──────┬──────┘                        │
│                              │                               │
│                       ┌──────▼──────┐                        │
│                       │             │                        │
│                       │ API Client  │                        │
│                       │             │                        │
│                       └──────┬──────┘                        │
│                              │                               │
└──────────────────────────────┼───────────────────────────────┘
                               │
                      ┌────────▼───────┐
                      │                │
                      │  API Gateway   │
                      │                │
                      └────────────────┘
```

## Deployment Architecture

The Deployment diagram shows how UniBase is deployed on AWS infrastructure.

```
┌─────────────────────────────────────────────────────────────┐
│                        AWS Cloud                             │
│                                                             │
│  ┌────────────────────────────────────────────────────────┐ │
│  │                      VPC                               │ │
│  │                                                        │ │
│  │  ┌──────────────┐         ┌──────────────────────┐    │ │
│  │  │ Public Subnet│         │  Private Subnet      │    │ │
│  │  │              │         │                      │    │ │
│  │  │ ┌──────────┐ │         │  ┌──────────────┐    │    │ │
│  │  │ │          │ │         │  │              │    │    │ │
│  │  │ │  ALB     │ │         │  │  ECS/EKS     │    │    │ │
│  │  │ │          │ │         │  │  Containers  │    │    │ │
│  │  │ └─────┬────┘ │         │  │              │    │    │ │
│  │  │       │      │         │  └──────┬───────┘    │    │ │
│  │  └───────┼──────┘         └─────────┼────────────┘    │ │
│  │          │                          │                  │ │
│  │          │                 ┌────────▼──────────────┐   │ │
│  │          │                 │   Private Subnet      │   │ │
│  │          │                 │                       │   │ │
│  │          │                 │   ┌───────────────┐   │   │ │
│  │          └─────────────────┼──►│               │   │   │ │
│  │                            │   │  RDS Cluster  │   │   │ │
│  │                            │   │               │   │   │ │
│  │                            │   └───────────────┘   │   │ │
│  │                            │                       │   │ │
│  │                            │   ┌───────────────┐   │   │ │
│  │                            │   │               │   │   │ │
│  │                            │   │  Redis Cache  │   │   │ │
│  │                            │   │               │   │   │ │
│  │                            │   └───────────────┘   │   │ │
│  │                            └───────────────────────┘   │ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                             │
│  ┌────────────┐    ┌─────────────┐    ┌────────────────┐   │
│  │            │    │             │    │                │   │
│  │ S3 Buckets │    │ CloudFront  │    │ CloudWatch     │   │
│  │            │    │             │    │                │   │
│  └────────────┘    └─────────────┘    └────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Data Flow Diagrams

### User Authentication Flow

```
┌────────┐    1. Login Request    ┌────────────┐
│        ├───────────────────────►│            │
│ Client │                        │ API Gateway │
│        │◄───────────────────────┤            │
└────────┘    2. JWT Token        └──────┬─────┘
                                         │
                                         │ 3. Validate
                                         │
                                    ┌────▼─────┐
                                    │          │
                                    │  Auth    │
                                    │  Service │
                                    │          │
                                    └────┬─────┘
                                         │
                                         │ 4. Lookup
                                         │
                                    ┌────▼─────┐
                                    │          │
                                    │ Database │
                                    │          │
                                    └──────────┘
```

### Farm Game Data Flow

```
┌────────┐    1. Harvest Request    ┌────────────┐
│        ├─────────────────────────►│            │
│ Client │                          │ API Gateway │
│        │◄─────────────────────────┤            │
└────────┘    6. Updated Resources   └──────┬─────┘
                                           │
                                           │ 2. Route
                                           │
                                      ┌────▼─────┐
                                      │          │
                                      │  Farm    │
                                      │  Service │
                                      │          │
                                      └────┬─────┘
                                           │
                                           │ 3. Process
                                           │
                        ┌─────────────────┐│┌─────────────────┐
                        │                 ││                  │
                 4. Read │                 ▼▼                  │ 5. Write
                        │          ┌──────────────┐           │
                        │          │              │           │
                        └─────────►│   Database   ├───────────┘
                                   │              │
                                   └──────────────┘
```

## Notes on Architecture Evolution

The architecture is designed to evolve with the following considerations:

1. **Horizontal Scaling**: All components can scale horizontally to handle increased load.
2. **Microservices Transition**: The service layer is designed to transition from a monolithic to a microservices architecture as needed.
3. **Multi-Region Support**: The architecture can be replicated across AWS regions for global availability.
4. **Hybrid Cloud Options**: While initially AWS-focused, the architecture can adapt to hybrid or multi-cloud scenarios.

## References

- [AWS Architecture Center](https://aws.amazon.com/architecture/)
- [Flutter Architecture Best Practices](https://flutter.dev/docs/resources/architectural-overview)
- [C4 Model for Visualizing Software Architecture](https://c4model.com/) 