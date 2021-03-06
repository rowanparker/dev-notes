# Symfony


### Access environment variables

To access `MY_VAR=foo` from `.env*` files in a Twig template use `{{ app.request.server.get('MY_VAR') }}`

# Checking HTTP status codes in functional tests

If you are using exceptions to generate responses (e.g. such as throwing a `BadRequestHttpException` for a 400 error page) then you need to tell the `client` not to catch them. This let's them flow through to the test it self, where you should check that the actual exception is thrown:

    $this->expectException(BadRequestHttpException::class);
    $client = static::createClient();
    $client->client->catchExceptions(false);
    $client->request('GET', '/foo');

If you are manually creating response objects in your controller (e.g. setting the content and status codes) you should use the in-built BrowserKit methods:

    $client = static::createClient();
    $client->request('GET', '/bar');
    $this->assertStatusCode(403);

# Database Testing

Always use different database schema name (e.g. `_test` suffix) for the test enviroment.
This will ensure the symfony command line arguments will be less ambigious when you migrate or apply fixtures.

In the production environment, you want to allow the database to handle generation of IDs (e.g. primary keys). This should be done using the `@GeneratedValue` annotation. This becomes a problem in the test environment, because you want to use data fixtures that specify IDs (for retrieval in functional or unit tests).

Therefore, in the data fixture, you should use the following lines to temporarily disable the `@GeneratedValue` annotation, only during the fixture processing.

        // Enforce specified record ID
        $metadata = $manager->getClassMetaData(MyEntity::class);
        $metadata->setIdGeneratorType(\Doctrine\ORM\Mapping\ClassMetadata::GENERATOR_TYPE_NONE);
        
A related problem that occurs when using data fixtures is that it attempts to use the `TRUNCATE` SQL command to empty the existing database. This will cause SQL integrity constraint exceptions if you are using foreign keys. You will need to drop the database, create it again, run existing migrations, then load fixtures. For simplicity create the following two files. Don't forget to set line endings to `LF`.

`bin/reset-db-dev`:

    #!/usr/bin/env sh
    php bin/console doctrine:database:drop --force --env=dev
    php bin/console doctrine:database:create --env=dev
    php bin/console doctrine:migrations:migrate --env=dev
    php bin/console doctrine:fixtures:load --env=dev

`bin/reset-db-test`:

    #!/usr/bin/env sh
    php bin/console doctrine:database:drop --force --env=test
    php bin/console doctrine:database:create --env=test
    php bin/console doctrine:migrations:migrate --env=test
    php bin/console doctrine:fixtures:load --env=test

# Determining current environment

To access the current environment, use the Kernel Interface:

    private KernelInterface $kernel;

    public function __construct(KernelInterface $kernel)
    {
        $this->kernel = $kernel;
    }
    
    //...
    $kernel->getEnvironment();
    //...
    
    
### Symfony Make Migration Metdata Storage Sync Error

Fix this by appending the correct serverVersion string to the database URL.

    The metadata storage is not up to date, please run the sync-metadata-storage command to fix this issue

    DATABASE_URL=mysql://root:@127.0.0.1:3306/testtest?serverVersion=10.4.11
    to
    DATABASE_URL=mysql://root:@127.0.0.1:3306/testtest?serverVersion=mariadb-10.4.11




