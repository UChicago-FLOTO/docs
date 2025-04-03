# **FLOTO Platform Documentation**

## **I. Introduction**

This section introduces the FLOTO platform's purpose, architecture, and helps users navigate the documentation based on their role.

### **A. Platform Purpose and Goals**

FLOTO is a discovery testbed that enables researchers to deploy applications on distributed devices for measuring internet access networks. It provides a unified infrastructure for broadband research, education, and policy initiatives, offering easy deployment, management, and data collection capabilities.

### **B. System Architecture**

The FLOTO platform is built on two primary components:

1. **Device Management Layer**: Built on openBalena, manages connections to devices via VPN, enabling remote command execution and application deployment.  
2. **Application Layer**: Built on Kubernetes, orchestrates application deployments across distributed devices. Devices run a Kubernetes worker service that connects them to a larger cluster, allowing for containerized application deployment.

### **C. Documentation Navigation Guide**

This documentation is organized by user roles and workflows:

* **Section II (Device Layer)**: For device operators managing hardware deployment and configuration  
* **Section III (Application Layer)**: For researchers developing and deploying applications  
* **Section IV (Data Layer)**: For data analysts working with collected measurements
* **Section V (API Integration)**: For programmatic access to FLOTO services
* **Section VI (Best Practices)**: For optimizing your FLOTO experience
* **Section VII (Reference Materials)**: For additional resources and specifications

## **II. Device Management**

### **A. Device Acquisition**

Learn how to request and receive pre-configured FLOTO devices for your research project.

