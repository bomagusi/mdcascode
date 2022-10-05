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
