# Choosing an architecture

There are several configurations available for Puppet Enterprise. The configuration you use depends on the number of nodes in your environment and the resources required to serve agent catalogs. All PE installations begin with the standard configuration, and scale up by adding additional components as needed.

|Configuration|Description|Node limit|
|-------------|-----------|----------|
|Standard installation \(Recommended\)|All infrastructure components are installed on the master. This installation type is the easiest to install, upgrade, and troubleshoot.|Up to 4,000|
|Large installation|Similar to a standard installation, plus one or more compilers and a load balancer which help distribute the agent catalog compilation workload.|4,000–20,000|
|Extra-large installation|Similar to a large installation, plus a separate node which hosts the PE-PostgreSQL instance.|More than 20,000|

**Tip:** You can add high availability to an installation with or without compilers by configuring a replica of your master. High availability isn't supported with standalone PE-PostgreSQL.

## Standard installation

![Graphic showing the standard reference architecture, where end users interact with a single master, and the master interacts with multiple agents.](standard.png)

## Large installation

![Graphic showing a large reference architecture, where end users interact with a single master. The master interacts with multiple compilers and multiple agents.](large.png)

## Extra-large installation

![Graphic showing an extra-large reference architecture, where end users interact with a single master. The master interacts with multiple compilers, multiple agents, and a standalone PE-PostgreSQL node.](xl.png)

