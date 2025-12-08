# n8n Database Schema Documentation

## Overview

n8n uses TypeORM as its Object-Relational Mapping (ORM) layer to interact with various database systems. The database schema is designed to support workflow automation, user management, credential storage, execution tracking, and more.

## Supported Databases

n8n supports the following database systems:

1. **SQLite** (default) - File-based database, ideal for development and small deployments
2. **PostgreSQL** - Recommended for production deployments
3. **MySQL** - Alternative production database
4. **MariaDB** - MySQL-compatible database system

## Database Configuration

The database configuration is centralized in the `@n8n/db` package:

- **Location**: `packages/@n8n/db/src/connection/`
- **Main Classes**:
  - `DbConnectionOptions` - Configures connection options for different database types
  - `DbConnection` - Manages database connection lifecycle and health checks

### Connection Options

Each database type has specific configuration:

- **SQLite**: Database file path, WAL mode, optional connection pooling
- **PostgreSQL**: Host, port, username, password, SSL configuration, connection timeout, pool size, schema
- **MySQL/MariaDB**: Host, port, username, password, pool size, timezone (UTC)

## Database Structure

### Core Entities

The database schema is organized into the following main entity categories:

#### 1. **Workflow Management**

| Entity | Table | Description |
|--------|-------|-------------|
| `WorkflowEntity` | `workflow_entity` | Stores workflow definitions including nodes, connections, and settings |
| `WorkflowHistory` | `workflow_history` | Tracks version history of workflows |
| `WorkflowPublishHistory` | `workflow_publish_history` | Records workflow publishing events |
| `WorkflowStatistics` | `workflow_statistics` | Stores workflow execution statistics |
| `WorkflowDependency` | `workflow_dependency` | Tracks dependencies between workflows |
| `WorkflowTagMapping` | `workflow_tag_mapping` | Junction table for workflow-tag relationships |

**WorkflowEntity Key Columns:**
- `id` (varchar, PK) - Unique workflow identifier (NanoID)
- `name` (varchar(128)) - Workflow name (unique index)
- `description` (text, nullable) - Workflow description
- `active` (boolean) - Whether workflow is active (deprecated, use `activeVersionId`)
- `isArchived` (boolean) - Soft delete flag
- `nodes` (json) - Workflow node definitions
- `connections` (json) - Node connections
- `settings` (json, nullable) - Workflow settings
- `staticData` (json, nullable) - Persistent workflow data
- `meta` (json, nullable) - Frontend metadata
- `pinData` (json, nullable) - Pinned test data
- `versionId` (varchar(36)) - Current version identifier
- `activeVersionId` (varchar(36), nullable) - Active version reference
- `versionCounter` (integer) - Version counter
- `triggerCount` (integer) - Number of triggers
- `parentFolderId` (varchar, nullable) - Parent folder reference
- `createdAt` (timestamp) - Creation timestamp
- `updatedAt` (timestamp) - Last update timestamp

#### 2. **Execution Management**

| Entity | Table | Description |
|--------|-------|-------------|
| `ExecutionEntity` | `execution_entity` | Stores execution metadata and status |
| `ExecutionData` | `execution_data` | Stores execution input/output data |
| `ExecutionMetadata` | `execution_metadata` | Additional key-value metadata for executions |
| `ExecutionAnnotation` | `execution_annotation` | Execution annotations (EE feature) |

**ExecutionEntity Key Columns:**
- `id` (auto-increment, PK) - Unique execution identifier
- `workflowId` (varchar, nullable) - Associated workflow ID
- `finished` (boolean) - Execution completion status (deprecated)
- `status` (varchar) - Execution status (running, success, error, waiting, etc.)
- `mode` (varchar) - Execution mode (trigger, webhook, manual, etc.)
- `retryOf` (varchar, nullable) - Original execution if this is a retry
- `retrySuccessId` (varchar, nullable) - Successful retry execution ID
- `createdAt` (timestamp) - Execution creation time
- `startedAt` (timestamp, nullable) - Execution start time
- `stoppedAt` (timestamp, nullable) - Execution stop time
- `deletedAt` (timestamp, nullable) - Soft delete timestamp
- `waitTill` (timestamp, nullable) - Wait until timestamp for paused executions

