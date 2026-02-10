# Agrinext AWS Architecture Diagram

## High-Level AWS Infrastructure

```mermaid
graph TB
    subgraph "User Layer"
        Users[Rural Farmers<br/>Mobile App]
    end
    
    subgraph "Edge & DNS"
        R53[Route 53<br/>DNS]
        CF[CloudFront<br/>CDN & Edge Cache]
    end
    
    subgraph "API Layer"
        APIGW[API Gateway<br/>REST API v1]
        Cognito[AWS Cognito<br/>Authentication]
    end
    
    subgraph "Load Balancing"
        ALB[Application Load Balancer<br/>Multi-AZ]
    end
    
    subgraph "Compute - Application Services"
        subgraph "ECS Fargate Cluster"
            Advisory[Advisory Service<br/>ECS Task]
            Scheme[Government Scheme Service<br/>ECS Task]
            UserSvc[User Service<br/>ECS Task]
            Community[Community Service<br/>ECS Task]
            Weather[Weather Service<br/>ECS Task]
        end
    end
    
    subgraph "AI/ML Services"
        DiseaseAPI[Disease Detection API<br/>Lambda Function]
        SageMaker[SageMaker Endpoint<br/>ML Model Inference]
        NLP[NLP Translation<br/>Lambda Function]
        Rekognition[Amazon Rekognition<br/>Image Quality Check]
    end
    
    subgraph "Messaging & Notifications"
        SQS[SQS Queue<br/>Notification Queue]
        SNS[SNS<br/>Push & SMS]
    end
    
    subgraph "Data Layer"
        subgraph "Relational Database"
            RDS[(RDS PostgreSQL<br/>Multi-AZ<br/>User Data)]
            RDSRead[(Read Replica<br/>PostgreSQL)]
        end
        
        subgraph "NoSQL Database"
            DynamoDB[(DynamoDB<br/>Advisory History<br/>Community Posts)]
        end
        
        subgraph "Cache Layer"
            Redis[(ElastiCache Redis<br/>Session & Cache)]
        end
        
        subgraph "Object Storage"
            S3Images[S3 Bucket<br/>Crop Disease Images]
            S3Models[S3 Bucket<br/>ML Models]
            S3Backup[S3 Bucket<br/>Database Backups]
        end
    end
    
    subgraph "Security & Secrets"
        Secrets[AWS Secrets Manager<br/>API Keys & Credentials]
        WAF[AWS WAF<br/>Web Application Firewall]
    end
    
    subgraph "Monitoring & Logging"
        CloudWatch[CloudWatch<br/>Logs & Metrics]
        XRay[X-Ray<br/>Distributed Tracing]
    end
    
    subgraph "External Integrations"
        IMD[IMD Weather API<br/>External]
        GovAPI[Government Schemes API<br/>External]
        SMSGateway[SMS Gateway<br/>External]
    end
    
    %% User Flow
    Users --> R53
    R53 --> CF
    CF --> APIGW
    
    %% API Gateway Flow
    APIGW --> Cognito
    APIGW --> WAF
    APIGW --> ALB
    
    %% Load Balancer to Services
    ALB --> Advisory
    ALB --> Scheme
    ALB --> UserSvc
    ALB --> Community
    ALB --> Weather
    
    %% Service to Database Connections
    UserSvc --> RDS
    UserSvc --> RDSRead
    Advisory --> DynamoDB
    Community --> DynamoDB
    Scheme --> Redis
    Weather --> Redis
    
    %% Disease Detection Flow
    APIGW --> DiseaseAPI
    DiseaseAPI --> Rekognition
    DiseaseAPI --> SageMaker
    DiseaseAPI --> S3Images
    SageMaker --> S3Models
    
    %% Translation Service
    Advisory --> NLP
    Scheme --> NLP
    
    %% Notification Flow
    Advisory --> SQS
    Weather --> SQS
    DiseaseAPI --> SQS
    Scheme --> SQS
    SQS --> SNS
    SNS --> SMSGateway
    
    %% External API Integrations
    Weather --> IMD
    Scheme --> GovAPI
    
    %% Security
    Secrets -.-> Advisory
    Secrets -.-> Scheme
    Secrets -.-> Weather
    Secrets -.-> DiseaseAPI
    
    %% Monitoring
    Advisory -.-> CloudWatch
    DiseaseAPI -.-> CloudWatch
    SageMaker -.-> CloudWatch
    APIGW -.-> CloudWatch
    ALB -.-> XRay
    
    %% Backup
    RDS -.-> S3Backup
    DynamoDB -.-> S3Backup
    
    style Users fill:#e1f5ff
    style R53 fill:#ff9900
    style CF fill:#ff9900
    style APIGW fill:#ff9900
    style Cognito fill:#dd344c
    style ALB fill:#ff9900
    style Advisory fill:#ff9900
    style Scheme fill:#ff9900
    style UserSvc fill:#ff9900
    style Community fill:#ff9900
    style Weather fill:#ff9900
    style DiseaseAPI fill:#ff9900
    style SageMaker fill:#01a88d
    style NLP fill:#ff9900
    style RDS fill:#3b48cc
    style DynamoDB fill:#3b48cc
    style Redis fill:#3b48cc
    style S3Images fill:#569a31
    style S3Models fill:#569a31
    style CloudWatch fill:#ff9900
```

## Detailed Component Breakdown

### 1. **Edge & DNS Layer**
- **Route 53**: DNS management and health checks
- **CloudFront**: CDN for static assets, API caching, and edge locations across India

