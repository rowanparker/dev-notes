When running functional unit tests against an **internal** API, check that the actual exception is thrown:

    $this->expectException(AccessDeniedException::class);
    $client = static::createClient();
    $client->client->catchExceptions(false);
    $client->request('GET', '/foo');

When running against an **external** API, use the in-built BrowserKit methods:

    $client = static::createClient();
    $client->request('GET', '/foo');
    $this->assertStatusCode(403);

# Database Testing

Always use different database schema name (e.g. `_test` suffix) for the test enviroment.
This will ensure the symfony command line arguments will be less ambigious when you migrate or apply fixtures.

After setting up an appropriate test environment.

    php bin/console --env=test doctrine:migrations:migrate
    php bin/console --env=test doctrine:fixtures:load

# Data Fixtures

Entities should have the `@GeneratedValue` annotation on the `id` property. If you are using data fixtures for testing, you will probably want to access specific entities by their `id`. To allow the data fixture to set the `id` use the following lines to override the generator type.

        // Enforce specified record ID
        $metadata = $manager->getClassMetaData(MyEntity::class);
        $metadata->setIdGeneratorType(\Doctrine\ORM\Mapping\ClassMetadata::GENERATOR_TYPE_NONE);