**Indexes:**
- `(workflowId, id)` - Fast workflow execution lookup
- `(waitTill, id)` - Efficient querying of waiting executions
- `(finished, id)` - Legacy index for finished executions
- `(workflowId, finished, id)` - Combined workflow/status queries
- `(workflowId, waitTill, id)` - Combined workflow/wait queries

#### 3. **User & Authentication**

| Entity | Table | Description |
|--------|-------|-------------|
| `User` | `user` | User account information |
| `Role` | `role` | Role definitions (global, project, workflow, credential) |
| `AuthIdentity` | `auth_identity` | External authentication provider identities |
| `AuthProviderSyncHistory` | `auth_provider_sync_history` | LDAP/SAML sync history |
| `ApiKey` | `user_api_keys` | API keys for authentication |
| `InvalidAuthToken` | `invalid_auth_token` | Blacklisted tokens |

**User Key Columns:**
- `id` (uuid, PK) - Unique user identifier
- `email` (varchar(254), unique, nullable) - User email (lowercase)
- `firstName` (varchar(32), nullable) - First name
- `lastName` (varchar(32), nullable) - Last name
- `password` (varchar, nullable) - Hashed password
- `personalizationAnswers` (json, nullable) - Onboarding survey answers
- `settings` (json, nullable) - User preferences
- `roleSlug` (varchar) - Foreign key to Role
- `disabled` (boolean) - Account disabled flag
- `mfaEnabled` (boolean) - MFA enabled flag
- `mfaSecret` (varchar, nullable) - MFA secret
- `mfaRecoveryCodes` (simple-array) - MFA recovery codes
- `lastActiveAt` (date, nullable) - Last activity timestamp
- `createdAt` (timestamp) - Account creation time
- `updatedAt` (timestamp) - Last update time

**Role Key Columns:**
- `slug` (varchar, PK) - Role identifier (e.g., 'global:owner', 'project:admin')
- `displayName` (varchar) - Human-readable role name
- `description` (varchar, nullable) - Role description
- `systemRole` (boolean) - System-managed role flag
- `roleType` (varchar) - Role category (global, project, workflow, credential)
- `createdAt` (timestamp) - Creation timestamp
- `updatedAt` (timestamp) - Update timestamp

#### 4. **Project & Collaboration**

| Entity | Table | Description |
|--------|-------|-------------|
| `Project` | `project` | Project/team workspaces |
| `ProjectRelation` | `project_relation` | User-project membership with roles |
| `SharedWorkflow` | `shared_workflow` | Workflow sharing within projects |
| `SharedCredentials` | `shared_credentials` | Credential sharing within projects |

**Project Key Columns:**
- `id` (varchar, PK) - Unique project identifier (NanoID)
- `name` (varchar(255)) - Project name
- `type` (varchar(36)) - Project type ('personal' or 'team')
- `icon` (json, nullable) - Project icon (emoji or icon type)
- `description` (varchar(512), nullable) - Project description
- `creatorId` (varchar, nullable) - User who created the project
- `createdAt` (timestamp) - Creation timestamp
- `updatedAt` (timestamp) - Last update timestamp

**ProjectRelation Key Columns:**
- `userId` (uuid, PK) - User identifier
- `projectId` (varchar, PK) - Project identifier
- `role` (varchar) - User's role in the project
- `createdAt` (timestamp) - Relation creation time
- `updatedAt` (timestamp) - Last update time

#### 5. **Credentials**

| Entity | Table | Description |
|--------|-------|-------------|
| `CredentialsEntity` | `credentials_entity` | Encrypted credential data |
| `SharedCredentials` | `shared_credentials` | Credential sharing permissions |

**CredentialsEntity Key Columns:**
- `id` (varchar, PK) - Unique credential identifier (NanoID)
- `name` (varchar(128)) - Credential name
- `type` (varchar(128), indexed) - Credential type (e.g., 'slackApi', 'googleSheetsOAuth2')
- `data` (text) - Encrypted credential data
- `isManaged` (boolean) - System-managed flag
- `isGlobal` (boolean) - Available to all users flag
- `createdAt` (timestamp) - Creation timestamp
- `updatedAt` (timestamp) - Last update timestamp

#### 6. **Tags & Organization**

