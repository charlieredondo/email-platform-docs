# Database Schema - Email Platform

## Overview

The Email Platform uses Microsoft SQL Server as the primary database with a multitenant architecture that ensures complete data isolation between clients through row-level security and tenant-specific filtering.

## Schema Design Principles

1. **Tenant Isolation**: Every table includes `tenant_id` for complete data separation
2. **Audit Trail**: All tables include created/updated timestamps and user tracking
3. **Scalability**: Optimized indexes for high-performance queries
4. **Data Integrity**: Foreign key constraints and check constraints
5. **Compliance**: Retention policies and audit logging

## Core Tables

### Tenants Table
Manages client organizations and their configurations.

```sql
CREATE TABLE Tenants (
    tenant_id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    name NVARCHAR(255) NOT NULL,
    subdomain NVARCHAR(100) UNIQUE,
    settings NVARCHAR(MAX), -- JSON config
    retention_policy_days INT DEFAULT 30,
    status NVARCHAR(20) DEFAULT 'active',
    created_at DATETIME2 DEFAULT GETUTCDATE(),
    updated_at DATETIME2 DEFAULT GETUTCDATE(),
    
    CONSTRAINT CK_Tenants_Status CHECK (status IN ('active', 'suspended', 'deleted')),
    CONSTRAINT CK_Tenants_Retention CHECK (retention_policy_days IN (30, 60, 90))
);

-- Indexes
CREATE INDEX IX_Tenants_Subdomain ON Tenants(subdomain);
CREATE INDEX IX_Tenants_Status ON Tenants(status);
```

### TenantUsers Table
Maps users to tenants with role-based access control.

```sql
CREATE TABLE TenantUsers (
    user_id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    tenant_id UNIQUEIDENTIFIER NOT NULL REFERENCES Tenants(tenant_id),
    cognito_user_id NVARCHAR(255) UNIQUE NOT NULL,
    email NVARCHAR(255) NOT NULL,
    role NVARCHAR(50) NOT NULL,
    is_active BIT DEFAULT 1,
    last_login DATETIME2,
    created_at DATETIME2 DEFAULT GETUTCDATE(),
    updated_at DATETIME2 DEFAULT GETUTCDATE(),
    
    CONSTRAINT CK_TenantUsers_Role CHECK (role IN ('admin', 'editor', 'viewer')),
    CONSTRAINT UQ_TenantUsers_Email_Tenant UNIQUE (email, tenant_id)
);

-- Indexes
CREATE INDEX IX_TenantUsers_TenantId ON TenantUsers(tenant_id);
CREATE INDEX IX_TenantUsers_CognitoUserId ON TenantUsers(cognito_user_id);
CREATE INDEX IX_TenantUsers_Email ON TenantUsers(email);
```

### EmailTemplates Table
Stores XSLT templates with versioning support.

```sql
CREATE TABLE EmailTemplates (
    template_id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    tenant_id UNIQUEIDENTIFIER NOT NULL REFERENCES Tenants(tenant_id),
    type NVARCHAR(100) NOT NULL,
    version INT DEFAULT 1,
    subject_template NVARCHAR(500) NOT NULL,
    from_email NVARCHAR(255) NOT NULL,
    from_name NVARCHAR(255),
    xslt_content NVARCHAR(MAX) NOT NULL,
    html_preview NVARCHAR(MAX),
    is_active BIT DEFAULT 1,
    created_by UNIQUEIDENTIFIER NOT NULL REFERENCES TenantUsers(user_id),
    created_at DATETIME2 DEFAULT GETUTCDATE(),
    updated_at DATETIME2 DEFAULT GETUTCDATE(),
    
    CONSTRAINT CK_EmailTemplates_Type CHECK (type IN (
        'forgot_password', 'welcome', 'order_confirmation', 'shipment_confirmation',
        'password_reset', 'account_activation', 'subscription_renewal'
    )),
    CONSTRAINT UQ_EmailTemplates_Type_Tenant_Version UNIQUE (tenant_id, type, version)
);

-- Indexes
CREATE INDEX IX_EmailTemplates_TenantId_Type ON EmailTemplates(tenant_id, type);
CREATE INDEX IX_EmailTemplates_Active ON EmailTemplates(is_active) WHERE is_active = 1;
CREATE INDEX IX_EmailTemplates_CreatedBy ON EmailTemplates(created_by);
```

### EmailConfigurations Table
Manages email-specific configurations per tenant.

