# n8n Database Quick Reference Guide

A quick reference for developers working with the n8n database.

## Database Package Location

```
packages/@n8n/db/
```

## Quick Start

### Import Database Classes

```typescript
import {
  User,
  WorkflowEntity,
  ExecutionEntity,
  CredentialsEntity,
  Project,
  DataSource,
} from '@n8n/db';
```

### Get Database Connection

```typescript
import { Container } from '@n8n/di';
import { DataSource } from '@n8n/typeorm';

const dataSource = Container.get(DataSource);
```

### Use Repositories

```typescript
import { Container } from '@n8n/di';
import { UserRepository, WorkflowRepository } from '@n8n/db';

const userRepository = Container.get(UserRepository);
const workflowRepository = Container.get(WorkflowRepository);
```

## Common Entity Reference

### User Entity

```typescript
{
  id: string;              // UUID
  email: string;           // Lowercase, unique
  firstName: string;
  lastName: string;
  password: string | null; // Bcrypt hashed
  roleSlug: string;        // FK to Role
  disabled: boolean;
  mfaEnabled: boolean;
  createdAt: Date;
  updatedAt: Date;
}
```

### WorkflowEntity

```typescript
{
  id: string;                // NanoID
  name: string;              // Unique
  description: string | null;
  active: boolean;           // Deprecated - use activeVersionId
  isArchived: boolean;       // Soft delete
  nodes: INode[];            // JSON
  connections: IConnections; // JSON
  settings?: IWorkflowSettings;
  versionId: string;
  activeVersionId: string | null;
  versionCounter: number;
  triggerCount: number;
  parentFolderId: string | null;
  createdAt: Date;
  updatedAt: Date;
}
```

### ExecutionEntity

```typescript
{
  id: string;              // Auto-increment as string
  workflowId: string;
  status: ExecutionStatus; // 'running' | 'success' | 'error' | 'waiting' | ...
  mode: WorkflowExecuteMode;
  finished: boolean;       // Deprecated - use status
  createdAt: Date;
  startedAt: Date | null;
  stoppedAt: Date | null;
  deletedAt: Date | null;  // Soft delete
  waitTill: Date | null;
  retryOf: string | null;
  retrySuccessId: string | null;
}
```

### CredentialsEntity

```typescript
{
  id: string;        // NanoID
  name: string;
  type: string;      // Credential type (indexed)
  data: string;      // Encrypted JSON
  isManaged: boolean;
  isGlobal: boolean;
  createdAt: Date;
  updatedAt: Date;
}
```

### Project

```typescript
{
  id: string;                    // NanoID
  name: string;
  type: 'personal' | 'team';
  icon: { type: 'emoji' | 'icon'; value: string } | null;
  description: string | null;
  creatorId: string | null;
  createdAt: Date;
  updatedAt: Date;
}
```

## Common Query Patterns

### Find User by Email

```typescript
const user = await userRepository.findOne({
  where: { email: email.toLowerCase() },
  relations: ['role'],
});
```

### Find Workflows by Project

```typescript
const workflows = await workflowRepository.find({
  where: {
    shared: {
      projectId,
    },
  },
  relations: ['shared', 'shared.project'],
});
```

### Find Active Workflows

```typescript
const activeWorkflows = await workflowRepository.find({
  where: {
    activeVersionId: Not(IsNull()),
    isArchived: false,
  },
});
```

### Find Recent Executions

```typescript
const executions = await executionRepository.find({
  where: { workflowId },
  order: { createdAt: 'DESC' },
  take: 10,
});
```

### Find Executions with Data

```typescript
const execution = await executionRepository.findOne({
  where: { id: executionId },
  relations: ['executionData', 'metadata'],
});
```

### Find User's Projects

```typescript
const projects = await projectRepository.find({
  where: {
    projectRelations: {
      userId,
    },
  },
  relations: ['projectRelations', 'projectRelations.role'],
});
```

## Transaction Usage

```typescript
import { withTransaction } from '@n8n/db';

await withTransaction(async (entityManager) => {
  // Multiple operations in single transaction
  const user = await entityManager.save(User, userData);
  const project = await entityManager.save(Project, projectData);
  await entityManager.save(ProjectRelation, {
    userId: user.id,
    projectId: project.id,
    role: 'project:admin',
  });
});
```

## Database Type Helpers

### JSON Columns

```typescript
import { JsonColumn } from '@n8n/db';

@JsonColumn()
data: object;

@JsonColumn({ nullable: true })
metadata?: object;
```

### DateTime Columns

```typescript
import { DateTimeColumn } from '@n8n/db';

@DateTimeColumn()
timestamp: Date;

@DateTimeColumn({ nullable: true })
completedAt?: Date;
```

### String ID Generation

```typescript
import { generateNanoId } from '@n8n/db';

const id = generateNanoId(); // Generates short, URL-safe ID
```

