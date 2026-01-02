# Lifecycle Hooks — Sonata Admin 4.x

Lifecycle hooks remain important. In 4.x the signatures may be stricter (typed `AdminInterface`, typed objects).

Example — set createdAt on insert with typed parameters:

```php
use Sonata\AdminBundle\Admin\AdminInterface;

public function prePersist(AdminInterface $admin, object $object): void
{
    if (method_exists($object, 'setCreatedAt')) {
        $object->setCreatedAt(new \DateTimeImmutable());
    }
}
```

Guidelines:
- Use DTOs or typed entities where possible.
- Avoid side-effects that perform long-running tasks; use message buses for async work.
