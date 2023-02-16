# Day 1

## Opening remarks
1:00 pm → 5 min


## How Terraform Helps Us Catch Bad Guys at Sea
1:05 pm → 30 min, Jonathann Zenou; Terraform 

The company is called Windward, AI platform for maritime data & insights.
They take data and perform it into insights.
Data to Entities, Entities to Voyages, Voyages to Patterns, Pattern into Recommendations.

Example ship name ELOIKA, a cargo ship.
The ship needs to have a transponder, that reports the location of the ship.
They see when transponder is seeing and when not.
They also see the identify (ownership) of the ship
If they see gaps and identity changes.
This might signal shady behavior but could also be regular behavior.
But checking the Geo location history it shows a high probability of fraudulant behavior. 

DL Model life cycle: Model deployment, project creation, micro service and queue architecture.

Model development: problem from product, underatand the problem, investigate in jupyter node books, identify pain points, data and usecases, check for existing models.
In case of deep learning use case a GPU will be required.
Issue with GPU: very expensive, spot instances are rare.
Solution make efficient use of the GPU.
Share the env and pause the instances in case of no usage.
They used saltstack in the initial approach.
It had a lot of limitation, especially issues with the EBS disks.
They introduced terraform.
Using self build AMI, pod templates and IAM roles.
Monitoring of this is also managed with terraform.
Also managed: Route53.

Project creation: create github repo from repo template, configure repo, add Jenkins pipelines.
They use the Jenkins provider to manage Jenkins resources.
They also use the Github provider.
They manage protection rules of repos with this provider.
They have ~600 repos.

Micro Services and Queuing Architecture.
They use eks, helm, prometheus, karpenter and keda.sh.
Keda is an event-driven auto scaller.
200 actives queues.
60 topics.
All monitored with cloud watch, managed with terraform.
The whole setup changes a lot, so the devops team as a QA bottleneck caused problems.
So they build a json file to generate resources based on this.
Now developers can change queues without knowing terraform.
Jenkins job applies to after QA from DevOps.
The Jenkins job can not run destroys, as a safety measure.

They are checking env0 right now, to mange the tf infrastructure.
Terraform enabled them to manage the infra with less errors and in a fast fashion.

(Hashidays are coming up, European event)

## An Introduction to "Packering"
1:35 pm → 30 min, Tom Howarth; Packer 

