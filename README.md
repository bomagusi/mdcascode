# Microsoft Defender for Cloud as Code
The purpose of this project is to provide tools to enable automatic deployment of Microsoft defender for cloud through Github Actions. 
The project has several folders for each of the different Microsoft Defender for cloud  components that can be configured (**Onboard** **Plans**, **Multicloud** **Onboarding** **AWS** & **GCP**, **Workbooks**, **Workflow Automation**) plus folders for script helpers and Github Action YAML pipelines. 
In this README we explain some of the basics for each of them but we encourage you to visit each of the folders for more details on how to use the tools.

Microsoft Defender for Cloud (MDC) provides organizations with Cloud Security Posture Management (CSPM) and Cloud Workload Protection (CWP) capabilities for their Azure, multicloud and hybrid workloads. Top of mind for both Microsoft partners and customers is how to programmatically deploy and manage MDC. Some common pain points, we hear from Microsoft partners and customers, are the following:
1)	How can I automate onboarding of subscriptions to MDC?
2)	How can I enable Defender for Cloud plans across multiple Azure subscriptions and tenants at scale?
3)	As a Microsoft partner, how can I programmatically configure parts of MDC, like workflow automations, that can be leveraged in multiple customer deployments? 
This article aims to answer these questions and more. It teaches you how to programmatically deploy and manage MDC at scale using a variety of options, including:
	Infrastructure as Code
	Azure Policy
	REST APIs
	Azure CLI 
	PowerShell  
You might already be familiar with some of these options, as they’re commonly used for automating deployment and management tasks in Azure. Infrastructure as Code (IaC) is particularly interesting as it allows you to use Bicep, Azure Resource Manager (ARM), Terraform, and CloudFormation templates to describe the infrastructure of your cloud environment as code. There are benefits of adopting this approach, as you can use version control for your IaC templates. This means that whenever there is a change to your IaC templates, you’re able to track these changes using version control. You are free to use any version control that you like. To teach, in this article we use GitHub. 

In addition to tracking changes to your IaC templates, it’s important to test and deploy code from these templates to your Azure environment. This is widely referred to as Continuous Integration/Continuous Delivery (CI/CD). For CI/DC you can use DevOps tools and combine it with IaC templates and version control. The idea behind using this approach is to firstly use IaC to describe the desired state of your environment and put it under version control, in this case GitHub. Then whenever there is a change to IaC you put under version control, you can use DevOps tools to programmatically deploy these changes to your Azure environment. The CI/CD tool we use in this article is GitHub actions. 

Obviously, this approach can be applied in general to other Azure services too. However, in this article we teach you how manage and deploy Microsoft Defender for Cloud at scale, in combination with GitHub actions. We do this by guiding you through the following areas: 
 
1)	Pre-deployment best practice guidance 
2)	Guided inventory of the configurable components within MDC 
3)	Automated way of deploying MDC at scale  
4)	Leveraging DevOps automation for this process (GitHub Actions)
We recommend going one by one to understand how it works. 
1)	Pre-deployment best practice guidance
Before starting your MDC journey, consider the following best practice guidance:
	Ensuring that you have a Management Group hierarchy in the Azure environment according to the organization’s needs, to understand where Azure subscriptions are residing and what can you manage at scale in these subscriptions.
	MDC is a service that you enable on a subscription. With that being said, Management Groups help to more easily deploy MDC at scale. 
	Understanding Role-based access control (RBAC) and different roles available within MDC (adding here a link with more information for your reference). Security Reader is the least privilege role when it comes to consuming information from MDC. Security Admin is a role suited for users who enable components of MDC (i.e. perform enabling/disabling od Microsoft Defender for Cloud plans, dismissing of security alerts, etc.).  

After you consider this best practice guidance, you can proceed with exploring what components of MDC are configurable. 
2)	Guided inventory of the configurable components within MDC 
Defender for Cloud has a lot of capabilities that are mapped to different components you can configure. For simplicity’s sake in this article, we focus on configuring the following: 
Component	Possible to automate with
Onboarding subscriptions to MDC	REST API, Azure Policy, Azure CLI, PowerShell
Enabling Defender for Cloud plans	REST API, Azure Policy, Azure CLI, PowerShell
Connecting AWS/GCP environment  	REST API
Configuring Auto provisioning settings 	REST API, Azure Policy, PowerShell
Continuous Export 	REST API, Azure Policy, PowerShell
Workflow automation 	REST API

