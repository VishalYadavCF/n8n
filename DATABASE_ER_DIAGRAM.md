# n8n Database Entity-Relationship Diagram

This document contains Entity-Relationship diagrams for the n8n database schema using Mermaid notation.

## Core Schema Overview

```mermaid
erDiagram
    User ||--o{ ProjectRelation : "has"
    User ||--o{ AuthIdentity : "has"
    User ||--o{ ApiKey : "has"
    User ||--|| Role : "has global role"
    
    Project ||--o{ ProjectRelation : "has"
    Project ||--o{ SharedWorkflow : "contains"
    Project ||--o{ SharedCredentials : "contains"
    Project ||--o{ Variables : "owns"
    Project ||--|| User : "created by"
    
    ProjectRelation }o--|| Role : "has project role"
    
    WorkflowEntity ||--o{ ExecutionEntity : "executes"
    WorkflowEntity ||--o{ SharedWorkflow : "shared in"
    WorkflowEntity ||--o{ WorkflowHistory : "versioned in"
    WorkflowEntity ||--o{ WorkflowStatistics : "tracks"
    WorkflowEntity ||--o{ WorkflowTagMapping : "tagged with"
    WorkflowEntity ||--o{ WebhookEntity : "registers"
    WorkflowEntity ||--o{ TestRun : "tested by"
    WorkflowEntity ||--o{ WorkflowDependency : "depends on"
    WorkflowEntity }o--|| Folder : "organized in"
    WorkflowEntity }o--|| WorkflowHistory : "active version"
    
    CredentialsEntity ||--o{ SharedCredentials : "shared in"
    
    ExecutionEntity ||--|| ExecutionData : "has data"
    ExecutionEntity ||--o{ ExecutionMetadata : "has metadata"
    ExecutionEntity ||--o| ExecutionAnnotation : "has annotation"
    
    ExecutionAnnotation ||--o{ AnnotationTagMapping : "tagged with"
    
    TagEntity ||--o{ WorkflowTagMapping : "tags"
    TagEntity ||--o{ FolderTagMapping : "tags"
    TagEntity ||--o{ AnnotationTagMapping : "tags"
    
    Folder ||--o{ Folder : "contains"
    Folder ||--o{ WorkflowEntity : "contains"
    Folder ||--o{ FolderTagMapping : "tagged with"
    Folder }o--|| Project : "belongs to"
    
    WorkflowHistory ||--o{ WorkflowPublishHistory : "published as"
    
    TestRun ||--o{ TestCaseExecution : "contains"
    
    Role ||--o{ Scope : "has scopes"
```

## User & Authentication Schema

```mermaid
erDiagram
    User {
        uuid id PK
        varchar email UK "Unique, lowercase"
        varchar firstName
        varchar lastName
        varchar password "Hashed"
        json personalizationAnswers
        json settings
        varchar roleSlug FK
        boolean disabled
        boolean mfaEnabled
        varchar mfaSecret
        array mfaRecoveryCodes
        date lastActiveAt
        timestamp createdAt
        timestamp updatedAt
    }
    
    Role {
        varchar slug PK "e.g. global:owner"
        varchar displayName
        varchar description
        boolean systemRole
        varchar roleType "global|project|workflow|credential"
        timestamp createdAt
        timestamp updatedAt
    }
    
    Scope {
        varchar slug PK
        varchar displayName
        varchar description
    }
    
    AuthIdentity {
        varchar providerId PK
        varchar providerType PK "ldap|saml|email"
        varchar userId FK
        timestamp createdAt
        timestamp updatedAt
    }
    
    ApiKey {
        varchar id PK
        varchar userId FK
        varchar label
        array scopes
        varchar apiKey UK "Hashed"
        varchar audience "public-api|mcp"
        timestamp createdAt
        timestamp updatedAt
    }
    
    InvalidAuthToken {
        varchar token PK
        varchar userId FK
        timestamp expiresAt
    }
    
    User ||--|| Role : has
    User ||--o{ AuthIdentity : has
    User ||--o{ ApiKey : has
    User ||--o{ InvalidAuthToken : blacklists
    Role }o--o{ Scope : has
```

## Project & Collaboration Schema

