# Architecture — Sonata Admin 4.x (Overview)

Sonata Admin 4.x modernizes the bundle to work with PHP 8.2+ and recent Symfony versions. Key platform constraints used by Sonata 4.x: PHP ^8.2 and Symfony components ^6.4 || ^7.3 || ^8.0.

In Sonata Admin 4.x, the same high-level customization points remain but the recommended patterns are modernized:

- AdminExtensionInterface  Cross-admin, reusable logic (methods may use stronger typing)
- AbstractAdmin subclasses  Entity-specific configuration
- Custom controllers (non-container-aware)  HTTP & workflow customization, prefer constructor injection

When to use what?

| Use case                            | Recommended approach                 |
| ----------------------------------- | ------------------------------------ |
| Reusable logic across many admins   | `AdminExtensionInterface`            |
| Entity-specific admin behavior (UI & rules) | `AbstractAdmin`                      |
| Custom routes, workflows, responses | Custom controller (no container access) |
| Business rules on save/delete       | Lifecycle hooks (`prePersist`, etc.) |
| Data validation                      | Admin / Extension                    |
| Pre/Post persistence hooks           | Admin / Extension                    |

Key differences from 3.x:

- PHP 8.x typed signatures and return types across many extension points — update your code to match stricter types.
- Controllers should use constructor injection instead of accessing the container.
- Service registration now commonly relies on autowiring and autoconfigure (Symfony Flex style).
- Some deprecated APIs (container helpers, older menu helpers) have been removed or replaced.

Example modern admin skeleton:

```php
use Sonata\AdminBundle\Admin\AbstractAdmin;

final class UserAdmin extends AbstractAdmin
{
    // ... typed method signatures where applicable
}
```
