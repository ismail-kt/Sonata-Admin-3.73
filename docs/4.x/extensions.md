# Admin Extensions — Sonata Admin 4.x

Admin extensions in 4.x have the same goal but expect stronger typing and modern DI practices.

## Example Admin Extension (4.x style)

```php
namespace App\Admin\Extension;

use Sonata\AdminBundle\Admin\AdminExtensionInterface;
use Sonata\AdminBundle\Form\FormMapper;
use Sonata\AdminBundle\Admin\AdminInterface;
use Sonata\AdminBundle\Route\RouteCollection;

final class TimestampAdminExtension implements AdminExtensionInterface
{
    public function configureFormFields(FormMapper $form): void
    {
        $form->add('createdAt');
    }

    public function configureListFields($list): void {}
    public function configureDatagridFilters($filter): void {}
    public function configureShowFields($show): void {}
    public function configureRoutes(AdminInterface $admin, RouteCollection $collection): void {}
    public function configureTabMenu(...): void {}

    // lifecycle/validation methods with modern typing where available
}
```

## Registering the Extension (autoconfigure + services.yaml)

```yaml
services:
  App\Admin\Extension\:
    resource: '../src/Admin/Extension/*'
    tags: ['sonata.admin.extension']
    autoconfigure: true
    autowire: true
```

## Notes for 4.x

- Update method signatures to match the Sonata 4.x types — PHPStan or an IDE will help find mismatches.
- Prefer autowiring and constructor injection for any collaborators you use inside your extension.
