# Sonata Admin 3.x – Extension & Customization Guide
PHP 7.4 · Symfony 4.4 · sonata-project/admin-bundle ^3.73

## 1. Purpose of This Document

This document explains supported extension points in Sonata Admin 3.73 and how to customize the admin layer.

Covering:
- AbstractAdmin
- AdminExtensionInterface
- CRUDController
- Lifecycle hooks
- Routing, security, validation, and workflows

Target audience:
- Senior PHP / Symfony developers
- Architects
- Maintenance teams working on legacy Sonata stacks

## 2. High-Level Architecture

In Sonata Admin 3.x, customization happens at three distinct levels:

- AdminExtensionInterface ← Cross-admin, reusable logic
- AbstractAdmin subclasses ← Entity-specific configuration
- CRUDController extensions ← HTTP & workflow customization

When to use what?

| Use case                            | Recommended approach                 |
| ----------------------------------- | ------------------------------------ |
| Reusable logic across many admins   | `AdminExtensionInterface`            |
| Entity-specific admin behavior (UI & rules) | `AbstractAdmin`                      |
| Custom routes, workflows, responses | Custom `CRUDController`              |
| Business rules on save/delete       | Lifecycle hooks (`prePersist`, etc.) |
| Data validation                      | Admin / Extension                    |
| Pre/Post persistence hooks           | Admin / Extension                    |

## 3. AbstractAdmin (Core Admin Class)

Every Sonata admin must extend `AbstractAdmin`.

```php
use Sonata\AdminBundle\Admin\AbstractAdmin;

final class UserAdmin extends AbstractAdmin
{
    // ...
}
```

Core responsibilities:
- Configure forms, lists, filters, show views
- Define routes
- Control permissions / security rules
- Hook into entity lifecycle
- Bind to controllers

## 4. AdminExtensionInterface (Cross-Admin Extensions)

Admin extensions allow you to inject behavior into multiple admins without inheritance.

### 4.1 Creating an Admin Extension

```php
use Sonata\AdminBundle\Admin\AdminExtensionInterface;
use Sonata\AdminBundle\Form\FormMapper;

final class TimestampAdminExtension implements AdminExtensionInterface
{
    public function configureFormFields(FormMapper $form): void
    {
        $form->add('createdAt');
    }

    public function configureListFields($list): void {}
    public function configureDatagridFilters($filter): void {}
    public function configureShowFields($show): void {}
    public function configureRoutes($admin, $collection): void {}
    public function configureTabMenu(...): void {}
    // configureSideMenu() is deprecated in later Sonata versions
    public function validate(...): void {}
    public function configureQuery(...): void {}
    public function alterNewInstance(...): void {}
    public function alterObject(...): void {}
    public function getPersistentParameters(...): array {}
    public function prePersist(...): void {}
    public function postPersist(...): void {}
    public function preUpdate(...): void {}
    public function postUpdate(...): void {}
    public function preRemove(...): void {}
    public function postRemove(...): void {}
}
```

### 4.2 Registering the Extension (Symfony 4.4)

```yaml
services:
  App\Admin\Extension\TimestampAdminExtension:
    tags:
      - { name: sonata.admin.extension }
```

This will automatically apply the extension to all admins.

## 5. AdminExtensionInterface – Method Reference

### 5.1 UI Configuration

`configureFormFields(FormMapper)` — used for create & edit forms:

```php
$form->add('email');
```

`configureListFields(ListMapper)` — defines list (table) columns:

```php
$list->addIdentifier('id')->add('name');
```

`configureDatagridFilters(DatagridMapper)` — adds filters:

```php
$filter->add('status');
```

`configureShowFields(ShowMapper)` — controls the show (read-only) view:

```php
$show->add('email');
```

### 5.2 Routes & Menus

`configureRoutes(AdminInterface, RouteCollection)`:

```php
$collection->add(
    'approve',
    $admin->getRouterIdParameter() . '/approve'
);
```

`configureTabMenu(...)` — adds items to the top admin menu:

```php
$menu->addChild('Audit Log', ['uri' => $admin->generateUrl('history')]);
```

Note: `configureSideMenu()` is deprecated in many setups — prefer `configureTabMenu` or menu integration supported by your Sonata version.

### 5.3 Query Customization

`configureQuery(AdminInterface, ProxyQueryInterface, $context)`:

```php
$query->andWhere('o.deleted = false');
```

### 5.4 Lifecycle Hooks (Critical)

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

Example:

```php
public function prePersist($admin, $object): void
{
    $object->setCreatedAt(new \DateTime());
}
```

### 5.5 Validation

```php
public function validate($admin, $errorElement, $object): void
{
    if ($object->getAmount() < 0) {
        $errorElement
            ->with('amount')
            ->addViolation('Amount cannot be negative')
            ->end();
    }
}
```

Note: Validation via `validate()` is still valid in Sonata 3.73 but may be deprecated in later major versions — confirm before upgrading.

## 6. AbstractAdmin – Advanced Usage

### 6.1 Common Overrides

```php
protected function configureFormFields($form) {}
protected function configureListFields($list) {}
protected function configureDatagridFilters($filter) {}
protected function configureShowFields($show) {}
protected function configureRoutes($collection) {}
```

### 6.2 Security Configuration

```php
protected function configureSecurityInformation(): array
{
    return [
        'EDIT'   => ['ROLE_ADMIN'],
        'DELETE' => ['ROLE_SUPER_ADMIN'],
    ];
}
```

### 6.3 Batch Actions

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

## 7. CRUDController Customization

### 7.1 Why Extend CRUDController?

Use when you need:
- Custom workflows
- External service calls
- JSON responses
- Custom redirects

### 7.2 Custom Controller Example

```php
use Sonata\AdminBundle\Controller\CRUDController;
use Symfony\Component\HttpFoundation\Request;

final class UserAdminController extends CRUDController
{
    protected function preCreate(Request $request, $object)
    {
        $this->get('logger')->info('Creating user');
        return null;
    }

    public function approveAction($id)
    {
        $object = $this->admin->getObject($id);
        $object->approve();

        $this->admin->update($object);

        return $this->redirectToList();
    }
}
```

### 7.3 Bind Controller to Admin

```php
protected $baseControllerName = UserAdminController::class;
```

## 8. ContainerAwareInterface (Legacy)

Sonata 3.x controllers may implement or rely on container-aware behavior.

```php
$this->container->get('doctrine');
```

✔ Valid for Symfony 4.4  
❌ Deprecated in Symfony 5+ — prefer constructor injection where possible.

## 9. Best Practices (Recommended)

Do:
- Use AdminExtensions for cross-cutting logic
- Keep controllers thin
- Use lifecycle hooks for business rules
- Centralize permissions in Admin classes

Avoid:
- Heavy logic in controllers
- DB writes in `configure*()` methods
- Duplicate logic across admins

## 10. Compatibility Matrix

| Component               | Status |
|------------------------:|:------:|
| PHP 7.4                 | ✅ Supported |
| Symfony 4.4             | ✅ LTS |
| Sonata Admin 3.73       | ✅ Stable |
| ContainerAware          | ⚠ Legacy |
| AdminExtensionInterface | ✅ Recommended |

## 11. Summary Cheat Sheet

| Feature             | Location               |
|--------------------:|------------------------|
| UI configuration    | Admin / Extension      |
| Reusable logic      | AdminExtension         |
| Lifecycle hooks     | Both                   |
| Routes              | Admin / Controller     |
| Workflows           | CRUDController         |
| Validation          | Admin / Extension      |
| Security            | Admin                  |

---