```mermaid
erDiagram
    Project {
        varchar id PK
        varchar name
        varchar type "personal|team"
        json icon
        varchar description
        varchar creatorId FK
        timestamp createdAt
        timestamp updatedAt
    }
    
    ProjectRelation {
        uuid userId PK,FK
        varchar projectId PK,FK
        varchar role FK "Role slug"
        timestamp createdAt
        timestamp updatedAt
    }
    
    SharedWorkflow {
        varchar workflowId PK,FK
        varchar projectId PK,FK
        varchar role "workflow:owner|workflow:editor"
        timestamp createdAt
        timestamp updatedAt
    }
    
    SharedCredentials {
        varchar credentialsId PK,FK
        varchar projectId PK,FK
        varchar role "credential:owner|credential:user"
        timestamp createdAt
        timestamp updatedAt
    }
    
    Variables {
        varchar id PK
        text key
        text type
        text value
        varchar projectId FK "Nullable for global vars"
    }
    
    User ||--o{ ProjectRelation : member
    Project ||--o{ ProjectRelation : has
    Project ||--o{ SharedWorkflow : shares
    Project ||--o{ SharedCredentials : shares
    Project ||--o{ Variables : owns
    Project }o--|| User : "created by"
    Role ||--o{ ProjectRelation : assigns
    WorkflowEntity ||--o{ SharedWorkflow : "shared via"
    CredentialsEntity ||--o{ SharedCredentials : "shared via"
```

## Workflow Management Schema

```mermaid
erDiagram
    WorkflowEntity {
        varchar id PK
        varchar name UK
        text description
        boolean active "Deprecated"
        boolean isArchived
        json nodes
        json connections
        json settings
        json staticData
        json meta
        json pinData
        varchar versionId
        varchar activeVersionId FK "to WorkflowHistory"
        int versionCounter
        int triggerCount
        varchar parentFolderId FK
        timestamp createdAt
        timestamp updatedAt
    }
    
    WorkflowHistory {
        varchar versionId PK
        varchar workflowId FK
        json nodes
        json connections
        varchar authors "Comma-separated user IDs"
        varchar name
        varchar description
        timestamp createdAt
        timestamp updatedAt
    }
    
    WorkflowPublishHistory {
        varchar id PK
        varchar workflowId FK
        varchar versionId FK
        varchar publishedBy FK "User ID"
        timestamp publishedAt
    }
    
    WorkflowStatistics {
        varchar workflowId PK,FK
        varchar name
        int count
        varchar latestEvent
        timestamp createdAt
        timestamp updatedAt
    }
    
    WorkflowDependency {
        varchar workflowId PK,FK
        varchar dependsOnWorkflowId PK,FK
    }
    
    WorkflowTagMapping {
        varchar workflowId PK,FK
        varchar tagId PK,FK
    }
    
    WebhookEntity {
        varchar webhookPath PK
        text method PK "GET|POST|etc"
        varchar workflowId FK
        varchar node
        varchar webhookId
        int pathLength
    }
    
    Folder {
        varchar id PK
        varchar name
        varchar parentFolderId FK "Self-reference"
        varchar projectId FK
        timestamp createdAt
        timestamp updatedAt
    }
    
    WorkflowEntity ||--o{ WorkflowHistory : versions
    WorkflowEntity }o--|| WorkflowHistory : "active version"
    WorkflowEntity ||--o{ WorkflowStatistics : statistics
    WorkflowEntity ||--o{ WorkflowDependency : dependencies
    WorkflowEntity ||--o{ WorkflowTagMapping : tags
    WorkflowEntity ||--o{ WebhookEntity : webhooks
    WorkflowEntity }o--|| Folder : "in folder"
    WorkflowHistory ||--o{ WorkflowPublishHistory : publications
    Folder ||--o{ Folder : "sub-folders"
    Folder }o--|| Project : "home project"
```

## Execution Tracking Schema