```sql
CREATE TABLE EmailConfigurations (
    config_id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    tenant_id UNIQUEIDENTIFIER NOT NULL REFERENCES Tenants(tenant_id),
    email_type NVARCHAR(100) NOT NULL,
    additional_recipients NVARCHAR(MAX), -- JSON array
    cc_recipients NVARCHAR(MAX), -- JSON array
    bcc_recipients NVARCHAR(MAX), -- JSON array
    reply_to_email NVARCHAR(255),
    return_path NVARCHAR(255),
    enabled BIT DEFAULT 1,
    priority_level NVARCHAR(20) DEFAULT 'normal',
    created_at DATETIME2 DEFAULT GETUTCDATE(),
    updated_at DATETIME2 DEFAULT GETUTCDATE(),
    
    CONSTRAINT CK_EmailConfigurations_Priority CHECK (priority_level IN ('low', 'normal', 'high', 'critical')),
    CONSTRAINT UQ_EmailConfigurations_Type_Tenant UNIQUE (tenant_id, email_type)
);

-- Indexes
CREATE INDEX IX_EmailConfigurations_TenantId_Type ON EmailConfigurations(tenant_id, email_type);
CREATE INDEX IX_EmailConfigurations_Enabled ON EmailConfigurations(enabled);
```

### EmailLogs Table
Comprehensive logging of all email sends.

```sql
CREATE TABLE EmailLogs (
    log_id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    tenant_id UNIQUEIDENTIFIER NOT NULL REFERENCES Tenants(tenant_id),
    template_id UNIQUEIDENTIFIER REFERENCES EmailTemplates(template_id),
    email_type NVARCHAR(100) NOT NULL,
    recipient_email NVARCHAR(255) NOT NULL,
    recipient_name NVARCHAR(255),
    subject NVARCHAR(500) NOT NULL,
    message_id NVARCHAR(255), -- SES Message ID
    ses_configuration_set NVARCHAR(255),
    sent_at DATETIME2 DEFAULT GETUTCDATE(),
    status NVARCHAR(50) DEFAULT 'queued',
    retry_count INT DEFAULT 0,
    error_message NVARCHAR(MAX),
    metadata NVARCHAR(MAX), -- JSON con datos del contexto
    
    CONSTRAINT CK_EmailLogs_Status CHECK (status IN (
        'queued', 'sending', 'sent', 'delivered', 'bounced', 'complained', 'failed'
    ))
);

-- Indexes
CREATE CLUSTERED INDEX IX_EmailLogs_TenantId_SentAt ON EmailLogs(tenant_id, sent_at);
CREATE INDEX IX_EmailLogs_MessageId ON EmailLogs(message_id);
CREATE INDEX IX_EmailLogs_RecipientEmail ON EmailLogs(recipient_email);
CREATE INDEX IX_EmailLogs_Status ON EmailLogs(status);
CREATE INDEX IX_EmailLogs_EmailType ON EmailLogs(email_type);

-- Partitioning for large datasets
-- CREATE PARTITION FUNCTION pf_EmailLogs_SentAt (DATETIME2) AS RANGE RIGHT FOR VALUES (...)
```

### EmailEvents Table
Tracks all email interaction events from SES.

```sql
CREATE TABLE EmailEvents (
    event_id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    log_id UNIQUEIDENTIFIER NOT NULL REFERENCES EmailLogs(log_id),
    event_type NVARCHAR(50) NOT NULL,
    timestamp DATETIME2 NOT NULL,
    bounce_type NVARCHAR(50), -- permanent, transient
    bounce_subtype NVARCHAR(50), -- suppressed, mailbox-full, etc.
    complaint_type NVARCHAR(50), -- abuse, auth-failure, fraud, etc.
    click_url NVARCHAR(2000), -- for click events
    metadata NVARCHAR(MAX), -- JSON con detalles del evento
    ip_address NVARCHAR(45),
    user_agent NVARCHAR(500),
    created_at DATETIME2 DEFAULT GETUTCDATE(),
    
    CONSTRAINT CK_EmailEvents_EventType CHECK (event_type IN (
        'send', 'reject', 'bounce', 'complaint', 'delivery', 'open', 'click', 'renderingFailure'
    ))
);

-- Indexes
CREATE INDEX IX_EmailEvents_LogId_EventType ON EmailEvents(log_id, event_type);
CREATE INDEX IX_EmailEvents_Timestamp ON EmailEvents(timestamp);
CREATE INDEX IX_EmailEvents_EventType ON EmailEvents(event_type);
```

### TemplateVariables Table
Documentation and validation for template variables.

