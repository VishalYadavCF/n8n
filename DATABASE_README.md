# n8n Database Schema Investigation - Documentation Index

This directory contains comprehensive documentation of the n8n database schema.

## 📚 Documentation Files

### 1. [DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md) - Complete Schema Documentation
**Start here for a comprehensive overview**

This is the main documentation file covering:
- ✅ Overview of n8n's database architecture
- ✅ Supported database systems (SQLite, PostgreSQL, MySQL, MariaDB)
- ✅ Database configuration and connection management
- ✅ Complete entity reference with all tables and columns
- ✅ Relationship diagrams and explanations
- ✅ Migration structure and history
- ✅ Data types across different databases
- ✅ Performance optimizations
- ✅ Security considerations
- ✅ Best practices and troubleshooting

**Size**: ~22KB | **Estimated Reading Time**: 30-45 minutes

---

### 2. [DATABASE_ER_DIAGRAM.md](./DATABASE_ER_DIAGRAM.md) - Visual Schema Diagrams
**For visual learners and system architects**

This file contains:
- ✅ Interactive Mermaid ER diagrams
- ✅ Core schema overview diagram
- ✅ Detailed diagrams by functional area:
  - User & Authentication
  - Project & Collaboration
  - Workflow Management
  - Execution Tracking
  - Credentials
  - Tags & Organization
  - Settings & Configuration
  - Binary Data
  - Testing (Enterprise Edition)
- ✅ Index documentation
- ✅ Junction tables reference
- ✅ Cascade behavior documentation
- ✅ Column naming conventions

**Size**: ~16KB | **Estimated Reading Time**: 20-30 minutes

---

### 3. [DATABASE_QUICK_REFERENCE.md](./DATABASE_QUICK_REFERENCE.md) - Developer Quick Reference
**For developers working with the database**

This is a practical guide containing:
- ✅ Quick start code examples
- ✅ Common entity reference
- ✅ Query pattern examples
- ✅ Transaction usage
- ✅ Migration commands
- ✅ Environment variables
- ✅ TypeORM helpers and decorators
- ✅ Performance tips
- ✅ Common pitfalls
- ✅ Useful SQL queries for debugging
- ✅ Database maintenance examples

**Size**: ~14KB | **Estimated Reading Time**: 15-20 minutes

---

## 🎯 Use Cases - Which Document to Read?

### I want to understand the overall database architecture
→ Read [DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md) from start to finish

### I need to visualize how tables relate to each other
→ Check [DATABASE_ER_DIAGRAM.md](./DATABASE_ER_DIAGRAM.md) for Mermaid diagrams

### I'm writing code that queries the database
→ Use [DATABASE_QUICK_REFERENCE.md](./DATABASE_QUICK_REFERENCE.md) for code examples

### I'm investigating a specific entity (like WorkflowEntity)
→ Search in [DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md) for detailed column info

### I need to write a migration
→ Check [DATABASE_QUICK_REFERENCE.md](./DATABASE_QUICK_REFERENCE.md) for commands, then [DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md) for migration structure

### I'm debugging database performance issues
→ See performance sections in [DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md) and [DATABASE_QUICK_REFERENCE.md](./DATABASE_QUICK_REFERENCE.md)

---

## 🗂️ Database Package Structure

```
packages/@n8n/db/
├── src/
│   ├── connection/           # Database connection management
│   │   ├── db-connection.ts
│   │   └── db-connection-options.ts
│   ├── entities/            # TypeORM entity definitions (40+ entities)
│   │   ├── user.ts
│   │   ├── workflow-entity.ts
│   │   ├── execution-entity.ts
│   │   ├── credentials-entity.ts
│   │   ├── project.ts
│   │   └── ... (see entity index)
│   ├── migrations/          # Database migrations
│   │   ├── postgresdb/      # PostgreSQL-specific migrations
│   │   ├── mysqldb/         # MySQL/MariaDB-specific migrations
│   │   └── sqlite/          # SQLite-specific migrations
│   ├── repositories/        # Data access layer (35+ repositories)
│   ├── services/            # Business logic services
│   ├── subscribers/         # TypeORM event subscribers
│   └── utils/              # Utility functions
└── package.json
```

---

## 🔑 Key Database Entities

### Core Workflow Tables
- **WorkflowEntity** - Workflow definitions (nodes, connections, settings)
- **ExecutionEntity** - Execution metadata and status
- **ExecutionData** - Execution input/output data (separate for performance)
- **CredentialsEntity** - Encrypted credentials

### User & Access Control
- **User** - User accounts
- **Role** - RBAC roles (global, project, workflow, credential)
- **Project** - Team/personal workspaces
- **ProjectRelation** - User-project membership
- **SharedWorkflow** - Workflow sharing permissions
- **SharedCredentials** - Credential sharing permissions

