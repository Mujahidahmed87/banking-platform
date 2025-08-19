# Technical Architecture Document: UAE Banking Platform

## 1. Architecture Overview

The banking platform is designed as a cloud-native, microservices-based architecture deployed on AWS, emphasizing security, scalability, and reliability. The architecture follows a multi-layered approach with clear service boundaries and robust inter-service communication patterns.

## 2. Component Analysis

### 2.1 Client Tier

#### Web Application
- **Technology Stack**: 
  - React 18.x with Server Components
  - TypeScript 5.x with strict mode
  - Vite for build optimization
  - Redux Toolkit for state management
  - React Query for data fetching
  - Jest + React Testing Library for testing

- **Key Features**:
  - Progressive Web App with Workbox:
    - Dynamic caching strategies
    - Background sync for offline transactions
    - Push notifications integration
    - Cached API responses with versioning
  
  - OpenAPI SDK Integration:
    - Auto-generated TypeScript clients
    - Request/response interceptors
    - Automatic retry with exponential backoff
    - Circuit breaker pattern implementation
  
  - Performance Optimizations:
    - Code splitting at route level
    - Dynamic imports for heavy components
    - React.lazy for component lazy loading
    - Service Worker for asset caching
    - Compression (Brotli/Gzip)

- **Technical Implementation**:
  ```typescript
  // Circuit Breaker Implementation
  class CircuitBreaker {
    private failures: number = 0;
    private lastFailure: number = 0;
    private readonly threshold: number = 5;
    private readonly resetTimeout: number = 60000;

    async execute<T>(fn: () => Promise<T>): Promise<T> {
      if (this.isOpen()) {
        throw new Error('Circuit breaker is open');
      }
      try {
        const result = await fn();
        this.reset();
        return result;
      } catch (error) {
        this.recordFailure();
        throw error;
      }
    }
  }
  ```

- **Cost Analysis**:
  | Component | AWS Service | Monthly Cost* |
  |-----------|------------|---------------|
  | Static Hosting | S3 + CloudFront | $0.50/GB + $0.085/GB |
  | Route53 DNS | Route53 | $0.50/hosted zone |
  | WAF Rules | AWS WAF | $5/rule/month |
  *Based on 100K monthly active users

- **Technical Decisions**:
  1. React 18 over Vue/Angular:
     - Concurrent rendering support
     - Server Components for better SEO
     - Larger ecosystem for banking components
  
  2. TypeScript Strict Mode:
     - Compile-time type checking
     - Better IDE support
     - Reduced runtime errors
     
  3. Vite over Webpack:
     - Faster HMR (Hot Module Replacement)
     - Better build performance
     - ES modules support

#### Mobile Application
- **Technology Stack**: 
  - React Native 0.72 with Hermes Engine
  - TypeScript 5.x for type safety
  - Redux Toolkit for state management
  - Realm for offline database
  - Firebase Cloud Messaging for push notifications
  - React Navigation 6.x for routing

- **Key Features & Implementation**:
  1. Biometric Authentication:
     ```typescript
     // Biometric Authentication Handler
     class BiometricAuth {
       async authenticate(): Promise<boolean> {
         const available = await LocalAuthentication.hasHardwareAsync();
         if (!available) return false;
         
         return await LocalAuthentication.authenticateAsync({
           promptMessage: 'Authenticate for Banking',
           fallbackLabel: 'Use Passcode',
           disableDeviceFallback: false,
           cancelLabel: 'Cancel'
         });
       }
     }
     ```

  2. Offline Transaction Support:
     - Realm Database Schema:
     ```typescript
     const TransactionSchema = {
       name: 'Transaction',
       properties: {
         id: 'string',
         amount: 'double',
         type: 'string',
         status: 'string',
         timestamp: 'date',
         syncStatus: 'string'
       },
       primaryKey: 'id'
     };
     ```
     - Background Sync Implementation
     - Conflict Resolution Strategy
     - Queue-based Transaction Processing

  3. Security Measures:
     - SSL Certificate Pinning
     - App Security Checklist:
       - Root Detection
       - Emulator Detection
       - Code Obfuscation
       - Secure Storage (Keychain/Keystore)
       - Screenshot Prevention
       - Clipboard Protection

- **Technical Architecture**:
  ```mermaid
  graph TD
    A[UI Layer] --> B[Business Logic]
    B --> C[Data Layer]
    C --> D[Local Storage]
    C --> E[API Client]
    F[Security Layer] --> B
    G[Navigation] --> A
  ```

- **Performance Optimizations**:
  - Hermes Engine Benefits:
    - Reduced APK Size: ~30% smaller
    - Faster App Launch: ~50% improvement
    - Lower Memory Usage: ~20% reduction
  - Image Caching & Optimization
  - Memory Leak Prevention
  - Network Request Optimization

- **Cost Analysis**:
  | Component | AWS/Service | Monthly Cost* |
  |-----------|------------|---------------|
  | Push Notifications | SNS/FCM | $0.50/million msgs |
  | API Gateway | AWS API Gateway | $3.50/million calls |
  | Content Delivery | CloudFront | $0.085/GB |
  | Analytics | Pinpoint | $1.20/1000 active users |
  *Based on 50K MAU

- **Technical Decisions & Trade-offs**:
  1. React Native over Native Development:
     - Pros:
       - Shared codebase (~70% code sharing)
       - Faster development cycles
       - Consistent UI/UX across platforms
     - Cons:
       - Slightly larger app size
       - Bridge overhead for native features
       - Third-party dependency management

  2. Realm over SQLite:
     - Pros:
       - Better offline-first capabilities
       - Real-time synchronization
       - Object-oriented data model
     - Cons:
       - Larger database size
       - Migration complexity
       - Learning curve for team

  3. FCM over AWS SNS Direct:
     - Pros:
       - Better battery optimization
       - Reliable delivery
       - Rich notification support
     - Cons:
       - Google dependency
       - Additional configuration
       - Cross-platform complexity

- **Monitoring & Analytics**:
  - Firebase Analytics Integration
  - Custom Event Tracking
  - Performance Monitoring
  - Crash Reporting
  - User Flow Analysis

