# Client

A machine entity that consumes one or more Marain [Services](#service).


# Deployment

One specific set of binaries and other artifacts put into a [Deployment Context](#deployment-context) at a particular moment in time.

# Deployment Context

A container for a set of resources (code, infrastructure, configuration etc.) that are deployed and managed as a unit, independent of any other Deployment Context.

Although a typical Deployment Context will contain a mixture of services, storage, compute capability, any of these may be absent. For example, you could conceive of a storage-only Deployment Context.

The Deployment Context does not necessarily have logically colocated resources. (A single context might include an AWS bucket in Australia, a Google Tensorflow compute node in South Carolina, and an Azure Function in Cardiff.)


# Resource

Some non-Marain [Service](#service) used by a Marain service (e.g., a container in a CosmosDB).


# Service

A well-defined capability of a system exposed to a [Client](#client) through a specific API.


# Stack

(tenantive) synonym for Deployment Context.


NEXT TIME:
working through the various terms used on https://app.mural.co/t/experiment8478/m/experiment8478/1684847460599/c98576005b9847b98c5551336cab8ee673009d78?sender=u648e627be65f5500bd3a2762