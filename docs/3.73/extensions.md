# Admin Extensions (AdminExtensionInterface)

Admin extensions let you inject behavior into multiple admins without inheritance.

## Creating an Admin Extension

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
    // configureSideMenu() is deprecated in many setups
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

## Registering the Extension (Symfony 4.4)

```yaml
services:
  App\Admin\Extension\TimestampAdminExtension:
    tags:
      - { name: sonata.admin.extension }
```

This will automatically apply the extension to all admins.

## Method Reference — UI & Menus

- `configureFormFields(FormMapper)` — used for create & edit forms
- `configureListFields(ListMapper)` — defines list (table) columns
- `configureDatagridFilters(DatagridMapper)` — adds filters
- `configureShowFields(ShowMapper)` — controls the show (read-only) view
- `configureRoutes(AdminInterface, RouteCollection)` — add custom routes
- `configureTabMenu(...)` — top admin menu; prefer over deprecated `configureSideMenu()`

Example route registration:

```php
$collection->add(
    'approve',
    $admin->getRouterIdParameter() . '/approve'
);
```

Query customization example:

```php
public function configureQuery($admin, $query, $context): void
{
    $query->andWhere('o.deleted = false');
}
```

Validation example (Sonata 3.x style):

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

Note: `validate()` is valid in Sonata 3.73 but may be deprecated in future major versions.
