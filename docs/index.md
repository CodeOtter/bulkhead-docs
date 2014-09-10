# Bulkhead

Bulkhead is a library for SailJS that allows a developer to compartmentalize the functionality of their web application into individual service-oriented components.

Traditionally, MVC patterns tend to make developers stuff as much business logic into the controller...

![Traditional MVC flowchart](../assets/start1.png)

However, this significantly reduces the extendability and scalability of a web application as business logic frequently needs to be accessed in an agnostic fashion.  (Web API, CLI, unit testing, reporting/mapReducing, cron, application components, etc.)

![Problematic scenarios](../assets/start2.png)

But with Bulkhead, developers can modularize services and NPM packages to perform a specific task and easily install it into a SailsJS project.

![Flexible solution via Bulkhead](../assets/start3.png)

So, how would you like to get started? :D

* [Put all of my code into reusable services](services.md)
* [Setup a unit testing harness with database access, fixtures, and REST testing](testing.md)
* [Convert an NPM package into a SailsJS plugin](plugin.md)
* [Create a brand new SailsJS plugin](plugin.md)
* [Learn more about Bulkhead](toc.md)