### 2.2 Edge Layer

#### CloudFront CDN
- **Detailed Configuration**:
  ```json
  {
    "Distribution": {
      "Origins": [
        {
          "S3Origin": {
            "DomainName": "banking-static.s3.amazonaws.com",
            "OriginAccessIdentity": "enabled"
          }
        },
        {
          "CustomOrigin": {
            "DomainName": "api.banking.com",
            "OriginProtocolPolicy": "https-only",
            "HTTPPort": 80,
            "HTTPSPort": 443,
            "OriginSSLProtocols": ["TLSv1.2", "TLSv1.3"]
          }
        }
      ],
      "DefaultCacheBehavior": {
        "AllowedMethods": ["GET", "HEAD", "OPTIONS"],
        "CachedMethods": ["GET", "HEAD"],
        "CachingOptimizedForUncompressedObjects": false,
        "DefaultTTL": 86400,
        "MaxTTL": 31536000,
        "MinTTL": 0
      }
    }
  }
  ```

- **DDoS Protection Strategy**:
  1. AWS Shield Advanced Integration:
     - Layer 3/4 DDoS protection
     - 10TB DDoS protection
     - 24/7 DDoS Response Team (DRT)
  
  2. Cost Optimization:
     ```yaml
     Monthly Costs:
       Base Shield Advanced: $3,000
       Data Transfer:
         First 10TB: Included
         Additional: $0.25/GB
       DDoS Protection:
         Layer 3/4: Included
         Layer 7: WAF Rules
     ```

  3. Performance Metrics:
     - Time to First Byte (TTFB): < 100ms
     - Cache Hit Ratio: > 85%
     - Error Rate: < 0.1%

#### WAF (Web Application Firewall)
- **Rule Implementation**:
  ```json
  {
    "Rules": [
      {
        "Name": "SQLInjectionRule",
        "Priority": 1,
        "Action": { "Block": {} },
        "OverrideAction": { "None": {} },
        "Statement": {
          "SqliMatchStatement": {
            "FieldToMatch": { "Body": {} },
            "TextTransformations": [
              { "Priority": 1, "Type": "HTML_ENTITY_DECODE" },
              { "Priority": 2, "Type": "LOWERCASE" }
            ]
          }
        }
      }
    ]
  }
  ```

- **Rate Limiting Implementation**:
  ```python
  def calculate_rate_limit(client_ip, path):
      base_limit = 10000  # 10K RPS
      if path.startswith('/api/payments'):
          return min(base_limit, 1000)  # Stricter for sensitive endpoints
      return base_limit
  ```

- **Cost Analysis**:
  | Rule Type | Monthly Requests | Cost/Month |
  |-----------|-----------------|------------|
  | Basic Rules | 1B | $1/million |
  | Advanced Rules | 500M | $2/million |
  | Bot Control | 100M | $4/million |

#### API Gateway
- **Technical Implementation**:
  1. API Types & Usage:
     ```yaml
     REST APIs:
       - Account Operations: /v1/accounts/*
       - Payment Services: /v1/payments/*
       - Utility Services: /v1/utilities/*
     
     WebSocket APIs:
       - Real-time Balance Updates
       - Transaction Notifications
       - Session Management
     ```

  2. Custom Domain Configuration:
     ```json
     {
       "DomainName": "api.banking.com",
       "Certificate": {
         "Type": "ACM",
         "ARN": "arn:aws:acm:region:account:certificate/id"
       },
       "SecurityPolicy": "TLS_1_2",
       "EndpointConfiguration": {
         "Types": ["REGIONAL"]
       }
     }
     ```

  3. Throttling Configuration:
     ```json
     {
       "BurstLimit": 5000,
       "RateLimit": 10000,
       "ThrottlingResponses": {
         "429": {
           "ResponseTemplates": {
             "application/json": "{\"message\":\"Rate limit exceeded\"}"
           }
         }
       }
     }
     ```

- **Cost Optimization Strategy**:
  | Component | Requests/Month | Cost |
  |-----------|---------------|------|
  | REST API Calls | 100M | $350 |
  | WebSocket Messages | 10M | $250 |
  | Data Transfer | 5TB | $450 |
  | Cache Usage | 50GB | $40 |

- **Technical Decisions & Trade-offs**:
  1. Regional vs. Edge Optimized:
     - Chose Regional for:
       - Lower latency within UAE
       - Better control over data residency
       - Cost optimization
     
  2. REST vs. HTTP APIs:
     - Chose REST for:
       - Advanced request validation
       - API key support
       - Custom authorizers
     - Trade-off: Higher cost but better security

  3. Custom Domain Strategy:
     - Wildcard certificates
     - Regional endpoint association
     - Route53 health checks

### 2.3 Security Layer

#### Cognito (Identity Management)
- **Architecture Design**:
  ```mermaid
  graph TD
    A[User] --> B[Cognito User Pool]
    B --> C[JWT Token]
    C --> D[API Gateway]
    B --> E[Identity Pool]
    E --> F[Temporary AWS Credentials]
  ```

- **User Pool Configuration**:
  ```json
  {
    "UserPool": {
      "AdminCreateUserConfig": {
        "AllowAdminCreateUserOnly": false,
        "InviteMessageTemplate": {
          "EmailMessage": "Custom welcome message",
          "SMSMessage": "Your username is {username} and temp password is {####}"
        }
      },
      "AutoVerifiedAttributes": ["email", "phone_number"],
      "MfaConfiguration": "ON",
      "EnabledMfaTypes": ["SMS", "TOTP"],
      "PasswordPolicy": {
        "MinimumLength": 12,
        "RequireUppercase": true,
        "RequireLowercase": true,
        "RequireNumbers": true,
        "RequireSymbols": true,
        "TemporaryPasswordValidityDays": 3
      },
      "CustomAttributes": [
        {
          "Name": "accountStatus",
          "Type": "String",
          "Mutable": true
        },
        {
          "Name": "kycStatus",
          "Type": "String",
          "Mutable": true
        }
      ]
    }
  }
  ```

