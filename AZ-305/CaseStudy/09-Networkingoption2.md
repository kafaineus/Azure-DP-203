---
casestudy:
    title: 'Design a network solution -BI enterprise application'
    module: 'Network infrastructure solutions'
---
# Design a network infrastructure solution  

## Requirements

As the Tailwind Traders Enterprise IT team prepares to define the strategy to migrate some of company’s workloads to Azure, it must identify the required networking components and design a network infrastructure necessary to support them. Considering the global scope of its operations, Tailwind Traders will be using multiple Azure regions to host its applications. Most of these applications have dependencies on infrastructure and data services, which will also reside in Azure. Internal applications migrated to Azure must remain accessible to Tailwind Traders users. Internet-facing applications migrated to Azure must remain accessible to any external customer. 

## Key takeaways
    * Multiple regions are used to deploy the solution. 
    * Multiple data services native to Azure could be used in the solution.
    * There are 2 types of applications that are being migrated. Some are internal only apps that need connectivity from the on-premises locations, and external apps that are exposed to customers via the Internet. 
    
To put together the initial networking design, the Tailwind Traders Enterprise IT team chose two key applications, which are representative of the most common categories of workloads that are expected to be migrated to Azure.  

## Design - BI enterprise application 

![BI enterprise application architecture](media/compute.png)

##Key takeaways
    * IIS is used to host the applications. Clearly scaling these is a problem. The fact that servers are idle off-hours and can't handle load during peak usage suggests using an autoscaling solution when we move to Azure. 
    * The front end layer is separated from the API layer and can be hosted/scaled separate from the front-end tier. Since we have an API layer, the Azure solution can leverage capabilities like APIM (API Management). 
    * The database layer seems to have two types of use-cases - Transactional workloads that cater to the usage of the web-application, and data-warehouse type workloads since there us usage of SSAS (Sql Server Analysis Services).
    
-	An internal, Windows-based, three-tier business intelligence (BI) enterprise application with the front-end tier running IIS web servers, the middle tier hosting .NET Framework-based business logic, and the back-end tier implemented as a SQL Server Always On Availability Group database. 

-	This application is categorized as mission-critical and requires high availability provisions with the availability SLA of 99.99% and disaster recovery provisions, with 10-minute RPO and 2-hour RTO.

    * Since the SLAs are pretty high, we need to have a deployment in multiple regions. 
    * Azure Site Recovery with Recovery Services vault is a requirement to meet the RPO / RTO requirements. 
    * Since we have to support the database in 2 separate regions, we can use auto-failover groups and active geo-replication.
    * Since performance is a pretty high concern, we need to use the Business Critical Tier. 
    * The transactional workloads can be supported in the SQL Server as it is fit for purpose. It might be better to move the data from this transactional store to an analytical store to support analytical workloads. Azure Synapse might be a good candidate for this use-case. 
    
-	To provide connectivity to internal apps migrated to Azure, Tailwind Traders will need to establish hybrid connectivity from their on-premises datacenters. The Enterprise IT group already established that such connectivity will be implemented by using ExpressRoute circuit from its main Seattle datacenter, however, at this point it is not clear yet what would be failover solution in case that circuit becomes unavailable. The Tailwind Traders CFO wants to avoid paying for another, redundant ExpressRoute circuit. 

    * The on-premise network has been connected to Azure via the Express route circuit. 
    * As we can see from the diagram below, Express route can be setup to have a redundant VPN pathway. If the connection via Express route has an issue, then the VPN pathway will act as a redundant path for traffic. 
    * This is a simple case where the deployment is with one region. Since this is a more complex deployment where we need to support multiple regions with express route, we need to take a more elaborate approach as shown here https://learn.microsoft.com/en-us/azure/expressroute/designing-for-disaster-recovery-with-expressroute-privatepeering

![Express Route and VPN](media/expressroute-vpn-failover.png)

- There are additional considerations that apply to on-premises connectivity to internal apps migrated to Azure. Since the Tailwind Traders Azure environment will consist of multiple subscriptions and, effectively, multiple virtual networks, to minimize cost, it is important to minimize the number of Azure resources required to implement core networking capabilities. Such capabilities include hybrid connectivity to on-premises locations as well as traffic filtering. Incidentally, this need to minimize cost aligns with the Information Security and Risk requirements, which state that all traffic between on-premises locations and Azure virtual networks must flow via a single virtual network, which will be hosting components responsible for hybrid connectivity and traffic filtering. 

    * Azure recommends a hub-and-spoke architecture for network design. This is copied from the article 

    * Hub and spoke is a networking model for efficiently managing common communication or security requirements. It also helps avoid Azure subscription limitations. This model addresses the following concerns:

    * Saving on costs and efficient management: Centralize services that can be shared by multiple workloads, like network virtual appliances (NVAs) and DNS servers. With a single location for services, IT can minimize redundant resources and management effort.

    * Overcoming subscription limits: Large cloud-based workloads might require using more resources than a single Azure subscription contains. Peering workload virtual networks from different subscriptions to a central hub can overcome these limits. For more information, see Azure subscription limits.

    * Instituting a separation of concerns: You can deploy individual workloads between central IT teams and workload teams.
    * More info https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/hub-spoke-network-topology

![Hub and Spoke](media/network-hub-spoke-high-level.png)

-	As per requirements defined by the Tailwind Traders Information Security and Risk teams, all communication between Azure VMs in different tiers that are part of the same application must allow only the ports required to run and maintain the application. However, due to IP address space limitations, it might not be possible to allocate dedicated subnets to each tier. Enterprise IT group needs to identify the optimal way to configure source and destination for traffic filtering that would not require directly referencing IP addresses or IP address ranges.

    * Need to research on this requirement more.


## Tasks - BI enterprise application 

1. Design a 3-tier network solution for the BI Application. Your design could include Azure ExpressRoute, VPN Gateways, Application Gateways, Azure Firewall, and Azure Load Balancers. Your networking components should be grouped into virtual networks and network security groups should be considered. Be prepared to explain why you chose each component of the solution. 
    * Most of these components are used in the diagram below. Copied directly from Azure Arch center.

2. Based on your architect solution from the compute case study how would this impact the network design? Would you need any additional networking resources to secure access to the modernized application? Would you no longer need some of the recommended solutions implemented in your original network design? 

3. Based on your storage (relational) case study how would you update the network design to secure access to the storage account and ensure only select users have access to the storage account?
    * https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure?view=azuresql&tabs=azure-powershell
    * The high level steps to use AD to control access to specific groups is shown in this article. 
    * We set the admin for the SQL server, and then create users in the SQL server to reference the groups/roles in AD. 

4. Based on the modernizing of the SQL backend how do you plan to enable pragmatic access to the data base so that the front end has no hard coded secrets in its code base?
    * Managed idenities can be assigned to the VMs that are running the API layer and need access to the database layer. 
    * Since the same identity needs to be shared by multiple VMs we need to have a user managed identity that can be shared across these VMs

How are you incorporating the Well Architected Framework pillars to produce a high quality, stable, and efficient cloud architecture?


## Possible Solution

![Hub and Spoke](media/multi-region-sql-server.png)
