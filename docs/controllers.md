# Controllers & CRUDController Customization

Use a custom controller when you need custom workflows, external service calls, JSON responses, or custom redirects.

## Example Custom Controller

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

## Binding Controller to Admin

In your admin class:

```php
protected $baseControllerName = UserAdminController::class;
```

## Container-aware behavior (legacy)

Sonata 3.x controllers may rely on container-aware behavior:

```php
$this->container->get('doctrine');
```

This is valid for Symfony 4.4 but deprecated in Symfony 5+. Prefer constructor injection when possible.