- **Authentication Flow Implementation**:
  ```typescript
  class AuthenticationService {
    async signIn(username: string, password: string): Promise<AuthResult> {
      try {
        // Initial authentication
        const authResult = await this.cognitoClient.initiateAuth({
          AuthFlow: 'USER_PASSWORD_AUTH',
          AuthParameters: { USERNAME: username, PASSWORD: password }
        });

        // Check for MFA challenge
        if (authResult.ChallengeName === 'SOFTWARE_TOKEN_MFA') {
          return { status: 'MFA_REQUIRED', session: authResult.Session };
        }

        // Handle custom challenge for risk assessment
        if (authResult.ChallengeName === 'CUSTOM_CHALLENGE') {
          return this.handleRiskBasedAuthentication(authResult);
        }

        return { status: 'SUCCESS', tokens: authResult.AuthenticationResult };
      } catch (error) {
        this.handleAuthError(error);
      }
    }

    private async handleRiskBasedAuthentication(authResult: any): Promise<AuthResult> {
      // Implement risk scoring based on:
      // - Device fingerprint
      // - Location
      // - Time of day
      // - Transaction history
    }
  }
  ```

- **Cost Analysis**:
  | Feature | Monthly Active Users | Cost/Month |
  |---------|---------------------|------------|
  | User Pool | 100K | $0.0055/MAU |
  | MFA (SMS) | 50K | $0.100/message |
  | Advanced Security | 100K | $0.050/MAU |

#### Secrets Manager
- **Implementation Architecture**:
  ```typescript
  class SecretsRotationManager {
    async rotateSecret(secretId: string): Promise<void> {
      const stages = ['createSecret', 'setSecret', 'testSecret', 'finishSecret'];
      
      for (const stage of stages) {
        await this.executeRotationStep(secretId, stage);
      }
    }

    private async executeRotationStep(secretId: string, step: string): Promise<void> {
      switch (step) {
        case 'createSecret':
          // Generate new credentials
          break;
        case 'setSecret':
          // Update the service with new credentials
          break;
        case 'testSecret':
          // Verify new credentials work
          break;
        case 'finishSecret':
          // Mark rotation as complete
          break;
      }
    }
  }
  ```

- **Secret Types and Rotation Policies**:
  ```yaml
  Database Credentials:
    RotationSchedule: 30 days
    Type: aurora-credentials
    ReplicaRegions:
      - RegionName: me-south-1
      - RegionName: eu-west-1

  API Keys:
    RotationSchedule: 90 days
    Type: api-key
    AutoRotation: true

  SSL Certificates:
    RotationSchedule: 30 days
    Type: certificate
    NotificationTarget: sns:certificate-rotation
  ```

- **Cost Optimization**:
  | Component | Number of Secrets | Cost/Month |
  |-----------|------------------|------------|
  | Secret Storage | 100 | $0.40/secret |
  | API Calls | 100K | $0.05/10K calls |
  | Rotation | 50 rotations | Included |

#### KMS (Key Management Service)
- **Key Hierarchy Implementation**:
  ```typescript
  class EncryptionService {
    async encryptData(data: Buffer, context: Record<string, string>): Promise<Buffer> {
      // Generate data key
      const dataKey = await this.kms.generateDataKey({
        KeyId: this.masterKeyId,
        KeySpec: 'AES_256',
        EncryptionContext: context
      });

      // Encrypt data with data key
      const cipher = crypto.createCipheriv('aes-256-gcm', dataKey.Plaintext, iv);
      const encryptedData = Buffer.concat([
        cipher.update(data),
        cipher.final(),
        cipher.getAuthTag()
      ]);

      // Store encrypted data key with encrypted data
      return Buffer.concat([dataKey.CiphertextBlob, encryptedData]);
    }
  }
  ```

