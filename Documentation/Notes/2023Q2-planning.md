## Current situation - where are we?

### What have we done recently?
- Rework of Corvus storage and tenanted storage libraries
  - Used in production in Career Canvas
  - Updated to use latest Azure SDK
  - Separated out core storage approach from tenanted storage approach

- Expansion of Corvus Identity
  - Used in production in Career Canvas
  - Enabling a more uniform approach to secret management (motivated by changes to Corvus storage, but useful elsewhere)
  - Support a wider range of service authentication models, e.g. both user defined and system defined managed identities in addition to storage keys.

- Tenant management v3
  - Used in Career Canvas; Smartr need to upgrade their code to use it
  - Introduced ability to configure the same service differently when used multiple times (e.g. directly and as a dependency of another service, such as when using operations and workflow directly, operations is also used by workflow).

- Updated Marain services to use updated storage libraries
  - Deployed for Career Canvas (check - in production?) and in test environments at Smartr
  - Operations
  - Tenancy
  - Claims

- Experiment: Least privilege storage credentials
  - Goal: Ensure that application code never has access to credentials that allow it to do more than it should - e.g. our current blob storage code would allow a tenant to access other tenant containers if they were in the same storage account, meaning the security boundary is the storage account, not the container.
  - Working POC for blob storage
  - Discovered that Cosmos hierarchical partition keys are not as helpful as we wanted them to be
  - Highlighted but didn't solve the fact that JWT sizes become a problem with this approach; we can't use them to pass arbitrary amounts of information around (although we can use them for smaller amounts of data, such as a key to access the data we might have put in there).

- Experiment: Mini-Marain
  - Goal: Determine if it's possible to host multiple Marain services in a single process
  - It is.
  - https://github.com/marain-dotnet/experiment-mini-marain

- Menes rework to support hosting outside Azure functions
  - Demonstrated hosting of Marain services in Service Fabric
  - Demonstrated production use in ASP.NET Core hosted in Azure Container Apps

- Corvus.Deployment

### What is currently in progress?
- Migration from Json.Net -> System.Text.Json
  - Corvus.Extensions.System.Text.Json (including refactoring common code out of Corvus.Extensions.Newtonsoft.Json)
  - Marain.Tenancy has a branch in progress
  - Marain.Operations has a branch in progress
  - Marain.Workflow has a branch in progress
  - Working on multiple services in parallel to discover what common functionality we need in Corvus.Extensions.System.Text.Json


## Current questions/concerns?

- None of the more recent Corvus.Deployment changes have been applied to other Corvus/Marain projects. We can't call it DevOps if the Dev doesn't talk to the Ops.
 
- Have any of these things addressed the core concerns around Marain performance in Career Canvas and at Smartr?
  - Cold start
  - Warm start
  - Density and cost
  - Service to service performance concerns

- Complexity of local dev when working with Marain services

- Marain Deployment is very complex due to the dependencies between services.

- No agreed way of versioning Marain services

- No easy/quick way of getting changes into production use

- Documentation is sparse for most of our IP

## Mini projects

- Bring all Corvus libraries up to date with our latest DevOps tooling/approach
  - What about https://github.com/orgs/corvus-dotnet/projects/12/views/1? Talk to Charlotte.
  - Projects to update:
    - Corvus.AzureFunctionsKeepAlive
    - Corvus.Configuration
    - Corvus.ContentHandling
    - Corvus.Deployment
    - Corvus.Deployment.PowerBi.Cli
    - Corvus.DotLiquidAsync
    - Corvus.EventStore
    - Corvus.Extensions
    - Corvus.Extensions.CosmosDb
    - Corvus.Extensions.Newtonsoft.Json
    - Corvus.Extensions.System.Text.Json
    - Corvus.Globbing
    - Corvus.Identity
    - Corvus.JsonSchema
    - Corvus.Leasing
    - Corvus.Monitoring
    - Corvus.Retry
    - Corvus.Storage
    - Corvus.Tenancy
    - Corvus.Testing
    - Corvus.UriTemplates
  - Projects to ignore
    - Corvus.Caching
    - Corvus.Extensions.CommandLine (archived)
    - nodatime

- Rework Marain deployment
  - Current deployment assumes a set of interdependent services are deployed as a single unit of work.
    - Is this still the right approach?
    - How can we bring this up to date with our latest scripted build/GitHub workflows?
  - How do we do local development?
  - How do we version the services?
  - How do we think we should host things?
    - Quantify and set targets for existing Marain
      - Cold start
      - Warm start
      - Density and cost
      - Service to service performance concerns
    - when hosted in
      - Azure Functions (consumption plan)
      - Azure Functions (premium plan)
      - Azure Container Apps (consumption plan)
      - Azure Container Apps (defined pricing plan)
      - Firecracker?
      - AKS
    - but not in
      - Web Applications
      - ACI
      - Service Fabric

- Documentation
  - Marain - general
    - What is a Marain service? e.g.
      - Buys into the Marain tenancy model (even if for a single tenant)
      - Fits in with the standard Marain way of communicating its dependencies to consumers
      - Fits in with the standard deployment model/management tooling
      - etc
  - Marain.Tenancy
  - Marain.Claims
  - Marain.Operations
  - Marain.Workflow
  - Marain.UserNotifications
  - For each project:
    - Conceptual docs
    - How it's intended to be used
    - How it works with other Marain services
    - How-to guides


- System.Text.Json all the things - https://github.com/orgs/marain-dotnet/projects/5

- Multitenancy in Azure
  - Automated/self-service tenant onboard/offboarding/management
  - Least privilege
    - storage credentials
    - Service-to-service credentials
    - Calls to other Azure services (e.g. ARM, Graph, KV, App Config Svc)
    - Calls to 3rd party services
  - Next step: ?

- Hosting Reaqtor and other non-Marain services
  - What is a non-Marain service?

- Things we haven't fully discussed
  - Marain demo project- what could this be? Need something that uses all the services.
  - Marain Cron - how do we do timed jobs in Marain? This is needed in the Content Platform now.

## Other changes

- Determine the best way of involving JD/ME-L in future so we don't get so out of sync with our devops best practice again (without them having to do all the work)


## Reactor

Assumptions about its environment
- persistent, reliable storage is available (key value store - essentially a replicated in-memory store distributed across the cluster)
  - all other storage options we have are Azure services which have order-of-magnitude differences in perf characteristics
  - retrieving values from service fabric storage is microseconds, retrieving from e.g. Azure Storage is milliseconds
- possible to spin up new instances of a service programatically from inside an existing service
  - not much of this - use for historical processing mode
  - could possibly live without it
- relying on the characteristics of service fabric remoting
  - very efficient transmission of binary data
  - great integration with tasks and cancellation
  - all internal to the cluster
- existing large scale deployments have a large number of query engine instances, which scale up/down based on demand. They have a smaller number of compute nodes; this approach could have major impacts on cost with some hosting options - e.g. ACA.

Notes
- Some work has also been done to use gRPC - and enough problems have been solved that service fabric remoting could be replaced with gRPC across the board.
- We want to provide various multi-tenancy isolation guarantees, onboarding, offboarding, etc, as per any other Marain service.
  - E.g. ensuring query engines are spun up as per-tenant processes, or sets of processes across nodes (for isolation)
