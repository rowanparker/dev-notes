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
