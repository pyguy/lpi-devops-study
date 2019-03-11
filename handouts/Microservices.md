## Microservices

The microservice architectural style is **an approach to developing a single application as a suite of small services**, each running in its own process and communicating with lightweight mechanisms, often an **HTTP resource API**. These services are built around business capabilities and independently deployable by fully automated deployment machinery.

 ### Monolithic application

Monolithic application built as a single unit. Enterprise Applications are often built in **three main parts**: 

* **a client-side user interface** (consisting of HTML pages and javascript running in a browser on the user's machine) 

* **a database** (consisting of many tables inserted into a common, and usually relational, database management system), 

* **a server-side application** (handling HTTP requests, executing domain logic, retrieving and updating data from the database)

 This server-side application is a *monolith* - a **single logical executable**. Any changes to the system involve building and deploying a new version of the server-side application. You can horizontally scale the monolith by running many instances behind a load-balancer.

 * Change cycles are tied together
 * A change made to a small part of the application, requires the entire monolith to be rebuilt and deployed. 
 * Over time it's often hard to keep a good modular structure, 
 * Scaling requires scaling of the entire application rather than parts of it that require greater resource.

 ![codebase deploys](../img/MonolithsandMicroservices.png)

 ### Characteristics of a Microservice Architecture

 There isn't a formal definition of the microservices architectural style, but we can attempt to describe what we see as common characteristics for architectures.

 #### Componentization via Services

 