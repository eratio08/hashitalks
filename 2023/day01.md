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

Work at seaplane.io

Waypoint: modern workflow to release across platforms.
Can be extended with plugins.

Seaplane - a multi-cloud platform with a global network.
Abstract away all the difficulties of a global deployment.

Use case a simple web app in python.

Waypoint has 3 Steps: Build to build the container and push it, Deploy the app, Release.

There is a plugin template that is provided on github.

3 Maturity levels of plugins.
1 - Local execution form HCL File
2 - Local execution from plugin framework
3 - Run on the platform in the waypoint framework

Step 1 - ??? (did not get was this was about).
Step 2 (still hard to get what this is all about).
The whole topic is hard to understand without knowing what waypoint does.

So they implemented a plugin so you can use waypoint to deploy to seaplane.
The details are not too interesting.

Waypoint has a nice UI.

They show a demo.
`waypoint init` and `waypoint up`.
The app will be deployed to seaplane.
`waypoint destroy` to remove it again.

Complex deployments to seaplane are not yet supported.


## Scale Your Cloud Network to Infinity and Beyond
4:05 pm → 30 min, Du'An Lightfoot; Terraform 

AWS.

What's a VPC: Virtual network in the cloud.
Mimics a traditional network.
How many VPC are needed?
Related to accounts and environments.
How to go from 1 VPC to multi VPC?
Larger VPC with large accounts vs smaller vpc and smaller accounts.
Less accounts and networks to manage.
Tighter controls within the account or vps.
On smaller accounts more accounts to setup and more infrastructure.
Big accounts make billing complex and blat radius is bigger.
Smaller accounts are billed easily and have a way smaller blast radius.

Interconnecting VPC with peering.
Point to point connection, with acceptor and requester.
Routing needs to be defined on both VPC.
Transitive routing is not supported.
VPC can not have overlapping IPs.
Can not use jumbo frame and multiple paring between the same vpcs.

Transient Gateway: regional virtual router for traffic flowing between vpc.
Hub and spoke style.
Before transitive gateway you would need a full mesh n:n peerings/
The Transit Gateway can be used to reduce to a central router.

Cloud9 a cloud base IDE is used for the demo.
The demo shows some multi vpc setup that is connected via a transient gateway.
There is a VPC modules by AWS with best practices.
https://registry.terraform.io/browse/modules?provider=aws

## Self-Service IOC: People, process, product
Juan Herreros Elorza, Banking Circles

Issue: We need an environment, this one guy know how, there is a script that does it.
Instructions where hand written notes, great.
DevOps: Devops is the unioin of peoples, process and product.
The original process:
Peoples where waiting too much, devops was doing to much, communication was too late.
Process - ops team was the bottleneck. 
Different solutions to the same problem.
Environment drifts.

Add devops to the teams.
High cognitive load for devops.
More points of communications.

Goal: move from sequential process to a parallel process.
Book - Team Topologies.
Mentions Platform team.

Shared re-usable self-service building blocks.
Blocks are easy to use, standard, compliant.
DevOps (platform) team provides, maintains and supports blocks.
Dev use the blocks.

Terraform modules & Azure DevOps pipeline templates.

The project team take modules from the shelf and runs it in a pipeline (instance of a pipeline template).

Use `terratest` to test the blocks and instances of the blocks.

Projects have a governance model.
Owner/maintainer, contributor and user.

> Do the same things in the same way.

They use https://www.checkov.io for static code analysis.

They use a terraform module repository to make it version safe.
They generate documentation with `terraform-docs` and `Mkdocs`.


## Break
5:05 pm → 10 min


## Designing for Equity
4:35 pm → 30 min, Jasmine Whaley; Community Culture Other 

was skipped


## Terraform: Don't Reinvent the Modules
5:15 pm → 30 min, Lays Rodrigues; Terraform 

Works at Pagar.me.

Also advocated for using official AWS modules.
Benefits are they are open source and implemented by aws with best practices.
The modules are nicely documented and have examples.
Wide spread usage.


## Advanced Terraform Techniques
5:45 pm → 30 min, John McDonough; Terraform 

Works at fortinet.

Its about: code reuse, data=driven code, for_each > count, implicit dependencies.

