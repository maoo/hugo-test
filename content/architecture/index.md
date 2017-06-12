---
date: 2017-06-12T00:11:02+01:00
title: Architecture
weight: 20
---

This page guides developers to create their first Symphony Integration, also known as **_Symphony Integration Bridge_**.

## Integration Bridge architecture

The Integration Bridge is responsible for managing active integrations and provides key services to allow third party services the ability to post messages into a configurable set of streams

The key services provided to the registered integrations are:

* Authentication proxy - each integration should be configured with credentials, but the implementation never needs to
 deal with them. Once bootstrapped, the integration can use integration bridge services as if it's unauthenticated. The bridge itself proxies those services to the cloud with the proper authentication.
* Send messages to a stream
* Read and write configuration information to the cloud configuration services
* Read user information to the cloud user services
* Health check

### Definitions

A list of definitions also serves to introduce the main components of the integrations system.

* **Integration** - An interface to be implemented by an specific integration. These are bootstrapped into the
Integration Bridge via config file. They have a lifecycle to be managed by the Integration Bridge.

* **Webhook Integration** - A specific type of integration which can be configured to listen for specific webhook events
 from external services. This is also a superclass of particular implementations of the webhook integrations which support specific services.
 For instance, a Zapier webhook integration would extend this, and provide logic to translate the event information
 into a particular message to the streams.

* **Integration Config API** - Cloud API responsible to manage the webhook instances configured by the user.

* **Configurator** - An application (listed in the Symphony App Store) which allows a user to
configure instances of an integration type. Using Zapier as an example, an user would have the configuration app
available in the app store, and once opened would be able to configure a new instance of the Zapier webhook
integration, get the URL for the JIRA cloud services to ping with webhook events, configure streams which the
integration instance should post to, as well as the name of the instance. The configurator app can also be used to
manage existing instances.

### System overview
Following below how the message sent by third party services flows through the system overall.

![System Overview](/Integration_Bridge_Overview.png)

The message sent by the third party service is processed by the integration parser and translated to MessageML. After that, the Integration Bridge posts the result MessageML through the Agent Message API's.

## Webhook Integration architecture
In this section we'll detail what is the general workflow when the core receives a message from an integrated
application, let's say Zapier, for this example:

> 1. Expose an endpoint where it will receive the message.
> 2. Identify where this message is coming from through the URL parameters it received (configurationId and instanceId)
> 3. If the message is trying to reach a valid integration and a configured instance, it will delegate the message to the specific integration code implemented separately across the other Integration repositories.
> 4. The integration Zapier logic will now determine which event it is dealing with through the received message header parameter, and based on this will determine which [parser](#parsers) it must use to treat the message properly.
> 5. The parser will then convert the message to a [Message ML format](#the-message-ml-format), extracting the needed information from the payload received.
> 6. The parsed message will return to the Integration Core and post the message to the Symphony platform

### Parsers
Integrations will most of the times need a parser to work properly.
Those special classes will need to deal with the content coming from the related application, parsing this data into a format readable by the Symphony platform.

This format is called Symphony Message ML and it may contain a set of tags. More details below.

### The Message ML format
A Message ML is a Symphony XML format that defines XML elements and attributes necessary to compose a message that can be posted within a chat room.
The most basic message one can send may be as simple as ``<messageML>simple message</messageML>`` or as detailed as it can get. What determines this is what system we are integrating with.

These elements and attributes will be briefly detailed in the next topics as reference. The specific integration formats can be found in their separate repositories "Readme" files.

### Entity (MessageMLv1.0)
_**note: MessageMLv1.0 has been superseded by MessageMLv2.0. MessageMLv1.0 is still supported, however MessageMLv2.0 will allow you to create a rich render in a more seamlessly manner, with less steps and no front end code. Please see a JIRA ticket rendered using messageMLv2.0 here for as a complex example: https://symphonyoss.atlassian.net/wiki/display/WGFOS/Single+Jira+Ticket+-+Templated+PresentationML & see [Zapier](https://github.com/symphonyoss/App-Integrations-Zapier) as a simple example of a richly rendererd message._**

An entity is a special element contained in a ``<messageML>``, it may also be nested within other entities as another element, and so on.

Entities must have a "type" and a "version", and may also have a "name" for itself, all of those as XML attributes.

The first entity in a messageML MUST have an element called "presentationML".

The ``<presentationML>`` is a special element that must hold all content that would be otherwise drawn on Symphony by other elements, represented as a single string on its content.
This particular text must follow the rules presented [here](https://rest-api.symphony.com/docs/message-format/).

It is important that it contains matching information as it is used for visualising a message when a specific renderer is not present, on Symphony mobile apps or content export.

Entities may also have ``<attribute>``s as their XML elements, which in turn must have a "name", a "type" and a "value" as attributes.

Here's an example of a valid MessageML, containing all of the mentioned above:

```xml
<messageML>
    <entity type="sample.event.core" version="1.0">
        <presentationML>test message for:<br/>application core</presentationML>
        <attribute name="message" type="org.symphonyoss.string" value="test message"/>
        <entity name="application" type="sample.application">
            <attribute name="appName" type="org.symphonyoss.string" value="core"/>
        </entity>
    </entity>
</messageML>
```