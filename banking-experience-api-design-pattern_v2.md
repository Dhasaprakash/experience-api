# Banking Experience API Design Pattern: Home/Landing Page Journey

## Problems

This design pattern addresses several key challenges in the current banking architecture, specifically for home/landing page journeys:

1. **Journey Fragmentation**: Mobile and web applications must invoke multiple disjointed APIs (accounts, transactions, offers, notifications) to populate a single home/landing page, creating unnecessary complexity and performance bottlenecks.

2. **Security Inconsistency**: Different authentication and authorization mechanisms across channels lead to potentially inconsistent security controls and compliance issues.

3. **Business Logic Redundancy**: The same business logic (e.g., calculating account balances, determining offers relevancy) is duplicated across different client channels.

4. **Incorrect Orchestration Layer**: Business logic orchestration happens within each client rather than in the Process API layer, creating inconsistencies and making updates difficult.

5. **Client Complexity**: Clients are burdened with coordinating multiple API calls with various error handling strategies and retries, increasing development complexity.

6. **Data Redundancy**: Multiple API calls result in overlapping data being transferred repeatedly to assemble the home page view.

7. **Personalization Challenges**: Difficulty in implementing consistent personalization logic across channels for tailored home/landing page experiences.

## Context

The bank operates internationally, serving distinct customer segments:

- **Retail Banking Customers**: Primarily concerned with everyday banking needs
- **Affluent Banking Customers**: Higher-value clients with more complex financial requirements and personalized service

The bank offers services across various domains:
- Current Accounts and Savings Accounts (CASA)
- Fixed and Term Deposits
- Investment Products
- Personal and Business Loans
- Credit and Debit Cards
- Domestic and International Payments

The existing architecture has evolved with:
- Separate mobile and web applications with their own channel-specific APIs
- Diverse core banking systems across regions
- Siloed development teams for different channels
- Inconsistent personalization approaches for home/landing pages

The bank has adopted an API-led connectivity approach with three distinct layers:
1. **Experience APIs** (EAPI): Client-facing APIs designed for specific user experiences
2. **Process APIs** (PAPI): Orchestrating business processes and logic
3. **System APIs** (SAPI): Abstracting underlying system complexities

For the home/landing page journey specifically, mobile apps currently make 5-8 separate API calls to assemble all required data, while web applications make 4-6 calls, with significant redundancy between implementations.

## Scope

This design pattern focuses on creating an Experience API specifically for the home/landing page journey. It covers:

- The design principles for a unified home/landing page Experience API serving both retail and affluent segments
- Integration patterns between this Experience API and various Process APIs
- Security patterns for consistent authentication and authorization
- Response aggregation and transformation tailored for home page contexts
- Personalization strategies for different user segments
- Caching mechanisms to optimize performance

The pattern does not cover:
- Specific implementation technologies or frameworks
- Detailed API specifications or schemas
- Internal Process API or System API designs
- DevOps practices for API deployment

## Solution

### Experience API Design Pattern: Unified Home Page Facade

The proposed design pattern creates a facade that presents a single, comprehensive API endpoint to client applications for assembling complete home/landing page data while orchestrating the necessary Process API calls behind the scenes.

#### Architecture Overview