| Entity | Table | Description |
|--------|-------|-------------|
| `TagEntity` | `tag_entity` | Tag definitions |
| `WorkflowTagMapping` | `workflow_tag_mapping` | Workflow-tag associations |
| `Folder` | `folder` | Folder hierarchy for workflow organization |
| `FolderTagMapping` | `folder_tag_mapping` | Folder-tag associations |
| `AnnotationTagEntity` | `annotation_tag_entity` | Execution annotation tags (EE) |
| `AnnotationTagMapping` | `annotation_tag_mapping` | Annotation-tag associations (EE) |

**Folder Key Columns:**
- `id` (varchar, PK) - Unique folder identifier (NanoID)
- `name` (varchar) - Folder name
- `parentFolderId` (varchar, nullable) - Parent folder for hierarchy
- `createdAt` (timestamp) - Creation timestamp
- `updatedAt` (timestamp) - Last update timestamp

#### 7. **Webhooks**

| Entity | Table | Description |
|--------|-------|-------------|
| `WebhookEntity` | `webhook_entity` | Active webhook registrations |

**WebhookEntity Key Columns:**
- `webhookPath` (varchar, PK) - Webhook URL path
- `method` (text, PK) - HTTP method (GET, POST, etc.)
- `workflowId` (varchar) - Associated workflow
- `node` (varchar) - Node name that registered the webhook
- `webhookId` (varchar, nullable) - Webhook identifier for dynamic paths
- `pathLength` (integer, nullable) - Path segment count for routing

**Index:** `(webhookId, method, pathLength)` - Efficient webhook routing

#### 8. **Settings & Configuration**

| Entity | Table | Description |
|--------|-------|-------------|
| `Settings` | `settings` | Application-wide settings |
| `Variables` | `variables` | Environment variables (global and project-scoped) |
| `EventDestinations` | `event_destinations` | Event bus destinations |

**Variables Key Columns:**
- `id` (varchar, PK) - Unique variable identifier (NanoID)
- `key` (text) - Variable name
- `type` (text) - Variable type (default: 'string')
- `value` (text) - Variable value
- `project` (relation, nullable) - Project association (null for global variables)

#### 9. **Binary Data**

| Entity | Table | Description |
|--------|-------|-------------|
| `BinaryDataFile` | `binary_data_file` | Binary file metadata and storage |
| `ProcessedData` | `processed_data` | Processed binary data tracking |

#### 10. **Testing (Enterprise Edition)**

| Entity | Table | Description |
|--------|-------|-------------|
| `TestRun` | `test_run` | Test execution runs |
| `TestCaseExecution` | `test_case_execution` | Individual test case results |

#### 11. **Permissions & Authorization**

| Entity | Table | Description |
|--------|-------|-------------|
| `Scope` | `scope` | Permission scopes for RBAC |

## Entity Relationships

### Key Relationships Diagram

```
User ────────┐
             │
             ├──── ProjectRelation ──── Project ────┐
             │           │                          │
             │           └──── Role                 │
             │                                      │
             ├──── AuthIdentity                     │
             ├──── ApiKey                           │
             │                                      │
             └──── (legacy direct relations)        │
                                                    │
Project ─────────────────────────────────────────────┤
             │                                      │
             ├──── SharedWorkflow ──── WorkflowEntity
             │                              │
             ├──── SharedCredentials ──── CredentialsEntity
             │                              │
             └──── Variables                │
                                            │
WorkflowEntity ──────────────────────────────┤
             │                              │
             ├──── ExecutionEntity ──── ExecutionData
             │           │
             │           └──── ExecutionMetadata
             │
             ├──── WorkflowHistory ──── WorkflowPublishHistory
             │
             ├──── WorkflowStatistics
             │
             ├──── WorkflowTagMapping ──── TagEntity
             │
             ├──── WebhookEntity
             │
             ├──── Folder
             │
             └──── TestRun ──── TestCaseExecution
```

### Relationship Details

1. **User-Project Relationship**:
   - Many-to-Many through `ProjectRelation` with roles
   - Each user has a personal project automatically created
   - Users can be members of multiple team projects

2. **Workflow Sharing**:
   - Workflows belong to projects via `SharedWorkflow`
   - Multiple projects can share the same workflow with different roles

3. **Credential Sharing**:
   - Similar to workflows, credentials are shared through `SharedCredentials`
   - Project-level credential access control