### 2. **API & Security Layer**
- **API Gateway**: RESTful API endpoint management, rate limiting (100 req/min per user)
- **AWS Cognito**: User authentication with mobile OTP
- **AWS WAF**: Protection against common web exploits

### 3. **Compute Layer (ECS Fargate)**
- **Advisory Service**: Provides personalized farming advice
- **Government Scheme Service**: Manages scheme information and applications
- **User Service**: User profile and preference management
- **Community Service**: Forum and peer-to-peer communication
- **Weather Service**: Weather data aggregation and advisories

### 4. **AI/ML Layer**
- **Lambda (Disease Detection API)**: Serverless API for disease detection requests
- **SageMaker Endpoint**: Hosts TensorFlow/PyTorch models for disease classification
- **Lambda (NLP Translation)**: Translates content to 10+ Indian languages
- **Amazon Rekognition**: Validates image quality before ML processing

### 5. **Data Layer**
- **RDS PostgreSQL (Multi-AZ)**: User data, scheme applications, preferences
- **Read Replica**: Offload read queries for better performance
- **DynamoDB**: Advisory history, community posts, disease detection logs
- **ElastiCache Redis**: Session management, weather cache, scheme cache
- **S3 Buckets**: 
  - Crop disease images
  - ML model artifacts
  - Database backups

### 6. **Messaging & Notifications**
- **SQS**: Decouples notification generation from delivery
- **SNS**: Sends push notifications and SMS alerts

### 7. **Monitoring & Operations**
- **CloudWatch**: Centralized logging, metrics, and alarms
- **X-Ray**: Distributed tracing for performance optimization

### 8. **External Integrations**
- **IMD Weather API**: Real-time weather data
- **Government Schemes API**: Latest scheme information
- **SMS Gateway**: SMS delivery for critical alerts

## AWS Services Summary

| Service | Purpose | Configuration |
|---------|---------|---------------|
| Route 53 | DNS Management | Health checks, failover routing |
| CloudFront | CDN | Edge caching, HTTPS, India edge locations |
| API Gateway | API Management | REST API, rate limiting, API keys |
| Cognito | Authentication | Mobile OTP, JWT tokens |
| WAF | Security | SQL injection, XSS protection |
| ALB | Load Balancing | Multi-AZ, health checks |
| ECS Fargate | Container Orchestration | Auto-scaling, 2-10 tasks per service |
| Lambda | Serverless Compute | Disease detection, NLP translation |
| SageMaker | ML Model Hosting | Real-time inference endpoint |
| RDS PostgreSQL | Relational Database | Multi-AZ, automated backups |
| DynamoDB | NoSQL Database | On-demand capacity, global tables |
| ElastiCache Redis | Caching | Cluster mode, Multi-AZ |
| S3 | Object Storage | Versioning, lifecycle policies |
| SQS | Message Queue | Standard queue, DLQ enabled |
| SNS | Notifications | SMS, push notifications |
| Secrets Manager | Secret Management | Automatic rotation |
| CloudWatch | Monitoring | Logs, metrics, alarms |
| X-Ray | Tracing | Service map, latency analysis |

## Scalability & High Availability

### Auto-Scaling Configuration
- **ECS Services**: Scale 2-10 tasks based on CPU (70%) and memory (80%)
- **SageMaker**: Auto-scaling based on invocations per instance
- **DynamoDB**: On-demand capacity mode for unpredictable traffic
- **ElastiCache**: Cluster mode with 3-6 shards

### High Availability
- **Multi-AZ Deployment**: RDS, ALB, ECS tasks across 3 AZs
- **Read Replicas**: PostgreSQL read replica for query offloading
- **Health Checks**: ALB health checks every 30 seconds
- **Failover**: Automatic RDS failover in <2 minutes

### Disaster Recovery
- **RDS Backups**: Automated daily backups, 7-day retention
- **DynamoDB**: Point-in-time recovery enabled
- **S3 Versioning**: Enabled on all buckets
- **Cross-Region Replication**: S3 backup bucket replicated to secondary region

## Cost Optimization

1. **Compute**: ECS Fargate Spot for non-critical workloads (50% cost savings)
2. **Storage**: S3 Intelligent-Tiering for images (automatic cost optimization)
3. **Database**: RDS Reserved Instances (40% savings)
4. **Cache**: ElastiCache Reserved Nodes (30% savings)
5. **Lambda**: Provisioned concurrency only for disease detection API
6. **Data Transfer**: CloudFront reduces data transfer costs

## Security Architecture

1. **Network Security**: VPC with private subnets for databases and compute
2. **Encryption**: 
   - At rest: RDS, DynamoDB, S3 (AWS KMS)
   - In transit: TLS 1.2+ for all communications
3. **Access Control**: IAM roles with least privilege
4. **Secrets**: Secrets Manager with automatic rotation
5. **WAF Rules**: OWASP Top 10 protection
6. **DDoS Protection**: AWS Shield Standard (automatic)

## Estimated Monthly Cost (for 100,000 active users)

| Service | Estimated Cost |
|---------|----------------|
| ECS Fargate | $500 |
| Lambda | $200 |
| SageMaker | $800 |
| RDS PostgreSQL | $400 |
| DynamoDB | $300 |
| ElastiCache | $200 |
| S3 | $150 |
| CloudFront | $100 |
| API Gateway | $150 |
| SNS/SQS | $100 |
| Other Services | $100 |
| **Total** | **~$3,000/month** |

*Note: Costs scale with usage. Estimated for 100K active users with 1M API calls/day.*