- **Key Policy Example**:
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "Enable IAM User Permissions",
        "Effect": "Allow",
        "Principal": {
          "AWS": "arn:aws:iam::ACCOUNT-ID:root"
        },
        "Action": "kms:*",
        "Resource": "*"
      },
      {
        "Sid": "Allow EKS service role to use the key",
        "Effect": "Allow",
        "Principal": {
          "AWS": "arn:aws:iam::ACCOUNT-ID:role/eks-service-role"
        },
        "Action": [
          "kms:Encrypt",
          "kms:Decrypt",
          "kms:ReEncrypt*",
          "kms:GenerateDataKey*",
          "kms:DescribeKey"
        ],
        "Resource": "*"
      }
    ]
  }
  ```

- **Monitoring and Alerting**:
  ```yaml
  CloudWatch Alarms:
    UnauthorizedOperationAttempts:
      Threshold: 5
      Period: 5 minutes
      Action: SNS:SecurityTeam
    
    KeyDeletion:
      Threshold: 1
      Period: 1 minute
      Action: SNS:SecurityTeam,Lambda:BlockDeletion
  ```

- **Cost Analysis**:
  | Component | Usage | Cost/Month |
  |-----------|-------|------------|
  | Customer Master Key | 5 CMKs | $1/key |
  | API Requests | 1M requests | $0.03/10K requests |
  | Key Rotation | Automatic | Included |
  | Custom Key Store | HSM-backed | $1000/month |

### 2.4 Application Layer

#### Application Load Balancer (ALB)
- **Advanced Configuration**:
  ```json
  {
    "LoadBalancerAttributes": {
      "idle_timeout.timeout_seconds": "60",
      "routing.http2.enabled": "true",
      "access_logs.s3.enabled": "true",
      "deletion_protection.enabled": "true"
    },
    "Listeners": [
      {
        "Protocol": "HTTPS",
        "Port": 443,
        "Certificates": [
          {
            "CertificateArn": "arn:aws:acm:region:account:certificate/id"
          }
        ],
        "SslPolicy": "ELBSecurityPolicy-TLS-1-2-2017-01",
        "DefaultActions": [
          {
            "Type": "authenticate-cognito",
            "AuthenticateCognitoConfig": {
              "UserPoolArn": "arn:aws:cognito-idp:region:account:userpool/id",
              "UserPoolClientId": "client_id",
              "UserPoolDomain": "domain"
            }
          }
        ]
      }
    ]
  }
  ```

- **Target Group Configuration**:
  ```yaml
  TargetGroups:
    AccountService:
      HealthCheck:
        Path: /health
        Port: 8080
        Protocol: HTTP
        IntervalSeconds: 15
        TimeoutSeconds: 5
        HealthyThresholdCount: 2
        UnhealthyThresholdCount: 3
      Attributes:
        deregistration_delay.timeout_seconds: 30
        slow_start.duration_seconds: 30
        load_balancing.algorithm.type: least_outstanding_requests

    PaymentService:
      # Similar configuration with specific health check paths
  ```

- **Cost Analysis**:
  | Component | Usage | Cost/Month |
  |-----------|-------|------------|
  | ALB Hours | 730 hours | $16.43 |
  | LCU Usage | 100 LCU | $5.48 |
  | Data Processed | 100 GB | $4.53 |

#### EKS Cluster
- **Cluster Architecture**:
  ```mermaid
  graph TD
    A[Route53] --> B[ALB]
    B --> C[EKS Control Plane]
    C --> D[Node Group 1]
    C --> E[Node Group 2]
    C --> F[Node Group 3]
    G[AWS Auth ConfigMap] --> C
    H[Cluster Autoscaler] --> C
  ```

- **Node Group Specifications**:
  ```yaml
  NodeGroups:
    General:
      InstanceType: m5.2xlarge
      MinSize: 3
      MaxSize: 10
      DesiredCapacity: 5
      Labels:
        role: general
      Taints: []
      
    MemoryOptimized:
      InstanceType: r5.2xlarge
      MinSize: 2
      MaxSize: 8
      DesiredCapacity: 3
      Labels:
        role: memory-optimized
      Taints:
        - key: dedicated
          value: memory
          effect: NoSchedule

    Monitoring:
      InstanceType: c5.xlarge
      MinSize: 2
      MaxSize: 4
      DesiredCapacity: 2
      Labels:
        role: monitoring
  ```

- **Kubernetes Configuration**:
  ```yaml
  # Network Policy Example
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: payment-service-policy
  spec:
    podSelector:
      matchLabels:
        app: payment-service
    policyTypes:
    - Ingress
    - Egress
    ingress:
    - from:
      - podSelector:
          matchLabels:
            app: api-gateway
      ports:
      - protocol: TCP
        port: 8080
    egress:
    - to:
      - podSelector:
          matchLabels:
            app: account-service
      ports:
      - protocol: TCP
        port: 8080

  # HPA Configuration
  apiVersion: autoscaling/v2
  kind: HorizontalPodAutoscaler
  metadata:
    name: payment-service-hpa
  spec:
    scaleTargetRef:
      apiVersion: apps/v1
      kind: Deployment
      name: payment-service
    minReplicas: 3
    maxReplicas: 10
    metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
  ```

- **Cost Optimization Strategy**:
  | Component | Configuration | Monthly Cost |
  |-----------|--------------|--------------|
  | Control Plane | HA across 3 AZs | $73/cluster |
  | Node Groups (On-Demand) | 8 m5.2xlarge | $1,751.04 |
  | Node Groups (Spot) | 4 m5.2xlarge | $525.31 |
  | EBS Volumes | 100 GB/node | $80 |
  | Load Balancer | 1 ALB | $16.43 |

- **Performance Monitoring**:
  ```yaml
  PrometheusRules:
    ClusterHealth:
      - alert: NodeCPUUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 15m
        labels:
          severity: warning
      - alert: NodeMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 85
        for: 15m
        labels:
          severity: warning
  ```

- **Technical Decisions & Trade-offs**:
  1. EKS vs. ECS:
     ```yaml
     EKS Benefits:
       - Industry standard Kubernetes
       - Rich ecosystem of tools
       - Better multi-tenant isolation
       - Advanced deployment strategies
     EKS Challenges:
       - Higher operational complexity
       - Steeper learning curve
       - Higher cost
     
     ECS Benefits:
       - Simpler management
       - Lower cost
       - Tighter AWS integration
     ECS Limitations:
       - Limited deployment strategies
       - AWS-specific tooling
       - Basic orchestration features
     ```

  2. Node Group Strategy:
     ```yaml
     Dedicated vs Mixed Workloads:
       Chosen: Dedicated Node Groups
       Rationale:
         - Better resource isolation
         - Optimized instance types
         - Simplified capacity management
       Trade-offs:
         - Higher cost
         - More complex management
         - Better performance

#### Microservices

##### Account Service
- **Technology**: Java 17, Spring Boot 3.x
- **Resources**: 8GB RAM, 4 vCPU, Min 3 replicas
- **Responsibilities**:
  - Account management
  - Balance inquiries
  - Transaction history
  - Statement generation
- **Requirements Alignment**: Directly implements the Account Management capabilities

##### Payment Service
- **Technology**: Java 17, Spring Boot 3.x
- **Resources**: 8GB RAM, 4 vCPU, Min 3 replicas
- **Responsibilities**:
  - Domestic transfers
  - International SWIFT payments
  - Beneficiary management
  - Payment limits and approvals
- **Requirements Alignment**: Implements the Payments and Transfers capabilities

##### Notification Service
- **Technology**: Node.js 18 LTS, NestJS
- **Resources**: 4GB RAM, 2 vCPU, Min 2 replicas
- **Responsibilities**:
  - Push notification management
  - SMS integration
  - Email delivery
  - Preference management
- **Requirements Alignment**: Fulfills the Notification System requirements

##### Utility Service
- **Technology**: Java 17, Spring Boot 3.x
- **Resources**: 4GB RAM, 2 vCPU, Min 2 replicas
- **Responsibilities**:
  - Biller aggregation
  - Utility payments
  - Bill presentment
- **Requirements Alignment**: Implements the Utilities Payments capabilities

### 2.5 Data Layer