4. **Workflow Versioning**:
   - `WorkflowEntity.versionId` tracks current draft version
   - `WorkflowEntity.activeVersionId` references published version in `WorkflowHistory`
   - `WorkflowHistory` stores historical versions
   - `WorkflowPublishHistory` tracks when versions were published

5. **Execution Tracking**:
   - `ExecutionEntity` stores metadata and status
   - `ExecutionData` stores full execution data (separate for performance)
   - One-to-many: Workflow → Executions

## Database Migrations

Migrations are database-specific and located in:

- `packages/@n8n/db/src/migrations/postgresdb/` - PostgreSQL migrations
- `packages/@n8n/db/src/migrations/mysqldb/` - MySQL/MariaDB migrations
- `packages/@n8n/db/src/migrations/sqlite/` - SQLite migrations

### Migration Strategy

- Migrations are executed using TypeORM's migration system
- Each migration is run within a transaction (`transaction: 'each'`)
- Migration tracking table: `<prefix>migrations` (default prefix is empty)
- Migrations are wrapped with error handling via `wrapMigration` helper

### Notable Migration History

1. **Initial Migration** (2020-04-24)
   - Created base tables for workflows, executions, credentials

2. **User Management** (2022-02-07)
   - Introduced multi-user support with roles
   - Added `User`, `Role`, `SharedWorkflow`, `SharedCredentials`

3. **Project-based Architecture** (Recent)
   - Migrated from direct user ownership to project-based sharing
   - Introduced `Project` and `ProjectRelation` entities

4. **Workflow Versioning** (2022-11-28)
   - Added `versionId` and `activeVersionId` columns
   - Created `WorkflowHistory` table

## Data Types by Database

The `abstract-entity.ts` module handles database-specific type mapping:

| Type | SQLite | PostgreSQL | MySQL/MariaDB |
|------|--------|------------|---------------|
| JSON | `simple-json` (text) | `json` | `json` |
| DateTime | `datetime` | `timestamptz` | `datetime` |
| Binary | `blob` | `bytea` | `longblob` |
| Timestamp | `STRFTIME('%Y-%m-%d %H:%M:%f', 'NOW')` | `CURRENT_TIMESTAMP(3)` | `CURRENT_TIMESTAMP(3)` |

## Special Features

### 1. **Soft Deletes**
- `ExecutionEntity` uses `deletedAt` column for soft deletion
- Workflows use `isArchived` flag

### 2. **Automatic Timestamps**
- Base classes provide `createdAt` and `updatedAt` automatically
- `WithTimestamps` mixin adds both columns
- `WithTimestampsAndStringId` adds timestamps + NanoID primary key

### 3. **NanoID Generation**
- Most entities use NanoID instead of auto-increment or UUID
- Generated via `generateNanoId()` utility
- Provides short, URL-safe unique identifiers

### 4. **JSON Column Handling**
- Database-specific JSON storage (simple-json for SQLite, native JSON for others)
- Custom transformers for object retrieval and SQLite compatibility

### 5. **Indexing Strategy**
- Composite indexes on execution tables for common query patterns
- Unique indexes on names and identifiers
- Foreign key indexes for join optimization

## Repository Pattern

Each entity has a corresponding repository in `packages/@n8n/db/src/repositories/`:

- Repositories extend TypeORM's `Repository` class
- Custom query methods for complex operations
- Transaction support via `withTransaction` helper
- Type-safe query builders

## Security Considerations

1. **Credential Encryption**:
   - Credential `data` field is always encrypted before storage
   - Encryption happens at the application layer, not database layer

2. **Password Hashing**:
   - User passwords are hashed using bcrypt
   - Stored in `User.password` column

3. **MFA Support**:
   - TOTP secrets stored in `User.mfaSecret`
   - Recovery codes in `User.mfaRecoveryCodes`

4. **API Key Security**:
   - API keys stored hashed in `ApiKey.apiKey`
   - Scoped permissions via `ApiKey.scopes`

## Performance Optimizations

1. **Execution Data Separation**:
   - `ExecutionEntity` and `ExecutionData` are separate tables
   - Allows querying execution metadata without loading large data payloads

2. **Composite Indexes**:
   - Strategic indexes on frequently queried columns
   - Support for pagination and filtering

3. **Connection Pooling**:
   - Configurable pool sizes for PostgreSQL and MySQL
   - SQLite pooled mode for concurrent access

