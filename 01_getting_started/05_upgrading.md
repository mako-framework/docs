# Upgrading

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako 9.0.x to 9.1.x.

There are no breaking changes in this release but there are some deprecations. Check out the changelog to check out the new features included in the release. Follow this upgrade guide to future-proof your application.

--------------------------------------------------------

### Command Bus

The command bus library has been deprecated and replaced by the new ["bus"](:base_url:/docs/:version:/learn-more:command-event-and-query-buses) library that includes command, event and query bus implementations. You should consider upgrading now as the old library will be removed in Mako 10.

### Events

The event class has been deprecated and replaced by the new ["bus"](:base_url:/docs/:version:/learn-more:command-event-and-query-buses) library that includes command, event and query bus implementations. You should consider upgrading now as the old event class will be removed in Mako 10.
