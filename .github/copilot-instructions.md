# WebMethods System Analyzer - AI Coding Agent Instructions

## Project Purpose
**wm-system-analyzer** is a diagnostic and monitoring toolkit for IBM WebMethods Integration Server. It captures, organizes, and analyzes system state across four layers: static configuration, runtime metrics, historical analytics, and deployment profiles.

## Architecture Overview

### Data Collection & Organization
The project follows a **layered data snapshot architecture**:

- **`/config`** - Static system state (server config, services, connections, security settings)
- **`/runtime`** - Dynamic runtime metrics (threads, pools, scheduler state, usage statistics)
- **`/logs`** - Historical analytics (CSV time-series data like `stats.csv`)
- **`/profiles`** - JVM deployment configurations (wrapper.conf for containerization)

**Key Insight**: Each layer serves a specific purpose in system diagnostics. Changes should maintain this separation of concerns.

## Critical Components & Integration Points

### Universal Messaging (Primary)
- **Connection Alias**: `IS_UM_CONNECTION` (defined in `config/MessagingConnectionAliasInfo.txt`)
- **Architecture**: 3-node nsp cluster (10.129.3.125:9000, 10.129.3.117:9000, 10.129.3.89:9000)
- **Features**: Client-Side Queueing (CSQ) enabled, ordered delivery, master-follower topology
- **Related Files**: `config/JMSTriggersInfo.txt`, `runtime/MessagingTriggersInfo.txt`, `runtime/JMSTriggersInfo.txt`

### JDBC & Database
- Managed through `config/JDBCPools.txt` and `runtime/JDBCPools.txt` (stateless in most services)
- Trigger configurations stored in trigger info files

### Services & Packages
- **Reference**: `config/ServicesInfo.txt` (2540+ lines, lists all installed services with audit/caching settings)
- **Adapter Packages**: MCUMConsumer package (100+ adapter services visible)
- **License**: Tracked per service in `config/LicenseInfo.txt`

### System Properties & Security
- **Properties**: `config/SysProps.txt` (943 lines of JVM/platform properties)
- **Encryption**: AES-encrypted credentials (e.g., `WEBM_CLUSTERUSER={AES}...`)
- **Security**: JCE settings in `config/JCESecurityInfo.txt`

### Deployment Infrastructure
- **Wrapper Config**: `profiles/config/wrapper.conf` (227 lines, Java/OSGI bootstrap)
- **Container Support**: Set OSGI paths to `/opt/sag/products/profiles/IS_default`
- **Custom Settings**: `profiles/config/custom_wrapper.conf` for environment-specific tuning
- **AppDynamics Integration**: Node name pattern `IntegrationServer-AZ1-2`, tier `UMF-v1011-PROD-IntegrationServer`

## File Format Conventions

### Header Pattern
All files include timestamps and descriptive headers:
```
2025-05-12 09:12:22 UTC   <Component> Configuration Information
---------------------------------------------------
```

### Configuration Files (`.txt`, `.cnf`)
- Property-based format with key=value pairs (see `config/Server.cnf`)
- Boolean values: `true`/`false`, missing values: `null`
- Multiline values use escaped newlines or continuation patterns

### XML Configuration Files
Located in `config/Auditing/`, `config/Caching/`, `config/TCDB/` - follow Software AG naming:
- `AuditConfig.xml` - Audit logging policy
- `SoftwareAG-IS-*.xml` - Component-specific cache managers (Core, Services, IData, WMN, ART)

### CSV Analytics
- `logs/stats.csv` - Time-series metrics (moved from root in recent restructuring commit c8ebd44)

## Developer Workflows

### Data Capture & Analysis
1. Extract system diagnostics via analyzer tool
2. Organize into `/config` (static), `/runtime` (dynamic), `/logs` (history)
3. Parse connection aliases and trigger configurations for integration point analysis
4. Cross-reference services in `ServicesInfo.txt` against triggers and connections

