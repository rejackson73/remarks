name: terraform-user-management
class: title, shelf, no-footer, fullbleed
background-image: url(https://hashicorp.github.io/field-workshops-assets/assets/bkgs/HashiCorp-Title-bkg.jpeg)
count: false

# Terraform User Management
## Access, Permissions, and Integrations

![:scale 15%](https://hashicorp.github.io/field-workshops-assets/assets/logos/logo_terraform.png)

???
Of course any implementation of a tool like Terraform, in an enterprise environment, requires some sort of single signon user managemnet.

---
layout: true

.footer[
- Copyright © 2019 HashiCorp
- ![:scale 100%](https://hashicorp.github.io/field-workshops-assets/assets/logos/HashiCorp_Icon_Black.svg)
]

---
name: terraform-slides-link
# The Slide Show
## You can follow along on your own computer at this link:
### tbd

???
Here is a link to the slides so you can follow along, but please don't look ahead!

---
name: Notes
Full support for Security Assertion Markup Language (SAML) 2.0
Can operate as a Service Provider/Relay Provider or integrate with a third party identity provider (idP)
Terraform has the concepts of teams, with policies tied to each team
Teams can be managed within TFE or mapped using SAML 

Need attributes to identify username, site admin designation, and team attribute name (memberof)





---
name: What is a Module
# What is a Module
Within programming, large, complex programs are broken out in to functions and reusable libraries.
<br>
Infrastructure should be structured similarly, with reusable Modules.
![:scale 100%](images/ProgramFunctionCalls.png)
???
Modules in Terraform Infrastructure are very similar to functions or subroutines in programming languages.  Self contained packages of Terraform code that can be reused across multiple workspaces.

---
name: What is a Module - 2
# What is a Module (2)
* Abstration layer enabling easy access to bundles of resources
* Developed by HashiCorp, Community, or Internally
* Available through a private registry, or public registry (registry.terraform.io)
<br>
## A Module is NOT
* A thin wrapper around an existing resource
???
Modules provide an abstraction layer for Terraform operators and users to create groups of resources in a similar, predefined, method.  The Terraform registry provides thousands of modules eveloped internally or through the community.  You can even write and submit your own.  Note that private module registries are accessible to all within an organization, similar to most code repository structures.  Do not create a module simply to deploy a single resource, this is better performed using variables.

---
name: Module Details
# Module Details
Module details are available through the registry (public modules) or the Terraform UI for Private Modules
<br>
# Participation!
* Go to https://registry.terraform.io
* Find a module of interest
* Find the version of the module, inputs, and outputs

???
We're going to be using the public registry, however, for most instances and larger companies a private registry is more beneficial.  There may be specific resources, bundles, version, or other constraints to be defined for internal usage.  Question the students about their module and relevant information.  Open discussion about usage of private modules.

---
name: Module Details - 2
#  Module Details
Images of public and private module details
.left-side[![:scale 95%](images/ModuleDetails.png)]
.right-side[![:scale 100%](images/PrivateModuleDetails.png)]



???
Here you can see a public module and a private module, side by side. Can you spot the differences?  Note the source.  The structure of the Private Module Repository is managed by the code repository - bitbucket in this case.

---
name: Module Structure
# Module Structure
.left-column[
* Similar to the main code, with a 'main,' 'variables,'  and 'outputs.'
* Each module should include a README describing usage
* Each module should include a version
* Most modules require input variables
* Modules can call other modules, recommended not to go more than 2 levels deep
]
.right-column[![:scale 100%](images/ModuleStructure.png)]
???
Modules are defined very similarly to core Terraform code. Actually, several people refer to the core of the Infrastructure as the 'Core Module'.  Each module should include a main tf file, a variables tf file, and an outputs file.  The readme file should include details regarding the appropriate version and necessary input variables.

---
name: Configuration Designer
# Configuration Designer
Massive time saver for infrastructure definition
* Outline a configuration for a new workspace using predefined modules
* Define versions and variables using the Terraform UI
* Generate the core module 'main.tf' to add to a new workspace and repository

???
The Configuration Designer is a huge benefit for the Private Module Registry.  Search and select modules for your infrastructure, set versions and variables, and create your main.tf.  Note that you MUST save your main.tf file off, and manually create your workspace.  You still have to do something to earn your pay.

---
name: Configuration Designer - 2
#  Configuration Designer
Images of Configuration Designer
.left-side[![:scale 95%](images/ConfigurationDesigner1.png)]
.right-side[![:scale 100%](images/ConfigurationDesigner2.png)]



???
here are some screen captures for the configuration designer, all through the Private Terraform UI.  Here is where you filter and select your modules, version, and variables, ultimately to create your main terraform file.

---
name: Configuration Designer - 3
#  Configuration Designer
Images of Configuration Designer Code Creation
![:scale 80%](images/ConfigurationDesigner3.png)]


???
Ultimately you'll end up with your terraform output that you can use to create your own workspace and infrastructure.  Several large companies are doing this to make it easier to self service infrastructure.

---
name: References
#  References and Follow Up

https://www.terraform.io/docs/registry/index.html
https://learn.hashicorp.com/terraform/getting-started/modules

???
Terraform documentation for the registry services.