#### Aurora MySQL
- **Cluster Architecture**:
  ```mermaid
  graph TD
    A[Primary Instance] --> B[Reader 1]
    A --> C[Reader 2]
    A --> D[Reader 3]
    E[Proxy Fleet] --> A
    E --> B
    E --> C
    E --> D
    F[Backup/Restore] --> G[S3]
    H[Parameter Group] --> A
  ```

- **Configuration Details**:
  ```json
  {
    "DBCluster": {
      "Engine": "aurora-mysql",
      "EngineVersion": "8.0.mysql_aurora.3.03.1",
      "DBClusterParameterGroup": {
        "innodb_buffer_pool_size": "{DBInstanceClassMemory*0.75}",
        "max_connections": 5000,
        "innodb_flush_log_at_trx_commit": 1,
        "sync_binlog": 1,
        "innodb_flush_method": "O_DIRECT",
        "binlog_format": "ROW",
        "binlog_row_image": "FULL"
      },
      "ScalingConfiguration": {
        "MinCapacity": 2,
        "MaxCapacity": 64,
        "AutoPause": false
      }
    }
  }
  ```

- **Performance Optimization**:
  ```sql
  -- Optimized Table Structure Example
  CREATE TABLE transactions (
    transaction_id BINARY(16) PRIMARY KEY,
    account_id BINARY(16),
    transaction_type ENUM('DEBIT', 'CREDIT'),
    amount DECIMAL(20,2),
    status VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_account_date (account_id, created_at),
    INDEX idx_status (status, created_at)
  ) ENGINE=InnoDB ROW_FORMAT=COMPRESSED;

  -- Partition Strategy
  ALTER TABLE transactions
  PARTITION BY RANGE (UNIX_TIMESTAMP(created_at)) (
    PARTITION p_2024_q1 VALUES LESS THAN (UNIX_TIMESTAMP('2024-04-01 00:00:00')),
    PARTITION p_2024_q2 VALUES LESS THAN (UNIX_TIMESTAMP('2024-07-01 00:00:00')),
    PARTITION p_2024_q3 VALUES LESS THAN (UNIX_TIMESTAMP('2024-10-01 00:00:00')),
    PARTITION p_2024_q4 VALUES LESS THAN (UNIX_TIMESTAMP('2025-01-01 00:00:00')),
    PARTITION p_future VALUES LESS THAN MAXVALUE
  );
  ```

- **High Availability Setup**:
  ```yaml
  Availability:
    Primary:
      AZ: me-south-1a
      Instance: db.r6g.2xlarge
    Replicas:
      - AZ: me-south-1b
        Instance: db.r6g.2xlarge
        Purpose: Read-scaling
      - AZ: me-south-1c
        Instance: db.r6g.2xlarge
        Purpose: Failover-target
    
  BackupStrategy:
    AutomatedBackup:
      RetentionPeriod: 35
      BackupWindow: "00:00-03:00"
    PointInTimeRecovery: true
    ContinuousBackup: enabled
  ```

- **Cost Analysis**:
  | Component | Configuration | Monthly Cost |
  |-----------|--------------|--------------|
  | Primary Instance | db.r6g.2xlarge | $1,226.40 |
  | Read Replicas (2) | db.r6g.2xlarge | $2,452.80 |
  | Storage (6TB) | io1, 100K IOPS | $800 |
  | I/O Operations | 100K IOPS | $1,000 |
  | Backup Storage | 3TB | $300 |

#### DynamoDB
- **Table Design**:
  ```json
  {
    "TableName": "UserSessions",
    "AttributeDefinitions": [
      {
        "AttributeName": "SessionId",
        "AttributeType": "S"
      },
      {
        "AttributeName": "UserId",
        "AttributeType": "S"
      },
      {
        "AttributeName": "ExpirationTime",
        "AttributeType": "N"
      }
    ],
    "KeySchema": [
      {
        "AttributeName": "SessionId",
        "KeyType": "HASH"
      }
    ],
    "GlobalSecondaryIndexes": [
      {
        "IndexName": "UserIndex",
        "KeySchema": [
          {
            "AttributeName": "UserId",
            "KeyType": "HASH"
          },
          {
            "AttributeName": "ExpirationTime",
            "KeyType": "RANGE"
          }
        ],
        "Projection": {
          "ProjectionType": "ALL"
        }
      }
    ],
    "BillingMode": "PAY_PER_REQUEST",
    "StreamSpecification": {
      "StreamEnabled": true,
      "StreamViewType": "NEW_AND_OLD_IMAGES"
    }
  }
  ```

- **DAX Configuration**:
  ```yaml
  DAXCluster:
    NodeType: dax.r5.xlarge
    ReplicationFactor: 3
    AvailabilityZones:
      - me-south-1a
      - me-south-1b
      - me-south-1c
    Parameters:
      query-ttl-millis: 60000
      record-ttl-millis: 86400000
    MaintenanceWindow: sun:05:00-sun:09:00
    SecurityGroupIds:
      - sg-12345678
  ```

- **Access Patterns and Modeling**:
  ```typescript
  interface UserPreference {
    UserId: string;            // Partition Key
    PreferenceType: string;    // Sort Key
    Value: string;
    LastUpdated: number;
    Version: number;
  }

  interface NotificationSettings {
    UserId: string;            // Partition Key
    ChannelType: string;       // Sort Key
    Enabled: boolean;
    Frequency: string;
    Filters: string[];
    LastUpdated: number;
  }

  // Transaction Example
  const transactionItems = {
    TransactItems: [
      {
        Update: {
          TableName: "UserPreferences",
          Key: { UserId, PreferenceType },
          UpdateExpression: "SET #val = :newVal, Version = :newVersion",
          ConditionExpression: "Version = :oldVersion",
          ExpressionAttributeNames: { "#val": "Value" },
          ExpressionAttributeValues: {
            ":newVal": newValue,
            ":newVersion": currentVersion + 1,
            ":oldVersion": currentVersion
          }
        }
      }
    ]
  };
  ```

- **Cost Optimization Strategy**:
  | Component | Usage Pattern | Monthly Cost |
  |-----------|--------------|--------------|
  | Write Capacity | On-demand | $1.25/million |
  | Read Capacity | On-demand | $0.25/million |
  | DAX Cluster | 3 nodes | $912.24 |
  | Backup Storage | Point-in-time | $0.20/GB |
  | Global Tables | 2 regions | $1.875/million replicated writes |