## Migration Commands

### Create New Migration

```bash
# For PostgreSQL
pnpm --filter @n8n/db migration:create -n MigrationName

# For MySQL
pnpm --filter @n8n/db migration:create -n MigrationName

# For SQLite
pnpm --filter @n8n/db migration:create -n MigrationName
```

### Run Migrations

```bash
# Migrations run automatically on application start
# Or manually via:
pnpm --filter @n8n/db migration:run
```

### Revert Migration

```bash
pnpm --filter @n8n/db migration:revert
```

## Environment Variables

### Database Configuration

```bash
# Database Type
DB_TYPE=postgresdb  # sqlite | postgresdb | mysqldb | mariadb

# PostgreSQL
DB_POSTGRESDB_HOST=localhost
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=n8n
DB_POSTGRESDB_SCHEMA=public

# MySQL
DB_MYSQLDB_HOST=localhost
DB_MYSQLDB_PORT=3306
DB_MYSQLDB_DATABASE=n8n
DB_MYSQLDB_USER=n8n
DB_MYSQLDB_PASSWORD=n8n

# SQLite
DB_SQLITE_DATABASE=database.sqlite

# Table Prefix
DB_TABLE_PREFIX=  # Optional prefix for all tables

# Logging
DB_LOGGING_ENABLED=false
DB_LOGGING_OPTIONS=query,error
DB_LOGGING_MAX_QUERY_EXECUTION_TIME=1000  # ms
```

## Entity Relationships Quick Reference

```typescript
// One-to-Many
@OneToMany(() => SharedWorkflow, (sw) => sw.workflow)
sharedWorkflows: SharedWorkflow[];

// Many-to-One
@ManyToOne(() => WorkflowEntity, (w) => w.sharedWorkflows)
workflow: WorkflowEntity;

// Many-to-Many
@ManyToMany(() => TagEntity)
@JoinTable({ name: 'workflow_tag' })
tags: TagEntity[];

// One-to-One
@OneToOne(() => ExecutionData, (ed) => ed.execution)
executionData: ExecutionData;
```

## Common Indexes

```typescript
// Simple Index
@Index()
@Column()
field: string;

// Unique Index
@Index({ unique: true })
@Column()
email: string;

// Composite Index
@Index(['field1', 'field2'])
@Entity()
class MyEntity { ... }

// Named Index
@Index('idx_custom_name', ['field'])
@Entity()
class MyEntity { ... }
```

## Validation Decorators

```typescript
import { Length, IsString, IsEmail } from 'class-validator';
import { NoXss, NoUrl } from '@n8n/db';

@Length(3, 128)
@IsString()
@NoXss()
@Column()
name: string;

@IsEmail()
@Column()
email: string;

@NoUrl()
@Column()
description: string;
```

## Custom Transformers

```typescript
import { lowerCaser, objectRetriever, idStringifier } from '@n8n/db';

// Lowercase transformer
@Column({ transformer: lowerCaser })
email: string;

// Object retriever (for JSON columns)
@JsonColumn({ transformer: objectRetriever })
data: object;

// ID stringifier (converts number to string)
@Column({ transformer: idStringifier })
id: string;
```

## Base Classes

```typescript
import {
  WithStringId,
  WithTimestamps,
  WithTimestampsAndStringId,
} from '@n8n/db';

// Entity with NanoID
class MyEntity extends WithStringId {
  // id: string is already defined
}

// Entity with timestamps
class MyEntity extends WithTimestamps {
  // createdAt: Date and updatedAt: Date are defined
}

// Entity with both
class MyEntity extends WithTimestampsAndStringId {
  // id: string, createdAt: Date, updatedAt: Date are defined
}
```

## Testing Utilities

```typescript
import { mockInstance } from '@n8n/db';

// Mock entity manager for tests
const mockEntityManager = mockInstance(EntityManager);
```

## Performance Tips

1. **Use indexes** on frequently queried columns
2. **Select specific columns** instead of entire entities when possible
3. **Use pagination** for large result sets
4. **Avoid N+1 queries** by using proper relations
5. **Use composite indexes** for multi-column queries
6. **Separate read and write operations** when appropriate
7. **Cache frequently accessed, rarely changing data**
8. **Use execution data separation** (ExecutionEntity vs ExecutionData)

## Common Pitfalls

1. ❌ **Don't use `any` type** - Use proper TypeORM types
2. ❌ **Don't forget transactions** for multi-table operations
3. ❌ **Don't load execution data** when only metadata is needed
4. ❌ **Don't use direct SQL** unless absolutely necessary
5. ❌ **Don't forget to add indexes** to migration files
6. ❌ **Don't hardcode database types** - Use type-agnostic code
7. ❌ **Don't forget to test** on all supported databases
8. ❌ **Don't expose sensitive data** in toJSON() methods