```
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│ Mobile Client │     │   Web Client  │     │ Other Clients │
└───────┬───────┘     └───────┬───────┘     └───────┬───────┘
        │                     │                     │
        └─────────────┬───────┴─────────────┬───────┘
                      │                     │
           ┌──────────▼─────────┐  ┌────────▼──────────┐
           │ Identity & Access  │  │ API Gateway Layer │
           │    Management     │  │   (Rate limiting,  │
           └──────────┬────────┘  │  Analytics, etc.)  │
                      │           └─────────┬──────────┘
                      │                     │
┌─────────────────────▼─────────────────────▼──────────────────────┐
│                                                                  │
│                         Experience API Layer                     │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐      │
│  │            Home/Landing Page Experience API            │      │
│  │                                                        │      │
│  │  ┌──────────────────┐  ┌─────────────────────────┐     │      │
│  │  │ Retail Segment   │  │ Affluent Segment        │     │      │
│  │  │ View Adapter     │  │ View Adapter            │     │      │
│  │  └─────────┬────────┘  └────────────┬────────────┘     │      │
│  │            │                        │                  │      │
│  │  ┌─────────▼────────────────────────▼────────────┐     │      │
│  │  │           Orchestration Engine                │     │      │
│  │  └─────────┬────────────────────────┬────────────┘     │      │
│  │            │                        │                  │      │
│  │  ┌─────────▼────────┐    ┌──────────▼───────────┐      │      │
│  │  │ Response         │    │ Cache Manager        │      │      │
│  │  │ Aggregator       │    │                      │      │      │
│  │  └─────────┬────────┘    └──────────────────────┘      │      │
│  └────────────┼───────────────────────────────────────────┘      │
│               │                                                  │
├───────────────┼──────────────────────────────────────────────────┤
│               │                                                  │
│  ┌────────────▼─────────────────────────────────────────────┐    │
│  │                     Process API Layer                    │    │
│  │                                                          │    │
│  │  ┌──────────┐  ┌───────────┐  ┌─────────┐  ┌──────────┐  │    │
│  │  │ Account  │  │ Transaction│  │ Offer  │  │Notification│ │    │
│  │  │ Process  │  │ Process    │  │ Process │  │ Process   │ │    │
│  │  │ API      │  │ API        │  │ API     │  │ API       │ │    │
│  │  └────┬─────┘  └─────┬─────┘  └────┬────┘  └─────┬─────┘  │    │
│  └───────┼──────────────┼────────────┼───────────── │────────┘    │
│          │              │            │              │             │
├──────────┼──────────────┼────────────┼──────────────┼─────────────┤
│          │              │            │              │             │
│  ┌───────▼──────┐ ┌─────▼─────┐ ┌────▼────┐ ┌───────▼─────┐       │
│  │Account System│ │Transaction │ │ Offer  │ │Notification │       │
│  │API           │ │System API  │ │System  │ │System API   │       │
│  └───────┬──────┘ └─────┬─────┘ │API     │ └───────┬─────┘       │
│          │              │       └────┬────┘         │             │
└──────────┼──────────────┼────────────┼──────────────┼─────────────┘
           │              │            │              │
  ┌────────▼──────┐ ┌─────▼─────┐ ┌────▼────┐ ┌───────▼─────┐
  │Core Banking   │ │Transaction │ │Marketing│ │Notification │
  │Systems        │ │Systems     │ │Systems  │ │Systems      │
  └───────────────┘ └───────────┘ └─────────┘ └─────────────┘
```

#### Home/Landing Page Experience API Components

1. **Segment-Specific View Adapters**: 
   - Retail and affluent adapters that customize information density and presentation
   - Apply segment-specific business rules for content prioritization
   - Select relevant data attributes based on segment needs

2. **Orchestration Engine**:
   - Coordinates calls to all necessary Process APIs for home page assembly
   - Implements parallel execution strategies for performance optimization
   - Applies intelligent loading patterns (critical path first, then enhancement data)

3. **Response Aggregator**:
   - Combines responses from multiple Process APIs into a cohesive view model
   - Structures data in a way that minimizes client-side transformation
   - Implements fallback strategies when partial data is unavailable

4. **Cache Manager**:
   - Implements multi-level caching (user-specific, segment-specific, global)
   - Applies different TTL strategies based on data volatility
   - Manages cache invalidation through event-based triggers

### Example Implementation for Home/Landing Page Journey

