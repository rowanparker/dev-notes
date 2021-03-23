When running functional unit tests against an **internal** API, check that the actual exception is thrown:

$this->expectException(AccessDeniedException::class);
$client = static::createClient();
$client->client->catchExceptions(false);
$client->request('GET', '/foo');

When running against an **external** API, use the in-built BrowserKit methods:

$client = static::createClient();
$client->request('GET', '/foo');
$this->assertStatusCode(403);
