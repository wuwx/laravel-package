# Laravel package for prooph components

## Overview
This is a Laravel package for prooph components to get started out of the box with message bus, CQRS, event sourcing and 
snapshots. It uses Doctrine DBAL.

It provides all [service definitions and a default configuration](config "Laravel Package Resources"). 
See the [documentation](http://getprooph.org/) for more details of the prooph components.

### Available services
* `Prooph\ServiceBus\CommandBus`: Dispatches commands
* `Prooph\ServiceBus\EventBus`: Dispatches events
* `Prooph\EventStoreBusBridge\TransactionManager`: Transaction manager for service bus and event store
* `Prooph\EventStoreBusBridge\EventPublisher`: Publishes events on the event bus
* `Prooph\EventStore\Adapter\Doctrine\DoctrineEventStoreAdapter`: Doctrine adapter for event store
* `Prooph\EventStore\Snapshot\SnapshotStore`: Event store snapshot adapter
* `Prooph\EventStore\Snapshot\Adapter\Doctrine\DoctrineSnapshotAdapter`: Doctrine snapshot adapter

## Installation

You can install prooph/prooph-package via composer by adding `"proophsoftware/prooph-package": "^0.1"` as requirement to 
your composer.json. Setup your database [migrations](https://github.com/prooph/event-store-doctrine-adapter#database-set-up)
for the Event Store.

### Service Provider
In your application configuration add `Monii\Interop\Container\Laravel\ServiceProvider` for container-interop support and
`Prooph\Package\ProophServiceProvider` to your providers array. Then you have access to the services above.

## Example
You have only to define your models (Entities, Repositories) and commands/routes. You find all these things in the 
[prooph components documentation](http://getprooph.org/ "prooph components documentation"). Here is an example config
from the [proophessor-do example app](https://github.com/prooph/proophessor-do "prooph components in action").

> Run `artisan vendor:publish` to add your configuration for the prooph components

```php
// in your config/prooph.php
return [
    'event_store' => [
        // list of aggregate repositories
        'user_collection' => [
            'repository_class' => 'Prooph\ProophessorDo\Infrastructure\Repository\EventStoreUserCollection',
            'aggregate_type' => 'Prooph\ProophessorDo\Model\User\User',
            'aggregate_translator' => 'Prooph\EventSourcing\EventStoreIntegration\AggregateTranslator',
        ],
    ],
    'service_bus' => [
        'command_bus' => [
            'router' => [
                'routes' => [
                    // list of commands with corresponding command handler
                    'Prooph\ProophessorDo\Model\User\Command\RegisterUser' => 'Prooph\ProophessorDo\Model\User\Handler\RegisterUserHandler'
                ]
            ]
        ],
        'event_bus' => [
            'router' => [
                'routes' => [
                    // list of events with a list of projectors
                    'Prooph\ProophessorDo\Model\User\Event\UserWasRegistered' => [
                        'Prooph\ProophessorDo\Projection\User\UserProjector'
                    ]
                ]
            ]
        ]
    ]
];
```

Here is an example of the corresponding service XML configuration with container-interop for the proophessor-do example.

```php
// in your config/dependencies.php
return [
    'factories' => [
        // Model
        \Prooph\ProophessorDo\Model\User\Handler\RegisterUserHandler::class => \Prooph\ProophessorDo\Container\Model\User\RegisterUserHandlerFactory::class,
        \Prooph\ProophessorDo\Model\User\UserCollection::class => \Prooph\ProophessorDo\Container\Infrastructure\Repository\EventStoreUserCollectionFactory::class,
        // Projections
        \Prooph\ProophessorDo\Projection\User\UserProjector::class => \Prooph\ProophessorDo\Container\Projection\User\UserProjectorFactory::class,
        \Prooph\ProophessorDo\Projection\User\UserFinder::class => \Prooph\ProophessorDo\Container\Projection\User\UserFinderFactory::class,
    ]
];
```

Here is an example how to call the `RegisterUser` command:

```php
    /* @var $container \Illuminate\Container\Container */
    
    /* @var $commandBus \Prooph\ServiceBus\CommandBus */
    $commandBus = $container->make(Prooph\ServiceBus\CommandBus::class);

    $command = new \Prooph\ProophessorDo\Model\User\Command\RegisterUser(
        [
            'user_id' => \Rhumsaa\Uuid\Uuid::uuid4()->toString(),
            'name' => 'prooph',
            'email' => 'my@domain.com',
        ]
    );

    $commandBus->dispatch($command);
```

Here is an example how to get a list of all users from the example above:

```php
    /* @var $container \Illuminate\Container\Container */
    $userFinder = $container->make(Prooph\ProophessorDo\Projection\User\UserFinder::class);

    $users = $userFinder->findAll();
```

## Support

- Ask questions on [prooph-users](https://groups.google.com/forum/?hl=de#!forum/prooph) mailing list.
- File issues at [https://github.com/proophsoftware/prooph-package/issues](https://github.com/proophsoftware/prooph-package/issues).
- Say hello in the [prooph gitter](https://gitter.im/prooph/improoph) chat.

## Contribute

Please feel free to fork and extend existing or add new plugins and send a pull request with your changes!
To establish a consistent code quality, please provide unit tests for all your changes and may adapt the documentation.

## License

Released under the [New BSD License](LICENSE.md).