4. **WAL Mode (SQLite)**:
   - Write-Ahead Logging enabled for better concurrency
   - Configurable via `sqlite.enableWAL`

## Database Health Monitoring

The `DbConnection` class implements automatic health checks:

- Periodic ping every `database.pingIntervalSeconds`
- SELECT 1 query with 5-second timeout
- Connection recovery detection and logging
- Error reporting integration

## Configuration Options

Key configuration options from `@n8n/config`:

```typescript
{
  type: 'sqlite' | 'postgresdb' | 'mysqldb' | 'mariadb',
  tablePrefix: string, // Default: ''
  
  sqlite: {
    database: string,    // Default: 'database.sqlite'
    enableWAL: boolean,  // Default: true
    poolSize: number,    // Default: 0 (no pooling)
  },
  
  postgresdb: {
    host: string,
    port: number,
    database: string,
    user: string,
    password: string,
    schema: string,
    poolSize: number,
    connectionTimeoutMs: number,
    idleTimeoutMs: number,
    ssl: {
      enabled: boolean,
      ca: string,
      cert: string,
      key: string,
      rejectUnauthorized: boolean,
    },
  },
  
  mysqldb: {
    host: string,
    port: number,
    database: string,
    user: string,
    password: string,
    poolSize: number,
  },
  
  logging: {
    enabled: boolean,
    options: string, // 'query', 'error', 'schema', 'warn', 'info', 'log', 'all'
    maxQueryExecutionTime: number, // Log slow queries
  },
}
```

## Module Organization

The `@n8n/db` package structure:

```
packages/@n8n/db/src/
├── connection/           # Database connection management
│   ├── db-connection.ts
│   └── db-connection-options.ts
├── entities/            # TypeORM entity definitions
│   ├── abstract-entity.ts
│   ├── user.ts
│   ├── workflow-entity.ts
│   ├── execution-entity.ts
│   └── ...
├── migrations/          # Database migrations
│   ├── common/         # Shared migration utilities
│   ├── dsl/           # Migration DSL helpers
│   ├── postgresdb/    # PostgreSQL-specific migrations
│   ├── mysqldb/       # MySQL-specific migrations
│   └── sqlite/        # SQLite-specific migrations
├── repositories/        # Data access layer
│   ├── user.repository.ts
│   ├── workflow.repository.ts
│   └── ...
├── services/           # Business logic services
├── subscribers/        # TypeORM event subscribers
└── utils/             # Utility functions
    ├── generators.ts   # NanoID generation
    ├── transformers.ts # Column transformers
    └── validators/     # Validation decorators
```

## Best Practices

1. **Always use repositories** instead of direct entity access
2. **Use transactions** for multi-table operations
3. **Index frequently queried columns** but avoid over-indexing
4. **Separate read-heavy and write-heavy operations** when possible
5. **Use database-agnostic TypeORM features** to maintain compatibility
6. **Test migrations** on all supported database types
7. **Keep execution data separate** from execution metadata
8. **Use soft deletes** for audit trail requirements
9. **Implement proper error handling** for database operations
10. **Monitor query performance** using logging configuration

## Extending the Schema

To add new entities:

1. Create entity file in `packages/@n8n/db/src/entities/`
2. Export from `packages/@n8n/db/src/entities/index.ts`
3. Create repository in `packages/@n8n/db/src/repositories/`
4. Create migrations for all database types
5. Update this documentation

For migrations:

1. Create migration files for each database type
2. Use the migration DSL from `packages/@n8n/db/src/migrations/dsl/`
3. Test on all supported databases
4. Add to migration index files

## Troubleshooting

Common database issues:

1. **Connection timeout**: Increase `postgresdb.connectionTimeoutMs`
2. **Too many connections**: Adjust pool size settings
3. **Slow queries**: Enable logging and check indexes
4. **Migration failures**: Check migration order and dependencies
5. **SQLite locked**: Enable WAL mode or increase timeout
6. **Large binary data**: Increase MySQL's `max_allowed_packet`

## Additional Resources

- TypeORM Documentation: https://typeorm.io/
- n8n Documentation: https://docs.n8n.io/
- Database Configuration: See `packages/@n8n/config/src/configs/database.config.ts`
- Migration Helpers: See `packages/@n8n/db/src/migrations/migration-helpers.ts`
