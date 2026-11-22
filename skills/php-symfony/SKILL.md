---
name: php-symfony
description:
  Guides attribute-first Symfony service wiring and container verification. Use
  when editing Symfony services, dependency injection, autowiring, service
  discovery, Messenger handlers, Mailer wiring, or test service overrides.
---

# PHP Symfony

Use with the `php` skill. Check the project's Symfony version and service
discovery rules before changing wiring.

## Put service wiring in the owning class

- Prefer constructor type hints for dependencies Symfony can resolve directly.
  Use `#[Autowire(env: '…')]` for an environment variable and
  `#[Autowire(service: '…')]` to select a specific service, such as a named rate
  limiter. Do not also bind the same argument in configuration.
- Use a specific attribute such as `#[AsMessageHandler(bus: 'command_bus')]`
  when it expresses real wiring. Do not add an empty `#[Autoconfigure]` or an
  explicit `set()` for a class already discovered with the required defaults.
- Keep configuration for discovery, environment-specific substitutions, vendor
  services, and factories that attributes cannot express. Attributes do not
  register a class outside a discovered directory. Do not broaden discovery to
  all test classes just to register one service.
- Keep services and aliases private unless a caller needs direct container
  access. Injection does not require a public alias.

## Example: a test-only replacement

Put the capture path and the visibility required for direct test lookups on the
class. Keep registration and the replacement alias in test-only configuration.

```php
use Symfony\Component\DependencyInjection\Attribute\Autoconfigure;
use Symfony\Component\DependencyInjection\Attribute\Autowire;

#[Autoconfigure(public: true)]
final class TestMailer implements MailerInterface
{
    public function __construct(
        #[Autowire(env: 'TEST_MAIL_CAPTURE_PATH')]
        private readonly string $capturePath,
    ) {
    }

    // Implement MailerInterface here.
}
```

```php
$services->set(TestMailer::class)->autowire()->autoconfigure();
$services->alias(MailerInterface::class, TestMailer::class);
```

Use the project's existing defaults if they also apply to that configuration
file. Do not assume defaults in `services.php` apply to a separate package
configuration file.

## Verify the actual service

- Compile the affected container from a fresh cache and resolve the changed
  service. A health endpoint or successful kernel boot alone does not prove a
  lazily instantiated service has valid wiring.
- Verify the real consumer uses the intended implementation in each affected
  environment. For mail tests, use a fake. If an HTTP server runs in a separate
  process, use a capture mechanism both processes can read, not in-memory state.
- Do not route Mailer through a default Messenger bus without checking its
  middleware. If delivery must be synchronous and callable without a signed-in
  user, configure that explicitly rather than weakening bus authorization.