### Organization
- **Folder** - Workflow folder hierarchy
- **TagEntity** - Workflow and folder tags
- **Variables** - Environment variables (global and project-scoped)

### Versioning
- **WorkflowHistory** - Workflow version history
- **WorkflowPublishHistory** - Publishing events

### Support
- **Settings** - Application settings
- **WebhookEntity** - Active webhook registrations
- **ApiKey** - API authentication

---

## 🗄️ Supported Databases

| Database | Status | Use Case | Configuration Complexity |
|----------|--------|----------|-------------------------|
| **SQLite** | ✅ Default | Development, small deployments | Low |
| **PostgreSQL** | ✅ Recommended | Production, large scale | Medium |
| **MySQL** | ✅ Supported | Production alternative | Medium |
| **MariaDB** | ✅ Supported | MySQL-compatible production | Medium |

---

## 📊 Database Statistics

- **Total Entities**: 40+ entities
- **Total Repositories**: 35+ repositories
- **Junction Tables**: 6+ (for many-to-many relationships)
- **Migration Files**: 60+ per database type
- **Indexed Columns**: 15+ strategic indexes
- **Primary Key Types**: UUID, NanoID, Auto-increment

---

## 🔐 Security Features

- ✅ Encrypted credential storage
- ✅ Bcrypt password hashing
- ✅ API key authentication
- ✅ Multi-factor authentication (MFA) support
- ✅ Role-Based Access Control (RBAC)
- ✅ Project-level permissions
- ✅ Soft deletes for audit trails
- ✅ OAuth/LDAP/SAML integration

---

## 🚀 Performance Optimizations

- ✅ Composite indexes on frequently queried columns
- ✅ Separate execution metadata from data
- ✅ Connection pooling for PostgreSQL/MySQL
- ✅ WAL mode for SQLite concurrency
- ✅ Strategic use of JSON columns
- ✅ Efficient webhook routing
- ✅ Pagination support

---

## 🛠️ Developer Tools

- **TypeORM**: ORM layer with full TypeScript support
- **Migrations**: Automatic schema versioning
- **Query Builder**: Type-safe query construction
- **Repositories**: Abstraction layer for data access
- **Validators**: Input validation decorators
- **Transformers**: Column data transformers

---

## 📈 Schema Evolution

The n8n database has evolved significantly:

1. **Initial Schema** (2020): Basic workflow and execution tables
2. **User Management** (2022): Multi-user support with roles
3. **Project Architecture** (2023-2024): Team collaboration features
4. **Workflow Versioning** (2022): Version control for workflows
5. **Folder System** (Recent): Hierarchical organization
6. **Testing Framework** (EE): Automated testing support

---

## 🔗 Related Resources

### Internal Documentation
- Source Code: `packages/@n8n/db/`
- Configuration: `packages/@n8n/config/src/configs/database.config.ts`
- Backend Guide: `packages/cli/scripts/backend-module/backend-module-guide.md`
- Repository Agents: `AGENTS.md`

### External Resources
- [TypeORM Documentation](https://typeorm.io/)
- [n8n Official Documentation](https://docs.n8n.io/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [MySQL Documentation](https://dev.mysql.com/doc/)

---

## 🤝 Contributing

When working with the database:

1. **Always test migrations** on all supported database types
2. **Follow naming conventions** for consistency
3. **Add indexes** for frequently queried columns
4. **Update documentation** when schema changes
5. **Use TypeORM features** for database-agnostic code
6. **Write tests** for new entities and repositories

---

## ❓ Getting Help

1. **Read the documentation** - Start with [DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md)
2. **Check code examples** - See [DATABASE_QUICK_REFERENCE.md](./DATABASE_QUICK_REFERENCE.md)
3. **Review entity relationships** - Use [DATABASE_ER_DIAGRAM.md](./DATABASE_ER_DIAGRAM.md)
4. **Examine existing code** - Look at similar repositories and entities
5. **Enable query logging** - Debug with `DB_LOGGING_ENABLED=true`

---

## 📝 Documentation Version

- **Created**: December 2024
- **Based on**: n8n codebase snapshot
- **Last Updated**: December 8, 2024
- **Coverage**: All entities in `packages/@n8n/db/src/entities/`

---

## 🎓 Learning Path

**For Beginners:**
1. Read "Overview" section in DATABASE_SCHEMA.md
2. Look at ER diagrams in DATABASE_ER_DIAGRAM.md
3. Try simple queries from DATABASE_QUICK_REFERENCE.md
4. Explore entity files in the codebase

**For Intermediate Developers:**
1. Study entity relationships in DATABASE_SCHEMA.md
2. Review migration patterns
3. Practice with complex queries
4. Understand transaction usage

**For Advanced Developers:**
1. Master migration creation
2. Optimize query performance
3. Implement custom repositories
4. Extend the schema with new entities

---

**Happy coding! 🎉**

For questions or improvements to this documentation, please open an issue or PR in the repository.