### Validation & Verification
- Check for required connection aliases: `IS_UM_CONNECTION`, `IS_DES_CONNECTION`, `IS_LOCAL_CONNECTION`
- Verify nsp cluster node health (3 nodes typically required for HA)
- Validate service stateless/caching configuration consistency
- Audit encrypted property formats (AES prefix `{AES}`)

### Containerization Workflow
- Update wrapper.conf OSGI paths when migrating to container environments
- Ensure custom_wrapper.conf includes environment-specific JVM args
- Verify AppDynamics nodeNames align with deployment topology

## Key Patterns & Conventions

### Naming Conventions
- **Service Paths**: `PackageName.adapters:serviceName` (e.g., `MCUMConsumer.adapters:addPublicHoliday`)
- **Connection Aliases**: `IS_<PROVIDER>_CONNECTION` (UM, DES, LOCAL)
- **Properties**: Prefixes indicate scope: `watt.*` (core), `com.softwareag.*` (platform), `appdynamics.*` (monitoring)

### Configuration Inheritance
Connection and service settings cascade: System Properties → Server Config → Specific Component Configs
Example: `SysProps.txt` (global) → `Server.cnf` (server-wide) → `MessagingConnectionAliasInfo.txt` (component-specific)

### Cluster Awareness
- **Cluster ID**: Stored in `com.softwareag.clusterid` (SysProps.txt)
- **Terracotta**: NOT used (verified absent from configs) - clustering via Universal Messaging native architecture
- **High Availability**: Configured in connection aliases with `Follow the Master` flags

## Integration Points to Consider

1. **Audit System** (`config/Auditing/`): Track service execution, errors, pipelines based on `ServicesInfo.txt` audit settings
2. **Caching** (`config/Caching/`): Multiple cache managers per subsystem - coordinate TTL and prefetch strategies
3. **Message Triggers** (`*TriggersInfo.txt`): Validate trigger type (JMS, Messaging, Scheduler) matches connection type
4. **License Enforcement**: Cross-check `LicenseInfo.txt` against enabled services to prevent violations

## Common Tasks & Examples

### Adding a Service Analysis
1. Extract service entry from `config/ServicesInfo.txt`
2. Check trigger type in `runtime/MessagingTriggersInfo.txt` or `runtime/JMSTriggersInfo.txt`
3. Verify connection alias exists in `MessagingConnectionAliasInfo.txt`
4. Document caching/audit settings from service configuration

### Troubleshooting Messaging Issues
1. Verify cluster nodes in `IS_UM_CONNECTION` nsp endpoints
2. Check CSQ settings and drain-in-order flags
3. Review `MessagingTriggersInfo.txt` for trigger enable/disable state
4. Cross-reference error patterns in `runtime/ServerStats.txt` Service Errors metric

### Container Deployment Preparation
1. Update wrapper.conf paths from hardcoded `/opt/sag/products` to container mount points
2. Externalize secrets (WEBM_CLUSTERUSER) from SysProps into environment variables
3. Adjust AppDynamics nodeNames to container instance identifiers
4. Validate custom_wrapper.conf memory/thread settings for container resource limits

## Recent Changes & Project Evolution

- **Latest Commit** (c8ebd44, Nov 27 2025): Restructuring - moved `stats.csv` from root to `logs/` directory
- **Project Status**: Under active containerization migration (folder structure: wm-system-analyzer)
- **Focus Area**: Integration Server diagnostics for microservices/cloud deployment

## Files You'll Reference Most

1. `config/Server.cnf` - Core configuration tuning
2. `config/ServicesInfo.txt` - Service inventory and audit/cache settings
3. `config/MessagingConnectionAliasInfo.txt` - Messaging topology and UM cluster details
4. `profiles/config/wrapper.conf` - Deployment and JVM bootstrap
5. `config/SysProps.txt` - System-wide properties and encryption keys
6. `runtime/ServerStats.txt` - Performance baseline and error metrics

---

**Last Updated**: November 29, 2025 | **Analyzer Version**: Supports WebMethods 10.11+