```
┌─────────────────┐
│  Mobile Client  │
└────────┬────────┘
         │
         │ Single API call: GET /retail/home-page
         │ or GET /affluent/home-page
         ▼
┌─────────────────────────────────────┐
│      Experience API Layer           │
│                                     │
│ 1. Authorize user and determine     │
│    segment (retail/affluent)        │
│ 2. Check cache for available data   │
│ 3. Orchestrate parallel Process     │
│    API calls for missing data       │
│ 4. Aggregate responses with         │
│    segment-specific prioritization  │
│ 5. Apply personalization rules      │
│ 6. Cache results appropriately      │
│ 7. Return unified response          │
└─────────────────┬─────────────────┘
                  │
    ┌─────────────┴─────────────────────────────────┐
    │             │               │                 │
    ▼             ▼               ▼                 ▼
┌─────────────┐ ┌───────────┐ ┌──────────┐ ┌────────────────┐
│ Account     │ │Transaction│ │ Offer    │ │ Notification   │
│ Process API │ │Process API│ │Process   │ │ Process API    │
└─────────────┘ └───────────┘ │API       │ └────────────────┘
                              └──────────┘
```

### Response Structure for Home/Landing Page

```json
{
  "customerInfo": {
    "name": "Jane Smith",
    "segmentType": "AFFLUENT",
    "relationshipManager": {  // Only for affluent customers
      "name": "John Davis",
      "contactNumber": "+1-555-123-4567",
      "availability": "Available now"
    }
  },
  "accounts": {
    "summary": {
      "totalBalance": 243500.00,
      "currencyCode": "USD"
    },
    "quickAccess": [  // Most frequently accessed accounts
      {
        "id": "AC-1234567",
        "nickname": "Main Checking",
        "accountType": "CURRENT",
        "balance": 12500.00,
        "currencyCode": "USD"
      },
      {
        "id": "SV-7654321",
        "nickname": "Emergency Fund",
        "accountType": "SAVINGS",
        "balance": 50000.00,
        "currencyCode": "USD"
      }
    ]
  },
  "recentTransactions": [
    {
      "id": "TX-987654",
      "date": "2025-05-09T14:30:00Z",
      "description": "ACME Corp Payroll",
      "amount": 5400.00,
      "type": "CREDIT"
    },
    {
      "id": "TX-987655",
      "date": "2025-05-08T09:15:00Z",
      "description": "Whole Foods Market",
      "amount": 89.43,
      "type": "DEBIT"
    }
  ],
  "offers": [
    {
      "id": "OF-123456",
      "title": "Premium Credit Card Upgrade",
      "description": "Upgrade to our Platinum card with enhanced travel benefits",
      "expiryDate": "2025-06-30T23:59:59Z",
      "ctaLink": "/offers/OF-123456"
    }
  ],
  "notifications": [
    {
      "id": "NT-345678",
      "type": "ALERT",
      "title": "Unusual transaction detected",
      "description": "We noticed a transaction that doesn't match your usual pattern",
      "timestamp": "2025-05-10T08:30:00Z",
      "isRead": false,
      "ctaLink": "/notifications/NT-345678"
    }
  ],
  "insights": {  // Only for affluent customers or retail opt-in
    "spendingAnalysis": {
      "monthlyTotal": 4378.29,
      "topCategory": "Dining",
      "percentChange": 5.2
    },
    "upcomingPayments": [
      {
        "description": "Auto Loan Payment",
        "dueDate": "2025-05-15T00:00:00Z",
        "amount": 450.00
      }
    ]
  },
  "marketUpdates": {  // Primarily for affluent customers
    "lastUpdated": "2025-05-11T09:30:00Z",
    "highlights": [
      {
        "title": "Markets rebound after Fed announcement",
        "summary": "Major indices up 2% following interest rate decision",
        "ctaLink": "/market-insights/fed-announcement"
      }
    ]
  }
}
```

## Drivers

1. **Improved Customer Experience**: Faster loading and more cohesive home/landing page experience.

2. **Segment-Specific Optimization**: Tailored experiences for retail vs. affluent customers with appropriate information density.

3. **Reduced Client Complexity**: Single API call greatly simplifies client development and maintenance.

4. **Performance Enhancement**: Parallel backend processing and intelligent caching reduce perceived latency.

5. **Consistent Security**: Standardized authentication and authorization across channels.

