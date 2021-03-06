# Serverless Fastify

Simple wrapper around [Fastify](https://www.fastify.io/) to use it with [Serverless](https://serverless.com/)

Disclaimer: very EARLY draft of the package, thus I may introduce breaking changes between versions

## Prerequisites
- TypeScript
- Fastify

## Install

Install with npm:

```
npm i serverless-fastify --save
```

## Example

hello-world-controller.ts
```ts
import { Get } from "serverless-fastify";
import { FastifyRequest, FastifyReply } from "fastify";

// Define the controller using the interface
export class HelloWorldController {
  @Get("/", {
    schema: {
      response: {
        200: {
          type: "object",
          properties: {
            msg: { type: "string" },
          },
        },
      },
    },
  })
  getMessage(request: FastifyRequest, reply: FastifyReply<any>) {
    return {
      msg: "Hello world",
    };
  }
}

```

sls-fastify-config.ts
```ts
import { SlsFastifyConfig } from "serverless-fastify";
import { HelloWorldController } from "./hello-world.controller";

// Pre handler hook plugin example ( may be turned into class + decorator later )
const preHandlerHook = (fastify, options, done) => {
  fastify.addHook("preHandler", (request, reply, done) => {
    console.log("Pre handler hook !");
    done();
  });
  done();
};

// Define the config here
const slsFastifyConfig = {
  // Define the routes
  routes: [
    {
      name: "helloworld", // This is the name of the handler in serverless.yml
      controller: HelloWorldController, // the actual controller for this route
      prefix: "v1/helloworld", // The prefix defined for api gateway
    },
  ],
  // Registering the fastify plugins ( the order matters )
  plugins: [preHandlerHook],
} as SlsFastifyConfig;

export { slsFastifyConfig };

```

handlers.ts
```ts
import { initHandlers } from "serverless-fastify";
import { slsFastifyConfig } from "./sls-fastify-config"; // Your config

// Register the handlers for serverless
export = {
  ...initHandlers(slsFastifyConfig, async () => {
    // Any async code before execution the handlers ( in serverless )
    // e.g initDatabaseConnection()
  }),
};

```

server.ts ( To run the server locally )
```ts
import { initApp } from "serverless-fastify";
import { slsFastifyConfig } from "./sls-fastify-config"; // Your config

async function start() {
  const PORT = Number(process.env.port) || 3000;
  const HOST = process.env.host || "127.0.0.1";

  const app = initApp(slsFastifyConfig);

  app.listen(PORT, HOST, async (err) => {
    if (err) console.error(err);
    console.log(`server listening on port ${PORT}`);
  });
}

start();

```

serverless.yml

```yml
service: sls-fastify

provider:
  name: aws
  runtime: nodejs12.x
  stage: ${opt:stage, 'dev'}
  region: eu-west-1
  stackName: ${self:service}-${self:provider.stage}
  apiName: ${self:service}-${self:provider.stage}
  timeout: 29

functions:
  helloworld:
    handler: handlers.helloworld
    events:
      - http: "ANY /v1/helloworld"
      - http: "ANY /v1/helloworld/{proxy+}"


plugins:
  - serverless-webpack
```

## License

Licensed under MIT
