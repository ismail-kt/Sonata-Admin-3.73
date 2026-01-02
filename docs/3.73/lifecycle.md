# Lifecycle Hooks

Lifecycle hooks are critical for enforcing business rules during persistence operations.

Hook            | When
--------------- | -------------------------
alterNewInstance| Object instantiation
alterObject     | Object loaded
prePersist      | Before insert
postPersist     | After insert
preUpdate       | Before update
postUpdate      | After update
preRemove       | Before delete
postRemove      | After delete

Example — set createdAt on insert:

```php
public function prePersist($admin, $object): void
{
    $object->setCreatedAt(new \DateTime());
}
```

Guidelines:

- Use lifecycle hooks for domain/business rules that must run on create/update/delete.
- Avoid heavy external I/O or long-running processes in hooks; delegate to async jobs if needed.