#### Redis Cache
- **Cluster Architecture**:
  ```yaml
  ElastiCache:
    Engine: redis
    EngineVersion: "7.0"
    ReplicationGroupDescription: "Banking Session Store"
    NumNodeGroups: 3
    ReplicasPerNodeGroup: 2
    NodeType: cache.r6g.xlarge
    AutomaticFailoverEnabled: true
    MultiAZ: true
    CacheParameterGroup:
      Parameters:
        maxmemory-policy: volatile-lru
        timeout: 0
        appendonly: "yes"
        appendfsync: everysec
    SecurityGroup:
      InboundRules:
        - IpProtocol: tcp
          FromPort: 6379
          ToPort: 6379
          SourceSecurityGroupId: sg-application
  ```

- **Data Structure Examples**:
  ```typescript
  // Rate Limiting Implementation
  class RateLimiter {
    private readonly redis: Redis;
    private readonly windowSize: number = 60; // seconds

    async isAllowed(userId: string, limit: number): Promise<boolean> {
      const now = Date.now();
      const key = `ratelimit:${userId}`;
      
      const pipeline = this.redis.pipeline();
      pipeline.zadd(key, now, `${now}`);
      pipeline.zremrangebyscore(key, 0, now - (this.windowSize * 1000));
      pipeline.zcard(key);
      pipeline.expire(key, this.windowSize);
      
      const results = await pipeline.exec();
      const requestCount = results[2][1];
      
      return requestCount <= limit;
    }
  }

  // Session Management
  interface SessionData {
    userId: string;
    permissions: string[];
    lastAccess: number;
    deviceInfo: {
      id: string;
      type: string;
    };
  }

  class SessionManager {
    async setSession(sessionId: string, data: SessionData): Promise<void> {
      await this.redis.setex(
        `session:${sessionId}`,
        3600, // 1 hour TTL
        JSON.stringify(data)
      );
    }
  }
  ```

- **Performance Monitoring**:
  ```yaml
  CloudWatchAlarms:
    MemoryUsage:
      Threshold: 80
      Period: 300
      EvaluationPeriods: 2
      ComparisonOperator: GreaterThanThreshold
    
    CPUUtilization:
      Threshold: 70
      Period: 300
      EvaluationPeriods: 2
      ComparisonOperator: GreaterThanThreshold
    
    CacheHitRate:
      Threshold: 80
      Period: 300
      EvaluationPeriods: 2
      ComparisonOperator: LessThanThreshold
  ```

- **Cost Analysis**:
  | Component | Configuration | Monthly Cost |
  |-----------|--------------|--------------|
  | Primary Nodes (3) | cache.r6g.xlarge | $1,094.40 |
  | Replica Nodes (6) | cache.r6g.xlarge | $2,188.80 |
  | Backup Storage | 100GB | $20 |
  | Data Transfer | 1TB | $90 |

#### S3 Storage
- **Bucket Configuration**:
  ```json
  {
    "Bucket": {
      "Versioning": "Enabled",
      "Lifecycle": {
        "Rules": [
          {
            "ID": "TransitionToIA",
            "Prefix": "statements/",
            "Status": "Enabled",
            "Transition": {
              "Days": 90,
              "StorageClass": "STANDARD_IA"
            }
          },
          {
            "ID": "TransitionToGlacier",
            "Prefix": "archive/",
            "Status": "Enabled",
            "Transition": {
              "Days": 365,
              "StorageClass": "GLACIER"
            }
          }
        ]
      },
      "Replication": {
        "Role": "arn:aws:iam::account:role/s3-replication-role",
        "Rules": [
          {
            "ID": "CriticalDataReplication",
            "Status": "Enabled",
            "Destination": {
              "Bucket": "arn:aws:s3:::backup-bucket",
              "StorageClass": "STANDARD"
            }
          }
        ]
      }
    }
  }
  ```

- **Security Configuration**:
  ```json
  {
    "ServerSideEncryption": {
      "SSEAlgorithm": "aws:kms",
      "KMSMasterKeyID": "arn:aws:kms:region:account:key/key-id"
    },
    "BlockPublicAccess": {
      "BlockPublicAcls": true,
      "IgnorePublicAcls": true,
      "BlockPublicPolicy": true,
      "RestrictPublicBuckets": true
    }
  }
  ```

- **Cost Analysis**:
  | Component | Usage | Monthly Cost |
  |-----------|-------|--------------|
  | Standard Storage | 2TB | $46 |
  | S3 IA Storage | 5TB | $62.50 |
  | Glacier Storage | 10TB | $40 |
  | PUT/COPY/POST/LIST | 100K requests | $0.50 |
  | GET/SELECT | 1M requests | $4.00 |
  | Data Transfer Out | 500GB | $42.50 |

### 2.6 Integration Layer

#### SNS (Simple Notification Service)
- **Architecture Overview**:
  ```mermaid
  graph TD
    A[Publisher Service] --> B[SNS FIFO Topic]
    B --> C[SQS Payment Queue]
    B --> D[SQS Notification Queue]
    B --> E[Lambda Handler]
    B --> F[EventBridge]
  ```

- **Topic Configuration**:
  ```json
  {
    "Topics": {
      "TransactionEvents": {
        "Type": "FIFO",
        "Name": "banking-transactions.fifo",
        "ContentBasedDeduplication": true,
        "Subscriptions": [
          {
            "Protocol": "sqs",
            "Endpoint": "arn:aws:sqs:region:account:payment-processing.fifo",
            "FilterPolicy": {
              "eventType": ["PAYMENT_INITIATED", "PAYMENT_COMPLETED"],
              "amount": [{"numeric": [">", 10000]}]
            }
          },
          {
            "Protocol": "https",
            "Endpoint": "https://api.banking.com/webhooks",
            "FilterPolicy": {
              "eventType": ["LARGE_TRANSACTION"]
            }
          }
        ]
      },
      "NotificationEvents": {
        "Type": "Standard",
        "Name": "customer-notifications",
        "Subscriptions": [
          {
            "Protocol": "sqs",
            "Endpoint": "arn:aws:sqs:region:account:email-notifications"
          },
          {
            "Protocol": "sqs",
            "Endpoint": "arn:aws:sqs:region:account:sms-notifications"
          }
        ]
      }
    }
  }
  ```

