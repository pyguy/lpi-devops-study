## The Twelve-Factor App

The twelve-factor app is a methodology for building software-as-a-service apps that:
* Use **declarative** formats for setup automation, to minimize time and cost
* Have a **clean contract** with the underlying operating system, offering **maximum portability**
* Are suitable for **deployment** on modern **cloud platforms**
* **Minimize divergence** between development and production
* And can **scale up** without significant changes to tooling, architecture, or development practices.

### 1. Codebase
> One codebase tracked in revision control, many deploys

  * If there are multiple codebases, it’s not an app – it’s a distributed system.
  * Multiple apps sharing the same code is a violation of twelve-factor
  * There is only one codebase per app, but there will be many deploys of the app

   ![codebase deploys](../img/codebase-deploys.png)

### 2. Dependencies
> Explicitly declare and isolate dependencies

 * Programming languages offer packaging system for distributing support libraries, such as **CPAN** for Perl or **Rubygems** for Ruby.
 * A twelve-factor app never relies on implicit existence of system-wide packages.

|  languages |  declaration | isolation       | 
|------------|--------------|-----------------|
| ruby       | Gemfile      | bundle exec    | 
| python     | pip          | virtualenv      | 
| C          | autoconf    | static linking |

### 3. Config
> Store config in the environment


* Resource handles to the database, Memcached, and other backing services
* Credentials to external services such as Amazon S3 or Twitter
* Per-deploy values such as the canonical hostname for the deploy

A litmus test for whether an app has all config correctly factored out of the code is whether the codebase could be made open source at any moment, without compromising any credentials.

Another approach to config is the use of config files which are not checked into revision control, such as `config/database.yml` in Rails. Further, these formats tend to be language- or framework-specific.

The twelve-factor app stores config in **environment variables** (often shortened to *env vars* or *env*).

They are a language- and OS-agnostic standard.

### 4. Backing services
> Treat backing services as attached resources

A backing service is any service the app consumes over the network as part of its normal operation. Examples include:

* datastores (such as MySQL or CouchDB)
* messaging/queueing systems (such as RabbitMQ or Beanstalkd)
* SMTP services for outbound email (such as Postfix)
* and caching systems (such as Memcached).

In addition to these locally-managed services, the app may also have services provided and managed by third parties. Examples include:
* SMTP services (such as Postmark)
* metrics-gathering services (such as New Relic or Loggly)
* binary asset services (such as Amazon S3)
* and even API-accessible consumer services (such as Twitter, Google Maps, or Last.fm).

**The code for a twelve-factor app makes no distinction between local and third party services.**  To the app, both are attached resources, accessed via a URL or other locator/credentials stored in the config.

The twelve-factor app treats these databases as attached resources, which indicates their loose coupling to the deploy they are attached to.

![attached resources](../img/attached-resources.png)

### 5. Build, release, run
> Strictly separate build and run stages

A codebase is transformed into a (non-development) deploy through three stages:

* The **build stage** is a transform which converts a code repo into an executable bundle known as a build. Using a version of the code at a commit specified by the deployment process, the build stage fetches vendors dependencies and compiles binaries and assets.
* The **release stage** takes the build produced by the build stage and combines it with the deploy’s current config. The resulting release contains both the build and the config and is ready for immediate execution in the execution environment.
* The **run stage** (also known as “runtime”) runs the app in the execution environment, by launching some set of the app’s processes against a selected release.

![releases](../img/release.png)

**The twelve-factor app uses strict separation between the build, release, and run stages.** For example, it is impossible to make changes to the code at runtime, since there is no way to propagate those changes back to the build stage.

Every release should always have a unique release ID, Releases are an append-only ledger and a release cannot be mutated once it is created. Any change must create a new release.

### 6. Processes
> Execute the app as one or more stateless processes

In the simplest case, the code is a stand-alone script, the execution environment is a developer’s local laptop with an installed language runtime, and the process is launched via the command line (for example, `python my_script.py`).

**Twelve-factor processes are stateless and share-nothing.** Any data that needs to persist must be stored in a stateful backing service, typically a database.

The twelve-factor app never assumes that anything cached in memory or on disk will be available on a future request or job.

Some web systems rely on “sticky sessions” – that is, caching user session data in memory of the app’s process and expecting future requests from the same visitor to be routed to the same process. **Sticky sessions are a violation of twelve-factor and should never be used or relied upon.** Session state data is a good candidate for a datastore that offers time-expiration, such as *Memcached* or *Redis*.

### 7. Port binding
> Export services via port binding

**The twelve-factor app is completely self-contained** and does not rely on runtime injection of a webserver into the execution environment to create a web-facing service. The web app **exports HTTP as a service by binding to a port**, and listening to requests coming in on that port.

 In deployment, a routing layer handles routing requests from a public-facing hostname to the port-bound web processes.

This is typically implemented by using **dependency declaration** to add a webserver library to the app, such as *Tornado* for Python, *Thin* for Ruby, or *Jetty* for Java and other JVM-based languages. This happens entirely in user space, that is, within the app’s code. The contract with the execution environment is binding to a port to serve requests.