To configure these components, the focus of this article is how to do it at scale and automate it in a programmatic way.  
3)	Automated way of deploying MDC at scale
Before going into how to automate MDC deployment at scale, we want to touch briefly on the sequence of steps you generally need to take, when automating deployment of MDC. These are the following: 
Steps how to automate Defender for Cloud deployment in general:
1. Enable Microsoft Defender for Cloud on all subscriptions
o	This can be accomplished via the Portal, REST API, Azure CLI, PowerShell and Azure Policy.
2. Ensure Azure Security Benchmark is assigned
o	Please note that the Azure Security Benchmark assignment can be done at the Management Group level.
3. Enable Defender for Cloud plans (depending on your environment)
o	Please note this is a subscription level setting. 
4. Configure Auto-Provisioning settings 
o	This let’s MDC automatically provision the LA/AMA agent (and other extensions) to all the VMs in that subscription (including even Azure Arc connected machines connected to that subscription) to all the machines in the subscriptions. 
5. Onboard AWS and/or GCP environment to MDC (depending on your environment) 
o	Please note to onboard AWS or GCP environment to MDC, you need to create a security connector. The security connector is the representation of your AWS or GCP environment in MDC. 
6. Configure Continuous Export 
o	Please note this is a subscription level setting. 
7. Configure Workflow Automation 

Steps how to automate Defender for Cloud deployment at scale:
1. Enable Microsoft Defender for Cloud on all subscriptions: 
o	This is done by triggering the registration of the Microsoft.Security resource provider to all subscriptions, using one of the following options:

Automation option	Guidance
REST API	To enable MDC on a subscription, make a POST request to:

POST https://management.azure.com/subscriptions/{subscriptionId}
/providers/{resourceProviderNamespace}/register?
api-version=2021-04-01
Azure Policy	Use the following policy to enable MDC: 

Enable Azure Security Center on your subscription

Azure CLI 	Use the following Azure CLI command:

az provider register --namespace 'Microsoft.Security'
Azure PowerShell	Use the following PowerShell command:

Register-AzResourceProvider 
-ProviderNamespace 'Microsoft.Security'

2. Ensure Azure Security Benchmark is assigned: 
o	Assigning Azure Security Benchmark policy initiative can happen at different scopes, i.e. on the Management Group scope or on the individual subscription scope. The Management Group scope is recommended for organizations who want to centrally control the Azure Security Benchmark policy initiative (adding here a link with guidance for your reference). All subscriptions within the management group automatically inherit the policies applied to the management group. In addition, each Azure tenant is given a single-level management group called root management group. This root management group is built into the hierarchy to have all management groups and subscriptions fold up to it. This group allows global policies to the applied to it, i.e. ensuring that any newly created subscription is onboarded onto Microsoft Defender for Cloud (adding a link here with guidance for your reference). Assigning the Azure Security Benchmark policy initiative to a Management Group can be done using several automation options (i.e., ARM template, Azure REST API, CLI or PowerShell – adding a link here with guidance for your reference).

Automation option	Guidance
REST API	To assign the Azure Security Benchmark policy initiative, make a PUT request to:

https://management.azure.com/{scope}/providers/
Microsoft.Authorization/policyAssignments/{policyAssignmentName}
?api-version=2020-09-01 with the following request body

with the following request body:

{
  "properties": {
    "policyDefinitionId": "/providers/Microsoft.Authorization/policySetDefinitions
/1f3afdf9-d0c9-4c3d-847f-89da613e70a8",
  }
}
Azure CLI 	Use the following Azure CLI command:

az policy assignment create --name ‘ASCDefault’ –policy-set-definition 1f3afdf9-d0c9-4c3d-847f-89da613e70a8
Azure PowerShell	Use the following Azure PowerShell command: 

$definition = Get-AzPolicySetDefinition -Name 1f3afdf9-d0c9-4c3d-847f-89da613e70a8 
New-AzPolicyAssignment -Name 'ASCDefault' 
-PolicySetDefinition $definition
ARM Template	To create a Microsoft.Authorization/policyAssignments resource, add the following JSON to the resources section of your ARM template:

{  
  "type": " Microsoft.Authorization/policyAssignments",  
  "name": "ASCDefault",  
  "apiVersion": "2019-09-01",  
  "properties": {  
    "policyDefinitionId": "1f3afdf9-d0c9-4c3d-847f-89da613e70a8"  
  }  
}

