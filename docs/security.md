# Security, Batch Actions & Compatibility

## Security Configuration (AbstractAdmin)

```php
protected function configureSecurityInformation(): array
{
    return [
        'EDIT'   => ['ROLE_ADMIN'],
        'DELETE' => ['ROLE_SUPER_ADMIN'],
    ];
}
```

## Batch Actions

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

## Compatibility Matrix

| Component               | Status |
|------------------------:|:------:|
| PHP 7.4                 | ✅ Supported |
| Symfony 4.4             | ✅ LTS |
| Sonata Admin 3.73       | ✅ Stable |
| ContainerAware          | ⚠ Legacy |
| AdminExtensionInterface | ✅ Recommended |

## Notes

- Verify method signatures and exact API names against Sonata Admin 3.73 docs before copy-pasting.
- If planning to upgrade to Symfony 5+, prepare migration notes: move away from container usage to constructor injection and update deprecated APIs.