```mermaid
erDiagram
    ExecutionEntity {
        int id PK "Auto-increment"
        varchar workflowId FK
        boolean finished "Deprecated"
        varchar status "running|success|error|waiting|etc"
        varchar mode "trigger|webhook|manual|etc"
        varchar retryOf FK
        varchar retrySuccessId FK
        timestamp createdAt
        timestamp startedAt
        timestamp stoppedAt
        timestamp deletedAt "Soft delete"
        timestamp waitTill
    }
    
    ExecutionData {
        varchar executionId PK,FK
        text data "Compressed execution data"
        json workflowData "Workflow snapshot"
    }
    
    ExecutionMetadata {
        int id PK
        varchar executionId FK
        text key
        text value
    }
    
    ExecutionAnnotation {
        varchar id PK
        varchar executionId FK,UK
        varchar vote "up|down"
        text note
        timestamp createdAt
        timestamp updatedAt
    }
    
    AnnotationTagMapping {
        varchar annotationId PK,FK
        varchar tagId PK,FK
    }
    
    WorkflowEntity ||--o{ ExecutionEntity : executes
    ExecutionEntity ||--|| ExecutionData : "has data"
    ExecutionEntity ||--o{ ExecutionMetadata : "has metadata"
    ExecutionEntity ||--o| ExecutionAnnotation : "has annotation"
    ExecutionAnnotation ||--o{ AnnotationTagMapping : "tagged with"
    TagEntity ||--o{ AnnotationTagMapping : tags
```

## Credentials Schema

```mermaid
erDiagram
    CredentialsEntity {
        varchar id PK
        varchar name
        varchar type "slackApi|googleSheetsOAuth2|etc"
        text data "Encrypted"
        boolean isManaged
        boolean isGlobal
        timestamp createdAt
        timestamp updatedAt
    }
    
    CredentialsEntity ||--o{ SharedCredentials : "shared via"
```

## Tag System Schema

```mermaid
erDiagram
    TagEntity {
        varchar id PK
        varchar name UK
        timestamp createdAt
        timestamp updatedAt
    }
    
    WorkflowTagMapping {
        varchar workflowId PK,FK
        varchar tagId PK,FK
    }
    
    FolderTagMapping {
        varchar folderId PK,FK
        varchar tagId PK,FK
    }
    
    AnnotationTagMapping {
        varchar annotationId PK,FK
        varchar tagId PK,FK
    }
    
    TagEntity ||--o{ WorkflowTagMapping : "tags workflows"
    TagEntity ||--o{ FolderTagMapping : "tags folders"
    TagEntity ||--o{ AnnotationTagMapping : "tags annotations"
    WorkflowEntity ||--o{ WorkflowTagMapping : "tagged by"
    Folder ||--o{ FolderTagMapping : "tagged by"
    ExecutionAnnotation ||--o{ AnnotationTagMapping : "tagged by"
```

## Settings & Configuration Schema

```mermaid
erDiagram
    Settings {
        varchar key PK
        varchar value
        boolean loadOnStartup
    }
    
    Variables {
        varchar id PK
        text key
        text type
        text value
        varchar projectId FK "Nullable"
    }
    
    EventDestinations {
        varchar id PK
        json destination
        timestamp createdAt
        timestamp updatedAt
    }
    
    Variables }o--o| Project : "scoped to"
```

## Binary Data Schema

```mermaid
erDiagram
    BinaryDataFile {
        varchar id PK
        varchar executionId FK
        text fileName
        text mimeType
        varchar sourceType "workflow|execution"
        bigint size
        timestamp createdAt
    }
    
    ProcessedData {
        varchar id PK
        varchar executionId FK
        varchar nodeId
        int outputIndex
        text data
        timestamp createdAt
    }
    
    BinaryDataFile }o--|| ExecutionEntity : "belongs to"
    ProcessedData }o--|| ExecutionEntity : "belongs to"
```

## Testing Schema (Enterprise Edition)

```mermaid
erDiagram
    TestRun {
        varchar id PK
        varchar workflowId FK
        varchar status "new|running|completed|error"
        varchar runBy FK "User ID"
        varchar completedAt
        timestamp createdAt
        timestamp updatedAt
    }
    
    TestCaseExecution {
        varchar id PK
        varchar testRunId FK
        varchar testCaseId
        varchar status "new|running|passed|failed"
        text errorMessage
        json testData
        timestamp createdAt
        timestamp updatedAt
    }
    
    WorkflowEntity ||--o{ TestRun : "tested by"
    TestRun ||--o{ TestCaseExecution : contains
    User ||--o{ TestRun : runs
```

## Authentication Provider Sync

```mermaid
erDiagram
    AuthProviderSyncHistory {
        varchar id PK
        varchar providerType "ldap|saml"
        timestamp startedAt
        timestamp endedAt
        varchar status "success|error"
        int created
        int updated
        int disabled
        text error
    }
```

## Database Indexes