1. **Request Process** The FLOTO team offers pre-configured Raspberry Pi 4 8GB devices available through a formal request process at [https://floto.cs.uchicago.edu/deployment/apply-for-floto-devices/](https://floto.cs.uchicago.edu/deployment/apply-for-floto-devices/). Decisions are made within 3 business days and devices ship within 2 weeks of approval.  
2. **Device Kit Contents** Each approved request receives devices with all necessary components for deployment, including a protective case, Ethernet cord, power supply, and pre-flashed OS.  
3. **Installation Requirements** Devices can be deployed in various environments (residential, commercial, campus) and require only power and internet connectivity via Ethernet.

### **B. Device Operator Responsibilities**

This section covers the tasks and responsibilities of device operators in the FLOTO ecosystem.

1. **Network Configuration** Devices require specific outbound network access to function:  
   * TCP/443: HTTPS traffic to {api,auth,k3s-ipv6,portal,registry,s3,tunnel,vpn}.floto.science  
   * TCP/3128: Remote command/control capabilities to vpn.floto.science  
   * TCP/6443: HTTPS traffic to k3s.floto.science (application control plane)  
   * UDP/51820: VPN connection to k3s.floto.science for application deployment  
   * No inbound access is required  
2. **Device Dashboard Operation** The FLOTO dashboard (portal.floto.science) provides a web interface for device management, including:  
   * Device status monitoring (online/offline, ready/not ready)  
   * Command execution  
   * Environment variable configuration  
   * Log access (last 1000 lines)  
   * Peripheral management  
3. **Monitoring and Maintenance** Regular monitoring of device status helps ensure reliable data collection. The dashboard shows key operational metrics including:  
   * Connection status (`is_online`, `is_ready`)  
   * Resource utilization (memory, storage, CPU)  
   * Current IP address and network configuration

### **C. Custom Device Registration**

For researchers with existing hardware who want to integrate with FLOTO.

1. **Compatibility Requirements** Custom device integration requires:  
   * Hardware supported by OpenBalena OS  
   * Communication with the FLOTO team at contact@floto.science  
   * Adding the device to FLOTO's k3 fleet for application deployment  
2. **Integration Process** To add your device to FLOTO:  
   * Contact the FLOTO team with information about your project and needs
   * Follow the specific instructions provided by the FLOTO team for your situation

## **III. Application Deployment**

### **A. Application Architecture**

Understanding how applications are structured and managed in the FLOTO ecosystem.

1. **Component Hierarchy** FLOTO uses a three-tier model for application deployment:  
   * **Services**: Container references (Docker images) for individual applications  
   * **Applications**: Orchestration of services (similar to docker-compose files)  
   * **Jobs**: Deployment of applications to specific devices on defined schedules  
2. **Privacy Controls** Each component (service, application, job) can be configured as public or private:  
   * Public components are visible to all FLOTO users  
   * Private components are visible only to their creator (project-wide)  
   * Environment variables can be set at the application or job level, allowing sensitive configuration to remain private

### **B. Application Development**

Guidelines for creating applications compatible with FLOTO's infrastructure.

1. **Development Environment** FLOTO is designed as a deployment (not development) platform. Developers should:  
   * Develop and test containers locally before submission  
   * Consider using CHI@Edge for testing in a similar environment  
   * Ensure compatibility with ARM architecture (Raspberry Pi)  
   * Follow container best practices for resource efficiency  
2. **Container Requirements** All containers deployed on FLOTO must:  
   * Be built for ARM architecture  
   * Be hosted on accessible registries  
   * Follow resource constraints appropriate for edge devices  
   * Define required peripherals and port configurations

### **C. Deployment Process**

Step-by-step guide to deploying applications on FLOTO devices.

1. **Service Creation** Services represent the container images your application uses:  
   * Register your container via the Services page in the dashboard  
   * Configure required peripherals and port forwarding  
   * Specify the registry URL for your container  
2. **Application Configuration** Applications define how services work together:  
   * Create a named application with a descriptive purpose  
   * Select one or more services to include  
   * Configure environment variables  
   * Determine if single-tenant (exclusive device use) or multi-tenant operation is required  
3. **Storage Access** Applications can use two types of storage:  
   * `/share`: Ephemeral volume shared between containers, deleted after job completion  
   * `/public`: Persistent volume shared across jobs and users, suitable for data collection  
4. **Job Scheduling** Jobs control when and where applications run:  
   * Select target devices from the available pool  
   * Define runtime duration  
   * Set job-specific environment variables  
   * Monitor execution through the dashboard
5. **Project Selection** Before creating jobs:
   * Ensure your project name is active in the top navigation bar
   * If not visible, go to your profile at portal.floto.science/dashboard/user
   * Click "Set Active Project" to select your project

### **D. Featured Applications**

Ready-to-use applications available to all FLOTO users.

1. **Netrics** A comprehensive network measurement application that:  
   * Provides a framework for executing network measurements
   * Collects broadband speed, latency, and network parameter data
   * Includes built-in measurements from Ookla, Measurement Lab, iPerf and more
   * Supports digital divide and internet equity research
   * Features flexible configuration formats (YAML or TOML)
   * Is available as version 0.0.2-post2 from [https://github.com/internet-innovation/netrics](https://github.com/internet-innovation/netrics)
   
2. **Data Uploader Options**:

   a. **FLOTO Data Uploader** A versatile data collection service that:  
   * Monitors directories for file changes
   * Uses rclone to automatically upload data to configured remote storage
   * Default configuration watches `/share/data` directory (shared across containers)
   * Requires rclone configuration via `RCLONE_CONFIG_JSON` environment variable
   * Is available as a Docker container from [https://github.com/UChicago-FLOTO/data_uploader](https://github.com/UChicago-FLOTO/data_uploader)
   
   b. **nm-mgmt-collect** An alternative data collection system that:
   * Supports the Netrics measurement framework
   * Provides HTTPS-based data collection to S3
   * Features both native and container deployment options
   * Automatically processes files with device IDs
   * Supports customization via environment variables (NM_ID, NM_PATH_DATA_PENDING, etc.)
   * Organizes data with a structured path: `netrics_mngd/{experiment}/{topic}/{device}/{date}/{extension}/{filename}`

### **E. Application Request Process**

Guidelines for requesting permission to deploy applications on FLOTO.

1. **Application Information Requirements** All deployment requests must include:  
   * Applicant details (name, institution, contact information)  
   * Project description (objectives, field of study, expected outcomes)  
   * Application specifications (network usage, data handling protocols)  
   * Resource requirements (device count, peripherals needed)  
2. **Review Process** The FLOTO team evaluates requests based on:  
   * Research merit and alignment with platform goals  
   * Resource requirements and availability  
   * Technical feasibility and security considerations  
   * Data handling and privacy safeguards

## **IV. Data Management**

### **A. Data Collection**

How application developers can collect and store data from FLOTO device deployments. Note that FLOTO primarily provides the infrastructure for running applications on devices - data collection, storage, and publishing is the responsibility of the application developer.

1. **Data Collection Responsibilities** 
   * FLOTO only provides infrastructure for running applications on devices
   * Application developers are responsible for their own data collection, storage, and access mechanisms
   * Currently, only measurement data from the default Netrics application is supported for inclusion in the FLOTO public dataset
   * All other applications must implement their own complete data pipeline

2. **Data Storage Options on Devices** 
   * Write data to local files in `/share` (ephemeral storage shared between containers in your application)
   * Store persistent data in `/public` (accessible across jobs and users)
   * Note that neither of these options automatically transfers data off the device

3. **Data Transfer Options**
   * Application developers must configure their own data transfer mechanisms
   * You can incorporate a pre-built data uploader service alongside your application
   * You can develop and deploy your own custom data transfer service
   * FLOTO provides no automatic data transfer off devices

4. **Available Data Uploader Tools** 
   * Off-the-shelf tools that can be configured and deployed alongside your application
   * You must configure these tools with your own storage endpoints (S3, etc.)
   * FLOTO provides no storage for your data unless you're running the default Netrics measurements
   * FLOTO support is limited to deployment assistance and troubleshooting device-side issues

5. **FLOTO Data Uploader Options**
   
   a. **rclone-based Uploader Configuration**:
      * Flexible tool that can upload to many remote storage types
      * Required `RCLONE_CONFIG_JSON` environment variable for remote storage configuration
      * Optional `DIRECTORY` variable (defaults to `/share/data`) to specify directory to watch
      * Optional `TARGET_BUCKET` to specify destination bucket name
      * Works with any non-empty file format
      * You are responsible for setting up the destination storage
   
   b. **nm-mgmt-collect Configuration**:
      * HTTPS-based uploader specifically designed for Netrics data format
      * Configure with environment variables like `NM_ID` (device identifier)
      * Set paths with `NM_PATH_DATA_PENDING` and `NM_PATH_DATA_ARCHIVE`
      * Specify experiment with `NM_EXP` and topic with `NM_TOPIC`
      * Organizes data using the path structure: `netrics_mngd/{experiment}/{topic}/{device}/{date}/{extension}/{filename}`
      * You are responsible for setting up the destination S3 bucket

6. **Netrics Data Format** 
   * Only relevant if using the nm-mgmt-collect uploader or the default Netrics application
   * Only data from the default Netrics application is currently eligible for inclusion in the FLOTO public dataset
   * Example structure:
     ```json
     {
       "Measurements": {
         "dns_latency": {
           "dns_query_avg_ms": 14,
           "dns_query_max_ms": 43,
           "error": false
         },
         "ping_latency": {
           "Chicago_rtt_avg_ms": 3.274,
           "Chicago_rtt_max_ms": 3.642
         }
       },
       "Meta": {
         "Id": "device-id",
         "Time": 1633974357.597229
       }
     }
     ```

### **B. Data Access and Sharing**

Application developers must implement their own data access and sharing mechanisms.

1. **Custom Data Access Solution Responsibility**
   * FLOTO does not provide data access services for custom applications
   * Developers must create their own data access solutions after collecting data from devices
   * Options include hosting your own API, database, or using cloud storage with appropriate access controls

2. **Shared Device Storage** 
   * Applications can share data between containers using the `/share` volume (ephemeral)
   * Applications can share data across jobs using the `/public` volume (persistent on device)
   * These options only facilitate data sharing on the same device
   
3. **Data Integration Patterns**
   * Producer/consumer application pairs on the same device
   * Scheduled data collection jobs and separate data processing jobs
   * External systems that retrieve and process data from your configured storage

## **V. API Integration**

### **A. API Overview**

Using the FLOTO API for programmatic interaction with the platform.

1. **Authentication Methods** The API supports two authentication methods:  
   * Globus auth for user-based access  
   * API tokens for application integration (found on your user profile at portal.floto.science/dashboard/user)  
   * Permission models based on user roles and project assignments  
2. **Core API Services** The FLOTO API provides endpoints for:  
   * Device management (status, commands, environment)  
   * Application deployment (services, applications, jobs)  
   * Data collection (datasets, collections)  
   * Project administration (access control, resource allocation)
3. **Base API Endpoint** The API is accessible at:
   * https://portal.floto.science/api/
4. **API Schema** Interactive API documentation is available at:
   * https://portal.floto.science/api/schema/swagger-ui/
5. **Example Usage** A basic API call using an API token:
   ```bash
   curl -L -X GET \
       -H "Authorization: Token $FLOTO_API_KEY" \
       https://portal.floto.science/api/services
   ```

### **B. Device Management API**

Programmatic control of devices on the FLOTO platform.

1. **Key Endpoints**  
   * `/api/devices/` \- List and retrieve device information  
   * `/api/devices/{id}/action/` \- Execute device actions  
   * `/api/devices/{id}/command/` \- Run commands on devices  
   * `/api/devices/{id}/environment/` \- Manage environment variables  
   * `/api/devices/{id}/logs/{count}/` \- Retrieve device logs  
   * `/api/devices/{id}/peripherals/` \- Configure device peripherals  
2. **Status Monitoring** API provides detailed device status information:  
   * `is_online`: Device connection to OpenBalena  
   * `is_ready`: Application framework status  
   * `api_heartbeat_state`: Communication status  
   * Hardware metrics (memory, storage, CPU utilization)
3. **Device Metadata** Applications can access device information via API:
   * The `FLOTO_DEVICE_UUID` environment variable is available to all applications
   * This UUID can be used with the device API to retrieve metadata such as location

### **C. Application Management API**

Deploying and managing applications via API calls.

1. **Deployment Endpoints**  
   * `/api/services/` \- Container reference management  
   * `/api/applications/` \- Application orchestration  
   * `/api/jobs/` \- Device assignment and scheduling  
   * `/api/jobs/{id}/events/` \- Job event tracking  
   * `/api/jobs/{id}/logs/` \- Log retrieval  
2. **Schedule Management**  
   * `/api/timeslots/` \- Schedule management  
   * Conflict resolution and resource allocation  
   * Device availability verification

## **VI. Best Practices and Troubleshooting**

### **A. Resource Utilization**

Guidelines for efficient use of FLOTO platform resources.

1. **Network Impact Considerations** When designing applications, consider:  
   * Bandwidth consumption limitations  
   * Data transfer volume and frequency  
   * Protocol efficiency  
   * Impact on host networks  
2. **Device Resource Management** Edge devices have limited resources; optimize for:  
   * Memory efficiency  
   * Storage utilization  
   * CPU usage patterns  
   * Power consumption

### **B. Troubleshooting Common Issues**

Solutions for frequently encountered problems with FLOTO deployments.

1. **Device Connectivity** When devices show offline:  
   * Verify physical network connection  
   * Check required port access for VPN and API communication  
   * Review authentication token validity  
   * Examine device logs for connection failures  
2. **Application Deployment Failures** Common causes of deployment issues:  
   * Container architecture mismatch (ARM vs x86)  
   * Resource constraints (memory, storage)  
   * Environment variable configuration errors  
   * Missing peripheral dependencies  
3. **Data Collection Problems** If data isn't being collected properly:  
   * Check that files are non-empty  
   * Verify daemon status for data uploader  
   * Confirm network connectivity to AWS endpoints  
   * Review logs for specific error messages

## **VII. Reference Materials**

### **A. Contact Information**

Who to contact for different types of assistance.

1. **Support Channels** For help with FLOTO:  
   * Email: contact@floto.science  
   * Web form: https://floto.cs.uchicago.edu/contact-us/  
   * Device enrollment: mtrichardson@uchicago.edu  
2. **Documentation Resources** Additional reference materials:  
   * GitHub repositories for applications and tools  
   * API documentation via Swagger UI  
   * Installation and configuration guides

### **B. Technical Specifications**

Detailed requirements and limitations of the FLOTO platform.

1. **Device Hardware** Standard FLOTO devices are based on:  
   * Raspberry Pi single-board computers  
   * Ethernet connectivity  
   * Configured for headless operation  
   * Pre-installed with OpenBalena OS  
2. **Application Constraints** Applications should work within these limits:  
   * ARM architecture compatibility  
   * Container-based deployment  
   * Kubernetes orchestration  
   * Resource-constrained edge environment

