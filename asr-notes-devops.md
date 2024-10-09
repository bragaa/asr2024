# Introduction
- due to the continuous changes in the software development industry, requirements/constraints get imposed on applications (essentially due to business and organizational reasons):
  - more and more testing requirements,
  - dev teams needs tigther collaboration,
  - need to shorten time to market, make rapid feature/bugfixes integration, while being able to rollback easily
  - do software versionning,
  - adapt to various reqs in production (distruibuted over heterogeneous deployment env, mostly in cloud, the rise of software defined trend, compliance requirements, etc.)
  - the need to always release new features/changes or make them gradually, etc.

- software design business adapted by taking the following technologies: (sometimes, the technology was the driver not the solution)
  - software architecture follows proved **design patterns**: n-tiers (model view controller is an n-tiers app pattern), microservices, service mesh, API gateway, message bus, etc.
    - modularity, reuse, incremental development.
    - so common components are reused (kafka, redis, proxies, for lb, etc.)
  - tools have emerged to handle every step in the **application lifecycle**, aka **SDLC**: dev, various tests, integration, delivery, deployment, telemetry, etc.
  - describe everything through code, **xaaC**: how to perform tests, desired state of the app, desired prod environment config, how to handle spikes in demand, and evert aspects in the SDLC.
  - **automate** everything
  - make use of **virtualization technologies** to accelerate provisioning of compute/storage/networking resources: linux containers (including docker), cloud environments, SDN, k8s.

- this caused new engineering roles to appear in the software industry:
  - **devops engineers**: handles these technologies, ensuring that the infra for dev is ok, more like a systems engineer focusing on dev environments. understand devops as "development operations".
  - **site reliability engineering** (SRE): out of scope, concerned with the engineering of application deployment in production and uses tools like proxies, load balancers, CDNs, DNS, API gateways, etc.
  
- The goal here is to expand on these technologies and how they are typically used to accelerate the SDLC.

# Example of an application architecture and development
- Web application component:
  - database,
  - application server (backend)
  - reverse proxy
  - ui + static files on a cdn

# Application deployment options
- an application lives in the following environment: kernel, **execution engine** (run time environment) and libraries.
  - three models are offered for application hosting: SaaS, PaaS, IaaS
    - SaaS is not a really for app hosting, the app is already offered as a paid service and is consumed directly by customers.
    - in IaaS, customer is responsible for implementing the entire infrastructure, the provider offers infrastructure objects, e.g. Azure, AWS
    - in PaaS, customer gets some level already offered, like a PHP environment, a containers engine, etc.
- **Managed** vs Self-operated. It is about balancing responsiblities between customer and provider/operator
  - think of the difference between an Azure Managed Kubernetes clutser and a self operated Kubernetes cluster *you* deploy on the Azure cloud.
- Infrastructure resources can also be consumed as a service withoug caring about installation and operation:
  - compute (CPU, GPU)
  - networking (sw, routers, fw)
  - file/block storage
- billing options (pay as you go)
- **Function as a service**: just logic, not data storage, no execution environment, moderate to easy complexity. Links to **Serverless** computing. Examples are LAmbda and Azure Functions.

# The Software development lifecycle
- modelled in different ways (waterfall, agile, RUP, etc.,) but essential activities are gathering reqs, design, eventually leading to a SRS (software reqs. specs), develop, build, test, release/deploy, operate and maintain. we only focus on the steps starting from the development.
- **automation** everywhere, tools for each step. described as a pipeline
- note: integration vs delivery vs deployment. all theses steps are done multiple times, hence the word **continuous** integration, cont. deliv, cont. depl.
  - **integration** refers to integrating new features/bugfixes into the final code: includes merging the new module to the codebase after having made the some tests and quality tests, and providing in one place a new version of the software (in source code).
  - **delivery** refers to the delivery of code to the **staging environment** for integration and elaborated tests, and manual deployment (authorized and audited based on test reports).
  - **deployment** refers to delivery of code to production (automatic **publishing authorization** for code deployment).

## build
- build is triggered by a pull request (request to integrate code into main release branch, aka merge request). a webhook is then automatically started on the automation server to start build, test, notify team lead (and the like).

## test
- the following aspects are tested and evaluated: modules/features taken in isolation (unit tests), do they fit correctly together and with environment, i.e. db/thrid party software (integration tests), does the code provides the needed functionnality (user acceptance), is peformance acceptable, compliance, and the user interface behavior.
- testing is usually expensive; coverage is thus taken into account.
- some other tests, like code style, code quality, the use of secret within the code, etc.
- **unit testing** tests a specific section of code to ensure the code does what it is expected to do. unit tests are prepared by developers during the development phase. At this stage, a static code
analysis, data flow analysis, code coverage, and other software verification processes can be applied.
- **static code analysis** and **security application static testing** examine source code to find patterns of errors. other source code centric tests include *detecting secret* embedded into code (like passwords, unusual byte streams, etc.), check libraries used for *knwon vulnerabilities* or *license issues*, make a **software bill of material** (SBoM).
- code is then completely built onto a **staging environment**. it undergoes integration tests, performance, user acceptance, **dynamic security application tesing** (fuzzing,) etc.
- upon deployment, an additional **canary** test might be done: a change is partially rolled out (on a limited subset of servers or audience), then evaluated against the perf. baseline to ensure it is operating at least as well as the old.
- unit tests are done before building the whole software, while integrationa nd acceptance tests are done after and before release.
## release
- release models: rolling releases vs point releases. rolligng is more complex?

## deployment
- one should assume that software might still contain errors/behaves incorrectly in production. so, provide for **rollback**. deployment migth be done with certain service interruption (downtime) or not (wait until ongoing session finishes and forward new clients to new instances)
- different deployment methods: 
  + deploy in place and replace code in the existing instances, implies downtime.
  + rolling release: replace code in some instances. canary deployment is a variation, where only a limited subset is upgraded while monitoring it. 
  + blue/green


# Everything as a Code
- every thing in the application is being described rather than typed interactively: libs (in requirements.json), database creation in ddl scripts, firewall rules, virtual hosts, installed packages, etc.
- advantages:
  - reproducable/consistent environment through diffrent stages
  - rapid dev, fewer errors, easy roolbacks, etc.
- Techynologies: ansible, Micrtosoft DSC, Chef, etc.

# Architecture of a devops environment 
## software dev automation server (aka CI server)
- automates integration, delivery/release and deployment as well as tests.
- examples: Jenkins, Gitlab, BitBucket, Spinnaker, Travis

## versions control systems
- previous tools: cvs, svn, and current is git, mercurial, etc. Tools: bitbucket, github, etc.
  - gitlab in the Gartner's magical quadrant
- maintains a file/code repository, accessible to multiple teammates and tracks their activities (add/remove/modify). allows having multiple versions of the same file/code.
- supports **pull requests** aka merge requests.
  - pull requests are sent to the automation server in order to trigger the CI pipeline and provide results.