### ExecutionEntity Indexes
- `idx_execution_workflow_id` on `(workflowId, id)`
- `idx_execution_wait_till` on `(waitTill, id)`
- `idx_execution_finished` on `(finished, id)`
- `idx_execution_workflow_finished` on `(workflowId, finished, id)`
- `idx_execution_workflow_wait` on `(workflowId, waitTill, id)`
- `idx_execution_stopped_at` on `(stoppedAt)`

### WebhookEntity Indexes
- `idx_webhook_routing` on `(webhookId, method, pathLength)`

### User Indexes
- `idx_user_email` on `(email)` - Unique

### WorkflowEntity Indexes
- `idx_workflow_name` on `(name)` - Unique

### CredentialsEntity Indexes
- `idx_credentials_type` on `(type)`

### TagEntity Indexes
- `idx_tag_name` on `(name)` - Unique

### ApiKey Indexes
- `idx_api_key_api_key` on `(apiKey)` - Unique
- `idx_api_key_user_label` on `(userId, label)` - Unique

## Junction Tables (Many-to-Many Relationships)

1. **workflows_tags**: WorkflowEntity ↔ TagEntity
2. **folder_tag**: Folder ↔ TagEntity
3. **role_scope**: Role ↔ Scope
4. **workflow_tag_mapping**: WorkflowEntity ↔ TagEntity (new structure)
5. **folder_tag_mapping**: Folder ↔ TagEntity (new structure)
6. **annotation_tag_mapping**: ExecutionAnnotation ↔ AnnotationTagEntity

## Column Naming Conventions

- **Primary Keys**: `id` (varchar/uuid/auto-increment)
- **Foreign Keys**: `{entity}Id` (e.g., `userId`, `workflowId`)
- **Timestamps**: `createdAt`, `updatedAt`, `deletedAt`, `stoppedAt`, `startedAt`
- **Boolean flags**: `is{Property}`, `{property}Enabled` (e.g., `isArchived`, `mfaEnabled`)
- **Role columns**: `role` or `roleSlug`

## Data Type Mapping

### Common Types
- `varchar` - Variable-length strings
- `text` - Long text content
- `json` - JSON objects (simple-json in SQLite)
- `timestamp`/`timestamptz`/`datetime` - Timestamps
- `boolean` - True/false flags
- `int`/`integer` - Numeric values
- `uuid` - UUID identifiers (user IDs)
- `simple-array` - Comma-separated values (TypeORM feature)

### Database-Specific
- **JSON**: `json` (PostgreSQL/MySQL), `simple-json` (SQLite)
- **Datetime**: `timestamptz` (PostgreSQL), `datetime` (MySQL/SQLite)
- **Binary**: `bytea` (PostgreSQL), `longblob` (MySQL), `blob` (SQLite)

## Cascade Behaviors

### ON DELETE CASCADE
- `ProjectRelation` → When Project deleted
- `SharedWorkflow` → When Workflow deleted
- `SharedCredentials` → When Credentials deleted
- `ExecutionData` → When ExecutionEntity deleted
- `ExecutionMetadata` → When ExecutionEntity deleted
- `WorkflowHistory` → When WorkflowEntity deleted
- `Folder` → When parent Folder deleted
- `Variables` → When Project deleted (for project-scoped vars)
- `ApiKey` → When User deleted

### ON DELETE SET NULL
- `Project.creator` → When User deleted
- `Variables.project` → When Project deleted (converts to global var)

## Transaction Isolation

- Migrations run in individual transactions (`transaction: 'each'`)
- Repository operations use default isolation level
- `withTransaction` helper available for multi-operation transactions

## Performance Considerations

1. **Execution queries** - Use composite indexes for common filtering patterns
2. **Workflow lookups** - Name is unique indexed for fast search
3. **Webhook routing** - Specialized index on `(webhookId, method, pathLength)`
4. **User authentication** - Email unique index for login
5. **Tag filtering** - Junction tables with both foreign keys indexed

## Notes

- This diagram represents the logical schema; actual table names may include a configurable prefix
- Some entities are Enterprise Edition (EE) only, marked with `.ee` suffix in filenames
- Soft deletes are implemented via `deletedAt` timestamp columns
- All timestamps use precision of 3 (milliseconds)
- NanoID is used for most string primary keys (shorter than UUID)
- Credential data is encrypted at the application layer before storage
