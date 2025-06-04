# API Sample Test

## Getting Started

This project requires a newer version of Node. Don't forget to install the NPM packages afterwards.

You should change the name of the ```.env.example``` file to ```.env```.

Run ```node app.js``` to get things started. Hopefully the project should start without any errors.

## Explanations

The actual task will be explained separately.

This is a very simple project that pulls data from HubSpot's CRM API. It pulls and processes company and contact data from HubSpot but does not insert it into the database.

In HubSpot, contacts can be part of companies. HubSpot calls this relationship an association. That is, a contact has an association with a company. We make a separate call when processing contacts to fetch this association data.

The Domain model is a record signifying a HockeyStack customer. You shouldn't worry about the actual implementation of it. The only important property is the ```hubspot```object in ```integrations```. This is how we know which HubSpot instance to connect to.

The implementation of the server and the ```server.js``` is not important for this project.

Every data source in this project was created for test purposes. If any request takes more than 5 seconds to execute, there is something wrong with the implementation.

## Execution Code - 2 hours work (Screenshot)
- I followed the "design code" from application and give my contributions on Debrief section
![honeystack-run](https://i.imgur.com/O8VMCFU.png)

## Debrief

### Code Quality and Readability
Configuration and Libs:
- The application needs some environment vars to startup correctly, for this reason I would put a env parse validator by runtime like [zod](https://zod.dev/) or [env-schema](https://www.npmjs.com/package/env-schema) ensuring application doesn't start without env vars;
- Update libraries from project.
	- The current version of [@hubspot/api-client](https://www.npmjs.com/package/@hubspot/api-client) is 13.0.0
	- Express don't need `body-parser` 
	- Nowadays [date-fns](https://date-fns.org/) is better than moment.js
  - Express current version: 5.1.0, project: 4.18.2

Bad Practices

```js
app.locals.moment = moment;
app.locals.version = process.env.version;
app.locals.NODE_ENV = NODE_ENV;
```

- Set "global" vars for Express APP is a bad thing to do for sure
- If you need set a API version use prefix in API 

Change to ES Modules
- Better readability
- Work as well with linters, type checkers and bundlers


### Project architecture
- Node.JS was made to work with modules, each file in Node.JS represents a module.
- The Hubspot integrations can would be moved to a single module like `src/libs/hubspot-client.ts`
- Despite the methods `processContacts` `processCompanies` `processMeetings` has integration with Hubspot, they are methods which implement business rules, adapt these methods for not depends on Hubspot client directly is a good approach.
- `Domain.js` represents a schema from mongodb not a real entity from system. Changing the name to `DomainSchema` can correct the misunderstanding
- Organizing structure folder like:
	- src/services/process-meetings.js
	- src/services/process-companies.js
	- src/services/process-contacts.js
	- src/libs/hubspot-api-client.js
	- src/http/app.js
	- src/server.js
	- src/config/env-config.js

### Performance, Tools and Observability
- Replace `forEach` to `await Promise.all()` or `await Promise.allSetled()` if functions doesn't depends each other.
	- `forEach` doesn't work with `await` and stuck the event loop
- Change the manual timestamps calculus to validate library 
- "Cheat" for rate limiting `offsetObject?.after >= 9900`, change this for a config property 
- Transform app to a `JobProcessorApplication` , a dedicated application which process async jobs and running cron jobs
	- Change the `queue` method from `async` lib to BullMQ with redis or any message broker with `node-cron` to trigger a new job (rabbitMQ, SNS/SQS, kafka)
	- Retry exponential back off
- Implementing structured logs, [pino](https://www.npmjs.com/package/pino) can be a good choice, very fast and low overhead Node.JS logger
- Alerts when job finished
- Use external tools to storage sensitive info (API Keys, Database Credentials) like Amazon Web Secrets or Vault from HashCorp 

### Bonus
If this application will receives HTTP Request to execute some action or do some job I would change the express to fastify
- Fastify uses fast-json-stringify by Matteo Collina with built-in functions based on schemas for serialize objects too fast
- Fastify uses a radix tree for routing - a compact and optimized tree built during server setup.
- It’s minimalist, with a strong focus on core performance
- Native validation using JSON Schema via Ajv

Express working as well but for optimization...

- Express uses JSON.stringify for serialize objects
- Linear routing based on route definition
- Minimalist but depends on directly middlewares
- No native validation

**Unit and integration tests.**
