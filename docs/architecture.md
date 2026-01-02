# Architecture — Sonata Admin 3.x (Overview)

In Sonata Admin 3.x, customization happens at three distinct levels:

- AdminExtensionInterface  Cross-admin, reusable logic
- AbstractAdmin subclasses  Entity-specific configuration
- CRUDController extensions  HTTP & workflow customization

When to use what?

| Use case                            | Recommended approach                 |
| ----------------------------------- | ------------------------------------ |
| Reusable logic across many admins   | `AdminExtensionInterface`            |
| Entity-specific admin behavior (UI & rules) | `AbstractAdmin`                      |
| Custom routes, workflows, responses | Custom `CRUDController`              |
| Business rules on save/delete       | Lifecycle hooks (`prePersist`, etc.) |
| Data validation                      | Admin / Extension                    |
| Pre/Post persistence hooks           | Admin / Extension                    |

Core responsibilities of `AbstractAdmin`:

- Configure forms, lists, filters, show views
- Define routes
- Control permissions / security rules
- Hook into entity lifecycle
- Bind to controllers

Example skeleton:

```php
use Sonata\AdminBundle\Admin\AbstractAdmin;

final class UserAdmin extends AbstractAdmin
{
    // ...
}
```

Best practices:

- Use AdminExtensions for cross-cutting logic
- Keep controllers thin
- Use lifecycle hooks for business rules
- Centralize permissions in Admin classes

Avoid:

- Heavy logic in controllers
- DB writes in `configure*()` methods
- Duplicate logic across admins