Packer is like terraform for machine images.
You can use json or HCL2 to configure.
Packer has not state files like terraform.
With source blocks you define the image.
With build blocks you setup additional steps like removal of packages or users.
HCP (Hash Cloud Platform).
It's demoed how the HCP Packer can be utilized.
(Not quite sure what he's doing there.)


## The Ups and Downs of Maintaining a Terraform Provider
2:05 pm → 30 min, Martin Wentzel; Terraform 

They wanted to implement something with terraform and found a bug in a terraform resource.
What's next?
Found a bug in tf what shall I do now?
Go to github and report the bug.

Talk will be about how to develop a provider, testing & ci/cd, feature development.

Why do we need a docker provider?
Not everyone is working with managed docker providers.
In development since 2015.

First understand terraform state.
Easy case: Normal container.
Setup Containers.
Containers to setup things and terminate after the work.
This does not match the regular case of a terraform resource.
Would be reported as missing on the next plan run.
Image pulls also do not match to regular state.

`watch docker inspect $ID`.
Match results to the tf state.
The docker daemon is the backend in this architecture.
The Client is the terraform provider.
They communicate via a client package, that wraps around the api to make it simpler to communicate.
The different version of the daemon make it a hard problem.
Checking API versions or Buildkit support.
Practically re-implementing docker cli.
Why not wrap the docker cli?
Advantage not having a dependency on the cli!
Mappers are required in both cases.
The client package give better control thus better performance.

Rushed decisions.
Talks about some decisions that are not too interesting.

CI/CD.
Client server testing is easy because runner already have a docker daemon.
Only additional dependencies: https & http registry.
Issues of long running jobs (15 min).
Used matrix build to run tests in parallel base on the suite and multiple terraform version.
Now 9min builds.

Supporting podman is hard.
The triangle: Usefulness, Complexity, Support-able.
Podman: 3 resources not supported by podman, 5 resources tests where failing, only 2 resources work fine.
So complexity is higher than expected.


## Zero Trust Security with Boundary and Vault
2:35 pm → 30 min, Japneet Sahni; Boundary Vault 

Works at Arctiq.

How to setup Zero Trust?
Traditional PAM: 
operator has ssh key, vpn credentials, app credentials, ip address.
Connect to bastion host/vpn.
Possible firewall to restrict traffic.
Connection to apps with credentials.
A bunch of information is needed by the user.
Credentials & Host names need to be known.

Challenges: On and Off boarding.
Key rotations.
Once the access is granted a wide access is granted in most cases.
Dynamic environments are hard to know ips.
Application credentials are expose to the user e.g. DB credentials.
Long-lived credentials are a high risk.
How to move away from static credentials?
Dynamic credentials, but how to do this?

Boundary + Vault.
Every login with IDP.
So management only in the IDP.
Authorization base on roles.
Roles are defined on a logical service.
Not bound to a physical service instance.
No bridging into the private network.
Integration with a KMS to generate short lived credentials.
Short TTL or OTP.

Boundary worker used for the proxy connection to the target service.

Control Plane and Workers.

Credentials Brokering: Boundary hand credentials to the client.

Credentials Injections: only in HCP, the credentials are hidden from the client.

Logical Components:
Global Scopes for admins.
Organizations scope.
Auth methods - IDP.
User/Groups - Principals.
Roles - Authorization.
Project scope to mange target and hosts.
Target are logical services.
Host makes up a target.
Host Catalog - contains hosts, can be static and dynamic.
Populated by tags in a dynamic case.
AWS and Azure supported.
Credential Store to manage credentials with credentials libraries.
Boundary Desktop App to help the end user to connect to boundary.

Use Cases.
1. Accessing Domain Joined Windows Servers using OpenLDAP secret engine.
Authenticate with Boundary using Azure Active Directory.
Select target.
Valut will use OpenLDAP secret engine to get the passwords.
OpenLDAP is domain joined with the service.
Boundary create proxy connection to the service.

2. Access Linux server with local accounts using ssh-otp secret engine.
Boundary login with IDP.
Vault will return OPT and connect to Linux server.
The agent on the linux server with authenticate OTP with Vault.

Boundary has some nice admin ui.

Connecting to a target will present a proxy URL and an OTP.
So you use SSH to connect to the server.
Sessions are logged in boundary.

All the vault and boundary config can be done with terraform.

Credential Templating can be used to not manage a secret per user in the credential library.


## Replatforming with Minimal Drama and Downtime
3:05 pm → 30 min, Elijah Voigt; Nomad 

Works at Lob.

Talks should function as an inspiration story.
Lob is a physical mail sending api provider.
70 services with mixed workloads, serverless, batch or micro services.

Started with a SASS product that gave an abstraction AWS.
Issues where discoverability (hard to understand, because of the wrapper), flexibility (fixed to aws), scalability (only supported ec2 workloads on all instances).

Replatforming with Nomad.
Build one platform for current and future services, flexible operations to meet service and cost demands.
Zero downtime and minor velocity impact.

Architecture migration took 6 month.
Did a testing run on dev an it worked out fine.

Architecture: AWS for everything here.
Route53 and ALB route trafix to Nomand Clients.
Clients have a traefik side car and a consul agent.
Consul is used as a service catalog.
Consul Server also used for orchestration.

Migration Plan:
Service by Service Migration, so gradual migration.
Focus on Lift & Shift, change as little as possible.
Traffic Shaping Customer Traffic at the load-balancer level to send some % of traffic to the new services.
Tune Scaling and Deploys.

Traffic Shaping is scoped to the loadbalancer.
Also provided a route without traffic shaping to isolate issues of the env.

Curated Experience.
Created a role the migration ambassadors per team.
This person was direct contact before, during and after the migration.
Create organization specific docs tailored for common and quirky use-cases, that show which part of nomad are relevant.
Build auto transformer to migrate old configuration to nomad.
Keep people informed on updates and progress.

The long Tail.
Start with some quick wins.
Tackle big fish halfway.
People will not get nervous if you do not keep it till the end.
It will take longer than expected.
Capture progress and wins.

How did it go?
95% migrated.
Mostly positive feedback.
21 Incidents/Downtimes. 
10 due to misconfigurations, 7 cluster misconfigurations, 4 operator errors.
All in the error budget.


## Writing Your First Waypoint Deploy Plugin
3:35 pm → 30 min, Bram Vogelaar, Fokke Dekker; Waypoint 

## Scale Your Cloud Network to Infinity and Beyond
4:05 pm → 30 min, Du'An Lightfoot; Terraform 

## Designing for Equity
4:35 pm → 30 min, Jasmine Whaley; Community Culture Other 

## Break
5:05 pm → 10 min

## Terraform: Don't Reinvent the Modules
5:15 pm → 30 min, Lays Rodrigues; Terraform 

## Advanced Terraform Techniques
5:45 pm → 30 min, John McDonough; Terraform 

## Building Scalable Enterprise Secrets Management with GitHub OIDC and HashiCorp Vault
6:15 pm → 30 min, Ari Kalfus; Vault 

## Running Cilium with Nomad
6:45 pm → 30 min, Dan Norris; Consul Nomad 

## Multi-Cloud WAN Federation with Consul and Kubernetes
7:15 pm → 30 min, Jacob Mammoliti; Consul 

## Multi-Cloud Image Pipeline
7:45 pm → 30 min, Marcelo Zambrana; Packer 

## Vault and Boundary - Managing Secrets at Home
8:15 pm → 30 min, Michael Greenlaw; Boundary Terraform Vault 

## Hidden Hazards: Unique Burnout Risks in Tech
8:45 pm → 30 min, Allessandria Polizzi; Community Culture Other 

## Break
9:15 pm → 10 min

## Enhancing Platform Teams Workflow with Infrastructure as Code
9:25 pm → 30 min, Sivamuthu Kumar; Terraform 

## How to Nomadify Your Kubernetes Manifests
9:55 pm → 30 min, Adriana Villela; Nomad 

## Autoscaling Workers for Boundary
10:25 pm → 30 min, Ned Bellavance; Boundary 

## Secure Developer Workflows with Vault & Github Actions
10:55 pm → 30 min, Kartik Lunkad; Vault 

## Why You Should Use Vault as Your Consul Certificate Authority
11:25 pm → 30 min, Thomas Kula; Consul 

## Fast-track Your Kubernetes Journey with EKS BluePrints for Terraform and Waypoint
11:55 pm → 30 min, Juan Peredo; Terraform Waypoint 

## Open Policy Agent in Terraform Cloud
12:25 am → 30 min, Peter ONeill; Terraform 

## Scaling HashiCorp's Writing Culture with Internal Tooling
12:55 am → 30 min, Anubhav Mishra, Josh Freda; Community Culture Other 