code reuse, he uses git submodules.
A submodule is a repository referenced by the current repo.
In github submodules are shown with the referenced commit hash.
Kepp generic resource code in a git repository.
Resource module is a distinct object.

`git submodule add <remote_url> <destination>`.
`git commit -m "added submodule"`
`git push`

To update the submodule:
...

How does a generic resource look like.
Variabalize every props and reuse the name of the original prop.

Have a module to use the generic resource and use for_each to loop over locals to create actual resources.

Locals + Generic Resources = data-driven code.
Only change locals to change the infrastructure.
Easy to reuse in different environments.

He stresses the might of the `for_each` to create NxM loops.
Use `setproduct` to create a list of NxM and loop over with `for_each`.

`count` use for conditionals, and for sized.
Count uses indices for reference but for_each uses key name. 
Don't use count if you can use for_each.

You can use for_each for conditional with the `if` condition.
Or use the ternary to loop over empty loop in the false case.
Use for_each with dynamic blocks.

Implicit dependencies.
This enforces ordering during plan.
Easier apply and destroy.

Search Github if we can find the referenced examples.
github.com/movingalog


## Building Scalable Enterprise Secrets Management with GitHub OIDC and HashiCorp Vault
6:15 pm → 30 min, Ari Kalfus; Vault 

Is security at digital ocean.

Manage secrets is hard.
In the use case of github actions this is a nice approach.
Secret Zero problem.
Where do you store the secret to the secret store.
What to do if the env is compromised.
Do not use github secretes.

OIDC solves the management complexity.

Github Secretes issues.
No lifecycle or rotation.
Granularity is coarse.
Auditing seems hard/missing.
What if one workflow is compromised.

OIDC is ephemeral to each run.
Lot more options than per-repo.
Great audibility.
Enables very short TTL/otp.

Vault can bind a role to any github oidc claim.
So a very fine granular is possible.

How to manage these new create roles at scale?
Reduce developer friction.
Provide a paved path for developers.
Secrets access should not be a hassle.
Solve developers concerns.
Have actions to set global default in github workflows.
Do not expose OIDC claim construction to the developers.
Possibly generate workflow files.

Try not to use the actor field for auditing.
Rather use user_claim.
The vault audit logs now show actionable information.

They published the actions in their repo with a DIY course.
There is also an article in the DO Blog.


## Running Cilium with Nomad
6:45 pm → 30 min, Dan Norris; Consul Nomad 

CNI Container Netwwork Interface: spec and libraries for plugins that configure networking for linux containers.
Allows for fine grained control over how tasks are networked.
They use CNI with Firecracker for Nomad.

Was is cilium.
CNI plugin to use eBPF.
eBPF is like a virtual machine in the kernel.
You can run a hook to every packet to a machine.
This is was cilium does.
Hubble is part of cilium and allows to inspect every egress and ingress traffic.
Cilium allows for network policies at L3,L4 and L7.
Can restrict service to service accessibility.
Can limit on CIDR or DNS and more.
Effectively a programmable firewall for containers.
Can use wireguard to encryption, so no TLS needed.

Cilium only works with K8s.
Most bookkeeping of cilium is done by a K8s operator.
So Nomad is not supported directly.
Nomad agents only finger prints during startup.
So Cilium needs to running before Nomad starts.

Cilium needs to run on every node.
They run cilium as docker containers.
The docker bridge needs a different cidr for this to work.
Endure all base CNI plugin are installed and provide a config file for cilium.
Insure to run cilium everywhere.
They use systemd to archive this.
Consul is required here as well and cilium has to use it.

They've written `netreap` to clean up ip allocations and sync policies.

A single cilium cluster manages single subnet across all node in the cluster.
netreap check that no ip spaces are exhausted.
It also removes stale agent from the cilium cluster.

Cilium uses CRDs to network policies in K8s.
In Nomad netreap can handle this.

Demo: shows network policies restrictions.
Egress policies are required because it's restrict all by default.


## Multi-Cloud WAN Federation with Consul and Kubernetes
7:15 pm → 30 min, Jacob Mammoliti; Consul 

What does it mean to be multi-cloud?
To use the service of at least 2 cloud providers.
The benefits are to use the best of each cloud.

Consul and Service Discovery.
To avoid management of ip or local dns names use consul.
Consul connect can be used for service to service connection (service mesh).
It uses envoy proxy cross clouds.
Manage traffic, app security also adds observability.

