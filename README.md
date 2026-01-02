# Sonata Admin 3.x – Extension & Customization Guide
PHP 7.4 · Symfony 4.4 · sonata-project/admin-bundle ^3.73

## 1. Purpose of This Document

  This document explains all supported extension points in Sonata Admin 3.73, 
  #### covering:
  
  - AbstractAdmin
  - AdminExtensionInterface
  - CRUDController
  - Lifecycle hooks
  - Routing, security, validation, and workflows

  #### Target audience:

  - Senior PHP / Symfony developers
  - Architects
  - Maintenance teams working on legacy Sonata stacks

## 2. High-Level Architecture
   
In Sonata Admin 3.x, customization happens at three distinct levels:

AdminExtensionInterface ← Cross-sdmin, reusable logic

AbstractAdmin subclasses ← Entity-specific configuration

CRUDController extensions ← HTTP & workflow customization


**When to use what?**
| Use case                            | Recommended approach                 |
| ----------------------------------- | ------------------------------------ |
| Reusable logic across many admins   | `AdminExtensionInterface`            |
| Entity-specific admin behavior UI & rules      | `AbstractAdmin`                      |
| Custom routes, workflows, responses | Custom `CRUDController`              |
| Business rules on save/delete       | Lifecycle hooks (`prePersist`, etc.) |
| Data validation       | Admin / Extension |
| Pre/Post persistence hooks     | Admin / Extension |

## 3. AbstractAdmin (Core Admin Class)
Every Sonata admin must extend AbstractAdmin.


        use Sonata\AdminBundle\Admin\AbstractAdmin;
                
        final class UserAdmin extends AbstractAdmin
        {
        }



#### Core Responsibilities:
- Configure forms, lists, filters, show views
- Define routes
- Control permissions / Security rules
- Hook into entity lifecycle
- Bind to controllers

## 4. AdminExtensionInterface (Cross-Admin Extensions)

Admin extensions allow you to inject behavior into multiple admins without inheritance.

### 4.1 Creating an Admin Extension

    use Sonata\AdminBundle\Admin\AdminExtensionInterface;
    use Sonata\AdminBundle\Form\FormMapper;

    final class TimestampAdminExtension implements AdminExtensionInterface
    {
        public function configureFormFields(FormMapper $form): void
        {
            $form->add('createdAt');
        }
    
        public function configureListFields(ListMapper $list): void {}
        public function configureDatagridFilters(DatagridMapper $filter): void {}
        public function configureShowFields(ShowMapper $show): void {}
        public function configureRoutes(AdminInterface $admin, RouteCollection $collection): void {}
        public function configureSideMenu(...) {}
        public function configureTabMenu(...) {}
        public function validate(...) {}
        public function configureQuery(...) {}
        public function alterNewInstance(...) {}
        public function alterObject(...) {}
        public function getPersistentParameters(...) {}
        public function prePersist(...) {}
        public function postPersist(...) {}
        public function preUpdate(...) {}
        public function postUpdate(...) {}
        public function preRemove(...) {}
        public function postRemove(...) {}
    }

### 4.2 Registering the Extension (Symfony 4.4)

    services:
      App\Admin\Extension\TimestampAdminExtension:
        tags:
          - { name: sonata.admin.extension }
✅ Automatically applied to all admins


## 5. AdminExtensionInterface – Method Reference
### 5.1 UI Configuration
   
   #### configureFormFields(FormMapper)
   Used for create & edit forms
   
        $form->add('email');

   #### configureListFields(ListMapper)
   Defines list (table) columns

        $list->addIdentifier('id')->add('name');

   #### configureDatagridFilters(DatagridMapper)
   Adds filters

        $filter->add('status');

   #### configureShowFields(ShowMapper)
   Controls show (read-only) view

        $show->add('email');
### 5.2 Routes & Menus
   #### configureRoutes(AdminInterface, RouteCollection)

        $collection->add(
            'approve',
            $admin->getRouterIdParameter() . '/approve'
        );

   #### configureTabMenu(...)
   Adds items to top admin menu

        $menu->addChild('Audit Log', ['uri' => $admin->generateUrl('history'),]);

  ⚠️ configureSideMenu() is deprecated
   
### 5.3 Query Customization
   #### configureQuery(AdminInterface, ProxyQueryInterface, context)

        $query->andWhere('o.deleted = false');

### 5.4 Lifecycle Hooks (Critical)
  ##### Hook	              ##### When
  alterNewInstance	        Object instantiation
  alterObject	              Object loaded
  prePersist	              Before insert
  postPersist	              After insert
  preUpdate	                Before update
  postUpdate	              After update
  preRemove	                Before delete
  postRemove	              After delete

  Example:

        public function prePersist(AdminInterface $admin, $object): void
        {
            $object->setCreatedAt(new \DateTime());
        }

### 5.5 Validation

        public function validate(AdminInterface $admin, ErrorElement $errorElement, $object)
        {
            if ($object->getAmount() < 0) {
                $errorElement
                    ->with('amount')
                    ->addViolation('Amount cannot be negative')
                    ->end();
            }
        }

⚠️ Deprecated but valid in Sonata 3.73

## 6. AbstractAdmin – Advanced Usage
### 6.1 Common Overrides

        protected function configureFormFields(FormMapper $form)
        protected function configureListFields(ListMapper $list)
        protected function configureDatagridFilters(DatagridMapper $filter)
        protected function configureShowFields(ShowMapper $show)
        protected function configureRoutes(RouteCollection $collection)

### 6.2 Security Configuration

        protected function configureSecurityInformation(): array
        {
            return [
                'EDIT' => ['ROLE_ADMIN'],
                'DELETE' => ['ROLE_SUPER_ADMIN'],
            ];
        }

### 6.3 Batch Actions

        protected function configureBatchActions(array $actions): array
        {
            $actions['approve'] = [
                'label' => 'Approve',
                'ask_confirmation' => true,
            ];
        
            return $actions;
        }

## 7. CRUDController Customization
### 7.1 Why Extend CRUDController?

#### Use when you need:
- Custom workflows
- External service calls
- JSON responses
- Custom redirects

### 7.2 Custom Controller Example
        use Sonata\AdminBundle\Controller\CRUDController;
        
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

### 7.3 Bind Controller to Admin

       protected $baseControllerName = UserAdminController::class;

## 8. ContainerAwareInterface (Legacy)
Sonata 3.x controllers implement ContainerAwareInterface.

        $this->container->get('doctrine');

✔ Valid for Symfony 4.4
❌ Deprecated in Symfony 5+

## 9. Best Practices (Recommended)
#### Do:
- Use AdminExtensions for cross-cutting logic
- Keep controllers thin
- Use lifecycle hooks for business rules
- Centralize permissions in Admin classes

#### Avoid:
- Heavy logic in controllers
- DB writes in configure*() methods
- Duplicate logic across admins

## 10. Compatibility Matrix

Component	Status

PHP 7.4	✅ Supported

Symfony 4.4	✅ LTS

Sonata Admin 3.73	✅ Stable

ContainerAware	⚠ Legacy

AdminExtensionInterface	✅ Recommended

## 11. Summary Cheat Sheet

Feature	Location

UI configuration	Admin / Extension

Reusable logic	AdminExtension

Lifecycle hooks	Both

Routes	Admin / Controller

Workflows	CRUDController

Validation	Admin / Extension

Security	Admin