6. **Personalization at Scale**: Centralized personalization logic that can evolve without client changes.

7. **Reduced Network Traffic**: Elimination of redundant data transfers between client and server.

8. **Analytics Opportunity**: Comprehensive view of home page usage patterns across channels.

## Controls

### Authentication & Authorization

1. **Segment-Based Authorization**
   - OAuth 2.0/OpenID Connect implementation with segment-specific scopes
   - Role-Based Access Control (RBAC) differentiating retail vs. affluent permissions
   - Step-up authentication for sensitive operations
   - Attribute-Based Access Control (ABAC) for fine-grained authorization decisions

2. **Request Validation**
   - Schema validation for incoming requests
   - Parameter sanitization to prevent injection attacks
   - Client identification and validation

3. **Traffic Management**
   - Tiered rate limiting based on customer segment
   - Priority queueing for affluent customer requests
   - Circuit breakers to prevent cascade failures
   - Load shedding strategies during peak periods

4. **Monitoring & Observability**
   - End-to-end transaction tracing with correlation IDs
   - Component-level performance metrics
   - Error rate monitoring by segment
   - Home page component performance tracking

5. **Data Protection**
   - Field-level encryption for PII and sensitive financial data
   - Data minimization in responses
   - Response filtering based on entitlements
   - Compliance with regional data protection regulations

### Caching Strategy

1. **Multi-Level Caching**
   - User-specific cache for personalized content
   - Segment-level cache for shared segment-specific content
   - Global cache for common reference data
   
2. **Time-To-Live (TTL) Strategy**
   - Account data: 2-5 minutes
   - Transactional data: 5-15 minutes
   - Offers and notifications: 15-60 minutes
   - Market updates: Based on market hours and volatility

3. **Cache Invalidation**
   - Event-based invalidation for critical updates
   - TTL-based expiration for less volatile data
   - Forced refresh mechanisms for user-initiated updates

## Instructions

### Design Principles for Home/Landing Page Experience API

1. **Customer-First Design**
   - Design the API from the perspective of what the home page needs to display
   - Prioritize information based on customer segment needs
   - Structure responses to match UI consumption patterns

2. **Response Optimization**
   - Return data in display-ready format to minimize client-side processing
   - Include all necessary metadata for rendering decisions
   - Implement progressive enhancement pattern (critical data first)

3. **Flexibility for Channel-Specific Needs**
   - Support query parameters for client customization
   - Allow optional sections to be requested or omitted
   - Provide expandable references for drill-down scenarios

4. **Versioning Strategy**
   - Use URL versioning for major changes: `/v1/retail/home-page`
   - Implement additive-only changes between major versions
   - Provide deprecation notices and timelines

### Implementation Guidelines

1. **User Journey Mapping**
   - Document distinct home page requirements for retail vs. affluent segments
   - Identify critical path components vs. enhancements
   - Map content priorities and personalization rules

2. **API Design**
   - Create OpenAPI/Swagger specifications with comprehensive documentation
   - Define clear request parameters and response schemas
   - Document error scenarios and fallback behaviors

3. **Security Implementation**
   - Integrate with the bank's IAM solution
   - Implement segment-based authorization checks
   - Apply appropriate data filtering based on entitlements

4. **Orchestration Implementation**
   - Use async/parallel processing patterns
   - Implement timeout handling with graceful degradation
   - Apply circuit breaker patterns for dependent services

5. **Caching Strategy**
   - Implement distributed caching infrastructure
   - Define cache keys based on user ID, segment, and context
   - Implement intelligent pre-warming for high-value customers

6. **Testing Strategy**
   - Create segment-specific test scenarios
   - Implement performance testing with realistic load patterns
   - Test degradation scenarios when backend services fail

## Relevant Industry Best Practices and Regulations

