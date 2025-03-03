# JS

#### Event Targets

`event.target` is the element that triggered the event (e.g. icon inside button)

`event.currentTarget` is the element that the listener is attached to (e.g. button iself).

#### Operators

```typescript
foo?.bar // optional chaining
foo!.bar // TypeScript non-null assertion
```

#### Nullish Coalescing vs Logical OR

**?? (Nullish Coalescing Operator)**
- Returns the right-hand operand only if the left-hand operand is null or undefined.
- It does not treat false, 0, NaN, or "" (empty string) as "nullish."

```typescript
const value = null ?? "default"; // "default"
const value2 = undefined ?? "default"; // "default"
const value3 = 0 ?? "default"; // 0 (not replaced)
const value4 = "" ?? "default"; // "" (not replaced)
```

**|| (Logical OR Operator)**
- Returns the right-hand operand if the left-hand operand is falsy (false, 0, NaN, "", null, or undefined).

```typescript
const value = null || "default"; // "default"
const value2 = undefined || "default"; // "default"
const value3 = 0 || "default"; // "default" (0 is falsy)
const value4 = "" || "default"; // "default" (empty string is falsy)
```

# Rush

 #### Clear pnpm-lock.yaml

```
pnpm why @foo/bar -r
git checkout origin/main -- common/config/rush/pnpm-lock.yaml
rush update
```
#### Missing `.eslintrc.js`

```
Plugin "react" was conflicted between "package.json » eslint-config-react-app » ...\node_modules\eslint-config-react-app\base.js" and "BaseConfig » ...\node_modules\.pnpm\eslint-config-react-app@7.0.1_7e5e6237d96334972dc90355c15ec900\node_modules\eslint-config-react-app\base.js".
```

The individual project within the monorepo is likely missing it's own `.eslintrc.js` file.

#### Invalid package.json

https://github.com/microsoft/rushstack/issues/2678#issuecomment-890270823

>Another problem I've encountered is if you have a trailing comma in one of your settings (e.g. "dependencies": { "x": "1.2.3", }) then rush install will process it okay, but PNPM will fail to install without printing any error message.

```
pnpm-store --no-prefer-frozen-lockfile --recursive --link-workspace-packages false
The command failed with exit code 1
```

# Symfony




## Access environment variables

### Controller

In `config/services.yaml`

    parameters:
        app.paramname: '%env(APP_PARAM)%'

In  `src/Controller/SomeController.php`

    $this->getParameter('app.paramname');

https://stackoverflow.com/questions/52151783/symfony-4-get-env-parameter-from-a-controller-is-it-possible-and-how

### Twig Template

`MY_VAR=foo` use `{{ app.request.server.get('MY_VAR') }}`

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
    
### Doctrine relation collections not loading during tests

Clear the Entity Manager cache in the test setup:

    $this->entityManager->clear();
    
Refer to https://stackoverflow.com/questions/18268464/doctrine-lazy-loading-in-symfony-test-environment
    

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
    
### Dynamic per-request Doctrine database connections

https://karoldabrowski.com/blog/dynamic-database-connection-based-on-request-symfony-and-doctrine/

Use the `wrapper_class` parameter and extend the `Connection` class.    
    
### Symfony Make Migration Metdata Storage Sync Error

Fix this by appending the correct serverVersion string to the database URL.

    The metadata storage is not up to date, please run the sync-metadata-storage command to fix this issue

    DATABASE_URL=mysql://root:@127.0.0.1:3306/testtest?serverVersion=10.4.11
    to
    DATABASE_URL=mysql://root:@127.0.0.1:3306/testtest?serverVersion=mariadb-10.4.11

### Acccess mapped entity in Symfony Form in Twig template

https://stackoverflow.com/questions/23868624/access-mapped-entity-from-form-in-twig

    {{ form.vars.data.firstName }}

### bin/console + xdebug + phpstorm

Add Run/Debug configuration

    File: /project_dir/bin/console
    Arguments: command_name
    
### Project Directory

`services.yaml`

    services:
        _defaults:
            bind:
                $projectDir: '%kernel.project_dir%'

`SomeService.php`

    public function __construct(private string $projectDir)
    
### Database connection error when running cache:clear

If the `server_version` value is not set in the configuration, Doctrine will try connecting to the server to determine it. 
https://stackoverflow.com/questions/34023813/symfony-2-7-cacheclear-command-checks-every-database-connection

    #config.yml

    doctrine:
        dbal:
        ...
            server_version:       5.6

### Loading fixtures with DateTimeImmutable fields

    # src/Faker/Provider/CustomProvider.php
    
    namespace App\Faker\Provider;

    use Carbon\CarbonImmutable;
    use Faker\Provider\Base;

    final class CustomProvider extends Base
    {
        public function carbonImmutable(string $dateTime): CarbonImmutable
        {
            return CarbonImmutable::parse($dateTime, 'utc');
        }
    }

    # fixtures/user.yml
    
    App\Entity\User:
        user_1:
            ...
            createdAt: <carbonImmutable('2020-01-01T00:00:00')>
            
            
### Formatting dates in twig via Carbon

    # src/Twig/AppExtension.php

    use Carbon\Carbon;
    use Carbon\CarbonInterval;
    use Twig\Extension\AbstractExtension;
    use Twig\TwigFilter;

    class AppExtension extends AbstractExtension
    {
        public function getFilters()
        {
            return [
                new TwigFilter('human_datetime', [$this, 'humanDatetime']),
                new TwigFilter('human_duration', [$this, 'humanDuration']),
            ];
        }

        public function humanDuration(mixed $duration)
        {
            return CarbonInterval::seconds((int) $duration)->cascade()->forHumans();
        }

        public function humanDatetime(\DateTime|\DateTimeImmutable $dateTime)
        {
            return (new Carbon($dateTime))->toDayDateTimeString();
        }
    }
    
    # templates/home/index.html.twig
        
    {{ dateFoo|human_datetime }}
