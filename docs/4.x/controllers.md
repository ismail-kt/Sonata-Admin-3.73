# Controllers & Customization — Sonata Admin 4.x

In 4.x you should avoid container access inside controllers. Use constructor injection and typed action signatures.

## Example modern controller

```php
namespace App\Controller\Admin;

use Sonata\AdminBundle\Controller\CRUDController;
use Psr\Log\LoggerInterface;
use Symfony\Component\HttpFoundation\Request;

final class UserAdminController extends CRUDController
{
    private LoggerInterface $logger;

    public function __construct(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    protected function preCreate(Request $request, object $object): ?
    {
        $this->logger->info('Creating user');
        return null;
    }

    public function approveAction(int $id)
    {
        $object = $this->admin->getObject($id);
        $object->approve();

        $this->admin->update($object);

        return $this->redirectToList();
    }
}
```

## Binding controller to admin

```php
protected static $baseControllerName = UserAdminController::class;
```

Notes:
- Ensure your controller service is registered and autowired.
- Replace any `$this->container->get()` calls with injected services.