- **Message Structure and Implementation**:
  ```typescript
  interface SNSMessage<T> {
    id: string;
    type: EventType;
    timestamp: string;
    version: string;
    data: T;
    metadata: {
      origin: string;
      correlation_id: string;
    };
  }

  // Message Publisher Implementation
  class EventPublisher {
    constructor(private readonly sns: AWS.SNS) {}

    async publishTransaction(event: TransactionEvent): Promise<void> {
      const message: SNSMessage<TransactionEvent> = {
        id: uuidv4(),
        type: EventType.TRANSACTION,
        timestamp: new Date().toISOString(),
        version: '1.0',
        data: event,
        metadata: {
          origin: 'payment-service',
          correlation_id: event.correlationId
        }
      };

      const params = {
        TopicArn: process.env.TRANSACTION_TOPIC_ARN,
        Message: JSON.stringify(message),
        MessageGroupId: event.accountId, // For FIFO topics
        MessageDeduplicationId: event.transactionId
      };

      try {
        await this.sns.publish(params).promise();
      } catch (error) {
        this.handlePublishError(error, message);
      }
    }

    private async handlePublishError(error: Error, message: SNSMessage<any>): Promise<void> {
      // Implement dead-letter queue logic
      // Implement retry with exponential backoff
      // Log error metrics
    }
  }
  ```

- **Message Filtering Example**:
  ```json
  {
    "FilterPolicy": {
      "event_type": ["transaction_completed", "transaction_failed"],
      "amount": [{"numeric": [">", 1000]}],
      "currency": ["AED", "USD"],
      "account_type": ["savings", "current"],
      "priority": ["high"]
    }
  }
  ```

- **Monitoring and Alerting**:
  ```yaml
  CloudWatch:
    Metrics:
      - Name: NumberOfMessagesPublished
        Threshold: 1000
        Period: 300
        EvaluationPeriods: 2
        
      - Name: NumberOfNotificationsDelivered
        Threshold: 950
        Period: 300
        EvaluationPeriods: 2
        
    Alarms:
      - Name: HighMessageFailureRate
        Metric: NumberOfMessageDeliveryFailures
        Threshold: 50
        Period: 300
        Action: SNS:OperationsTeam
  ```

- **Cost Analysis**:
  | Component | Usage | Monthly Cost |
  |-----------|-------|--------------|
  | Published Messages | 10M | $50.00 |
  | FIFO Messages | 5M | $75.00 |
  | HTTP/S Deliveries | 1M | $6.00 |
  | Filter Evaluations | 10M | $0.00 |

#### SQS (Simple Queue Service)
- **Queue Architecture**:
  ```yaml
  QueueTypes:
    FIFO:
      - Name: payment-processing.fifo
        MessageRetention: 14 days
        VisibilityTimeout: 300
        MaxReceiveCount: 3
        DeadLetterQueue: dlq-payment-processing.fifo
        
      - Name: transaction-processing.fifo
        MessageRetention: 14 days
        VisibilityTimeout: 600
        MaxReceiveCount: 5
        DeadLetterQueue: dlq-transaction-processing.fifo
        
    Standard:
      - Name: notification-dispatch
        MessageRetention: 7 days
        VisibilityTimeout: 180
        MaxReceiveCount: 3
        DeadLetterQueue: dlq-notification-dispatch
  ```

- **Queue Configuration Example**:
  ```json
  {
    "QueueConfiguration": {
      "FifoQueue": true,
      "ContentBasedDeduplication": true,
      "DeduplicationScope": "messageGroup",
      "FifoThroughputLimit": "perMessageGroupId",
      "VisibilityTimeout": 300,
      "MessageRetentionPeriod": 1209600,
      "MaximumMessageSize": 262144,
      "DelaySeconds": 0,
      "ReceiveMessageWaitTimeSeconds": 20,
      "RedrivePolicy": {
        "deadLetterTargetArn": "arn:aws:sqs:region:account:dlq-payment-processing.fifo",
        "maxReceiveCount": 3
      },
      "Tags": {
        "Environment": "Production",
        "Team": "Payments"
      }
    }
  }
  ```

- **Message Processing Implementation**:
  ```typescript
  class QueueProcessor {
    constructor(
      private readonly sqs: AWS.SQS,
      private readonly queueUrl: string,
      private readonly processor: MessageProcessor
    ) {}

    async startProcessing(): Promise<void> {
      while (true) {
        try {
          const messages = await this.receiveMessages();
          await this.processMessages(messages);
        } catch (error) {
          this.handleProcessingError(error);
        }
      }
    }

    private async processMessages(messages: AWS.SQS.Message[]): Promise<void> {
      const processPromises = messages.map(async (message) => {
        try {
          await this.processor.process(message);
          await this.deleteMessage(message);
        } catch (error) {
          if (this.shouldRetry(error)) {
            // Message will return to queue after visibility timeout
            this.logRetry(message, error);
          } else {
            // Move to DLQ after max retries
            await this.deleteMessage(message);
          }
        }
      });

      await Promise.allSettled(processPromises);
    }

    private shouldRetry(error: Error): boolean {
      // Implement retry decision logic
      // Consider error types, retry counts, etc.
      return error.isTransient === true;
    }
  }

  // Extended Batching Implementation
  class BatchMessageProcessor {
    private messageBuffer: AWS.SQS.Message[] = [];
    private readonly maxBatchSize = 10;
    private readonly maxBatchWindow = 5000; // ms

    async bufferMessage(message: AWS.SQS.Message): Promise<void> {
      this.messageBuffer.push(message);
      
      if (this.shouldProcessBatch()) {
        await this.processBatch();
      }
    }

    private async processBatch(): Promise<void> {
      const batch = this.messageBuffer.splice(0, this.maxBatchSize);
      // Implement batch processing logic
    }
  }
  ```