```sql
CREATE TABLE TemplateVariables (
    variable_id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    template_id UNIQUEIDENTIFIER NOT NULL REFERENCES EmailTemplates(template_id),
    variable_name NVARCHAR(100) NOT NULL,
    variable_type NVARCHAR(50) NOT NULL,
    description NVARCHAR(500),
    is_required BIT DEFAULT 0,
    default_value NVARCHAR(255),
    validation_regex NVARCHAR(500),
    created_at DATETIME2 DEFAULT GETUTCDATE(),
    
    CONSTRAINT CK_TemplateVariables_Type CHECK (variable_type IN (
        'string', 'number', 'date', 'datetime', 'boolean', 'object', 'array'
    )),
    CONSTRAINT UQ_TemplateVariables_Name_Template UNIQUE (template_id, variable_name)
);

-- Indexes
CREATE INDEX IX_TemplateVariables_TemplateId ON TemplateVariables(template_id);
```

## Security and Compliance Tables

### AuditLogs Table
Comprehensive audit trail for compliance.

```sql
CREATE TABLE AuditLogs (
    audit_id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    tenant_id UNIQUEIDENTIFIER NOT NULL REFERENCES Tenants(tenant_id),
    user_id UNIQUEIDENTIFIER REFERENCES TenantUsers(user_id),
    action NVARCHAR(100) NOT NULL,
    entity_type NVARCHAR(100) NOT NULL,
    entity_id UNIQUEIDENTIFIER,
    old_values NVARCHAR(MAX), -- JSON
    new_values NVARCHAR(MAX), -- JSON
    ip_address NVARCHAR(45),
    user_agent NVARCHAR(500),
    timestamp DATETIME2 DEFAULT GETUTCDATE(),
    
    CONSTRAINT CK_AuditLogs_Action CHECK (action IN (
        'CREATE', 'UPDATE', 'DELETE', 'LOGIN', 'LOGOUT', 'SEND_EMAIL', 'VIEW'
    ))
);

-- Indexes
CREATE CLUSTERED INDEX IX_AuditLogs_TenantId_Timestamp ON AuditLogs(tenant_id, timestamp);
CREATE INDEX IX_AuditLogs_UserId ON AuditLogs(user_id);
CREATE INDEX IX_AuditLogs_Action ON AuditLogs(action);
```

### ApiKeys Table
Manages API keys for external integrations.

```sql
CREATE TABLE ApiKeys (
    api_key_id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    tenant_id UNIQUEIDENTIFIER NOT NULL REFERENCES Tenants(tenant_id),
    name NVARCHAR(255) NOT NULL,
    key_hash NVARCHAR(255) NOT NULL, -- SHA256 hash
    permissions NVARCHAR(MAX), -- JSON array of permissions
    rate_limit_per_minute INT DEFAULT 1000,
    is_active BIT DEFAULT 1,
    expires_at DATETIME2,
    last_used_at DATETIME2,
    created_by UNIQUEIDENTIFIER NOT NULL REFERENCES TenantUsers(user_id),
    created_at DATETIME2 DEFAULT GETUTCDATE(),
    
    CONSTRAINT UQ_ApiKeys_Hash UNIQUE (key_hash)
);

-- Indexes
CREATE INDEX IX_ApiKeys_TenantId ON ApiKeys(tenant_id);
CREATE INDEX IX_ApiKeys_Hash ON ApiKeys(key_hash);
CREATE INDEX IX_ApiKeys_Active ON ApiKeys(is_active) WHERE is_active = 1;
```

## Views and Stored Procedures

### Email Analytics View
```sql
CREATE VIEW vw_EmailAnalytics AS
SELECT 
    el.tenant_id,
    el.email_type,
    CAST(el.sent_at AS DATE) as sent_date,
    COUNT(*) as total_sent,
    SUM(CASE WHEN el.status = 'delivered' THEN 1 ELSE 0 END) as delivered,
    SUM(CASE WHEN ee.event_type = 'open' THEN 1 ELSE 0 END) as opened,
    SUM(CASE WHEN ee.event_type = 'click' THEN 1 ELSE 0 END) as clicked,
    SUM(CASE WHEN el.status = 'bounced' THEN 1 ELSE 0 END) as bounced,
    SUM(CASE WHEN ee.event_type = 'complaint' THEN 1 ELSE 0 END) as complained
FROM EmailLogs el
LEFT JOIN EmailEvents ee ON el.log_id = ee.log_id
WHERE el.sent_at >= DATEADD(day, -90, GETUTCDATE())
GROUP BY el.tenant_id, el.email_type, CAST(el.sent_at AS DATE);
```