Consul was WAN federation to join networks in different clouds. 
It supports multi cloud fail-over.

The Demo shows cross cloud access with GCP and Azure.


## Multi-Cloud Image Pipeline
7:45 pm → 30 min, Marcelo Zambrana; Packer 

Multi Cloud image factories.
This images can be crafted with Packer.
As there is no standardization so packer can help here.
You can use github workflows and HCP to archive this.

Packer plugins help to support multi cloud images.

## Go build plugin for Vagrant
By Sphia Castellarin, Video from 2022 

Shows how to write go plugin for Vagrant.
Not so interesting...


## Vault and Boundary - Managing Secrets at Home
8:15 pm → 30 min, Michael Greenlaw; Boundary Terraform Vault 

From MonoByte.

HashiPass, a simple password management system.
For at-home use.
It should be cheap, simple, scalable secure and have zero trust.

Initial design. VPC, EKS, ALB, Bastion, Vault.
Bastion for SSH access.
Vault running in EKS.
(Example uses the AWS VPC/EKS module).

The whole setup is not simple as wished.

Now use Auth0 and boundary.
Ideally support MFA.
As boundary is managed via terraform.

It bit over powered for an at-home setup.


## Bootstraping Terraform Secrets with the 1Password CLI
Julian Morgan, 1 Password

The problem infrastructure requires credentials for management.
Envfile or tf var files are a risk.

Use 1Password cli to use if required.
Replace passwords in env files with a reference to the secret in 1 password.
Any changes to credentials will be synced to the team and version via source contrail.


`op-run --env-files tf apply` to replace reference on apply.


## Hidden Hazards: Unique Burnout Risks in Tech
8:45 pm → 30 min, Allessandria Polizzi; Community Culture Other 

They advocate for burnout resilience and related topics.

Covid, Was and other stuff have a negative impact.
2 out of 5 tech worker has experienced symptoms of burn out. 
42% tech workers anticipating leaving work due to burn out.
62% are concerned about the future.

Causes for burn out: Workload, absents of Control, In-Security, not aligned Values, not being Valued, having role clarity Clarity.

These are drivers but the actual cause is very personal.

Symptoms: Exhaustion, Cynicism, Cognition.
Cognition - things that we where good at seem to get more complicated.

