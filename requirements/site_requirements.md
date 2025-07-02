# Customer Questionnaire

## Architecture & Site Structure

### Q: Are all 600 sites independent of each other, or are there some dependencies?
The 600 or so sites will at a code level all utilize the same codebase (a monorepo).
- The monorepo will contain 3-4 website apps. One per "experience" these will each be slightly different, but utilize shared code modules.
- When deployed each will be deployed as its own K8s deployment. No site will "depend on another site".
- All the sites will depend on common shared Azure infra, as well as shared Microservices that we'll deploy in EIP.

### Q: What interaction is there between the CMS system and the individual sites?
The CMS is responsible for all (the vast majority) of content on the sites.
- The sites will interact with the CMS when server to server (as part of the Server Side Rendering process).
- There may also be some cases where we'll have clientside interaction with the CMS.
- It will always be read only.

### Q: Will each site have its own separate domain/subdomain or is path-based routing required?
- Some sites will have their own domain.
- A lot of sites will be on acme-brands.com – where path based routing will be required.
- Each domain will have a www. variant as well.

### Q: Will each site require a separate SSL certificate?
Yes, each site will have to have 2 SSL Certs:
- One for the base domain 
- Another for www
- We plan to use Azure Frontdoor managed certificates.

### Q: Does each site use a different base image, or are they the same images with different content and configuration?
Currently leaning towards a docker image per experience (3-4 in total) with different content and configuration passed in via K8s.

### Q: Are there any Kubernetes resources needed beyond the standard pods, services, ingress?
- Deployment
- akv2k8s – for syncing secrets from Azure Key Vault to the cluster
- KEDA – for scaling processor jobs based on Azure Storage Queue
- Azure Blob storage Container Storage Interface – in some cases for mounting azure blob storage as a volume

### Q: Is there a general traffic profile that is applicable to most sites, or does traffic vary for each?
Varies, this is why each site will be its own deployment, to allow us to scale properly.

## Security & Compliance

### Q: Is authentication required and if so how is that managed?
[No answer provided]

### Q: Do the sites require secrets in their configuration? How are these stored?
- Secrets via KeyVault and sync'd to the cluster using akv2k8s
- Nonsecret config will be via configmap (controlled via helm)

### Q: Are there any specific compliance requirements that need to be met?
[No answer provided]

### Q: Is there a process in place for security scanning of images?
Yes
- We use WIZ Container scanning at an EIP level to ensure no image with Critical or High vulnerabilities can be deployed.
- We also use Docker Hardened images to help reduce vulnerabilities.

## Networking & External Dependencies

### Q: Do the sites require access to external resources such as databases?
Potentially, though the vast majority of our resources are in Azure. The main exception is MongoDB Atlas.

### Q: Do the applications need to make any calls to external services or URLs?
Yes – there will be multiple services such as our CMS / SAP Customer Data Cloud / and other 3rd party integrations.

### Q: Are there any CDN requirements?
Yes
- All pages aside from auth'd pages will be heavily cached using Azure Front door.
- We will have a custom cache clearing and warming process to ensure it's always up to date with the latest changes from the CMS.

## Configuration & Deployment

### Q: Where is the configuration data for each site stored? Is it environment variables, configmaps or an external configuration store?
In a Github repo, either as configmaps / helm charts.

### Q: Is there a process in place for automated build and release of the images?
Yes, using Github Actions and a GitOps approach.

### Q: Are there any requirements for progressive deployments such as Blue/Green or Canary?
Not right now.

### Q: Is there any sort of feature flagging solution required?
Not right now.

## Data Management

### Q: Is there any data stored locally, does this data need to be persisted or backed up?
No, all data is either stored in 3rd party systems or in MongoDB Atlas.

### Q: Is there a requirement for local ephemeral storage? If so what size?
No.

### Q: Is there any performance requirement around the storage?
No.

## Observability & Monitoring

### Q: Do the sites expose any customer metrics that need to be collected?
- All customer metrics are tracked / handled via GTM and GA4.
- Some Realtime user monitoring metrics are handled via DataDog.

### Q: How is logging handled for each site?
DataDog AMP and observability is setup.

### Q: What level of monitoring for individual sites is required?
To be defined.

## Performance & Reliability
*Note: Additional information needed for this section*

### Q: What are the requirements for an uptime SLA?
[To be completed]

### Q: What are the requirements for RPO (Recovery Point Objective) & RTO (Recovery Time Objective)?
[To be completed]

### Q: Are there any requirements for response time and latency?
[To be completed]

## Testing & Quality Assurance

### Q: How is testing done prior to release?
[To be completed]

### Q: How are new releases signed off and pushed to production?
[To be completed]

### Q: What sort of testing and validation is done post release?
[To be completed]