1. **API Design Standards**
   - Follow RESTful API design principles with consistent resource naming
   - Apply [API Microservice Patterns](https://microservice-api-patterns.org/) like API Gateway and Backend Integration
   - Implement appropriate [API Responsibility Patterns](https://www.mulesoft.com/api/types-of-apis) for the Experience layer

2. **Banking Standards**
   - Comply with Open Banking standards where applicable
   - Align with BIAN (Banking Industry Architecture Network) service domains
   - Support ISO 20022 financial data standards

3. **Security Standards**
   - Implement OWASP API Security Top 10 protections
   - Apply Financial-grade API (FAPI) security profiles
   - Use TLS 1.3 with strong cipher suites

4. **Performance Standards**
   - Target response times under 500ms for initial critical data
   - Complete full response within 1-2 seconds
   - Support partial responses when some backend systems are unavailable

5. **Regulatory Compliance**
   - Ensure GDPR/CCPA compliance for personal data handling
   - Implement PSD2 requirements for European operations
   - Apply appropriate AML/KYC information controls

## Pattern Control Domain

The Home/Landing Page Experience API pattern operates within the "Digital Customer Experience Domain" of the banking architecture:

1. **Owns**:
   - Home/landing page experience definition
   - Segment-specific view models
   - Personalization rules for home content
   - Performance standards for home page loading

2. **Coordinates with**:
   - Security and Identity domain for authentication/authorization
   - Domain-specific Process API owners (accounts, payments, etc.)
   - Analytics domain for usage tracking
   - User Experience domain for alignment with design standards

3. **Governed by**:
   - API Governance Council for design standards
   - Digital Experience Committee for feature prioritization
   - Information Security for compliance and controls
   - Architecture Review Board for pattern consistency

## Pattern Layer

This pattern applies specifically to the Experience API Layer of the three-tiered API-led connectivity approach:

1. **Position**: Top layer (client-facing)
2. **Interfaces with**:
   - Client applications (mobile, web, etc.)
   - Process API layer (for business functionality)
3. **Characteristics**:
   - Customer journey-focused
   - Segment-aware (retail vs. affluent)
   - Coarse-grained aggregation
   - High cacheability
   - Personalization-enabled

## Rationale

The Home/Landing Page Experience API design pattern is optimal for the bank's modernization for several reasons:

1. **Dramatic Client Simplification**: Reduces 5-8 API calls to a single call, simplifying client development and improving reliability.

2. **Performance Optimization**: Server-side parallel processing and advanced caching strategies provide significantly better performance than client-side orchestration.

3. **Segment-Appropriate Experiences**: Built-in differentiation between retail and affluent customers ensures appropriate experiences without client complexity.

4. **Consistent Security Model**: Centralized security controls ensure uniform protection and easier compliance management.

5. **Backend Evolution Support**: Creates an abstraction layer allowing backend systems to evolve independently of client applications.

6. **Unified Analytics**: Provides a comprehensive view of home page usage patterns across channels.

7. **Personalization Engine**: Establishes a foundation for advanced personalization without requiring client changes.

8. **Reduced Network Overhead**: Minimizes data transfer volumes by eliminating redundant information across multiple calls.

## Consequences

### Positive Consequences

1. **Enhanced User Experience**: Faster, more coherent home/landing page experience across all channels.

2. **Development Efficiency**: Simplified client development with reduced testing complexity.

3. **Operational Benefits**: Easier monitoring, troubleshooting, and performance optimization.

4. **Consistent Experiences**: Users encounter the same behavior and information hierarchy regardless of channel.

5. **Future-Ready**: Foundation for advanced personalization, AI-driven insights, and new channels.

6. **Improved Reliability**: Fewer points of failure from the client perspective.

7. **Better Resource Utilization**: Optimized backend processing and network usage.

### Negative Consequences

1. **Increased Backend Complexity**: More sophisticated orchestration and aggregation logic in the Experience layer.

2. **Cache Management Challenges**: Need for advanced distributed caching strategies.

3. **Higher Initial Development Investment**: Greater upfront effort versus the traditional approach.

4. **Potential Single Point of Failure**: Requires robust resilience patterns.

5. **Team Skill Requirements**: Needs developers experienced in API design and distributed systems.

## Related Patterns

1. **API Gateway Pattern**: Used at the entry point for security, routing, and monitoring.

2. **Backend for Frontend (BFF)**: A specialized approach where Experience APIs are tailored for specific client types.

3. **Aggregator Pattern**: Core pattern used within the Experience API to combine data from multiple sources.

4. **Circuit Breaker Pattern**: Applied to prevent cascading failures when dependent services are unavailable.

5. **Cache-Aside Pattern**: Used for implementing the multi-level caching strategy.

6. **Façade Pattern**: Fundamental design pattern providing a simplified interface to complex subsystems.

7. **Command Query Responsibility Segregation (CQRS)**: Often used in Process APIs orchestrated by the Experience API.

8. **Strangler Pattern**: Useful when gradually migrating from legacy direct-to-backend APIs to the Experience API pattern.

## Responsibilities of Experience API

### 1. Authorization

The Home/Landing Page Experience API ensures the right resources are accessed by the right users with appropriate segment-based controls:

- **Token Validation**: Validate OAuth tokens and extract claims
- **Segment Determination**: Identify user as retail or affluent
- **Permission Enforcement**: Apply appropriate access controls based on segment
- **Entitlement Checking**: Verify access rights to specific data elements
- **Consent Management**: Respect user privacy preferences for data usage

**Implementation Strategy**:
- Integrate with the bank's Identity and Access Management (IAM) system
- Maintain segment-specific authorization rules
- Implement fine-grained data filtering based on entitlements
- Log all authorization decisions for audit purposes

### 2. Request Transformation

The Experience API transforms incoming requests to match Process API requirements:

- **Parameter Mapping**: Convert client parameters to backend formats
- **Context Enrichment**: Add user context, device information, and preferences
- **Request Augmentation**: Include additional parameters required by Process APIs
- **Segment-Specific Logic**: Apply different transformation rules based on segment
- **Location and Language Adaptation**: Adjust requests based on user locale

**Implementation Strategy**:
- Define transformation maps for each backend service
- Implement adapter patterns for different Process APIs
- Centralize transformation logic to ensure consistency
- Log transformation details for debugging

### 3. Orchestration

The Experience API orchestrates multiple Process API calls to assemble the complete home page data:

- **Parallel Execution**: Call independent Process APIs simultaneously
- **Critical Path Management**: Prioritize essential components
- **Conditional Processing**: Execute different flows based on segment and preferences
- **Error Handling**: Implement fallback strategies for component failures
- **Timeout Management**: Set appropriate timeouts for backend services

**Implementation Strategy**:
- Implement async programming patterns for parallel execution
- Use composition patterns to assemble the home page components
- Apply circuit breaker patterns for resilience
- Define clear dependency order for sequential operations

### 4. Response Aggregation

The Experience API aggregates and transforms responses from multiple Process APIs:

- **Data Consolidation**: Combine data from multiple sources into a unified view
- **Conflict Resolution**: Resolve data conflicts from different sources
- **Enrichment**: Add derived or calculated fields
- **Restructuring**: Organize data in client-friendly hierarchies
- **Personalization**: Apply user-specific adjustments to content

**Implementation Strategy**:
- Define clear response templates for each segment
- Implement composable response building
- Use progressive enhancement to build responses in priority order
- Apply consistent data formatting across all components

### 5. Attribute Minimization

The Experience API returns only the attributes required for the home/landing page:

- **Field Filtering**: Remove unnecessary fields from backend responses
- **Data Minimization**: Apply privacy and performance-driven data reduction
- **Segment-Specific Filtering**: Return different attribute sets for retail vs. affluent
- **Context-Aware Reduction**: Adjust detail level based on device and channel
- **Progressive Disclosure**: Support expandable references for detail views

**Implementation Strategy**:
- Define view models specific to each segment and context
- Implement whitelist-based attribute filtering
- Support optional expansion parameters for additional details
- Measure and optimize payload sizes