3. Enable Microsoft Defender for Cloud plans:
o	This is done by changing the pricing tier from “Free” to “Standard” for the particular Microsoft Defender for Cloud plan that you would like to enable. 

Automation option	Guidance
REST API	To enable MDC using it’s REST API, make a PUT request with the following request body:
{
  "properties": {
    "pricingTier": "Standard"
  }
}
for the following URL and add the relevant subscription ID and the pricingName: 
https://management.azure.com/subscriptions/{subscriptionId}
/providers/Microsoft.Security/pricings/{pricingName}
?api-version=2018-06-01

Note that {pricingName} can be any of the following: VirtualMachines, SqlServers, AppServices, StorageAccounts, SqlServerVirtualMachines, Containers, KeyVaults, Dns, Arm, etc.
Azure Policy	Use the following policies to enable the particular Defender for Cloud plans: 

	Azure Defender for servers should be enabled
	Azure Defender for App Service should be enabled
	Azure Defender for Azure SQL Database servers should be enabled
	Azure Defender for SQL servers on machines should be enabled
	Azure Defender for open-source relational databases should be enabled
	Azure Defender for Storage should be enabled
	Azure Defender for Containers should be enabled
	Azure Defender for Key Vault should be enabled
	Azure Defender for Resource Manager should be enabled
	Azure Defender for DNS should be enabled

Azure CLI 	Use the following Azure CLI command to enable MDC, e.g. for virtual machines:

az security pricing create -n VirtualMachines --tier 'standard'
Azure PowerShell	Use the following Azure PowerShell command to enable MDC, e.g. for virtual machines:

Set-AzSecurityPricing -Name "virtualmachines" -PricingTier "Standard"
ARM Template	To create a Microsoft.Security/pricings resource, add the following JSON to the resources section of your ARM template, e.g for virtual machines:

{
  "name": " VirtualMachines",
  "type": "Microsoft.Security/pricings",
  "apiVersion": "2018-06-01",
  "properties": {
     "pricingTier": "Standard"
  }
}

-	4. Configure Auto-provisioning settings:

o	This is done either through the Azure portal or by leveraging Azure Policy: 

Agent/Extension	Azure Policy
Log Analytics agent	Enable Security Center’s auto provisioning of the Log Analytics agent on your subscriptions with default workspace

Enable Security Center’s auto provisioning of the Log Analytics agent on your subscriptions with custom workspace

Log Analytics agent for Azure Arc machines	Configure Log Analytics extension on Azure Arc enabled Linux servers

Configure Log Analytics extension on Azure Arc enabled Windows servers

Vulnerability assessment for machines	[Preview]: Configure machines to receive a vulnerability asssessment provider

Guest configuration agent	Deploy prerequisites to enable Guest Configuration policies on virtual machines

Microsoft Defender for Containers components	[Preview]: Configure Azure Kubernetes Service clusters to enable Defender profile

[Preview]: Configure Azure Arc enabled Kubernetes clusters to install Azure Defender's extension

Deploy Azure Policy Add-on to Azure Kubernetes Service clusters

[Preview]: Configure Azure Arc enabled Kubernetes clusters to install the Azure Policy extension

	
5. Onboard AWS and/or GCP environment to MDC (depending on your environment)

o	This is done by leveraging the REST API: 

Automation option	Azure Policy
REST API	To create a security connector for AWS/GCP, make a PUT request to:  

https://management.azure.com/subscriptions/{subscriptionId}
/providers/Microsoft.Security/connectors/{connectorName}
?api-version=2020-01-01-preview

An example of the request body can be found here.


6. Configure Continuous Export 

o	This is done by leveraging one of the following options: 

Automation option	Azure Policy
Azure Policy	Deploy export to Log Analytics workspace for Azure Security Center alerts and recommendations

REST API	To configure Continuous Export, make PUT request to:  

https://management.azure.com/subscriptions/{subscriptionId}
/resourceGroups/{resourceGroupName}/providers/
Microsoft.Security/automations/{automationName}
?api-version=2019-01-01-preview

An example of the request body can be found here.


7. Configure Workflow Automation 

o	This is done by either leveraging Azure Policy or the REST API: 

Automation option	Azure Policy
Azure Policy	Deploy Workflow Automation for alerts

Deploy Workflow Automation for recommendations

REST API	To create a security connector for AWS/GCP, make a PUT request to:  

https://management.azure.com/subscriptions/{subscriptionId}
/providers/Microsoft.Security/connectors/{connectorName}
?api-version=2020-01-01-preview

An example of the request body can be found here.