## Useful SQL Queries (for debugging)

### PostgreSQL

```sql
-- List all tables
SELECT tablename FROM pg_tables WHERE schemaname = 'public';

-- Count executions by status
SELECT status, COUNT(*) FROM execution_entity GROUP BY status;

-- Find workflows with most executions
SELECT w.name, COUNT(e.id) as execution_count
FROM workflow_entity w
LEFT JOIN execution_entity e ON e."workflowId" = w.id
GROUP BY w.id, w.name
ORDER BY execution_count DESC
LIMIT 10;

-- Find active workflows
SELECT id, name FROM workflow_entity WHERE "activeVersionId" IS NOT NULL;
```

### MySQL

```sql
-- List all tables
SHOW TABLES;

-- Count executions by status
SELECT status, COUNT(*) FROM execution_entity GROUP BY status;

-- Find workflows with most executions
SELECT w.name, COUNT(e.id) as execution_count
FROM workflow_entity w
LEFT JOIN execution_entity e ON e.workflowId = w.id
GROUP BY w.id, w.name
ORDER BY execution_count DESC
LIMIT 10;
```

### SQLite

```sql
-- List all tables
SELECT name FROM sqlite_master WHERE type='table';

-- Count executions by status
SELECT status, COUNT(*) FROM execution_entity GROUP BY status;

-- Find workflows with most executions
SELECT w.name, COUNT(e.id) as execution_count
FROM workflow_entity w
LEFT JOIN execution_entity e ON e.workflowId = w.id
GROUP BY w.id, w.name
ORDER BY execution_count DESC
LIMIT 10;
```

## Database Maintenance

### Cleanup Old Executions

```typescript
// Soft delete executions older than 30 days
await executionRepository.update(
  {
    createdAt: LessThan(new Date(Date.now() - 30 * 24 * 60 * 60 * 1000)),
  },
  {
    deletedAt: new Date(),
  }
);
```

### Archive Workflows

```typescript
// Archive workflow
await workflowRepository.update(
  { id: workflowId },
  { isArchived: true }
);
```

### Check Database Connection

```typescript
import { Container } from '@n8n/di';
import { DbConnection } from '@n8n/db';

const dbConnection = Container.get(DbConnection);
if (dbConnection.connectionState.connected) {
  console.log('Database connected');
}
```

## TypeORM Query Builder Examples

### Complex Filtering

```typescript
const executions = await executionRepository
  .createQueryBuilder('execution')
  .where('execution.workflowId = :workflowId', { workflowId })
  .andWhere('execution.status = :status', { status: 'success' })
  .orderBy('execution.createdAt', 'DESC')
  .take(10)
  .getMany();
```

### Joins

```typescript
const workflows = await workflowRepository
  .createQueryBuilder('workflow')
  .leftJoinAndSelect('workflow.shared', 'shared')
  .leftJoinAndSelect('shared.project', 'project')
  .where('project.id = :projectId', { projectId })
  .getMany();
```

### Count

```typescript
const count = await executionRepository
  .createQueryBuilder('execution')
  .where('execution.workflowId = :workflowId', { workflowId })
  .getCount();
```

## Repository Method Reference

### Common Methods

- `find(options)` - Find multiple entities
- `findOne(options)` - Find single entity
- `findOneBy(criteria)` - Find by simple criteria
- `findOneByOrFail(criteria)` - Find or throw error
- `save(entity)` - Insert or update
- `update(criteria, partial)` - Update matching entities
- `delete(criteria)` - Delete matching entities
- `count(options)` - Count matching entities
- `createQueryBuilder()` - Create query builder

## Error Handling

```typescript
import { QueryFailedError } from '@n8n/typeorm';

try {
  await repository.save(entity);
} catch (error) {
  if (error instanceof QueryFailedError) {
    // Handle database errors
    if (error.message.includes('duplicate key')) {
      // Handle unique constraint violation
    }
  }
  throw error;
}
```

## Resources

- [TypeORM Documentation](https://typeorm.io/)
- [n8n Database Schema](./DATABASE_SCHEMA.md)
- [ER Diagram](./DATABASE_ER_DIAGRAM.md)
- Package source: `packages/@n8n/db/`

## Getting Help

1. Check TypeORM documentation for query issues
2. Review existing repository implementations
3. Check migration history for schema changes
4. Use database query logging for debugging
5. Test on all supported database types

## Version Compatibility

- TypeORM: Check `packages/@n8n/db/package.json`
- PostgreSQL: 11+
- MySQL: 8.0+
- MariaDB: 10.3+
- SQLite: 3.x

## Security Notes

- Credentials are encrypted before storage
- Passwords are hashed using bcrypt
- API keys are stored hashed
- Use parameterized queries to prevent SQL injection
- Validate all user inputs
- Don't log sensitive data
- Use HTTPS for database connections in production
