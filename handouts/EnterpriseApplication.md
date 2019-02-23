## Enterprise Application

Enterprise applications usually have **a lot of persistent data**, usually managed by some kind of database management system. Usually this database is relational, but increasingly we're seeing NoSQL alternatives. This data will usually be longer lasting and more valuable than the applications that process it.

This **data is accessed and manipulated concurrently**. The numbers vary a lot, in-house applications may have a few tens of users, but customer-facing web applications can easily have tens of thousands.

With so much data, enterprise applications have **a lot of user interface screens** to handle it.

You'll need to **integrate with other enterprise applications**. These systems are built by a wide range of teams, some from vendors who sell to many customers, others built internally just for your organization.

Even when different applications access the same data there is considerable **conceptual dissonance** between them, a customer may mean something quite different to the sales organization than it does to technical support. 

If you want to change business rules you need sixty-seven meetings and three vice-presidents retiring, Do this a thousand times and you have the **complex business "illogic"** that lies in the heart of many enterprise applications.