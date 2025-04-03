# **FLOTO Platform Documentation**

## **I. Introduction**

### **A. Platform Purpose**

FLOTO is a discovery testbed enabling researchers to deploy applications on distributed devices for measuring internet access networks. It provides unified infrastructure for broadband research, offering streamlined deployment, management, and data collection capabilities.

### **B. System Architecture**

FLOTO consists of two core components:

1. **Device Management Layer**: Built on openBalena, maintains VPN connections to devices for remote command execution and application deployment
2. **Application Layer**: Built on Kubernetes, orchestrates containerized application deployments across distributed devices via a worker service installed on each device

### **C. Documentation Structure**

* **Section II**: Device management for hardware operators
* **Section III**: Application deployment for researchers and developers
* **Section IV**: Data management for application developers
* **Section V**: API integration for programmatic access
* **Section VI**: Best practices and troubleshooting
* **Section VII**: Reference materials and specifications

## **II. Device Management**

### **A. Device Acquisition**

1. **Request Process**: Apply at [floto.cs.uchicago.edu/deployment/apply-for-floto-devices](https://floto.cs.uchicago.edu/deployment/apply-for-floto-devices). Decisions within 3 business days, shipping within 2 weeks.
2. **Kit Contents**: Pre-configured Raspberry Pi 4 8GB with protective case, Ethernet cord, power supply, and pre-flashed OS.
3. **Installation Requirements**: Requires only power and Ethernet connection. Compatible with residential, commercial, and campus environments.

### **B. Device Operation**

1. **Network Requirements**:
   * TCP/443: HTTPS to {api,auth,k3s-ipv6,portal,registry,s3,tunnel,vpn}.floto.science
   * TCP/3128: Command/control to vpn.floto.science
   * TCP/6443: HTTPS to k3s.floto.science (application control plane)
   * UDP/51820: VPN to k3s.floto.science (application deployment)
   * No inbound access required

2. **Dashboard Management** (portal.floto.science):
   * Monitor device status (online/offline, ready/not ready)
   * Execute commands
   * Configure environment variables and peripherals
   * Access logs (last 1000 lines)
   * View resource utilization (memory, storage, CPU)

### **C. Custom Device Integration**

1. **Compatibility**:
   * Hardware must support OpenBalena OS
   * Must be added to FLOTO's k3s fleet

2. **Process**:
   * Contact the FLOTO team (contact@floto.science)
   * Provide project information and requirements
   * Follow FLOTO team instructions for your specific setup

3. **Terms and Conditions** (to be documented):
   * Support for custom device images is provided on a case-by-case basis
   * FLOTO team reserves the right to decline custom device integration requests
   * Custom devices must meet minimum hardware requirements and security standards
   * Maintenance and troubleshooting support may be limited for custom devices

## **III. Application Deployment**

### **A. Application Architecture**

1. **Three-Tier Model**:
   * **Services**: Container references (Docker images)
   * **Applications**: Orchestration of services (similar to docker-compose)
   * **Jobs**: Scheduled application deployments to specific devices

2. **Privacy Controls**:
   * Components can be public (visible to all users) or private (project-specific)
   * Environment variables configurable at application or job level for security

3. **Multi-tenancy Support**:
   * Applications can be configured as "single-tenant" (exclusive device use) or "multi-tenant"
   * Multi-tenant applications can run alongside other applications on the same device
   * Shared persistent storage (`/public`) is accessible across applications in multi-tenant environments
   * Resource allocation and isolation are managed by the Kubernetes orchestration layer

### **B. Application Development**

1. **Development Guidelines**:
   * FLOTO is a deployment platform, not a development environment
   * Develop and test containers locally first
   * Consider CHI@Edge for testing in similar environments
   * Ensure ARM architecture compatibility

2. **Container Requirements**:
   * ARM architecture (Raspberry Pi compatible)
   * Hosted on accessible registries
   * Resource-efficient for edge devices
   * Properly configured peripherals and ports

### **C. Deployment Process**

1. **Service Registration**:
   * Register container via dashboard
   * Configure peripherals and port forwarding
   * Specify registry URL

2. **Application Configuration**:
   * Create named application
   * Select services to include
   * Configure environment variables
   * Choose single-tenant or multi-tenant operation

3. **Storage Options**:
   * `/share`: Ephemeral volume shared between containers
   * `/public`: Persistent volume shared across jobs and users

4. **Job Creation**:
   * Ensure active project selection in portal.floto.science
   * Select target devices
   * Define runtime duration
   * Set job-specific environment variables
   * Monitor execution via dashboard

### **D. Featured Applications**

1. **Netrics**: Network measurement application
   * Executes broadband speed, latency, and network parameter measurements
   * Includes built-in tools from Ookla, Measurement Lab, and iPerf
   * Supports digital divide research
   * Available at [github.com/internet-equity/netrics](https://github.com/internet-equity/netrics)

2. **Data Uploader Options**:

   a. **FLOTO Data Uploader**:
   * rclone-based tool for any storage endpoint
   * Monitors `/share/data` directory by default
   * Configurable via environment variables
   * Available at [github.com/UChicago-FLOTO/data_uploader](https://github.com/UChicago-FLOTO/data_uploader)
   
   b. **nm-mgmt-collect**:
   * HTTPS-based uploader for Netrics data
   * Supports native and container deployment
   * Configurable via environment variables
   * Organizes data by experiment, topic, device, and date

### **E. Application Request Process**

1. **Required Information**:
   * Applicant details (name, institution, contact)
   * Project description and objectives
   * Technical specifications (network usage, data handling)
   * Resource requirements (device count, peripherals)

2. **Evaluation Criteria**:
   * Research merit and platform alignment
   * Resource availability
   * Technical feasibility
   * Security and privacy controls

## **IV. Data Management**

### **A. Data Collection**

FLOTO provides infrastructure for running applications but *does not* handle data collection, storage, or publishing. These responsibilities belong to application developers.

1. **Developer Responsibilities**:
   * Implement your own data collection pipeline
   * Only default Netrics application data can be included in FLOTO public dataset
   * All other applications must handle their complete data lifecycle

2. **On-Device Storage Options**:
   * `/share`: Ephemeral storage shared between containers
   * `/public`: Persistent storage shared across jobs/users
   * Neither option automatically transfers data off-device

3. **Data Transfer Approaches**:
   * Add a data uploader service to your application
   * Develop custom transfer mechanisms
   * Configure your own storage endpoints (S3, etc.)

4. **Uploader Configurations**:

   a. **rclone-based Uploader**:
   * Works with multiple storage types
   * Configure with `RCLONE_CONFIG_JSON`, `DIRECTORY`, and `TARGET_BUCKET` variables
   * Monitors specified directories for changes

   b. **nm-mgmt-collect**:
   * Designed for Netrics data format
   * Configure with `NM_*` environment variables
   * Organizes data hierarchically by experiment/topic/device/date

5. **Uploader-Repository Compatibility Matrix** (to be documented):
   * Compatibility between uploaders and different storage repositories
   * Storage authentication requirements
   * Performance considerations

6. **Netrics Data Format and Upload Context**:
   * While nm-mgmt-collect can upload any file type, the Netrics Data Format is required for proper processing
   * By default, files are uploaded to the central Netrics S3 bucket, not your own storage
   * Redeployment with modified configuration is required to direct data to a different S3 bucket
   * Only data following this format can potentially be included in the public dataset
   * Example structure:
   ```json
   {
     "Measurements": {
       "dns_latency": {"dns_query_avg_ms": 14},
       "ping_latency": {"Chicago_rtt_avg_ms": 3.274}
     },
     "Meta": {
       "Id": "device-id",        /* Required for processing */
       "Time": 1633974357.597229 /* Unix timestamp */
     }
   }
   ```
   * Files must contain both a "Meta" object with "Id" and "Time" fields
   * The "Measurements" object contains the actual metric data
   * Files are processed based on the directory structure they're placed in
   * Empty files are ignored by the uploader

### **B. Data Access and Sharing**

1. **Access Mechanisms**:
   * Developers must create their own data access solutions
   * No FLOTO-provided data services for custom applications
   * Options: APIs, databases, cloud storage with access controls

2. **Integration Patterns**:
   * Producer/consumer application pairs on same device
   * Scheduled collection and processing jobs
   * External systems accessing your configured storage

3. **IEI Data Platform** (external resource):
   * Available for Netrics data management and visualization
   * Comprised of PostgreSQL data warehouse, Airflow for ETL, and Superset for visualization
   * Not directly integrated with FLOTO but can be used for analyzing collected Netrics data
   * Requires separate setup and configuration
   * Contact the FLOTO team for access details

## **V. API Integration**

### **A. API Overview**

1. **Authentication**:
   * Globus auth for user access
   * API tokens for application integration (found at portal.floto.science/dashboard/user)
   * Permission models based on roles and project assignments

2. **Core Services**:
   * Device management
   * Application deployment
   * Project administration

3. **Access Information**:
   * Base endpoint: https://portal.floto.science/api/
   * Documentation: https://portal.floto.science/api/schema/swagger-ui/
   * Example usage:
     ```bash
     curl -L -X GET -H "Authorization: Token $FLOTO_API_KEY" \
       https://portal.floto.science/api/services
     ```

4. **API Integration Examples** (to be documented):
   * Common patterns for combining API endpoints
   * Multi-step workflows for device management and application deployment
   * Example scripts for automating FLOTO operations
   * Integration with other systems and custom workflows

### **B. Device Management API**

1. **Key Endpoints**:
   * `/api/devices/` - List and retrieve devices
   * `/api/devices/{id}/action/` - Execute actions
   * `/api/devices/{id}/command/` - Run commands
   * `/api/devices/{id}/environment/` - Manage variables
   * `/api/devices/{id}/logs/{count}/` - Access logs
   * `/api/devices/{id}/peripherals/` - Configure peripherals

2. **Status Monitoring**:
   * `is_online`, `is_ready`, `api_heartbeat_state`
   * Hardware metrics (memory, storage, CPU)

3. **Device Metadata**:
   * `FLOTO_DEVICE_UUID` environment variable
   * Access location and other metadata via API

### **C. Application Management API**

1. **Deployment Endpoints**:
   * `/api/services/` - Container references
   * `/api/applications/` - Service orchestration
   * `/api/jobs/` - Device assignment
   * `/api/jobs/{id}/events/` - Job tracking
   * `/api/jobs/{id}/logs/` - Log access

2. **Scheduling**:
   * `/api/timeslots/` - Manage schedules
   * Conflict resolution
   * Device availability checks

## **VI. Best Practices and Troubleshooting**

### **A. Resource Optimization**

1. **Network Considerations**:
   * Minimize bandwidth consumption
   * Optimize data transfer frequency
   * Consider impact on host networks

2. **Device Resources**:
   * Optimize memory usage
   * Manage storage efficiently
   * Control CPU utilization
   * Monitor power consumption

### **B. Common Issues**

1. **Device Connectivity**:
   * Verify physical connections
   * Check required port access
   * Review authentication tokens
   * Examine connection logs

2. **Application Deployment**:
   * Ensure ARM architecture compatibility
   * Address resource constraints
   * Check environment variables
   * Verify peripheral dependencies

3. **Data Collection**:
   * Confirm file validity
   * Check uploader daemon status
   * Verify network connectivity
   * Review error logs

## **VII. Reference Materials**

### **A. Support Resources**

1. **Contact Channels**:
   * General support: contact@floto.science
   * Web form: floto.cs.uchicago.edu/contact-us
   * Device enrollment: mtrichardson@uchicago.edu

2. **Documentation**:
   * GitHub repositories: github.com/UChicago-FLOTO
   * API documentation: portal.floto.science/api/schema/swagger-ui
   * Getting started guides: portal.floto.science/dashboard

### **B. Technical Specifications**

1. **Devices**:
   * Raspberry Pi 4 8GB
   * Ethernet connectivity
   * Headless operation
   * OpenBalena OS

2. **Application Requirements**:
   * ARM architecture
   * Container-based
   * Kubernetes-compatible
   * Resource-efficient design