5 Hazardous for tech:
* Exhaustion - 56% have experienced it.
Think about it: How often do you struggle with letting go?
Change Curve.
How people experience changes.
Shock Denial Frustration Depression Experimentation Decision Integration.
There can be change fatigue.
To many changes to not recover from all.
Signs: Questioning Value, Apathy (less emotional investment), Info Hoarding (I don't want to share information), Cynicism, Decision Making (struggling to make a rescission).
* Constant Threats, Cyber attacks can cause PTSD.
* People make things messy.
* Impostor Syndrome.

What can you do?
Building courage: 
* Connect 
* Relate - understanding, personal, purposeful connections
* Collaborate - give / take, strengths, balanced voices, create history
* Support - empathy, solidarity, appreciation, sharing, assisting 


## Break
9:15 pm → 10 min


## Enhancing Platform Teams Workflow with Infrastructure as Code
9:25 pm → 30 min, Sivamuthu Kumar; Terraform 

Platform Engineering: prative of building & maintaining the infrastructure, tools and processes to support an organization's technology platform.
Focus on providing a seamless and efficient DX that enables teams to focus on delivering business value.

App Dev teams base on Platform Team, which base on the SRE Team.

Platform Team provide reusable modules, pipelines and tools.
Monitor the infrastructure for performance & issues.
Updating Infrastructure.

Automating Self-Serv Platform Request with Github IssueOps.
Infrastructure changes are automated by opening issues in github.
Uses github action to listen to github events.

Demo.
It uses Issue Forms.
Creating a new Issue will show issue templates base on what you need.
The issue forms make it very easy to get all required information.
He request the provisions of an EKS cluster with an issue.
The planner will check it.
A platform team member needs to approve and then the bot will apply the changes.


## How to Nomadify Your Kubernetes Manifests
9:55 pm → 30 min, Adriana Villela; Nomad 

Terminologies:
Container Runtime = Task Drive.
Pod = Task Group.
Container = Task.
Manifest = Jobspec.
K8s deployment = Nomad job.
Service = Service.

Conversion Process.
K8s Manigest to JobSpec.
Take a JobSpec Template and fill ports, services and tasks definitions.

The example an OTEL demo app.
Grab the Manifest from the OTEL repo.
Deployment Definition and the Service Definition.
JobSpec is crafted from this.
Set service nama.
Set network spec to expose ports.
Set the service definition.
One Service per port, one http, one grpc.
Service can be auto register to consul or nomad.
Expose the service with traefik by tagging the service.
Define health checks.

Now define the task.
This task uses the docker driver.
Image and ports are declared.

(stopped following at this point, as it very detailed about how to write the files)


## Autoscaling Workers for Boundary
10:25 pm → 30 min, Ned Bellavance; Boundary 

HasiCorp Ambassadors. NedInTheCloud.com

The history of boundary.
Getting rid of bastion hosts.
There was a reference architecture for AWS.
Boundary is made up of the control plane that uses an postgres to store session.
Worker nodes actually route the traffic.
He made a reference architecture for azure.
Like closely the same as the aws one.
The HCP product make it way simpler because the control plane is managed.
You might be responsible for the workers if you want.

HCP managed setup: Control Plane and managed workers.
Managed workers work for public endpoints.
Self-managed workers would be used for private resource in a private subnet.
Boundary Cost: Sessions, Data transfer fee, self-managed workers (cloud costs).

Reduce the worker cost by dynamically scale them.

Scaling Considerations: metrics - network or cpu?
Availability, recommended 3 worker for HA.
Have a scaling schedule not base on metrics solely.
Scale to zero?
Automatic Updates.
Scale up for upgrade, like a rolling update.
Because version have to match with the control plane.

Registration: Worker registers to the Control plane.
In OSS PKI and KMS.
In HCP only PKI.
Controller-led & Worker-led.
In Autosscaling case Worker-led.
1. Self managed worker will talk to the control plane, auth request
1. auth request token responded to the worker
1. a user principal take the request token, auth them self to the control plane 
1. run a command to the worker registration
1. Control plane hands credentials to the worker

Principal auth with password or OIDC.
OIDC requires interaction.
Automated worker registration will require password auth.
The worker will need quite some information to perform a self registration.
User Azure key store to provide this information to the nodes.

Demo: he shows `boundary.tmpl` that holds the worker script running at start up.
HCP uses a different bin as the OSS version.
He uses some service account to access the key store.
He generates the `worker.hcl`.
And generates the systemd file to use it as a service..
Start the systemd service.

github.com/ned1313


## Secure Developer Workflows with Vault & Github Actions
10:55 pm → 30 min, Kartik Lunkad; Vault 

Example: Netflix.
Vision to create Studio in the cloud.
Variety of the app teams had to many security things to do.
Strong authentication is our highest leverage control.
Killed the checklist approach.
Made trivial to integrate authN.

Security starts at the local dev env.
DX is the hack for adopting of secure practice.
Start with good enough security pattern.

Secure Secret Access Developer Workflow.
1. static secrets in your .net app
use token base auth and retrieve secrets from source code.
Secrets are generate into the code.
Less secure.
He Demos it.

1. dynamic secrets in a go application.
Dynamic access during runtime.
More secure, harder to use because they have a TTL.

1. manage secrets in the github actions workflows.
Uses token based auth, oidc would be more secure.

## Why You Should Use Vault as Your Consul Certificate Authority
11:25 pm → 30 min, Thomas Kula; Consul 

CA usage in Consul.
Consul Service Mesh using the CA.
mTLS (mutual TLS) at the core of Consul.
Each service has a cert to identify to each other.
Services access aka intentions are controlled with certs.

Who are you?
Are you allowed in?
All based on identify embedded in TLS certs.
So certs are very important. 
What do we need to protect.
We do not need to are about certs, at all!
A cert is only an identity assertion.
Certs are by definition a public document.
But we do care about the private key related to the cert, that signed the cert.
The private key is the proof of identity.
How can we trust a cert?
Certs are signed by higher up certificates, up to a CA. 
The root CA is trusted so we trust any derived cert.

What to we need to do to protect those private keys.
Protection is quite easy.
A private key is generate locally and it asked a ca to sign a cert.
The private key is only in memory.
It get harder for higher up CAs.
With the root ca private key we can sign anything.
There are many standard on how to protect CAs private keys.

How does Consul do it?
It's stored/protected with the RAFT protocol.
(RAFT is a distributed consensus algorithm).
Raft.db file is stored on a consul node. 
Checking the file we find a private key.
Generate the public key for the private key and compare it to the cert.
It the actual root CAs private key.
Consul mesh is an identify based networking tool.

Use Vault CA to secure the Consul CAs private key.
So no local storage of the CAs private key anymore.
All signing is handed off to Vault.
Vault encrypts every persistent storage used at a backend.

Vault will never hand out private keys.
There is a special endpoint to get this but it has to be configured with a high privileged token.

Try to access the token when vault is configured to save on disk. 
But vault will encrypt it, so raw disk access will not expose the key.

Read about PCI.


## Fast-track Your Kubernetes Journey with EKS BluePrints for Terraform and Waypoint
11:55 pm → 30 min, Juan Peredo; Terraform Waypoint 

Works at AWS.

Example: Joyce want to optimize her teams productivity with k8s.
Her company uses EKS.
Her problem is that k8s is complicated and has a lot of resources.
Security and Observeability has to be setup.
CNCF list 100+ projects which makes it not easy to apply it correctly.

How to solve
* cluster management: EKS Blueprints for Terraform.
* application management: Waypoint

The EKS blueprints helps to setup a well architected Cluster.
Integrated with ArgoCD.
The blueprints are build with terraform and helm.
https://aws-ia.github.io/terraform-aws-eks-blueprints/v4.24.0/

The Teams concept is interesting.
Platform team with admin access.
Application teams.
Each team in it's own namespace.
With own permissions.
Each team has a quota.
The quota is limiting the name space resources.
Installing addons, e.g. istio or the basic cni addons.
Observability addons e.g. fluentD.
Nice argo CD integration ❤️.

Waypoint to abstract building & deployment.
It's like docker compose up on steroids.
Can standardize how deployments are done in the organization.
Waypoint uses HCL declarative syntax.
Can deploy not only to K8s, but also ECS or lambda.
Powerful ui to check state.
How is the process to deploy to EKS.
1. setup the cluster
1. Create the `waypoint.hcl`
1. install & init the waypoint server in the cluster.
1. Run `waypoint up`

Nice abstraction that we could use for preview app.


## Open Policy Agent in Terraform Cloud
12:25 am → 30 min, Peter ONeill; Terraform 

OPA is a policy engine.
Externalize authority.
The service will ask OPA and OPA will compare the query to the policies that are defined.
Styra DAS is a control plane for OPA.

The creation of tf resources can also be restricted.

Demo: was really confusing and did not get what the goal was.


## Scaling HashiCorp's Writing Culture with Internal Tooling
12:55 am → 30 min, Anubhav Mishra, Josh Freda; Community Culture Other 

Over 2k employees and mostly remote.
works.hashicorp.com
Focus on the writing culture.

* Problem Requirements Document PRD: Defining the problem
* Request for Comment RFC: Proposing solutions.

Ideation -> PRD -> Request Review Approval -> RFC -> Request Review -> Approval -> Implement -> Release

Very manual process up until now.
Challenges
* scalling company from 100 to 2000 people
* Volume of documents
* consistency with writing process
* sharing & discovery
* on boarding

The switch to hermes. 
https://github.com/hashicorp-forge/hermes

Demo of Hermes.
Authenticate using google.
Nice pun in his docs, rewrite every thins in rust :D Rewrite vagrant in bash :D.
But interesting process.

The architecture:
A single go binary.
A server and a indexer.
Algolia is powering the search.

Challenges: unstructured data. 
Document are unstructured despite using templates.
Editing google docs with api is hard.
All is just a blob of json and indices are changing all the time.
Supporting the existing workflow and the new workflows.

There are published templates for the two document types.

https://docs.google.com/document/d/1Oz_7FhaWxdFUDEzKCC5Cy58t57C4znmC_Qr80BORy1U/edit
https://docs.google.com/document/d/1oS4q6IPDr3aMSTTk9UDdOnEcFwVWW9kT8ePCNqcg1P4/edit