- **Dead Letter Queue Handling**:
  ```typescript
  class DeadLetterQueueHandler {
    async processDeadLetters(): Promise<void> {
      const messages = await this.receiveDLQMessages();
      
      for (const message of messages) {
        const analysis = await this.analyzeFailure(message);
        
        if (analysis.isRecoverable) {
          await this.requeueMessage(message);
        } else {
          await this.logPermanentFailure(message);
        }
      }
    }

    private async analyzeFailure(message: AWS.SQS.Message): Promise<FailureAnalysis> {
      // Implement failure analysis logic
      // Consider message age, retry count, error patterns
    }
  }
  ```

- **Performance Monitoring**:
  ```yaml
  Metrics:
    - Name: ApproximateAgeOfOldestMessage
      Threshold: 300
      AlarmActions: NotifyOperations
      
    - Name: ApproximateNumberOfMessagesVisible
      Threshold: 1000
      AlarmActions: ScaleConsumers
      
    - Name: NumberOfMessagesDeleted
      Period: 300
      Threshold: 100
      AlarmActions: NotifyOperations
  ```

- **Cost Analysis**:
  | Component | Usage | Monthly Cost |
  |-----------|-------|--------------|
  | Standard Requests | 10M | $40.00 |
  | FIFO Requests | 5M | $50.00 |
  | Data Transfer | 100GB | $9.00 |
  | Extended Client | 1M messages | $0.00 |

- **Best Practices Implementation**:
  ```yaml
  MessageHandling:
    Idempotency:
      - Message deduplication IDs
      - Idempotency tokens in payload
      - Transaction logs
      
    ErrorHandling:
      - Circuit breaker pattern
      - Exponential backoff
      - Failure categorization
      
    Performance:
      - Long polling enabled
      - Batch processing
      - Concurrent consumers
      
    Monitoring:
      - Message age tracking
      - DLQ monitoring
      - Processing latency alerts
  ```

## 3. Service Interconnections

### 3.1 Client-Edge Communication
```mermaid
Clients -> CloudFront: HTTPS/WSS (TLS 1.3)
CloudFront -> WAF: L7 Filtering (10K RPS)
WAF -> API Gateway: REST/WebSocket
```

### 3.2 Authentication Flow
```mermaid
API Gateway -> Cognito: OAuth2/OIDC
Cognito -> Services: JWT Tokens
```

### 3.3 Service Communication
- **Synchronous**: REST APIs with circuit breakers
- **Asynchronous**: Event-driven via SNS/SQS
- **Caching**: Multi-level with Redis and DAX

### 3.4 Data Access Patterns
- **Read Paths**: 
  - Cache-aside with Redis
  - Write-through for critical data
- **Write Paths**:
  - Transactional writes to Aurora
  - Eventually consistent updates to DynamoDB

## 4. Technical Decisions and Trade-offs

### 4.1 Container Orchestration (EKS)
- **Why EKS over ECS**: 
  - Better multi-tenant isolation
  - Advanced deployment strategies
  - Kubernetes ecosystem
- **Trade-offs**:
  - Higher operational complexity
  - Increased management overhead
  - Higher cost compared to ECS

### 4.2 Database Choices
- **Aurora MySQL**:
  - Pros: ACID compliance, SQL flexibility
  - Cons: Higher cost, scaling complexity
- **DynamoDB**:
  - Pros: Auto-scaling, managed service
  - Cons: Limited transaction support, eventual consistency

### 4.3 Caching Strategy
- **Redis over ElastiCache Memcached**:
  - Advanced data structures
  - Multi-AZ with automatic failover
  - Better persistence options

### 4.4 Queue Implementation
- **FIFO Queues**:
  - Ensures ordered processing
  - Exactly-once delivery
  - Higher cost but necessary for financial transactions

## 5. Compliance and Security Measures

### 5.1 Data Protection
- Encryption at rest using KMS
- TLS 1.3 for all communications
- Field-level encryption for sensitive data

### 5.2 Access Control
- IAM roles for service accounts
- RBAC in Kubernetes
- Zero-trust network policies

### 5.3 Monitoring and Audit
- CloudWatch for operational metrics
- AWS CloudTrail for API auditing
- Custom audit logging for business events

## 6. Build vs. Buy Decisions

### 6.1 Build
1. **Core Banking Services**
   - Account Management
   - Payment Processing
   - Notification System
   - Rationale: Core business logic, requires full control

2. **Integration Layer**
   - Service Mesh
   - API Gateway Rules
   - Rationale: Custom requirements, security needs

### 6.2 Buy/Integrate
1. **SWIFT Gateway**
   - Third-party solution
   - Certified provider
   - Rationale: Compliance requirements, time-to-market

2. **Biller Aggregation**
   - Local UAE provider
   - Pre-built integrations
   - Rationale: Existing relationships, faster implementation

## 7. Scalability and Performance

### 7.1 Horizontal Scaling
- Auto-scaling groups for EKS nodes
- Read replicas for Aurora
- DAX clusters for DynamoDB

### 7.2 Performance Optimization
- CDN caching for static content
- Multi-level caching strategy
- Asynchronous processing for non-critical operations

## 8. Disaster Recovery and High Availability

### 8.1 Multi-AZ Setup
- Active-active deployment
- Automatic failover for databases
- Cross-zone load balancing

### 8.2 Backup Strategy
- Point-in-time recovery for Aurora
- Cross-region replication for critical data
- Regular snapshot backups

## 9. Cost Optimization

### 9.1 Compute
- Reserved instances for baseline load
- Spot instances for batch processing
- Auto-scaling based on demand

### 9.2 Storage
- S3 lifecycle policies
- Aurora storage auto-scaling
- DynamoDB on-demand mode

## 10. Future Considerations

### 10.1 Technology Evolution
- Serverless adoption where applicable
- GraphQL for API optimization
- Service mesh implementation

### 10.2 Scale Requirements
- Global expansion readiness
- Multi-region deployment
- Cross-region data replication