Note also that the port-binding approach means that one app can become the backing service for another app, by providing the URL to the backing app as a resource handle in the config for the consuming app.

### 8. Concurrency
> Scale out via the process model

**In the twelve-factor app, processes are a first class citizen.** Processes in the twelve-factor app take strong cues from the unix process model for running service daemons. Using this model, the developer can architect their app to handle diverse workloads by assigning each type of work to a process type. For example, HTTP requests may be handled by a web process, and long-running background tasks handled by a worker process.

![releases](../img/process-types.png)

Twelve-factor app processes should never daemonize or write PID files. Instead, rely on the operating system’s process manager (such as systemd) to manage output streams, respond to crashed processes, and handle user-initiated restarts and shutdowns.

### 9. Disposability
> Maximize robustness with fast startup and graceful shutdown

**The twelve-factor app’s processes are disposable, meaning they can be started or stopped at a moment’s notice.** This facilitates fast elastic scaling, rapid deployment of code or config changes, and robustness of production deploys.

Short startup time provides more agility for the release process and scaling up; and it aids robustness, because the process manager can more easily move processes to new physical machines when warranted.

Processes **shut down gracefully when they receive a SIGTERM signal** from the process manager. For a web process, graceful shutdown is achieved by ceasing to listen on the service port (thereby refusing any new requests), allowing any current requests to finish, and then exiting. 

For a worker process, graceful shutdown is achieved by returning the current job to the work queue. For example, on RabbitMQ the worker can send a `NACK`.

Processes should also be **robust against sudden death**, in the case of a failure in the underlying hardware. While this is a much less common occurrence than a graceful shutdown with `SIGTERM`, it can still happen. A recommended approach is use of a robust queueing backend, such as Beanstalkd, that returns jobs to the queue when clients disconnect or time out. 

### 10. Dev/prod parity
> Keep development, staging, and production as similar as possible

Historically, there have been substantial gaps between development (a developer making live edits to a local deploy of the app) and production (a running deploy of the app accessed by end users). These gaps manifest in three areas:

* **The time gap:** A developer may work on code that takes days, weeks, or even months to go into production.
* **The personnel gap:** Developers write code, ops engineers deploy it.
* **The tools gap:** Developers may be using a stack like Nginx, SQLite, and OS X, while the production deploy uses Apache, MySQL, and Linux.

**The twelve-factor app is designed for continuous deployment by keeping the gap between development and production small.**

|   |  Traditional app | Twelve-factor app       | 
|------------|--------------|-----------------|
| **Time between deploys**       | Weeks      | Hours   | 
| **Code authors vs code deployers**    | Different people  | Same people      |
| **Dev vs production environments**   | Divergent    | As similar as possible |

Developers sometimes find great appeal in using a lightweight backing service in their local environments, while a more serious and robust backing service will be used in production. For example, using SQLite locally and PostgreSQL in production; or local process memory for caching in development and Memcached in production. **The twelve-factor developer resists the urge to use different backing services between development and production.**

### 11. Logs
> Treat logs as event streams

Logs provide visibility into the behavior of a running app. In server-based environments they are commonly written to a file on disk (a “logfile”); but this is only an output format.

Logs are the stream of aggregated, time-ordered events collected from the output streams of all running processes and backing services.

**A twelve-factor app never concerns itself with routing or storage of its output stream.** 

In staging or production deploys, each process’ stream will be captured by the execution environment, collated together with all other streams from the app, and routed to one or more final destinations for viewing and long-term archival. These archival destinations are not visible to or configurable by the app, and instead are completely managed by the execution environment. Open-source log routers (such as Logplex and Fluentd) are available for this purpose.

These systems allow for great power and flexibility for introspecting an app’s behavior over time, including:

* Finding specific events in the past.
* Large-scale graphing of trends (such as requests per minute).
* Active alerting according to user-defined heuristics (such as an alert when the quantity of errors per minute exceeds a certain threshold).

### 12. Admin processes
> Run admin/management tasks as one-off processes

Developers will often wish to do one-off administrative or maintenance tasks for the app, such as:

* Running database migrations (e.g. `manage.py migrate` in Django, `rake db:migrate` in Rails).
* Running a console (also known as a REPL shell) to run arbitrary code or inspect the app’s models against the live database. Most languages provide a REPL by running the interpreter without any arguments (e.g. `python` or `perl`) or in some cases have a separate command (e.g. `irb` for Ruby, `rails console` for Rails).
* Running one-time scripts committed into the app’s repo (e.g. `php scripts/fix_bad_records.php`).

One-off admin processes should be run in an identical environment as the regular long-running processes of the app. They run against a release, using the same codebase and config as any process run against that release. Admin code must ship with application code to avoid synchronization issues.

Twelve-factor strongly favors languages which provide a REPL shell out of the box, and which make it easy to run one-off scripts. In a local deploy, developers invoke one-off admin processes by a direct shell command inside the app’s checkout directory. In a production deploy, developers can use ssh or other remote command execution mechanism provided by that deploy’s execution environment to run such a process.