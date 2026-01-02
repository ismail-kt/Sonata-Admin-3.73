# Security & Batch Actions — Sonata Admin 4.x

Security configuration continues to live in admin classes, but you can also rely on Symfony security voters and attributes.

## Example security in Admin (4.x)

```php
protected function configureSecurityInformation(): array
{
    return [
        'EDIT' => ['ROLE_ADMIN'],
        'DELETE' => ['ROLE_SUPER_ADMIN'],
    ];
}
```

## Batch actions (same concept)

```php
protected function configureBatchActions(array $actions): array
{
    $actions['approve'] = [
        'label' => 'Approve',
        'ask_confirmation' => true,
    ];

    return $actions;
}
```

## Compatibility Matrix (4.x target platforms)

| Component               | Supported Range |
|------------------------:|:--------------:|
| PHP                    | ^8.2 (8.2+)     |
| Symfony                | ^6.4 \| ^7.3 \| ^8.0 |
| Sonata Admin           | 4.x             |

Notes:
- If upgrading from 3.x, check for removed/deprecated APIs and adapt to typed method signatures and DI changes.