### Data Retention Procedure
```sql
CREATE PROCEDURE sp_PurgeOldEmailData
    @TenantId UNIQUEIDENTIFIER = NULL
AS
BEGIN
    SET NOCOUNT ON;
    
    DECLARE @RetentionDays INT;
    DECLARE @CutoffDate DATETIME2;
    
    IF @TenantId IS NOT NULL
    BEGIN
        SELECT @RetentionDays = retention_policy_days 
        FROM Tenants 
        WHERE tenant_id = @TenantId;
        
        SET @CutoffDate = DATEADD(day, -@RetentionDays, GETUTCDATE());
        
        -- Delete events first (FK constraint)
        DELETE ee FROM EmailEvents ee
        INNER JOIN EmailLogs el ON ee.log_id = el.log_id
        WHERE el.tenant_id = @TenantId AND el.sent_at < @CutoffDate;
        
        -- Delete email logs
        DELETE FROM EmailLogs 
        WHERE tenant_id = @TenantId AND sent_at < @CutoffDate;
        
        -- Delete old audit logs (keep longer retention)
        DELETE FROM AuditLogs 
        WHERE tenant_id = @TenantId AND timestamp < DATEADD(day, -365, GETUTCDATE());
    END
END;
```

## Row-Level Security Implementation

```sql
-- Enable RLS on key tables
ALTER TABLE EmailLogs ENABLE ROW LEVEL SECURITY;
ALTER TABLE EmailEvents ENABLE ROW LEVEL SECURITY;
ALTER TABLE EmailTemplates ENABLE ROW LEVEL SECURITY;
ALTER TABLE EmailConfigurations ENABLE ROW LEVEL SECURITY;
ALTER TABLE AuditLogs ENABLE ROW LEVEL SECURITY;

-- Create security policies
CREATE SECURITY POLICY tenant_isolation_policy_EmailLogs
ADD FILTER PREDICATE tenant_id = CAST(SESSION_CONTEXT(N'tenant_id') AS UNIQUEIDENTIFIER) ON EmailLogs;

CREATE SECURITY POLICY tenant_isolation_policy_EmailEvents
ADD FILTER PREDICATE 
    log_id IN (SELECT log_id FROM EmailLogs WHERE tenant_id = CAST(SESSION_CONTEXT(N'tenant_id') AS UNIQUEIDENTIFIER))
ON EmailEvents;

-- Similar policies for other tables...
```

## Backup and Recovery Strategy

### Full Backup Schedule
```sql
-- Full backup weekly
BACKUP DATABASE EmailPlatform 
TO DISK = 'S3://email-platform-backups/full/EmailPlatform_Full.bak'
WITH COMPRESSION, ENCRYPTION (
    ALGORITHM = AES_256,
    SERVER CERTIFICATE = EmailPlatformBackupCert
);

-- Differential backup daily
BACKUP DATABASE EmailPlatform 
TO DISK = 'S3://email-platform-backups/diff/EmailPlatform_Diff.bak'
WITH DIFFERENTIAL, COMPRESSION;

-- Transaction log backup every 15 minutes
BACKUP LOG EmailPlatform 
TO DISK = 'S3://email-platform-backups/log/EmailPlatform_Log.trn';
```

## Performance Optimization

### Key Performance Indicators
- Query response time < 100ms for 95th percentile
- Email log inserts > 10,000 per second
- Analytics queries < 5 seconds for 30-day periods
- Template retrieval < 50ms

### Indexing Strategy
- Clustered indexes on high-volume tables (EmailLogs, EmailEvents)
- Covering indexes for common query patterns
- Filtered indexes for active records
- Partitioning for time-series data

### Monitoring Queries
```sql
-- Email volume by tenant
SELECT 
    t.name,
    COUNT(*) as emails_last_24h,
    AVG(DATEDIFF(second, el.sent_at, ee.timestamp)) as avg_delivery_time
FROM Tenants t
LEFT JOIN EmailLogs el ON t.tenant_id = el.tenant_id
LEFT JOIN EmailEvents ee ON el.log_id = ee.log_id AND ee.event_type = 'delivery'
WHERE el.sent_at >= DATEADD(hour, -24, GETUTCDATE())
GROUP BY t.tenant_id, t.name
ORDER BY emails_last_24h DESC;
```

This database schema provides a robust foundation for the Email Platform with proper tenant isolation, comprehensive audit trails, and optimized performance for high-volume email processing.