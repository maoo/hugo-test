---
date: 2017-06-12T00:11:02+01:00
title: Quickstart - build your own integration
weight: 40
---

This page will guide a developer to build an integration module to be used with the [Integration Bridge](https://github.com/symphonyoss/App-Integrations-Core), using the [Symphony-Zapier integration](https://github.com/symphonyoss/App-Integrations-Zapier) as example:

## Prerequisites

Installation prerequisites:

* JDK 1.8
* [Maven 3.0.5+](https://maven.apache.org)
* Node 6.10+
* Gulp (globally installed)
* Webpack (globally installed)

Symphony Pod access prerequisites:

* Access to a Symphony Pod to be used for development and testing purposes
  * User Account (username/password credentials)
  * Service Account (*.p12 User Identity certificate and certificate password)
  * Symphony API (Agent, Pod, Sessionauth, Keymanager) endpoints (ask your Symphony Pod Administrator), reachable from your current workstation
* .p12 file is located in a local `certs/` folder (make sure you add `certs/` in your `.gitignore` file) and password is validated (use the command `openssl pkcs12 -info -in <certname.p12>`)
* an `env.sh` file is located in the root location of the project and contains all Symphony Pod endpoints and passwords mentioned above; start from [env.sh.sample](https://github.com/symphonyoss/App-Integrations-Zapier/blob/dev/local-run/env.sh.sample) running `cp local-run/env.sh.sample env.sh`

## Creating the code repository
Clone the source repository of an existing integration to use as skeleton or start from scratch.

```sh
git clone git@github.com:symphonyoss/App-Integrations-Zapier.git
cd App-Integrations-Zapier
mvn clean install
```

## Run locally

Make sure that:

- `certs/` folder contains a .p12 file
- `env.sh` values are all validated (see above)

To start the integration locally, just invoke `./run.sh` from the project root directory.

This command will create an `application.yaml` file in the project root folder, using [`local-run/application.yaml.template`](https://github.com/symphonyoss/App-Integrations-Zapier/blob/dev/local-run/application.yaml.template) as template.

## Expose local endpoint to a public host

In order to be able to create the app in the Foundation pod, you must provide a public `App Url`; you can use [ngrok](https://ngrok.com/) (or similar) to tunnel your local connection and expose it via a public DNS:
```
ngrok http 8080
```
Your local port 8080 is now accessible via `<dynamic_id>.ngrok.io`

If you have a paid subscription, you can also use
```
ngrok http -subdomain=my.static.subdomain 8080
```

## Add your locally running application to the Symphony Market

Adjust your [bundle.json](https://github.com/symphonyoss/App-Integrations-Zapier/blob/dev/src/main/webapp/bundle.json) located `src/main/webapp/` with the URL you are exposing via ngrok, the configuration and bot id's, and the application context.

**Note: The team is working on a integration-provisioning module that will automate this process; until further notice, please contact Symphony Support to get your configuration and bot id's.

For the application context, it should match what you have on [application-zapier.yml](https://github.com/symphonyoss/App-Integrations-Zapier/blob/dev/src/main/resources/application-zapier.yml)

For instance, see apps/zapier present in the URL's for the controller.html and appstore-logo.png, as well as in the **context** query parameter for the controller:

```json
{
  "applications": [
    {
      "type": "sandbox",
      "id": "devZapierWebHookIntegration",
      "name": "Zapier",
      "blurb": "Zapier Webhook Integration in Development Mode",
      "publisher": "Symphony",
      "url": "https://d74a790c.ngrok.io/apps/zapier/controller.html?configurationId=58598bf8e4b057438e69f517&botUserId=346621040656485&id=devZapierWebHookIntegration&context=apps/zapier",
      "icon": "https://d74a790c.ngrok.io/apps/zapier/img/appstore-logo.png",
      "domain": ".ngrok.io"
    }
  ]
}
```

Access the application icon on your browser to make sure it works and to accept any unsafe certificates (if necessary). In the above example, the URL to acces is `https://d74a790c.ngrok.io/img/appstore-logo.png`.

**Run your application again as indicated above, to get the new bundle.js information packaged.**

Launch the Symphony client on your browser, adding your bundle.js as path of the query parameters in the URL. For instance, using the Foundation Dev Pod with the above ngrok sample URL: `https://foundation-dev.symphony.com?bundle=https://d74a790c.ngrok.io/apps/zapier/bundle.json`.

Access the Symphony Market on the browser, and you should be notified to allow unauthorized apps. That is your development app added through bundle.json. Accept the notification and you should see your application in the application list, with the name and description provided in the bundle.json.

## Developing your own webhook configuration flow

The application added to the Symphony Market is commonly referred to as the Configurator Application, as it allows the user to configure the webhooks for a 3rd party integration.

Zapier Configurator Application is based on the out-of-the-box flow provided by [App-Integrations-FE-Commons](https://github.com/symphonyoss/App-Integrations-FE-Commons), which includes a complete set of views that allow users to manage the webhooks for an integration. Zapier Configurator App implementation involves the following files, and it is based on the [out-of-the-box-configurator sample](https://github.com/symphonyoss/App-Integrations-FE-Commons/tree/dev/samples/out-of-the-box-configurator):

- [.babelrc](https://github.com/symphonyoss/App-Integrations-Zapier/blob/dev/.babelrc)
- [.eslint](https://github.com/symphonyoss/App-Integrations-Zapier/blob/dev/.eslint)
- [package.json](https://github.com/symphonyoss/App-Integrations-Zapier/blob/dev/package.json)
- [webpack.config.js](https://github.com/symphonyoss/App-Integrations-Zapier/blob/dev/webpack.config.js)
- [webpack.config.prod.js](https://github.com/symphonyoss/App-Integrations-Zapier/blob/dev/webpack.config.prod.js)
- [src/main/webapp](https://github.com/symphonyoss/App-Integrations-Zapier/blob/dev/src/main/webapp)

It is also possible to customize the views and flows for your Configurator App. Check the [posting-location sample](https://github.com/symphonyoss/App-Integrations-FE-Commons/tree/dev/samples/posting-location-sample) for an example on how to do that.

### Running with Intellij
TODO

## Application healthcheck
To validate if your application were bootstrapped successfully you can reach on the application health check using
the URL `http://localhost:8080/integration/health`

The application health will indicate the connectivity and compatibility for each service you want to communicate like POD, Key Manager, and Agent. You also could check the application health, the configurator URL's, application flags, and latest posted message timestamp.

This is the result from healthcheck:

```json
{
  "status": "UP",
  "message": "Success",
  "version": "0.13.0-SNAPSHOT",
  "services": {
    "Agent": {
      "connectivity": "UP",
      "currentVersion": "1.46.0",
      "minVersion": "1.44.0",
      "compatibility": "OK"
    },
    "POD": {
      "connectivity": "UP",
      "currentVersion": "1.46.0",
      "minVersion": "1.44.0",
      "compatibility": "OK"
    },
    "Key Manager": {
      "connectivity": "UP",
      "currentVersion": "1.46.0",
      "minVersion": "1.44.0",
      "compatibility": "OK"
    }
  },
  "applications": [
    {
      "name": "zapier",
      "version": "0.13.0-SNAPSHOT",
      "status": "ACTIVE",
      "message": "Success",
      "configurator": {
        "loadUrl": "https://d74a790c.ngrok.io/apps/zapier/controller.html",
        "iconUrl": "https://d74a790c.ngrok.io/apps/zapier/img/appstore-logo.png"
      },
      "flags": {
        "parser_installed": "OK",
        "configurator_installed": "OK",
        "certificate_installed": "OK",
        "user_authenticated": "OK"
      },
      "latestPostTimestamp": "2017-05-29T16:30:44Z-0300"
    }
  ]
}
```

## YAML Configuration File
TODO

## Build a new integration
TODO

## Build the Configurator APP
TODO

## Inspect incoming payloads
TODO

## MessageML v2
TODO