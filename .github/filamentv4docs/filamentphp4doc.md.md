---
title: Getting started
---

---

<!-- Source: 01-getting-started.md -->

import Aside from "@components/Aside.astro"

## Introduction

While Filament comes with virtually any tool you'll need to build great apps, sometimes you'll need to add your own functionality either for just your app or as redistributable packages that other developers can include in their own apps. This is why Filament offers a plugin system that allows you to extend its functionality.

Before we dive in, it's important to understand the different contexts in which plugins can be used. There are two main contexts:

1. **Panel Plugins**: These are plugins that are used with [Panel Builders](../introduction/installation). They are typically used only to add functionality when used inside a Panel or as a complete Panel in and of itself. Examples of this are:
   1. A plugin that adds specific functionality to the dashboard in the form of Widgets.
   2. A plugin that adds a set of Resources / functionality to an app like a Blog or User Management feature.
2. **Standalone Plugins**: These are plugins that are used in any context outside a Panel Builder. Examples of this are:
   1. A plugin that adds custom fields to be used with the [Form Builders](../forms/installation/).
   2. A plugin that adds custom columns or filters to the [Table Builders](../tables/installation/).

Although these are two different mental contexts to keep in mind when building plugins, they can be used together inside the same plugin. They do not have to be mutually exclusive.

## Important concepts

Before we dive into the specifics of building plugins, there are a few concepts that are important to understand. You should familiarize yourself with the following before building a plugin:

1. [Laravel Package Development](https://laravel.com/docs/packages)
2. [Spatie Package Tools](https://github.com/spatie/laravel-package-tools)
3. [Filament Asset Management](../advanced/assets)

### The Plugin object

Filament introduces the concept of a Plugin object that is used to configure the plugin. This object is a simple PHP class that implements the `Filament\Contracts\Plugin` interface. This class is used to configure the plugin and is the main entry point for the plugin. It is also used to register Resources and Icons that might be used by your plugin.

While the plugin object is extremely helpful, it is not required to build a plugin. You can still build plugins without using the plugin object as you can see in the [building a panel plugin](building-a-panel-plugin) tutorial.

<Aside variant="info">
   The Plugin object is only used for Panel Providers. Standalone Plugins do not use this object. All configuration for Standalone Plugins should be handled in the plugin's service provider.
</Aside>

### Registering assets

All [asset registration](../advanced/assets), including CSS, JS and Alpine Components, should be done through the plugin's service provider in the `packageBooted()` method. This allows Filament to register the assets with the Asset Manager and load them when needed.

## Creating a plugin

While you can certainly build plugins from scratch, we recommend using the [Filament Plugin Skeleton](https://github.com/filamentphp/plugin-skeleton) to quickly get started. This skeleton includes all the necessary boilerplate to get you up and running quickly.

### Usage

To use the skeleton, simply go to the GitHub repo and click the "Use this template" button. This will create a new repo in your account with the skeleton code. After that, you can clone the repo to your machine. Once you have the code on your machine, navigate to the root of the project and run the following command:

```bash
php ./configure.php
```

This will ask you a series of questions to configure the plugin. Once you've answered all the questions, the script will stub out a new plugin for you, and you can begin to build your amazing new extension for Filament.

## Upgrading existing plugins

Since every plugin varies greatly in its scope of use and functionality, there is no one size fits all approaches to upgrading existing plugins. However, one thing to note, that is consistent to all plugins is the deprecation of the `PluginServiceProvider`.

In your plugin service provider, you will need to change it to extend the PackageServiceProvider instead. You will also need to add a static `$name` property to the service provider. This property is used to register the plugin with Filament. Here is an example of what your service provider might look like:

```php
class MyPluginServiceProvider extends PackageServiceProvider
{
    public static string $name = 'my-plugin';

    public function configurePackage(Package $package): void
    {
        $package->name(static::$name);
    }
}
```

### Helpful links

Please read this guide in its entirety before upgrading your plugin. It will help you understand the concepts and how to build your plugin.

1. [Filament Asset Management](../advanced/assets)
2. [Panel Plugin Development](../panels/plugins)
3. [Icon Management](../styling/icons)
4. [Colors Management](../styling/colors)
5. [CSS Hooks](../styling/css-hooks)

---

<!-- Source: 01-overview (2).md -->

import Aside from "@components/Aside.astro"

## Introduction

Resources are static classes that are used to build CRUD interfaces for your Eloquent models. They describe how administrators should be able to interact with data from your app using tables and forms.

## Creating a resource

To create a resource for the `App\Models\Customer` model:

```bash
php artisan make:filament-resource Customer
```

This will create several files in the `app/Filament/Resources` directory:

```
.
+-- Customers
|   +-- CustomerResource.php
|   +-- Pages
|   |   +-- CreateCustomer.php
|   |   +-- EditCustomer.php
|   |   +-- ListCustomers.php
|   +-- Schemas
|   |   +-- CustomerForm.php
|   +-- Tables
|   |   +-- CustomersTable.php
```

Your new resource class lives in `CustomerResource.php`.

The classes in the `Pages` directory are used to customize the pages in the app that interact with your resource. They're all full-page [Livewire](https://livewire.laravel.com) components that you can customize in any way you wish.

The classes in the `Schemas` directory are used to define the content of the [forms](../forms) and [infolists](../infolists) for your resource. The classes in the `Tables` directory are used to build the table for your resource.

<Aside variant="tip">
    Have you created a resource, but it's not appearing in the navigation menu? If you have a [model policy](#authorization), make sure you return `true` from the `viewAny()` method.
</Aside>

### Simple (modal) resources

Sometimes, your models are simple enough that you only want to manage records on one page, using modals to create, edit and delete records. To generate a simple resource with modals:

```bash
php artisan make:filament-resource Customer --simple
```

Your resource will have a "Manage" page, which is a List page with modals added.

Additionally, your simple resource will have no `getRelations()` method, as [relation managers](managing-relationships) are only displayed on the Edit and View pages, which are not present in simple resources. Everything else is the same.

### Automatically generating forms and tables

If you'd like to save time, Filament can automatically generate the [form](#resource-forms) and [table](#resource-tables) for you, based on your model's database columns, using `--generate`:

```bash
php artisan make:filament-resource Customer --generate
```

### Handling soft-deletes

By default, you will not be able to interact with deleted records in the app. If you'd like to add functionality to restore, force-delete and filter trashed records in your resource, use the `--soft-deletes` flag when generating the resource:

```bash
php artisan make:filament-resource Customer --soft-deletes
```

You can find out more about soft-deleting [here](deleting-records#handling-soft-deletes).

### Generating a View page

By default, only List, Create and Edit pages are generated for your resource. If you'd also like a [View page](viewing-records), use the `--view` flag:

```bash
php artisan make:filament-resource Customer --view
```

### Specifying a custom model namespace

By default, Filament will assume that your model exists in the `App\Models` directory. You can pass a different namespace for the model using the `--model-namespace` flag:

```bash
php artisan make:filament-resource Customer --model-namespace=Custom\\Path\\Models
```

In this example, the model should exist at `Custom\Path\Models\Customer`. Please note the double backslashes `\\` in the command that are required.

Now when [generating the resource](#automatically-generating-forms-and-tables), Filament will be able to locate the model and read the database schema.

### Generating the model, migration and factory at the same name

If you'd like to save time when scaffolding your resources, Filament can also generate the model, migration and factory for the new resource at the same time using the `--model`, `--migration` and `--factory` flags in any combination:

```bash
php artisan make:filament-resource Customer --model --migration --factory
```

## Record titles

A `$recordTitleAttribute` may be set for your resource, which is the name of the column on your model that can be used to identify it from others.

For example, this could be a blog post's `title` or a customer's `name`:

```php
protected static ?string $recordTitleAttribute = 'name';
```

This is required for features like [global search](global-search) to work.

<Aside variant="tip">
    You may specify the name of an [Eloquent accessor](https://laravel.com/docs/eloquent-mutators#defining-an-accessor) if just one column is inadequate at identifying a record.
</Aside>

## Resource forms

Resource classes contain a `form()` method that is used to build the forms on the [Create](creating-records) and [Edit](editing-records) pages.

By default, Filament creates a form schema file for you, which is referenced in the `form()` method. This is to keep your resource class clean and organized, otherwise it can get quite large:

```php
use App\Filament\Resources\Customers\Schemas\CustomerForm;
use Filament\Schemas\Schema;

public static function form(Schema $schema): Schema
{
    return CustomerForm::configure($schema);
}
```

In the `CustomerForm` class, you can define the fields and layout of your form:

```php
use Filament\Forms\Components\TextInput;
use Filament\Schemas\Schema;

public static function configure(Schema $schema): Schema
{
    return $schema
        ->components([
            TextInput::make('name')->required(),
            TextInput::make('email')->email()->required(),
            // ...
        ]);
}
```

The `components()` method is used to define the structure of your form. It is an array of [fields](../forms#available-fields) and [layout components](../schemas/layout#available-layout-components), in the order they should appear in your form.

Check out the Forms docs for a [guide](../forms) on how to build forms with Filament.

<Aside variant="tip">
    If you would prefer to define the form directly in the resource class, you can do so and delete the form schema class altogether:

    ```php
    use Filament\Forms\Components\TextInput;
    use Filament\Schemas\Schema;

    public static function form(Schema $schema): Schema
    {
        return $schema
            ->components([
                TextInput::make('name')->required(),
                TextInput::make('email')->email()->required(),
                // ...
            ]);
    }
    ```
</Aside>

### Hiding components based on the current operation

The `hiddenOn()` method of form components allows you to dynamically hide fields based on the current page or action.

In this example, we hide the `password` field on the `edit` page:

```php
use Filament\Forms\Components\TextInput;
use Filament\Support\Enums\Operation;

TextInput::make('password')
    ->password()
    ->required()
    ->hiddenOn(Operation::Edit),
```

Alternatively, we have a `visibleOn()` shortcut method for only showing a field on one page or action:

```php
use Filament\Forms\Components\TextInput;
use Filament\Support\Enums\Operation;

TextInput::make('password')
    ->password()
    ->required()
    ->visibleOn(Operation::Create),
```

## Resource tables

Resource classes contain a `table()` method that is used to build the table on the [List page](listing-records).

By default, Filament creates a table file for you, which is referenced in the `table()` method. This is to keep your resource class clean and organized, otherwise it can get quite large:

```php
use App\Filament\Resources\Customers\Schemas\CustomersTable;
use Filament\Tables\Table;

public static function table(Table $table): Table
{
    return CustomersTable::configure($table);
}
```

In the `CustomersTable` class, you can define the columns, filters and actions of the table:

```php
use Filament\Actions\BulkActionGroup;
use Filament\Actions\DeleteBulkAction;
use Filament\Actions\EditAction;
use Filament\Tables\Columns\TextColumn;
use Filament\Tables\Filters\Filter;
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;

public static function configure(Table $table): Table
{
    return $table
        ->columns([
            TextColumn::make('name'),
            TextColumn::make('email'),
            // ...
        ])
        ->filters([
            Filter::make('verified')
                ->query(fn (Builder $query): Builder => $query->whereNotNull('email_verified_at')),
            // ...
        ])
        ->recordActions([
            EditAction::make(),
        ])
        ->toolbarActions([
            BulkActionGroup::make([
                DeleteBulkAction::make(),
            ]),
        ]);
}
```

Check out the [tables](../tables) docs to find out how to add table columns, filters, actions and more.

<Aside variant="tip">
    If you would prefer to define the table directly in the resource class, you can do so and delete the table class altogether:

    ```php
    use Filament\Actions\BulkActionGroup;
    use Filament\Actions\DeleteBulkAction;
    use Filament\Actions\EditAction;
    use Filament\Tables\Columns\TextColumn;
    use Filament\Tables\Filters\Filter;
    use Filament\Tables\Table;
    use Illuminate\Database\Eloquent\Builder;
    
    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                TextColumn::make('name'),
                TextColumn::make('email'),
                // ...
            ])
            ->filters([
                Filter::make('verified')
                    ->query(fn (Builder $query): Builder => $query->whereNotNull('email_verified_at')),
                // ...
            ])
            ->recordActions([
                EditAction::make(),
            ])
            ->toolbarActions([
                BulkActionGroup::make([
                    DeleteBulkAction::make(),
                ]),
            ]);
    }
    ```
</Aside>

## Customizing the model label

Each resource has a "model label" which is automatically generated from the model name. For example, an `App\Models\Customer` model will have a `customer` label.

The label is used in several parts of the UI, and you may customize it using the `$modelLabel` property:

```php
protected static ?string $modelLabel = 'cliente';
```

Alternatively, you may use the `getModelLabel()` to define a dynamic label:

```php
public static function getModelLabel(): string
{
    return __('filament/resources/customer.label');
}
```

### Customizing the plural model label

Resources also have a "plural model label" which is automatically generated from the model label. For example, a `customer` label will be pluralized into `customers`.

You may customize the plural version of the label using the `$pluralModelLabel` property:

```php
protected static ?string $pluralModelLabel = 'clientes';
```

Alternatively, you may set a dynamic plural label in the `getPluralModelLabel()` method:

```php
public static function getPluralModelLabel(): string
{
    return __('filament/resources/customer.plural_label');
}
```

### Automatic model label capitalization

By default, Filament will automatically capitalize each word in the model label, for some parts of the UI. For example, in page titles, the navigation menu, and the breadcrumbs.

If you want to disable this behavior for a resource, you can set `$hasTitleCaseModelLabel` in the resource:

```php
protected static bool $hasTitleCaseModelLabel = false;
```

## Resource navigation items

Filament will automatically generate a navigation menu item for your resource using the [plural label](#plural-label).

If you'd like to customize the navigation item label, you may use the `$navigationLabel` property:

```php
protected static ?string $navigationLabel = 'Mis Clientes';
```

Alternatively, you may set a dynamic navigation label in the `getNavigationLabel()` method:

```php
public static function getNavigationLabel(): string
{
    return __('filament/resources/customer.navigation_label');
}
```

### Setting a resource navigation icon

The `$navigationIcon` property supports the name of any Blade component. By default, [Heroicons](https://heroicons.com) are installed. However, you may create your own custom icon components or install an alternative library if you wish.

```php
use BackedEnum;

protected static string | BackedEnum | null $navigationIcon = 'heroicon-o-user-group';
```

Alternatively, you may set a dynamic navigation icon in the `getNavigationIcon()` method:

```php
use BackedEnum;
use Illuminate\Contracts\Support\Htmlable;

public static function getNavigationIcon(): string | BackedEnum | Htmlable | null
{
    return 'heroicon-o-user-group';
}
```

### Sorting resource navigation items

The `$navigationSort` property allows you to specify the order in which navigation items are listed:

```php
protected static ?int $navigationSort = 2;
```

Alternatively, you may set a dynamic navigation item order in the `getNavigationSort()` method:

```php
public static function getNavigationSort(): ?int
{
    return 2;
}
```

### Grouping resource navigation items

You may group navigation items by specifying a `$navigationGroup` property:

```php
use UnitEnum;

protected static string | UnitEnum | null $navigationGroup = 'Shop';
```

Alternatively, you may use the `getNavigationGroup()` method to set a dynamic group label:

```php
public static function getNavigationGroup(): ?string
{
    return __('filament/navigation.groups.shop');
}
```

#### Grouping resource navigation items under other items

You may group navigation items as children of other items, by passing the label of the parent item as the `$navigationParentItem`:

```php
use UnitEnum;

protected static ?string $navigationParentItem = 'Products';

protected static string | UnitEnum | null $navigationGroup = 'Shop';
```

As seen above, if the parent item has a navigation group, that navigation group must also be defined, so the correct parent item can be identified.

You may also use the `getNavigationParentItem()` method to set a dynamic parent item label:

```php
public static function getNavigationParentItem(): ?string
{
    return __('filament/navigation.groups.shop.items.products');
}
```

<Aside variant="tip">
    If you're reaching for a third level of navigation like this, you should consider using [clusters](../navigation/clusters) instead, which are a logical grouping of resources and [custom pages](../navigation/custom-pages), which can share their own separate navigation.
</Aside>

## Generating URLs to resource pages

Filament provides `getUrl()` static method on resource classes to generate URLs to resources and specific pages within them. Traditionally, you would need to construct the URL by hand or by using Laravel's `route()` helper, but these methods depend on knowledge of the resource's slug or route naming conventions.

The `getUrl()` method, without any arguments, will generate a URL to the resource's [List page](listing-records):

```php
use App\Filament\Resources\Customers\CustomerResource;

CustomerResource::getUrl(); // /admin/customers
```

You may also generate URLs to specific pages within the resource. The name of each page is the array key in the `getPages()` array of the resource. For example, to generate a URL to the [Create page](creating-records):

```php
use App\Filament\Resources\Customers\CustomerResource;

CustomerResource::getUrl('create'); // /admin/customers/create
```

Some pages in the `getPages()` method use URL parameters like `record`. To generate a URL to these pages and pass in a record, you should use the second argument:

```php
use App\Filament\Resources\Customers\CustomerResource;

CustomerResource::getUrl('edit', ['record' => $customer]); // /admin/customers/edit/1
```

In this example, `$customer` can be an Eloquent model object, or an ID.

### Generating URLs to resource modals

This can be especially useful if you are using [simple resources](#simple-modal-resources) with only one page.

To generate a URL for an action in the resource's table, you should pass the `tableAction` and `tableActionRecord` as URL parameters:

```php
use App\Filament\Resources\Customers\CustomerResource;
use Filament\Actions\EditAction;

CustomerResource::getUrl(parameters: [
    'tableAction' => EditAction::getDefaultName(),
    'tableActionRecord' => $customer,
]); // /admin/customers?tableAction=edit&tableActionRecord=1
```

Or if you want to generate a URL for an action on the page like a `CreateAction` in the header, you can pass it in to the `action` parameter:

```php
use App\Filament\Resources\Customers\CustomerResource;
use Filament\Actions\CreateAction;

CustomerResource::getUrl(parameters: [
    'action' => CreateAction::getDefaultName(),
]); // /admin/customers?action=create
```

### Generating URLs to resources in other panels

If you have multiple panels in your app, `getUrl()` will generate a URL within the current panel. You can also indicate which panel the resource is associated with, by passing the panel ID to the `panel` argument:

```php
use App\Filament\Resources\Customers\CustomerResource;

CustomerResource::getUrl(panel: 'marketing');
```

## Customizing the resource Eloquent query

Within Filament, every query to your resource model will start with the `getEloquentQuery()` method.

Because of this, it's very easy to apply your own query constraints or [model scopes](https://laravel.com/docs/eloquent#query-scopes) that affect the entire resource:

```php
public static function getEloquentQuery(): Builder
{
    return parent::getEloquentQuery()->where('is_active', true);
}
```

### Disabling global scopes

By default, Filament will observe all global scopes that are registered to your model. However, this may not be ideal if you wish to access, for example, soft-deleted records.

To overcome this, you may override the `getEloquentQuery()` method that Filament uses:

```php
public static function getEloquentQuery(): Builder
{
    return parent::getEloquentQuery()->withoutGlobalScopes();
}
```

Alternatively, you may remove specific global scopes:

```php
public static function getEloquentQuery(): Builder
{
    return parent::getEloquentQuery()->withoutGlobalScopes([ActiveScope::class]);
}
```

More information about removing global scopes may be found in the [Laravel documentation](https://laravel.com/docs/eloquent#removing-global-scopes).

## Customizing the resource URL

By default, Filament will generate a URL based on the name of the resource. You can customize this by setting the `$slug` property on the resource:

```php
protected static ?string $slug = 'pending-orders';
```

## Resource sub-navigation

Sub-navigation allows the user to navigate between different pages within a resource. Typically, all pages in the sub-navigation will be related to the same record in the resource. For example, in a Customer resource, you may have a sub-navigation with the following pages:

- View customer, a [`ViewRecord` page](viewing-records) that provides a read-only view of the customer's details.
- Edit customer, an [`EditRecord` page](editing-records) that allows the user to edit the customer's details.
- Edit customer contact, an [`EditRecord` page](editing-records) that allows the user to edit the customer's contact details. You can [learn how to create more than one Edit page](editing-records#creating-another-edit-page).
- Manage addresses, a [`ManageRelatedRecords` page](managing-relationships#relation-pages) that allows the user to manage the customer's addresses.
- Manage payments, a [`ManageRelatedRecords` page](managing-relationships#relation-pages) that allows the user to manage the customer's payments.

To add a sub-navigation to each "singular record" page in the resource, you can add the `getRecordSubNavigation()` method to the resource class:

```php
use Filament\Resources\Pages\Page;

public static function getRecordSubNavigation(Page $page): array
{
    return $page->generateNavigationItems([
        ViewCustomer::class,
        EditCustomer::class,
        EditCustomerContact::class,
        ManageCustomerAddresses::class,
        ManageCustomerPayments::class,
    ]);
}
```

Each item in the sub-navigation can be customized using the [same navigation methods as normal pages](../navigation).

<Aside variant="tip">
    If you're looking to add sub-navigation to switch *between* entire resources and [custom pages](../navigation/custom-pages), you might be looking for [clusters](../navigation/clusters), which are used to group these together. The `getRecordSubNavigation()` method is intended to construct a navigation between pages that relate to a particular record *inside* a resource.
</Aside>

### Setting the sub-navigation position for a resource

The sub-navigation is rendered at the start of the page by default. You may change the position for all pages in a resource by setting the `$subNavigationPosition` property on the resource. The value may be `SubNavigationPosition::Start`, `SubNavigationPosition::End`, or `SubNavigationPosition::Top` to render the sub-navigation as tabs:

```php
use Filament\Pages\Enums\SubNavigationPosition;

protected static ?SubNavigationPosition $subNavigationPosition = SubNavigationPosition::End;
```

## Deleting resource pages

If you'd like to delete a page from your resource, you can just delete the page file from the `Pages` directory of your resource, and its entry in the `getPages()` method.

For example, you may have a resource with records that may not be created by anyone. Delete the `Create` page file, and then remove it from `getPages()`:

```php
public static function getPages(): array
{
    return [
        'index' => ListCustomers::route('/'),
        'edit' => EditCustomer::route('/{record}/edit'),
    ];
}
```

Deleting a page will not delete any actions that link to that page. Any actions will open a modal instead of sending the user to the non-existent page. For instance, the `CreateAction` on the List page, the `EditAction` on the table or View page, or the `ViewAction` on the table or Edit page. If you want to remove those buttons, you must delete the actions as well.

## Security

## Authorization

For authorization, Filament will observe any [model policies](https://laravel.com/docs/authorization#creating-policies) that are registered in your app. The following methods are used:

- `viewAny()` is used to completely hide resources from the navigation menu, and prevents the user from accessing any pages.
- `create()` is used to control [creating new records](creating-records).
- `update()` is used to control [editing a record](editing-records).
- `view()` is used to control [viewing a record](viewing-records).
- `delete()` is used to prevent a single record from being deleted. `deleteAny()` is used to prevent records from being bulk deleted. Filament uses the `deleteAny()` method because iterating through multiple records and checking the `delete()` policy is not very performant. When using a `DeleteBulkAction`, if you want to call the `delete()` method for each record anyway, you should use the `DeleteBulkAction::make()->authorizeIndividualRecords()` method. Any records that fail the authorization check will not be processed.
- `forceDelete()` is used to prevent a single soft-deleted record from being force-deleted. `forceDeleteAny()` is used to prevent records from being bulk force-deleted. Filament uses the `forceDeleteAny()` method because iterating through multiple records and checking the `forceDelete()` policy is not very performant. When using a `ForceDeleteBulkAction`, if you want to call the `forceDelete()` method for each record anyway, you should use the `ForceDeleteBulkAction::make()->authorizeIndividualRecords()` method. Any records that fail the authorization check will not be processed.
- `restore()` is used to prevent a single soft-deleted record from being restored. `restoreAny()` is used to prevent records from being bulk restored. Filament uses the `restoreAny()` method because iterating through multiple records and checking the `restore()` policy is not very performant. When using a `RestoreBulkAction`, if you want to call the `restore()` method for each record anyway, you should use the `RestoreBulkAction::make()->authorizeIndividualRecords()` method. Any records that fail the authorization check will not be processed.
- `reorder()` is used to control [reordering records in a table](listing-records#reordering-records).

### Skipping authorization

If you'd like to skip authorization for a resource, you may set the `$shouldSkipAuthorization` property to `true`:

```php
protected static bool $shouldSkipAuthorization = true;
```

### Protecting model attributes

Filament will expose all model attributes to JavaScript, except if they are `$hidden` on your model. This is Livewire's behavior for model binding. We preserve this functionality to facilitate the dynamic addition and removal of form fields after they are initially loaded, while preserving the data they may need.

<Aside variant="danger">
    While attributes may be visible in JavaScript, only those with a form field are actually editable by the user. This is not an issue with mass assignment.
</Aside>

To remove certain attributes from JavaScript on the Edit and View pages, you may override [the `mutateFormDataBeforeFill()` method](editing-records#customizing-data-before-filling-the-form):

```php
protected function mutateFormDataBeforeFill(array $data): array
{
    unset($data['is_admin']);

    return $data;
}
```

In this example, we remove the `is_admin` attribute from JavaScript, as it's not being used by the form.

---

<!-- Source: 01-overview (3).md -->

import Aside from "@components/Aside.astro"
import AutoScreenshot from "@components/AutoScreenshot.astro"

## Introduction

By default, Filament will register navigation items for each of your [resources](../resources), [custom pages](custom-pages), and [clusters](clusters). These classes contain static properties and methods that you can override, to configure that navigation item.

If you're looking to add a second layer of navigation to your app, you can use [clusters](clusters). These are useful for grouping resources and pages together.

## Customizing a navigation item's label

By default, the navigation label is generated from the resource or page's name. You may customize this using the `$navigationLabel` property:

```php
protected static ?string $navigationLabel = 'Custom Navigation Label';
```

Alternatively, you may override the `getNavigationLabel()` method:

```php
public static function getNavigationLabel(): string
{
    return 'Custom Navigation Label';
}
```

## Customizing a navigation item's icon

To customize a navigation item's [icon](../styling/icons), you may override the `$navigationIcon` property on the [resource](resources) or [page](pages) class:

```php
use BackedEnum;

protected static string | BackedEnum | null $navigationIcon = 'heroicon-o-document-text';
```

<AutoScreenshot name="panels/navigation/change-icon" alt="Changed navigation item icon" version="3.x" />

If you set `$navigationIcon = null` on all items within the same navigation group, those items will be joined with a vertical bar below the group label.

### Switching navigation item icon when it is active

You may assign a navigation [icon](../styling/icons) which will only be used for active items using the `$activeNavigationIcon` property:

```php
protected static ?string $activeNavigationIcon = 'heroicon-o-document-text';
```

<AutoScreenshot name="panels/navigation/active-icon" alt="Different navigation item icon when active" version="3.x" />

## Sorting navigation items

By default, navigation items are sorted alphabetically. You may customize this using the `$navigationSort` property:

```php
protected static ?int $navigationSort = 3;
```

Now, navigation items with a lower sort value will appear before those with a higher sort value - the order is ascending.

<AutoScreenshot name="panels/navigation/sort-items" alt="Sort navigation items" version="3.x" />

## Adding a badge to a navigation item

To add a badge next to the navigation item, you can use the `getNavigationBadge()` method and return the content of the badge:

```php
public static function getNavigationBadge(): ?string
{
    return static::getModel()::count();
}
```

<AutoScreenshot name="panels/navigation/badge" alt="Navigation item with badge" version="3.x" />

If a badge value is returned by `getNavigationBadge()`, it will display using the primary color by default. To style the badge contextually, return either `danger`, `gray`, `info`, `primary`, `success` or `warning` from the `getNavigationBadgeColor()` method:

```php
public static function getNavigationBadgeColor(): ?string
{
    return static::getModel()::count() > 10 ? 'warning' : 'primary';
}
```

<AutoScreenshot name="panels/navigation/badge-color" alt="Navigation item with badge color" version="3.x" />

A custom tooltip for the navigation badge can be set in `$navigationBadgeTooltip`:

```php
protected static ?string $navigationBadgeTooltip = 'The number of users';
```

Or it can be returned from `getNavigationBadgeTooltip()`:

```php
public static function getNavigationBadgeTooltip(): ?string
{
    return 'The number of users';
}
```

<AutoScreenshot name="panels/navigation/badge-tooltip" alt="Navigation item with badge tooltip" version="3.x" />

## Grouping navigation items

You may group navigation items by specifying a `$navigationGroup` property on a [resource](resources) and [custom page](custom-pages):

```php
use UnitEnum;

protected static string | UnitEnum | null $navigationGroup = 'Settings';
```

<AutoScreenshot name="panels/navigation/group" alt="Grouped navigation items" version="3.x" />

All items in the same navigation group will be displayed together under the same group label, "Settings" in this case. Ungrouped items will remain at the start of the navigation.

### Grouping navigation items under other items

You may group navigation items as children of other items, by passing the label of the parent item as the `$navigationParentItem`:

```php
use UnitEnum;

protected static ?string $navigationParentItem = 'Notifications';

protected static string | UnitEnum | null $navigationGroup = 'Settings';
```

You may also use the `getNavigationParentItem()` method to set a dynamic parent item label:

```php
public static function getNavigationParentItem(): ?string
{
    return __('filament/navigation.groups.settings.items.notifications');
}
```

As seen above, if the parent item has a navigation group, that navigation group must also be defined, so the correct parent item can be identified.

<Aside variant="tip">
    If you're reaching for a third level of navigation like this, you should consider using [clusters](clusters) instead, which are a logical grouping of resources and custom pages, which can share their own separate navigation.
</Aside>

### Customizing navigation groups

You may customize navigation groups by calling `navigationGroups()` in the [configuration](../panel-configuration), and passing `NavigationGroup` objects in order:

```php
use Filament\Navigation\NavigationGroup;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->navigationGroups([
            NavigationGroup::make()
                 ->label('Shop')
                 ->icon('heroicon-o-shopping-cart'),
            NavigationGroup::make()
                ->label('Blog')
                ->icon('heroicon-o-pencil'),
            NavigationGroup::make()
                ->label(fn (): string => __('navigation.settings'))
                ->icon('heroicon-o-cog-6-tooth')
                ->collapsed(),
        ]);
}
```

In this example, we pass in a custom `icon()` for the groups, and make one `collapsed()` by default.

#### Ordering navigation groups

By using `navigationGroups()`, you are defining a new order for the navigation groups. If you just want to reorder the groups and not define an entire `NavigationGroup` object, you may just pass the labels of the groups in the new order:

```php
$panel
    ->navigationGroups([
        'Shop',
        'Blog',
        'Settings',
    ])
```

#### Making navigation groups not collapsible

By default, navigation groups are collapsible.

<AutoScreenshot name="panels/navigation/group-collapsible" alt="Collapsible navigation groups" version="3.x" />

You may disable this behavior by calling `collapsible(false)` on the `NavigationGroup` object:

```php
use Filament\Navigation\NavigationGroup;

NavigationGroup::make()
    ->label('Settings')
    ->icon('heroicon-o-cog-6-tooth')
    ->collapsible(false);
```

<AutoScreenshot name="panels/navigation/group-not-collapsible" alt="Not collapsible navigation groups" version="3.x" />

Or, you can do it globally for all groups in the [configuration](../panel-configuration):

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->collapsibleNavigationGroups(false);
}
```

#### Adding extra HTML attributes to navigation groups

You can pass extra HTML attributes to the navigation group, which will be merged onto the outer DOM element. Pass an array of attributes to the `extraSidebarAttributes()` or `extraTopbarAttributes()` method, where the key is the attribute name and the value is the attribute value:

```php
NavigationGroup::make()
    ->extraSidebarAttributes(['class' => 'featured-sidebar-group']),
    ->extraTopbarAttributes(['class' => 'featured-topbar-group']),
```

The `extraSidebarAttributes()` will be applied to navigation group elements contained in the sidebar, and the `extraTopbarAttributes()` will only be applied to topbar navigation group dropdowns when using [top navigation](#using-top-navigation).

### Registering navigation groups with an enum

You can use an enum class to register navigation groups, which allows you to control their labels, icons, and order in a single place, without needing to register them in the [configuration](../panel-configuration).

To do this, you can create an enum class with cases for each group:

```php
enum NavigationGroup
{
    case Shop;
    
    case Blog;
    
    case Settings;
}
```

To order that the cases are defined in will control the order of the navigation groups.

To use an enum navigation group for a resource or custom page, you can set the `$navigationGroup` property to the enum case:

```php
protected static string | UnitEnum | null $navigationGroup = NavigationGroup::Shop;
```

You can also implement the `HasLabel` interface on the enum class, to define a custom label for each group:

```php
use Filament\Support\Contracts\HasLabel;

enum NavigationGroup implements HasLabel
{
    case Shop;
    
    case Blog;
    
    case Settings;

    public function getLabel(): string
    {
        return match ($this) {
            self::Shop => __('navigation-groups.shop'),
            self::Blog => __('navigation-groups.blog'),
            self::Settings => __('navigation-groups.settings'),
        };
    }
}
```

You can also implement the `HasIcon` interface on the enum class, to define a custom icon for each group:

```php
use Filament\Support\Contracts\HasIcon;

enum NavigationGroup implements HasIcon
{
    case Shop;
    
    case Blog;
    
    case Settings;

    public function getIcon(): ?string
    {
        return match ($this) {
            self::Shop => 'heroicon-o-shopping-cart',
            self::Blog => 'heroicon-o-pencil',
            self::Settings => 'heroicon-o-cog-6-tooth',
        };
    }
}
```

## Collapsible sidebar on desktop

To make the sidebar collapsible on desktop as well as mobile, you can use the [configuration](../panel-configuration):

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->sidebarCollapsibleOnDesktop();
}
```

<AutoScreenshot name="panels/navigation/sidebar-collapsible-on-desktop" alt="Collapsible sidebar on desktop" version="3.x" />

By default, when you collapse the sidebar on desktop, the navigation icons still show. You can fully collapse the sidebar using the `sidebarFullyCollapsibleOnDesktop()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->sidebarFullyCollapsibleOnDesktop();
}
```

<AutoScreenshot name="panels/navigation/sidebar-fully-collapsible-on-desktop" alt="Fully collapsible sidebar on desktop" version="3.x" />

### Navigation groups in a collapsible sidebar on desktop

<Aside variant="info">
    This section only applies to `sidebarCollapsibleOnDesktop()`, not `sidebarFullyCollapsibleOnDesktop()`, since the fully collapsible UI just hides the entire sidebar instead of changing its design.
</Aside>

When using a collapsible sidebar on desktop, you will also often be using [navigation groups](#grouping-navigation-items). By default, the labels of each navigation group will be hidden when the sidebar is collapsed, since there is no space to display them. Even if the navigation group itself is [collapsible](#making-navigation-groups-not-collapsible), all items will still be visible in the collapsed sidebar, since there is no group label to click on to expand the group.

These issues can be solved, to achieve a very minimal sidebar design, by [passing an `icon()`](#customizing-navigation-groups) to the navigation group objects. When an icon is defined, the icon will be displayed in the collapsed sidebar instead of the items at all times. When the icon is clicked, a dropdown will open to the side of the icon, revealing the items in the group.

When passing an icon to a navigation group, even if the items also have icons, the expanded sidebar UI will not show the item icons. This is to keep the navigation hierarchy clear, and the design minimal. However, the icons for the items will be shown in the collapsed sidebar's dropdowns though, since the hierarchy is already clear from the fact that the dropdown is open.

## Registering custom navigation items

To register new navigation items, you can use the [configuration](../panel-configuration):

```php
use Filament\Navigation\NavigationItem;
use Filament\Pages\Dashboard;
use Filament\Panel;
use function Filament\Support\original_request;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->navigationItems([
            NavigationItem::make('Analytics')
                ->url('https://filament.pirsch.io', shouldOpenInNewTab: true)
                ->icon('heroicon-o-presentation-chart-line')
                ->group('Reports')
                ->sort(3),
            NavigationItem::make('dashboard')
                ->label(fn (): string => __('filament-panels::pages/dashboard.title'))
                ->url(fn (): string => Dashboard::getUrl())
                ->isActiveWhen(fn () => original_request()->routeIs('filament.admin.pages.dashboard')),
            // ...
        ]);
}
```

## Conditionally hiding navigation items

You can also conditionally hide a navigation item by using the `visible()` or `hidden()` methods, passing in a condition to check:

```php
use Filament\Navigation\NavigationItem;

NavigationItem::make('Analytics')
    ->visible(fn(): bool => auth()->user()->can('view-analytics'))
    // or
    ->hidden(fn(): bool => ! auth()->user()->can('view-analytics')),
```

## Disabling resource or page navigation items

To prevent resources or pages from showing up in navigation, you may use:

```php
protected static bool $shouldRegisterNavigation = false;
```

Or, you may override the `shouldRegisterNavigation()` method:

```php
public static function shouldRegisterNavigation(): bool
{
    return false;
}
```

Please note that these methods do not control direct access to the resource or page. They only control whether the resource or page will show up in the navigation. If you want to also control access, then you should use [resource authorization](../resources#authorization) or [page authorization](custom-pages#authorization).

## Using top navigation

By default, Filament will use a sidebar navigation. You may use a top navigation instead by using the [configuration](../panel-configuration):

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->topNavigation();
}
```

<AutoScreenshot name="panels/navigation/top-navigation" alt="Top navigation" version="3.x" />

## Customizing the width of the sidebar

You can customize the width of the sidebar by passing it to the `sidebarWidth()` method in the [configuration](../panel-configuration):

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->sidebarWidth('40rem');
}
```

Additionally, if you are using the `sidebarCollapsibleOnDesktop()` method, you can customize width of the collapsed icons by using the `collapsedSidebarWidth()` method in the [configuration](../panel-configuration):

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->sidebarCollapsibleOnDesktop()
        ->collapsedSidebarWidth('9rem');
}
```

## Advanced navigation customization

The `navigation()` method can be called from the [configuration](../panel-configuration). It allows you to build a custom navigation that overrides Filament's automatically generated items. This API is designed to give you complete control over the navigation.

### Registering custom navigation items

To register navigation items, call the `items()` method:

```php
use App\Filament\Pages\Settings;
use App\Filament\Resources\Users\UserResource;
use Filament\Navigation\NavigationBuilder;
use Filament\Navigation\NavigationItem;
use Filament\Pages\Dashboard;
use Filament\Panel;
use function Filament\Support\original_request;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->navigation(function (NavigationBuilder $builder): NavigationBuilder {
            return $builder->items([
                NavigationItem::make('Dashboard')
                    ->icon('heroicon-o-home')
                    ->isActiveWhen(fn (): bool => original_request()->routeIs('filament.admin.pages.dashboard'))
                    ->url(fn (): string => Dashboard::getUrl()),
                ...UserResource::getNavigationItems(),
                ...Settings::getNavigationItems(),
            ]);
        });
}
```

<AutoScreenshot name="panels/navigation/custom-items" alt="Custom navigation items" version="3.x" />

### Registering custom navigation groups

If you want to register groups, you can call the `groups()` method:

```php
use App\Filament\Pages\HomePageSettings;
use App\Filament\Resources\Categories\CategoryResource;
use App\Filament\Resources\Pages\PageResource;
use Filament\Navigation\NavigationBuilder;
use Filament\Navigation\NavigationGroup;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->navigation(function (NavigationBuilder $builder): NavigationBuilder {
            return $builder->groups([
                NavigationGroup::make('Website')
                    ->items([
                        ...PageResource::getNavigationItems(),
                        ...CategoryResource::getNavigationItems(),
                        ...HomePageSettings::getNavigationItems(),
                    ]),
            ]);
        });
}
```

### Disabling navigation

You may disable navigation entirely by passing `false` to the `navigation()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->navigation(false);
}
```

<AutoScreenshot name="panels/navigation/disabled-navigation" alt="Disabled navigation sidebar" version="3.x" />

### Disabling the topbar

You may disable topbar entirely by passing `false` to the `topbar()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->topbar(false);
}
```

### Replacing the sidebar and topbar Livewire components

You may completely replace the Livewire components that are used to render the sidebar and topbar, passing your own Livewire component class name into the `sidebarLivewireComponent()` or `topbarLivewireComponent()` method:

```php
use App\Livewire\Sidebar;
use App\Livewire\Topbar;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->sidebarLivewireComponent(Sidebar::class)
        ->topbarLivewireComponent(Topbar::class);
}
```

## Disabling breadcrumbs

The default layout will show breadcrumbs to indicate the location of the current page within the hierarchy of the app.

You may disable breadcrumbs in your [configuration](../panel-configuration):

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->breadcrumbs(false);
}
```

## Reloading the sidebar and topbar

Once a page in the panel is loaded, the sidebar and topbar are not reloaded until you navigate away from the page, or until a menu item is clicked to trigger an action. You can manually reload these components to update them by dispatching a `refresh-sidebar` or `refresh-topbar` browser event.

To dispatch an event from PHP, you can call the `$this->dispatch()` method from any Livewire component, such as a page class, relation manager class, or widget class:

```php
$this->dispatch('refresh-sidebar');
```

When your code does not live inside a Livewire component, such as when you have a custom action class, you can inject the `$livewire` argument into a closure function, and call `dispatch()` on that:

```php
use Filament\Actions\Action;
use Livewire\Component;

Action::make('create')
    ->action(function (Component $livewire) {
        // ...
    
        $livewire->dispatch('refresh-sidebar');
    })
```

Alternatively, you can dispatch an event from JavaScript using the `$dispatch()` Alpine.js helper method, or the native browser `window.dispatchEvent()` method:

```html
<button x-on:click="$dispatch('refresh-sidebar')" type="button">
    Refresh Sidebar
</button>
```

```javascript
window.dispatchEvent(new CustomEvent('refresh-sidebar'));
```

---

<!-- Source: 01-overview (4).md -->

## Introduction

By default, all `App\Models\User`s can access Filament locally. To allow them to access Filament in production, you must take a few extra steps to ensure that only the correct users have access to the app.

## Authorizing access to the panel

To set up your `App\Models\User` to access Filament in non-local environments, you must implement the `FilamentUser` contract:

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\FilamentUser;
use Filament\Panel;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements FilamentUser
{
    // ...

    public function canAccessPanel(Panel $panel): bool
    {
        return str_ends_with($this->email, '@yourdomain.com') && $this->hasVerifiedEmail();
    }
}
```

The `canAccessPanel()` method returns `true` or `false` depending on whether the user is allowed to access the `$panel`. In this example, we check if the user's email ends with `@yourdomain.com` and if they have verified their email address.

Since you have access to the current `$panel`, you can write conditional checks for separate panels. For example, only restricting access to the admin panel while allowing all users to access the other panels of your app:

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\FilamentUser;
use Filament\Panel;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements FilamentUser
{
    // ...

    public function canAccessPanel(Panel $panel): bool
    {
        if ($panel->getId() === 'admin') {
            return str_ends_with($this->email, '@yourdomain.com') && $this->hasVerifiedEmail();
        }

        return true;
    }
}
```

## Authorizing access to Resources

See the [Authorization](../resources/overview#authorization) section in the Resource documentation for controlling access to Resource pages and their data records.

## Setting up user avatars

Out of the box, Filament uses [ui-avatars.com](https://ui-avatars.com) to generate avatars based on a user's name. However, if your user model has an `avatar_url` attribute, that will be used instead. To customize how Filament gets a user's avatar URL, you can implement the `HasAvatar` contract:

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\FilamentUser;
use Filament\Models\Contracts\HasAvatar;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements FilamentUser, HasAvatar
{
    // ...

    public function getFilamentAvatarUrl(): ?string
    {
        return $this->avatar_url;
    }
}
```

The `getFilamentAvatarUrl()` method is used to retrieve the avatar of the current user. If `null` is returned from this method, Filament will fall back to [ui-avatars.com](https://ui-avatars.com).

### Using a different avatar provider

You can easily swap out [ui-avatars.com](https://ui-avatars.com) for a different service, by creating a new avatar provider.

In this example, we create a new file at `app/Filament/AvatarProviders/BoringAvatarsProvider.php` for [boringavatars.com](https://boringavatars.com). The `get()` method accepts a user model instance and returns an avatar URL for that user:

```php
<?php

namespace App\Filament\AvatarProviders;

use Filament\AvatarProviders\Contracts;
use Filament\Facades\Filament;
use Illuminate\Contracts\Auth\Authenticatable;
use Illuminate\Database\Eloquent\Model;

class BoringAvatarsProvider implements Contracts\AvatarProvider
{
    public function get(Model | Authenticatable $record): string
    {
        $name = str(Filament::getNameForDefaultAvatar($record))
            ->trim()
            ->explode(' ')
            ->map(fn (string $segment): string => filled($segment) ? mb_substr($segment, 0, 1) : '')
            ->join(' ');

        return 'https://source.boringavatars.com/beam/120/' . urlencode($name);
    }
}
```

Now, register this new avatar provider in the [configuration](../panel-configuration):

```php
use App\Filament\AvatarProviders\BoringAvatarsProvider;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->defaultAvatarProvider(BoringAvatarsProvider::class);
}
```

## Configuring the user's name attribute

By default, Filament will use the `name` attribute of the user to display their name in the app. To change this, you can implement the `HasName` contract:

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\FilamentUser;
use Filament\Models\Contracts\HasName;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements FilamentUser, HasName
{
    // ...

    public function getFilamentName(): string
    {
        return "{$this->first_name} {$this->last_name}";
    }
}
```

The `getFilamentName()` method is used to retrieve the name of the current user.

## Authentication features

You can easily enable authentication features for a panel in the configuration file:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->login()
        ->registration()
        ->passwordReset()
        ->emailVerification()
        ->emailChangeVerification()
        ->profile();
}
```

Filament also supports multi-factor authentication, which you can learn about in the [Multi-factor authentication](multi-factor-authentication) section.

### Customizing the authentication features

If you'd like to replace these pages with your own, you can pass in any Filament page class to these methods.

Most people will be able to make their desired customizations by extending the base page class from the Filament codebase, overriding methods like `form()`, and then passing the new page class in to the configuration:

```php
use App\Filament\Pages\Auth\EditProfile;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->profile(EditProfile::class);
}
```

In this example, we will customize the profile page. We need to create a new PHP class at `app/Filament/Pages/Auth/EditProfile.php`:

```php
<?php

namespace App\Filament\Pages\Auth;

use Filament\Auth\Pages\EditProfile as BaseEditProfile;
use Filament\Forms\Components\TextInput;
use Filament\Schemas\Schema;

class EditProfile extends BaseEditProfile
{
    public function form(Schema $schema): Schema
    {
        return $schema
            ->components([
                TextInput::make('username')
                    ->required()
                    ->maxLength(255),
                $this->getNameFormComponent(),
                $this->getEmailFormComponent(),
                $this->getPasswordFormComponent(),
                $this->getPasswordConfirmationFormComponent(),
            ]);
    }
}
```

This class extends the base profile page class from the Filament codebase. Other page classes you could extend include:

- `Filament\Auth\Pages\Login`
- `Filament\Auth\Pages\Register`
- `Filament\Auth\Pages\EmailVerification\EmailVerificationPrompt`
- `Filament\Auth\Pages\PasswordReset\RequestPasswordReset`
- `Filament\Auth\Pages\PasswordReset\ResetPassword`

In the `form()` method of the example, we call methods like `getNameFormComponent()` to get the default form components for the page. You can customize these components as required. For all the available customization options, see the base `EditProfile` page class in the Filament codebase - it contains all the methods that you can override to make changes.

#### Customizing an authentication field without needing to re-define the form

If you'd like to customize a field in an authentication form without needing to define a new `form()` method, you could extend the specific field method and chain your customizations:

```php
use Filament\Schemas\Components\Component;

protected function getPasswordFormComponent(): Component
{
    return parent::getPasswordFormComponent()
        ->revealable(false);
}
```

### Email change verification

If you're using the `profile()` feature with the `emailChangeVerification()` feature, users that change their email address from the profile form will be required to verify their new email address before they can log in with it. This is done by sending a verification email to the new address, which contains a link that the user must click to verify their new email address. The email address in the database is not updated until the user clicks the link in the email.

The link that a user is sent is valid for 60 minutes. At the same time as the email to the new address is sent, an email to the old address is also sent, with a link to block the change. This is a security feature to potentially prevent a user from being affected by a malicious actor.

### Using a sidebar on the profile page

By default, the profile page does not use the standard page layout with a sidebar. This is so that it works with the [tenancy](tenancy) feature, otherwise it would not be accessible if the user had no tenants, since the sidebar links are routed to the current tenant.

If you aren't using [tenancy](tenancy) in your panel, and you'd like the profile page to use the standard page layout with a sidebar, you can pass the `isSimple: false` parameter to `$panel->profile()` when registering the page:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->profile(isSimple: false);
}
```

### Customizing the authentication route slugs

You can customize the URL slugs used for the authentication routes in the [configuration](../panel-configuration):

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->loginRouteSlug('login')
        ->registrationRouteSlug('register')
        ->passwordResetRoutePrefix('password-reset')
        ->passwordResetRequestRouteSlug('request')
        ->passwordResetRouteSlug('reset')
        ->emailVerificationRoutePrefix('email-verification')
        ->emailVerificationPromptRouteSlug('prompt')
        ->emailVerificationRouteSlug('verify')
        ->emailChangeVerificationRoutePrefix('email-change-verification')
        ->emailChangeVerificationRouteSlug('verify');
}
```

### Setting the authentication guard

To set the authentication guard that Filament uses, you can pass in the guard name to the `authGuard()` [configuration](../panel-configuration) method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->authGuard('web');
}
```

### Setting the password broker

To set the password broker that Filament uses, you can pass in the broker name to the `authPasswordBroker()` [configuration](../panel-configuration) method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->authPasswordBroker('users');
}
```

### Disabling revealable password inputs

By default, all password inputs in authentication forms are [`revealable()`](../forms/text-input#revealable-password-inputs). This allows the user can see a plain text version of the password they're typing by clicking a button. To disable this feature, you can pass `false` to the `revealablePasswords()` [configuration](../panel-configuration) method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->revealablePasswords(false);
}
```

You could also disable this feature on a per-field basis by calling `->revealable(false)` on the field object when [extending the base page class](#customizing-an-authentication-field-without-needing-to-re-define-the-form).

## Setting up guest access to a panel

By default, Filament expects to work with authenticated users only. To allow guests to access a panel, you need to avoid using components which expect a signed-in user (such as profiles, avatars), and remove the built-in Authentication middleware:

- Remove the default `Authenticate::class` from the `authMiddleware()` array in the panel configuration.
- Remove `->login()` and any other [authentication features](#authentication-features) from the panel.
- Remove the default `AccountWidget` from the `widgets()` array, because it reads the current user's data.

### Authorizing guests in policies

When present, Filament relies on [Laravel Model Policies](https://laravel.com/docs/authorization#generating-policies) for access control. To give read-access for [guest users in a model policy](https://laravel.com/docs/authorization#guest-users), create the Policy and update the `viewAny()` and `view()` methods, changing the `User $user` param to `?User $user` so that it's optional, and `return true;`. Alternatively, you can remove those methods from the policy entirely.

---

<!-- Source: 01-overview (5).md -->

import Aside from "@components/Aside.astro"

## Changing the colors

In the [configuration](../panel-configuration), you can easily change the colors that are used. Filament ships with 6 predefined colors that are used everywhere within the framework. They are customizable as follows:

```php
use Filament\Panel;
use Filament\Support\Colors\Color;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->colors([
            'danger' => Color::Rose,
            'gray' => Color::Gray,
            'info' => Color::Blue,
            'primary' => Color::Indigo,
            'success' => Color::Emerald,
            'warning' => Color::Orange,
        ]);
}
```

The `Filament\Support\Colors\Color` class contains color options for all [Tailwind CSS color palettes](https://tailwindcss.com/docs/customizing-colors).

You can also pass in a function to `register()` which will only get called when the app is getting rendered. This is useful if you are calling `register()` from a service provider, and want to access objects like the currently authenticated user, which are initialized later in middleware.

Alternatively, you may pass your own palette in as an array of OKLCH colors:

```php
$panel
    ->colors([
        'primary' => [
            50 => 'oklch(0.969 0.015 12.422)',
            100 => 'oklch(0.941 0.03 12.58)',
            200 => 'oklch(0.892 0.058 10.001)',
            300 => 'oklch(0.81 0.117 11.638)',
            400 => 'oklch(0.712 0.194 13.428)',
            500 => 'oklch(0.645 0.246 16.439)',
            600 => 'oklch(0.586 0.253 17.585)',
            700 => 'oklch(0.514 0.222 16.935)',
            800 => 'oklch(0.455 0.188 13.697)',
            900 => 'oklch(0.41 0.159 10.272)',
            950 => 'oklch(0.271 0.105 12.094)',
        ],
    ])
```

### Generating a color palette

If you want us to attempt to generate a palette for you based on a singular hex or RGB value, you can pass that in:

```php
$panel
    ->colors([
        'primary' => '#6366f1',
    ])

$panel
    ->colors([
        'primary' => 'rgb(99, 102, 241)',
    ])
```

## Changing the font

By default, we use the [Inter](https://fonts.google.com/specimen/Inter) font. You can change this using the `font()` method in the [configuration](../panel-configuration) file:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->font('Poppins');
}
```

All [Google Fonts](https://fonts.google.com) are available to use.

### Changing the font provider

[Bunny Fonts CDN](https://fonts.bunny.net) is used to serve the fonts. It is GDPR-compliant. If you'd like to use [Google Fonts CDN](https://fonts.google.com) instead, you can do so using the `provider` argument of the `font()` method:

```php
use Filament\FontProviders\GoogleFontProvider;

$panel->font('Inter', provider: GoogleFontProvider::class)
```

Or if you'd like to serve the fonts from a local stylesheet, you can use the `LocalFontProvider`:

```php
use Filament\FontProviders\LocalFontProvider;

$panel->font(
    'Inter',
    url: asset('css/fonts.css'),
    provider: LocalFontProvider::class,
)
```

## Creating a custom theme

Filament allows you to change the CSS used to render the UI by compiling a custom stylesheet to replace the default one. This custom stylesheet is called a "theme". Themes use [Tailwind CSS](https://tailwindcss.com).

To create a custom theme for a panel, you can use the `php artisan make:filament-theme` command:

```bash
php artisan make:filament-theme
```

If you have multiple panels, you can specify the panel you want to create a theme for:

```bash
php artisan make:filament-theme admin
```

By default, this command will use NPM to install dependencies. If you want to use a different package manager, you can use the `--pm` option:

```bash
php artisan make:filament-theme --pm=bun
````

This command generates a CSS file in the `resources/css/filament` directory.

Add the theme's CSS file to the Laravel plugin's `input` array in `vite.config.js`:

```js
input: [
    // ...
    'resources/css/filament/admin/theme.css',
]
```

Now, register the Vite-compiled theme CSS file in the panel's provider:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->viteTheme('resources/css/filament/admin/theme.css');
}
```

Finally, compile the theme with Vite:

```bash
npm run build
```

<Aside variant="info">
    Check the command output for the exact file path (e.g., `app/theme.css`), as it may vary depending on your panel's ID.
</Aside>

You can now customize the theme by editing the CSS file in `resources/css/filament`.

## Using Tailwind CSS classes in your Blade views or PHP files

Even though Filament uses Tailwind CSS to compile the framework, it is not set up to automatically scan for any Tailwind classes you use in your project, so these classes will not be included in the compiled CSS.

To use Tailwind CSS classes in your project, you need to set up a [custom theme](#creating-a-custom-theme) to customize the compiled CSS file in the panel. In the `theme.css` file of the theme, you will find two lines:

```css
@source '../../../../app/Filament';
@source '../../../../resources/views/filament';
```

These lines tell Tailwind to scan the `app/Filament` and `resources/views/filament` directories for any Tailwind classes you use in your project. You can [add any other directories](https://tailwindcss.com/docs/detecting-classes-in-source-files#explicitly-registering-sources) you want to scan for Tailwind classes here.

## Disabling dark mode

To disable dark mode switching, you can use the [configuration](../panel-configuration) file:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->darkMode(false);
}
```

## Changing the default theme mode

By default, Filament uses the user's system theme as the default mode. For example, if the user's computer is in dark mode, Filament will use dark mode by default. The system mode in Filament is reactive if the user changes their computer's mode. If you want to change the default mode to force light or dark mode, you can use the `defaultThemeMode()` method, passing `ThemeMode::Light` or `ThemeMode::Dark`:

```php
use Filament\Enums\ThemeMode;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->defaultThemeMode(ThemeMode::Light);
}
```

## Adding a logo

By default, Filament uses your app's name to render a simple text-based logo. However, you can easily customize this.

If you want to simply change the text that is used in the logo, you can use the `brandName()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->brandName('Filament Demo');
}
```

To render an image instead, you can pass a URL to the `brandLogo()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->brandLogo(asset('images/logo.svg'));
}
```

Alternatively, you may directly pass HTML to the `brandLogo()` method to render an inline SVG element for example:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->brandLogo(fn () => view('filament.admin.logo'));
}
```

```blade
<svg
    viewBox="0 0 128 26"
    xmlns="http://www.w3.org/2000/svg"
    class="h-full fill-gray-500 dark:fill-gray-400"
>
    <!-- ... -->
</svg>
```

If you need a different logo to be used when the application is in dark mode, you can pass it to `darkModeBrandLogo()` in the same way.

The logo height defaults to a sensible value, but it's impossible to account for all possible aspect ratios. Therefore, you may customize the height of the rendered logo using the `brandLogoHeight()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->brandLogo(fn () => view('filament.admin.logo'))
        ->brandLogoHeight('2rem');
}
```

## Adding a favicon

To add a favicon, you can use the [configuration](../panel-configuration) file, passing the public URL of the favicon:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->favicon(asset('images/favicon.png'));
}
```

---

<!-- Source: 01-overview (6).md -->

## Introduction

All examples in this guide will be written using [Pest](https://pestphp.com). To use Pest's Livewire plugin for testing, you can follow the installation instructions in the Pest documentation on plugins: [Livewire plugin for Pest](https://pestphp.com/docs/plugins#livewire). However, you can easily adapt this to PHPUnit, mostly by switching out the `livewire()` function from Pest with the `Livewire::test()` method.

Since all Filament components are mounted to a Livewire component, we're just using Livewire testing helpers everywhere. If you've never tested Livewire components before, please read [this guide](https://livewire.laravel.com/docs/testing) from the Livewire docs.

## Testing guides

Looking for a full example on how to test a panel resource? Check out the [Testing resources](testing-resources) section.

If you could like to learn the different methods available to test tables, check out the [Testing tables](../tables/testing) section.

If you need to test a schema, which encompasses both forms and infolists, check out the [Testing schemas](../schemas/testing) section.

If you would like to test an action, including actions that exist in tables or in schemas, check out the [Testing actions](../actions/testing) section.

If you would like to test a notification that you have sent, check out the [Testing notifications](../notifications/testing) section.

If you would like to test a custom page in a panel, these are Livewire components with no special behavior, so you should visit the [testing](https://livewire.laravel.com/docs/testing) section of the Livewire documentation.

## What is a Livewire component when using Filament?

When testing Filament, it is useful to understand which components are Livewire components and which aren't. With this information, you know which classes to pass to the `livewire()` function in Pest or the `Livewire::test()` method in PHPUnit.

Some examples of Livewire components are:

- Pages in a panel, including page classes in a resource's `Pages` directory
- Relation managers in a resource
- Widgets

Some examples of classes that are not Livewire components are:

- Resource classes
- Schema components
- Actions

These classes all interact with Livewire, but they are not Livewire components themselves. You can still test them, for example, by calling various methods and using the [Pest expectation API](https://pestphp.com/docs/expectations) to assert the expected behavior. However, the most useful tests will involve Livewire components, since they provide the best end-to-end testing coverage of your users' experience.

---

<!-- Source: 01-overview (7).md -->

## Introduction

Filament packages consume a set of core components that aim to provide a consistent and maintainable foundation for all interfaces. Some of these components are also available for use in your own applications and Filament plugins.

## Package components

The various packages in the Filament project can be used outside of a panel:

- [Action](action)
- [Form](form)
- [Infolist](infolist)
- [Notifications](notifications)
- [Schema](schema)
- [Table](table)
- [Widget](widget)

## Blade components

Aside from the core packages, all Filament projects can also consume the Blade components that Filament uses internally:

- [Avatar](avatar)
- [Badge](badge)
- [Button](button)
- [Breadcrumbs](breadcrumbs)
- [Checkbox](checkbox)
- [Dropdown](dropdown)
- [Fieldset](fieldset)
- [Icon button](icon-button)
- [Input](input)
- [Input wrapper](input-wrapper)
- [Link](link)
- [Loading indicator](loading-indicator)
- [Modal](modal)
- [Pagination](pagination)
- [Section](section)
- [Select](select)
- [Tabs](tabs)

---

<!-- Source: 01-overview.md -->

import Aside from "@components/Aside.astro"
import Disclosure from "@components/Disclosure.astro"

## Introduction

**Filament is a Server-Driven UI (SDUI) framework for Laravel.** It allows you to define user interfaces entirely in PHP using structured configuration objects, rather than traditional templating. Built on top of Livewire, Alpine.js, and Tailwind CSS, Filament empowers you to build full-featured interfaces like admin panels, dashboards, and form-based apps, all without writing custom JavaScript or frontend code.

<Disclosure>
    <span slot="summary">What is Server-Driven UI?</span>

    SDUI is a proven architecture used by companies like Meta, Airbnb, and Shopify. It moves control of the UI to the server, allowing for faster iteration, greater consistency, and centralized logic. Filament embraces this pattern for web development, letting you define interfaces declaratively using PHP classes that are rendered into HTML by the server.
    
    One key distinction to note is the difference between Server-Driven UI (SDUI) and Server-Rendered UI. While both approaches involve rendering content on the server, Server-Rendered UI relies on static templates (like traditional Blade views), where the structure and behavior of the UI are defined upfront in HTML or PHP files. In contrast, SDUI gives the server the power to dynamically generate the UI based on real-time configurations and business logic, allowing for more flexibility and reactivity without needing to modify frontend templates directly.
</Disclosure>

Thousands of developers use Filament to add admin panels to their Laravel applications, but it goes far beyond that. You can use Filament to build custom dashboards, user portals, CRMs, or even full applications with multiple panels. It integrates seamlessly with any frontend stack and works especially well alongside tools like Inertia.js, Livewire, and Blade.

If you're already using Blade views in your Laravel app, Filament components can enhance them too. You can drop Livewire components powered by Filament into any Blade view or route, and use the same schema-based builders for forms, tables, and more, all without switching your entire stack.

## Packages

The core of Filament comprises several packages:

- `filament/filament` - The core package for building panels (e.g., admin panels). This requires all other packages since panels often use many of their features.
- `filament/tables` - A data table builder that allows you to render an interactive table with filtering, sorting, pagination, and more.
- `filament/schemas` - A package that allows you to build UIs using an array of "component" PHP objects as configuration. This is used by many features in Filament to render UI. The package includes a base set of components that allow you to render content.
- `filament/forms` - A set of `filament/schemas` components for a large variety of form inputs (fields), complete with integrated validation.
- `filament/infolists` - A set of `filament/schemas` components for rendering "description lists". An infolist consists of "entries", which are key-value UI elements that can present read-only information like text, icons, and images. The data for an infolist can be sourced from anywhere but commonly comes from an individual Eloquent record.
- `filament/actions` - Action objects encapsulate the UI for a button, an interactive modal window that can be opened by the button, and the logic that should be executed when the modal window is submitted. They can be used anywhere in the UI and are commonly used to perform one-time actions like deleting a record, sending an email, or updating data in the database based on modal form input.
- `filament/notifications` - An easy way to send notifications to users in your app's UI. You can send a "flash" notification that appears immediately after a request to the server, a "database" notification which is stored in the database and rendered in a slide-over modal the user can open on demand, or a "broadcast" notification that is sent to the user in real-time over a websocket connection.
- `filament/widgets` - A set of dashboard "widgets" that can render anything, often statistical data. Charts, numbers, tables, and completely custom widgets can be rendered in a dashboard using this package.
- `filament/support` - This package contains a set of shared UI components and utilities used by all other packages. Users don't commonly install this directly, as it is a dependency of all other packages.

## Plugins

Filament is designed to be highly extensible, allowing you to add your own UI components and features to the framework. These extensions can live inside your codebase if they're specific to your application, or be distributed as Composer packages if they're general-purpose. In the Filament ecosystem, these Composer packages are called "plugins", and hundreds are available from the community. The Filament team also officially maintains several plugins that provide integration with popular third-party packages in the Laravel ecosystem.

The vast majority of plugins in the ecosystem are open-source and free to use. Some premium plugins are available for purchase, often offering enhanced customer support and quality.

<Aside variant="warning">
    Plugins not maintained by Filament team are created and managed by independent authors. While these plugins can enhance your experience, Filament cannot guarantee their quality, security, compatibility, or maintenance. We recommend reviewing the plugin's code, documentation, and user feedback before installation.
</Aside>

You can browse an extensive list of official and community plugins on the [Filament website](/plugins).

## Customizing the appearance

Tailwind CSS is a utility-based CSS framework that Filament uses as a token-based design system. Although Filament does not use Tailwind CSS utility classes directly in the HTML rendered by its components, it compiles Tailwind utilities into semantic CSS. These semantic classes can be targeted by Filament users with their own CSS to modify the appearance of components, creating a thin layer of overrides on top of the default Filament design.

A simple example demonstrating the power of this system is changing the border radius of all button components in Filament. By default, the following CSS code is used in the Filament codebase to style buttons using Tailwind utility classes:

```css
.fi-btn {
    @apply rounded-lg px-3 py-2 text-sm font-medium outline-none;
}
```

To decrease the [border radius in Tailwind CSS](https://tailwindcss.com/docs/border-radius), you can apply the `rounded-sm` (small) utility class to `.fi-btn` in your own CSS file:

```css
.fi-btn {
    @apply rounded-sm;
}
```

This overrides the default `rounded-lg` class with `rounded-sm` for all buttons in Filament while preserving the other styling properties of the button. This system provides a high level of flexibility to customize the appearance of Filament components without needing to write a complete custom stylesheet or maintain copies of HTML for each component.

For more information about customizing the appearance of Filament, visit the [Customizing styling documentation](../styling).

## Testing

The core packages in Filament undergo unit testing to ensure stability across releases. As a Filament user, you can write tests for applications built with the framework. Filament provides utilities for testing both functionality and UI components, compatible with either Pest or PHPUnit test suites. Testing is particularly crucial when customizing the framework or implementing custom functionality, though it's also valuable for verifying basic functionality works as intended.

For more information about testing Filament applications, visit the [Testing documentation](../testing).

## Alternatives to Filament

If you're reading this and concluding that Filament might not be the right choice for your project, that's okay! There are many other excellent projects in the Laravel ecosystem that might be a better fit for your needs. Here are a few we really like:

### Filament sounds too complicated

[Laravel Nova](https://nova.laravel.com) is an easy way to build an admin panel for your Laravel application. It's an official project maintained by the Laravel team, and purchasing it helps support the development of the Laravel framework.

### I do not want to use Livewire to customize anything

Many parts of Filament do not require you to touch Livewire at all, but building custom components might.

[Laravel Nova](https://nova.laravel.com) is built with Vue.js and Inertia.js, which might be a better fit for your project if it requires extensive customization and you have experience with these tools.

### I need an out-of-the-box CMS

[Statamic](https://statamic.com) is a CMS built on Laravel. It's a great choice if you need a CMS that's easy to set up and use, and you don't need to build a custom admin panel.

### I just want to write Blade views and handle the backend myself

[Flux](https://fluxui.dev) is the official Livewire UI kit and ships as a set of pre-built and pre-styled Blade components. It is maintained by the same team that maintains Livewire and Alpine.js, and purchasing it helps support the development of these projects.

---

<!-- Source: 01-render-hooks.md -->

## Introduction

Filament allows you to render Blade content at various points in the frameworks views. It's useful for plugins to be able to inject HTML into the framework. Also, since Filament does not recommend publishing the views due to an increased risk of breaking changes, it's also useful for users.

## Registering render hooks

To register render hooks, you can call `FilamentView::registerRenderHook()` from a service provider or middleware. The first argument is the name of the render hook, and the second argument is a callback that returns the content to be rendered:

```php
use Filament\Support\Facades\FilamentView;
use Filament\View\PanelsRenderHook;
use Illuminate\Support\Facades\Blade;

FilamentView::registerRenderHook(
    PanelsRenderHook::BODY_START,
    fn (): string => Blade::render('@livewire(\'livewire-ui-modal\')'),
);
```

You could also render view content from a file:

```php
use Filament\Support\Facades\FilamentView;
use Filament\View\PanelsRenderHook;
use Illuminate\Contracts\View\View;

FilamentView::registerRenderHook(
    PanelsRenderHook::BODY_START,
    fn (): View => view('impersonation-banner'),
);
```

## Available render hooks

### Panel Builder render hooks

```php
use Filament\View\PanelsRenderHook;
```

- `PanelsRenderHook::AUTH_LOGIN_FORM_AFTER` - After login form
- `PanelsRenderHook::AUTH_LOGIN_FORM_BEFORE` - Before login form
- `PanelsRenderHook::AUTH_PASSWORD_RESET_REQUEST_FORM_AFTER` - After password reset request form
- `PanelsRenderHook::AUTH_PASSWORD_RESET_REQUEST_FORM_BEFORE` - Before password reset request form
- `PanelsRenderHook::AUTH_PASSWORD_RESET_RESET_FORM_AFTER` - After password reset form
- `PanelsRenderHook::AUTH_PASSWORD_RESET_RESET_FORM_BEFORE` - Before password reset form
- `PanelsRenderHook::AUTH_REGISTER_FORM_AFTER` - After register form
- `PanelsRenderHook::AUTH_REGISTER_FORM_BEFORE` - Before register form
- `PanelsRenderHook::BODY_END` - Before `</body>`
- `PanelsRenderHook::BODY_START` - After `<body>`
- `PanelsRenderHook::CONTENT_AFTER` - After page content
- `PanelsRenderHook::CONTENT_BEFORE` - Before page content
- `PanelsRenderHook::CONTENT_END` - After page content, inside `<main>`
- `PanelsRenderHook::CONTENT_START` - Before page content, inside `<main>`
- `PanelsRenderHook::FOOTER` - Footer of the page
- `PanelsRenderHook::GLOBAL_SEARCH_AFTER` - After the [global search](../panels/resources/global-search) container, inside the topbar
- `PanelsRenderHook::GLOBAL_SEARCH_BEFORE` - Before the [global search](../panels/resources/global-search) container, inside the topbar
- `PanelsRenderHook::GLOBAL_SEARCH_END` - The end of the [global search](../panels/resources/global-search) container
- `PanelsRenderHook::GLOBAL_SEARCH_START` - The start of the [global search](../panels/resources/global-search) container
- `PanelsRenderHook::HEAD_END` - Before `</head>`
- `PanelsRenderHook::HEAD_START` - After `<head>`
- `PanelsRenderHook::LAYOUT_END` - End of the layout container, also [can be scoped](#scoping-render-hooks) to the page class
- `PanelsRenderHook::LAYOUT_START` - Start of the layout container, also [can be scoped](#scoping-render-hooks) to the page class
- `PanelsRenderHook::PAGE_END` - End of the page content container, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_FOOTER_WIDGETS_AFTER` - After the page footer widgets, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_FOOTER_WIDGETS_BEFORE` - Before the page footer widgets, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_FOOTER_WIDGETS_END` - End of the page footer widgets, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_FOOTER_WIDGETS_START` - Start of the page footer widgets, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_HEADER_ACTIONS_AFTER` - After the page header actions, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_HEADER_ACTIONS_BEFORE` - Before the page header actions, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_HEADER_WIDGETS_AFTER` - After the page header widgets, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_HEADER_WIDGETS_BEFORE` - Before the page header widgets, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_HEADER_WIDGETS_END` - End of the page header widgets, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_HEADER_WIDGETS_START` - Start of the page header widgets, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_START` - Start of the page content container, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_SUB_NAVIGATION_END_AFTER` - After the page sub navigation "end" sidebar position, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_SUB_NAVIGATION_END_BEFORE` - Before the page sub navigation "end" sidebar position, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_SUB_NAVIGATION_SELECT_AFTER` - After the page sub navigation select (for mobile), also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_SUB_NAVIGATION_SELECT_BEFORE` - Before the page sub navigation select (for mobile), also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_SUB_NAVIGATION_SIDEBAR_AFTER` - After the page sub navigation sidebar, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_SUB_NAVIGATION_SIDEBAR_BEFORE` - Before the page sub navigation sidebar, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_SUB_NAVIGATION_START_AFTER` - After the page sub navigation "start" sidebar position, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_SUB_NAVIGATION_START_BEFORE` - Before the page sub navigation "start" sidebar position, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_SUB_NAVIGATION_TOP_AFTER` - After the page sub navigation "top" tabs position, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::PAGE_SUB_NAVIGATION_TOP_BEFORE` - Before the page sub navigation "top" tabs position, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::RESOURCE_PAGES_LIST_RECORDS_TABLE_AFTER` - After the resource table, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::RESOURCE_PAGES_LIST_RECORDS_TABLE_BEFORE` - Before the resource table, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::RESOURCE_PAGES_LIST_RECORDS_TABS_END` - The end of the filter tabs (after the last tab), also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::RESOURCE_PAGES_LIST_RECORDS_TABS_START` - The start of the filter tabs (before the first tab), also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::RESOURCE_PAGES_MANAGE_RELATED_RECORDS_TABLE_AFTER` - After the relation manager table, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::RESOURCE_PAGES_MANAGE_RELATED_RECORDS_TABLE_BEFORE` - Before the relation manager table, also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::RESOURCE_RELATION_MANAGER_AFTER` - After the relation manager table, also [can be scoped](#scoping-render-hooks) to the page or relation manager class
- `PanelsRenderHook::RESOURCE_RELATION_MANAGER_BEFORE` - Before the relation manager table, also [can be scoped](#scoping-render-hooks) to the page or relation manager class
- `PanelsRenderHook::RESOURCE_TABS_END` - The end of the resource tabs (after the last tab), also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::RESOURCE_TABS_START` - The start of the resource tabs (before the first tab), also [can be scoped](#scoping-render-hooks) to the page or resource class
- `PanelsRenderHook::SCRIPTS_AFTER` - After scripts are defined
- `PanelsRenderHook::SCRIPTS_BEFORE` - Before scripts are defined
- `PanelsRenderHook::SIDEBAR_LOGO_AFTER` - After the logo in the sidebar
- `PanelsRenderHook::SIDEBAR_LOGO_BEFORE` - Before the logo in the sidebar
- `PanelsRenderHook::SIDEBAR_NAV_END` - In the [sidebar](../panels/navigation), before `</nav>`
- `PanelsRenderHook::SIDEBAR_NAV_START` - In the [sidebar](../panels/navigation), after `<nav>`
- `PanelsRenderHook::SIMPLE_LAYOUT_END` - End of the simple layout container, also [can be scoped](#scoping-render-hooks) to the page class
- `PanelsRenderHook::SIMPLE_LAYOUT_START` - Start of the simple layout container, also [can be scoped](#scoping-render-hooks) to the page class
- `PanelsRenderHook::SIMPLE_PAGE_END` - End of the simple page content container, also [can be scoped](#scoping-render-hooks) to the page class
- `PanelsRenderHook::SIMPLE_PAGE_START` - Start of the simple page content container, also [can be scoped](#scoping-render-hooks) to the page class
- `PanelsRenderHook::SIDEBAR_FOOTER` - Pinned to the bottom of the sidebar, below the content
- `PanelsRenderHook::STYLES_AFTER` - After styles are defined
- `PanelsRenderHook::STYLES_BEFORE` - Before styles are defined
- `PanelsRenderHook::TENANT_MENU_AFTER` - After the [tenant menu](../users/tenancy#customizing-the-tenant-menu)
- `PanelsRenderHook::TENANT_MENU_BEFORE` - Before the [tenant menu](../users/tenancy#customizing-the-tenant-menu)
- `PanelsRenderHook::TOPBAR_AFTER` - Below the topbar
- `PanelsRenderHook::TOPBAR_BEFORE` - Above the topbar
- `PanelsRenderHook::TOPBAR_END` - End of the topbar container
- `PanelsRenderHook::TOPBAR_LOGO_AFTER` - After the logo in the topbar
- `PanelsRenderHook::TOPBAR_LOGO_BEFORE` - Before the logo in the topbar
- `PanelsRenderHook::TOPBAR_START` - Start of the topbar container
- `PanelsRenderHook::USER_MENU_AFTER` - After the [user menu](../navigation/user-menu)
- `PanelsRenderHook::USER_MENU_BEFORE` - Before the [user menu](../navigation/user-menu)
- `PanelsRenderHook::USER_MENU_PROFILE_AFTER` - After the profile item in the [user menu](../navigation/user-menu)
- `PanelsRenderHook::USER_MENU_PROFILE_BEFORE` - Before the profile item in the [user menu](../navigation/user-menu)

### Table Builder render hooks

All these render hooks [can be scoped](#scoping-render-hooks) to any table Livewire component class. When using the Panel Builder, these classes might be the List or Manage page of a resource, or a relation manager. Table widgets are also Livewire component classes.

```php
use Filament\Tables\View\TablesRenderHook;
```

- `TablesRenderHook::FILTER_INDICATORS` - Replace the existing filter indicators, receives `filterIndicators` data as `array<Filament\Tables\Filters\Indicator>`
- `TablesRenderHook::SELECTION_INDICATOR_ACTIONS_AFTER` - After the "select all" and "deselect all" action buttons in the selection indicator bar
- `TablesRenderHook::SELECTION_INDICATOR_ACTIONS_BEFORE` - Before the "select all" and "deselect all" action buttons in the selection indicator bar
- `TablesRenderHook::HEADER_AFTER` - After the header container
- `TablesRenderHook::HEADER_BEFORE` - Before the header container
- `TablesRenderHook::TOOLBAR_AFTER` - After the toolbar container
- `TablesRenderHook::TOOLBAR_BEFORE` - Before the toolbar container
- `TablesRenderHook::TOOLBAR_END` - The end of the toolbar
- `TablesRenderHook::TOOLBAR_GROUPING_SELECTOR_AFTER` - After the [grouping](../tables/grouping) selector
- `TablesRenderHook::TOOLBAR_GROUPING_SELECTOR_BEFORE` - Before the [grouping](../tables/grouping) selector
- `TablesRenderHook::TOOLBAR_REORDER_TRIGGER_AFTER` - After the [reorder](../tables/overview#reordering-records) trigger
- `TablesRenderHook::TOOLBAR_REORDER_TRIGGER_BEFORE` - Before the [reorder](../tables/overview#reordering-records) trigger
- `TablesRenderHook::TOOLBAR_SEARCH_AFTER` - After the [search](../tables/overview#making-columns-sortable-and-searchable) container
- `TablesRenderHook::TOOLBAR_SEARCH_BEFORE` - Before the [search](../tables/overview#making-columns-sortable-and-searchable) container
- `TablesRenderHook::TOOLBAR_START` - The start of the toolbar
- `TablesRenderHook::TOOLBAR_COLUMN_MANAGER_TRIGGER_AFTER` - After the [column manager](../tables/columns/getting-started#toggling-column-visibility) trigger
- `TablesRenderHook::TOOLBAR_COLUMN_MANAGER_TRIGGER_BEFORE` - Before the [column manager](../tables/columns/getting-started#toggling-column-visibility) trigger

### Widgets render hooks

```php
use Filament\Widgets\View\WidgetsRenderHook;
```

- `WidgetsRenderHook::TABLE_WIDGET_END` - End of the [table widget](../widgets/overview#table-widgets), after the table itself, also [can be scoped](#scoping-render-hooks) to the table widget class
- `WidgetsRenderHook::TABLE_WIDGET_START` - Start of the [table widget](../widgets/overview#table-widgets), before the table itself, also [can be scoped](#scoping-render-hooks) to the table widget class

## Scoping render hooks

Some render hooks can be given a "scope", which allows them to only be output on a specific page or Livewire component. For instance, you might want to register a render hook for just 1 page. To do that, you can pass the class of the page or component as the second argument to `registerRenderHook()`:

```php
use Filament\Support\Facades\FilamentView;
use Filament\View\PanelsRenderHook;
use Illuminate\Support\Facades\Blade;

FilamentView::registerRenderHook(
    PanelsRenderHook::PAGE_START,
    fn (): View => view('warning-banner'),
    scopes: \App\Filament\Resources\Users\Pages\EditUser::class,
);
```

You can also pass an array of scopes to register the render hook for:

```php
use Filament\Support\Facades\FilamentView;
use Filament\View\PanelsRenderHook;

FilamentView::registerRenderHook(
    PanelsRenderHook::PAGE_START,
    fn (): View => view('warning-banner'),
    scopes: [
        \App\Filament\Resources\Users\Pages\CreateUser::class,
        \App\Filament\Resources\Users\Pages\EditUser::class,
    ],
);
```

Some render hooks for the [Panel Builder](#panel-builder-render-hooks) allow you to scope hooks to all pages in a resource:

```php
use Filament\Support\Facades\FilamentView;
use Filament\View\PanelsRenderHook;

FilamentView::registerRenderHook(
    PanelsRenderHook::PAGE_START,
    fn (): View => view('warning-banner'),
    scopes: \App\Filament\Resources\Users\UserResource::class,
);
```

### Retrieving the currently active scopes inside the render hook

The `$scopes` are passed to the render hook function, and you can use them to determine which page or component the render hook is being rendered on:

```php
use Filament\Support\Facades\FilamentView;
use Filament\View\PanelsRenderHook;

FilamentView::registerRenderHook(
    PanelsRenderHook::PAGE_START,
    fn (array $scopes): View => view('warning-banner', ['scopes' => $scopes]),
    scopes: \App\Filament\Resources\Users\UserResource::class,
);
```

## Passing data to render hooks

Render hooks can receive "data" from when the hook is rendered. To access data from a render hook, you can inject it using an `array $data` parameter to the hook's rendering function:

```php
use Filament\Support\Facades\FilamentView;
use Filament\Tables\View\TablesRenderHook;

FilamentView::registerRenderHook(
    TablesRenderHook::FILTER_INDICATORS,
    fn (array $data): View => view('filter-indicators', ['indicators' => $data['filterIndicators']]),
);
```

## Rendering hooks

Plugin developers might find it useful to expose render hooks to their users. You do not need to register them anywhere, simply output them in Blade like so:

```blade
{{ \Filament\Support\Facades\FilamentView::renderHook(\Filament\View\PanelsRenderHook::PAGE_START) }}
```

To provide [scope](#scoping-render-hooks) your render hook, you can pass it as the second argument to `renderHook()`. For instance, if your hook is inside a Livewire component, you can pass the class of the component using `static::class`:

```blade
{{ \Filament\Support\Facades\FilamentView::renderHook(\Filament\View\PanelsRenderHook::PAGE_START, scopes: $this->getRenderHookScopes()) }}
```

You can even pass multiple scopes as an array, and all render hooks that match any of the scopes will be rendered:

```blade
{{ \Filament\Support\Facades\FilamentView::renderHook(\Filament\View\PanelsRenderHook::PAGE_START, scopes: [static::class, \App\Filament\Resources\Users\UserResource::class]) }}
```

You can pass [data](#passing-data-to-render-hooks) to a render hook using a `data` argument to the `renderHook()` function:

```blade
{{ \Filament\Support\Facades\FilamentView::renderHook(\Filament\Tables\View\TablesRenderHook::FILTER_INDICATORS, data: ['filterIndicators' => $filterIndicators]) }}
```

---

<!-- Source: 02-action.md -->

import Aside from "@components/Aside.astro"

<Aside variant="warning">
    Before proceeding, make sure `filament/actions` is installed in your project. You can check by running:

    ```bash
    composer show filament/actions
    ```
    If it's not installed, consult the [installation guide](../introduction/installation#installing-the-individual-components) and configure the **individual components** according to the instructions.
</Aside>

## Setting up the Livewire component

First, generate a new Livewire component:

```bash
php artisan make:livewire ManagePost
```

Then, render your Livewire component on the page:

```blade
@livewire('manage-post')
```

Alternatively, you can use a full-page Livewire component:

```php
use App\Livewire\ManagePost;
use Illuminate\Support\Facades\Route;

Route::get('posts/{post}/manage', ManagePost::class);
```

You must use the `InteractsWithActions` and `InteractsWithSchemas` traits, and implement the `HasActions` and `HasSchemas` interfaces on your Livewire component class:

```php
use Filament\Actions\Concerns\InteractsWithActions;
use Filament\Actions\Contracts\HasActions;
use Filament\Schemas\Concerns\InteractsWithSchemas;
use Filament\Schemas\Contracts\HasSchemas;
use Livewire\Component;

class ManagePost extends Component implements HasActions, HasSchemas
{
    use InteractsWithActions;
    use InteractsWithSchemas;

    // ...
}
```

## Adding the action

Add a method that returns your action. The method must share the exact same name as the action, or the name followed by `Action`:

```php
use App\Models\Post;
use Filament\Actions\Action;
use Filament\Actions\Concerns\InteractsWithActions;
use Filament\Actions\Contracts\HasActions;
use Filament\Schemas\Concerns\InteractsWithSchemas;
use Filament\Schemas\Contracts\HasSchemas;
use Livewire\Component;

class ManagePost extends Component implements HasActions, HasSchemas
{
    use InteractsWithActions;
    use InteractsWithSchemas;

    public Post $post;

    public function deleteAction(): Action
    {
        return Action::make('delete')
            ->color('danger')
            ->requiresConfirmation()
            ->action(fn () => $this->post->delete());
    }
    
    // This method name also works, since the action name is `delete`:
    // public function delete(): Action
    
    // This method name does not work, since the action name is `delete`, not `deletePost`:
    // public function deletePost(): Action

    // ...
}
```

Finally, you need to render the action in your view. To do this, you can use `{{ $this->deleteAction }}`, where you replace `deleteAction` with the name of your action method:

```blade
<div>
    {{ $this->deleteAction }}

    <x-filament-actions::modals />
</div>
```

You also need `<x-filament-actions::modals />` which injects the HTML required to render action modals. This only needs to be included within the Livewire component once, regardless of how many actions you have for that component.

<Aside variant="info">
    `filament/actions` also includes the following packages:
    
    - `filament/forms`
    - `filament/infolists`
    - `filament/notifications`
    - `filament/support`
    
    These packages allow you to use their components within Livewire components.
    For example, if your action uses [Notifications](notifications), remember to include `@livewire('notifications')` in your layout and add `@import '../../vendor/filament/notifications/resources/css/index.css'` to your CSS file.
    
    If you are using any other [Filament components](overview#package-components) in your action, make sure to install and integrate the corresponding package as well.
</Aside>

## Passing action arguments

Sometimes, you may wish to pass arguments to your action. For example, if you're rendering the same action multiple times in the same view, but each time for a different model, you could pass the model ID as an argument, and then retrieve it later. To do this, you can invoke the action in your view and pass in the arguments as an array:

```php
<div>
    @foreach ($posts as $post)
        <h2>{{ $post->title }}</h2>

        {{ ($this->deleteAction)(['post' => $post->id]) }}
    @endforeach

    <x-filament-actions::modals />
</div>
```

Now, you can access the post ID in your action method:

```php
use App\Models\Post;
use Filament\Actions\Action;

public function deleteAction(): Action
{
    return Action::make('delete')
        ->color('danger')
        ->requiresConfirmation()
        ->action(function (array $arguments) {
            $post = Post::find($arguments['post']);

            $post?->delete();
        });
}
```

## Hiding actions in a Livewire view

If you use `hidden()` or `visible()` to control if an action is rendered, you should wrap the action in an `@if` check for `isVisible()`:

```blade
<div>
    @if ($this->deleteAction->isVisible())
        {{ $this->deleteAction }}
    @endif
    
    {{-- Or --}}
    
    @if (($this->deleteAction)(['post' => $post->id])->isVisible())
        {{ ($this->deleteAction)(['post' => $post->id]) }}
    @endif
</div>
```

The `hidden()` and `visible()` methods also control if the action is `disabled()`, so they are still useful to protect the action from being run if the user does not have permission. Encapsulating this logic in the `hidden()` or `visible()` of the action itself is good practice otherwise you need to define the condition in the view and in `disabled()`.

You can also take advantage of this to hide any wrapping elements that may not need to be rendered if the action is hidden:

```blade
<div>
    @if ($this->deleteAction->isVisible())
        <div>
            {{ $this->deleteAction }}
        </div>
    @endif
</div>
```

## Grouping actions in a Livewire view

You may [group actions together into a dropdown menu](grouping-actions) by using the `<x-filament-actions::group>` Blade component, passing in the `actions` array as an attribute:

```blade
<div>
    <x-filament-actions::group :actions="[
        $this->editAction,
        $this->viewAction,
        $this->deleteAction,
    ]" />

    <x-filament-actions::modals />
</div>
```

You can also pass in any attributes to customize the appearance of the trigger button and dropdown:

```blade
<div>
    <x-filament-actions::group
        :actions="[
            $this->editAction,
            $this->viewAction,
            $this->deleteAction,
        ]"
        label="Actions"
        icon="heroicon-m-ellipsis-vertical"
        color="primary"
        size="md"
        tooltip="More actions"
        dropdown-placement="bottom-start"
    />

    <x-filament-actions::modals />
</div>
```

## Chaining actions

You can chain multiple actions together, by calling the `replaceMountedAction()` method to replace the current action with another when it has finished:

```php
use App\Models\Post;
use Filament\Actions\Action;

public function editAction(): Action
{
    return Action::make('edit')
        ->schema([
            // ...
        ])
        // ...
        ->action(function (array $arguments) {
            $post = Post::find($arguments['post']);

            // ...

            $this->replaceMountedAction('publish', $arguments);
        });
}

public function publishAction(): Action
{
    return Action::make('publish')
        ->requiresConfirmation()
        // ...
        ->action(function (array $arguments) {
            $post = Post::find($arguments['post']);

            $post->publish();
        });
}
```

Now, when the first action is submitted, the second action will open in its place. The [arguments](#passing-action-arguments) that were originally passed to the first action get passed to the second action, so you can use them to persist data between requests.

If the first action is canceled, the second one is not opened. If the second action is canceled, the first one has already run and cannot be cancelled.

## Programmatically triggering actions

Sometimes you may need to trigger an action without the user clicking on the built-in trigger button, especially from JavaScript. Here is an example action which could be registered on a Livewire component:

```php
use Filament\Actions\Action;

public function testAction(): Action
{
    return Action::make('test')
        ->requiresConfirmation()
        ->action(function (array $arguments) {
            dd('Test action called', $arguments);
        });
}
```

You can trigger that action from a click in your HTML using the `wire:click` attribute, calling the `mountAction()` method and optionally passing in any arguments that you want to be available:

```blade
<button wire:click="mountAction('test', { id: 12345 })">
    Button
</button>
```

To trigger that action from JavaScript, you can use the [`$wire` utility](https://livewire.laravel.com/docs/alpine#controlling-livewire-from-alpine-using-wire), passing in the same arguments:

```js
$wire.mountAction('test', { id: 12345 })
```

---

<!-- Source: 02-assets.md -->

## Introduction

All packages in the Filament ecosystem share an asset management system. This allows both official plugins and third-party plugins to register CSS and JavaScript files that can then be consumed by Blade views.

## The `FilamentAsset` facade

The `FilamentAsset` facade is used to register files into the asset system. These files may be sourced from anywhere in the filesystem, but are then copied into the `/public` directory of the application when the `php artisan filament:assets` command is run. By copying them into the `/public` directory for you, we can predictably load them in Blade views, and also ensure that third party packages are able to load their assets without having to worry about where they are located.

Assets always have a unique ID chosen by you, which is used as the file name when the asset is copied into the `/public` directory. This ID is also used to reference the asset in Blade views. While the ID is unique, if you are registering assets for a plugin, then you do not need to worry about IDs clashing with other plugins, since the asset will be copied into a directory named after your plugin.

The `FilamentAsset` facade should be used in the `boot()` method of a service provider. It can be used inside an application service provider such as `AppServiceProvider`, or inside a plugin service provider.

The `FilamentAsset` facade has one main method, `register()`, which accepts an array of assets to register:

```php
use Filament\Support\Facades\FilamentAsset;

public function boot(): void
{
    // ...
    
    FilamentAsset::register([
        // ...
    ]);
    
    // ...
}
```

### Registering assets for a plugin

When registering assets for a plugin, you should pass the name of the Composer package as the second argument of the `register()` method:

```php
use Filament\Support\Facades\FilamentAsset;

FilamentAsset::register([
    // ...
], package: 'danharrin/filament-blog');
```

Now, all the assets for this plugin will be copied into their own directory inside `/public`, to avoid the possibility of clashing with other plugins' files with the same names.

## Registering CSS files

To register a CSS file with the asset system, use the `FilamentAsset::register()` method in the `boot()` method of a service provider. You must pass in an array of `Css` objects, which each represents a CSS file that should be registered in the asset system.

Each `Css` object has a unique ID and a path to the CSS file:

```php
use Filament\Support\Assets\Css;
use Filament\Support\Facades\FilamentAsset;

FilamentAsset::register([
    Css::make('custom-stylesheet', __DIR__ . '/../../resources/css/custom.css'),
]);
```

In this example, we use `__DIR__` to generate a relative path to the asset from the current file. For instance, if you were adding this code to `/app/Providers/AppServiceProvider.php`, then the CSS file should exist in `/resources/css/custom.css`.

Now, when the `php artisan filament:assets` command is run, this CSS file is copied into the `/public` directory. In addition, it is now loaded into all Blade views that use Filament. If you're interested in only loading the CSS when it is required by an element on the page, check out the [Lazy loading CSS](#lazy-loading-css) section.

### Using Tailwind CSS in plugins

Typically, registering CSS files is used to register custom stylesheets for your application. If you want to process these files using Tailwind CSS, you need to consider the implications of that, especially if you are a plugin developer.

Tailwind builds are unique to every application - they contain a minimal set of utility classes, only the ones that you are actually using in your application. This means that if you are a plugin developer, you probably should not be building your Tailwind CSS files into your plugin. Instead, you should provide the raw CSS files and instruct the user that they should build the Tailwind CSS file themselves. To do this, they probably just need to add your vendor directory into the `content` array of their `tailwind.config.js` file:

```js
export default {
    content: [
        './resources/**/*.blade.php',
        './vendor/filament/**/*.blade.php',
        './vendor/danharrin/filament-blog/resources/views/**/*.blade.php', // Your plugin's vendor directory
    ],
    // ...
}
```

This means that when they build their Tailwind CSS file, it will include all the utility classes that are used in your plugin's views, as well as the utility classes that are used in their application and the Filament core.

However, with this technique, there might be extra complications for users who use your plugin with the [Panel Builder](../panels). If they have a [custom theme](../panels/themes), they will be fine, since they are building their own CSS file anyway using Tailwind CSS. However, if they are using the default stylesheet which is shipped with the Panel Builder, you might have to be careful about the utility classes that you use in your plugin's views. For instance, if you use a utility class that is not included in the default stylesheet, the user is not compiling it themselves, and it will not be included in the final CSS file. This means that your plugin's views might not look as expected. This is one of the few situations where I would recommend compiling and [registering](#registering-css-files) a Tailwind CSS-compiled stylesheet in your plugin.

### Lazy loading CSS

By default, all CSS files registered with the asset system are loaded in the `<head>` of every Filament page. This is the simplest way to load CSS files, but sometimes they may be quite heavy and not required on every page. In this case, you can leverage the [Alpine.js Lazy Load Assets](https://github.com/tanthammar/alpine-lazy-load-assets) package that comes bundled with Filament. It allows you to easily load CSS files on-demand using Alpine.js. The premise is very simple, you use the `x-load-css` directive on an element, and when that element is loaded onto the page, the specified CSS files are loaded into the `<head>` of the page. This is perfect for both small UI elements and entire pages that require a CSS file:

```blade
<div
    x-data="{}"
    x-load-css="[@js(\Filament\Support\Facades\FilamentAsset::getStyleHref('custom-stylesheet'))]"
>
    <!-- ... -->
</div>
```

To prevent the CSS file from being loaded automatically, you can use the `loadedOnRequest()` method:

```php
use Filament\Support\Assets\Css;
use Filament\Support\Facades\FilamentAsset;

FilamentAsset::register([
    Css::make('custom-stylesheet', __DIR__ . '/../../resources/css/custom.css')->loadedOnRequest(),
]);
```

If your CSS file was [registered to a plugin](#registering-assets-for-a-plugin), you must pass that in as the second argument to the `FilamentAsset::getStyleHref()` method:

```blade
<div
    x-data="{}"
    x-load-css="[@js(\Filament\Support\Facades\FilamentAsset::getStyleHref('custom-stylesheet', package: 'danharrin/filament-blog'))]"
>
    <!-- ... -->
</div>
```

### Registering CSS files from a URL

If you want to register a CSS file from a URL, you may do so. These assets will be loaded on every page as normal, but not copied into the `/public` directory when the `php artisan filament:assets` command is run. This is useful for registering external stylesheets from a CDN, or stylesheets that you are already compiling directly into the `/public` directory:

```php
use Filament\Support\Assets\Css;
use Filament\Support\Facades\FilamentAsset;

FilamentAsset::register([
    Css::make('example-external-stylesheet', 'https://example.com/external.css'),
    Css::make('example-local-stylesheet', asset('css/local.css')),
]);
```

### Registering CSS variables

Sometimes, you may wish to use dynamic data from the backend in CSS files. To do this, you can use the `FilamentAsset::registerCssVariables()` method in the `boot()` method of a service provider:

```php
use Filament\Support\Facades\FilamentAsset;

FilamentAsset::registerCssVariables([
    'background-image' => asset('images/background.jpg'),
]);
```

Now, you can access these variables from any CSS file:

```css
background-image: var(--background-image);
```

## Registering JavaScript files

To register a JavaScript file with the asset system, use the `FilamentAsset::register()` method in the `boot()` method of a service provider. You must pass in an array of `Js` objects, which each represents a JavaScript file that should be registered in the asset system.

Each `Js` object has a unique ID and a path to the JavaScript file:

```php
use Filament\Support\Assets\Js;

FilamentAsset::register([
    Js::make('custom-script', __DIR__ . '/../../resources/js/custom.js'),
]);
```

In this example, we use `__DIR__` to generate a relative path to the asset from the current file. For instance, if you were adding this code to `/app/Providers/AppServiceProvider.php`, then the JavaScript file should exist in `/resources/js/custom.js`.

Now, when the `php artisan filament:assets` command is run, this JavaScript file is copied into the `/public` directory. In addition, it is now loaded into all Blade views that use Filament. If you're interested in only loading the JavaScript when it is required by an element on the page, check out the [Lazy loading JavaScript](#lazy-loading-javascript) section.

### Lazy loading JavaScript

By default, all JavaScript files registered with the asset system are loaded at the bottom of every Filament page. This is the simplest way to load JavaScript files, but sometimes they may be quite heavy and not required on every page. In this case, you can leverage the [Alpine.js Lazy Load Assets](https://github.com/tanthammar/alpine-lazy-load-assets) package that comes bundled with Filament. It allows you to easily load JavaScript files on-demand using Alpine.js. The premise is very simple, you use the `x-load-js` directive on an element, and when that element is loaded onto the page, the specified JavaScript files are loaded at the bottom of the page. This is perfect for both small UI elements and entire pages that require a JavaScript file:

```blade
<div
    x-data="{}"
    x-load-js="[@js(\Filament\Support\Facades\FilamentAsset::getScriptSrc('custom-script'))]"
>
    <!-- ... -->
</div>
```

To prevent the JavaScript file from being loaded automatically, you can use the `loadedOnRequest()` method:

```php
use Filament\Support\Assets\Js;
use Filament\Support\Facades\FilamentAsset;

FilamentAsset::register([
    Js::make('custom-script', __DIR__ . '/../../resources/js/custom.js')->loadedOnRequest(),
]);
```

If your JavaScript file was [registered to a plugin](#registering-assets-for-a-plugin), you must pass that in as the second argument to the `FilamentAsset::getScriptSrc()` method:

```blade
<div
    x-data="{}"
    x-load-js="[@js(\Filament\Support\Facades\FilamentAsset::getScriptSrc('custom-script', package: 'danharrin/filament-blog'))]"
>
    <!-- ... -->
</div>
```

#### Asynchronous Alpine.js components

Sometimes, you may want to load external JavaScript libraries for your Alpine.js-based components. The best way to do this is by storing the compiled JavaScript and Alpine component in a separate file, and letting us load it whenever the component is rendered.

Firstly, you should install [esbuild](https://esbuild.github.io) via NPM, which we will use to create a single JavaScript file containing your external library and Alpine component:

```bash
npm install esbuild --save-dev
```

Then, you must create a script to compile your JavaScript and Alpine component. You can put this anywhere, for example `bin/build.js`:

```js
import * as esbuild from 'esbuild'

const isDev = process.argv.includes('--dev')

async function compile(options) {
    const context = await esbuild.context(options)

    if (isDev) {
        await context.watch()
    } else {
        await context.rebuild()
        await context.dispose()
    }
}

const defaultOptions = {
    define: {
        'process.env.NODE_ENV': isDev ? `'development'` : `'production'`,
    },
    bundle: true,
    mainFields: ['module', 'main'],
    platform: 'neutral',
    sourcemap: isDev ? 'inline' : false,
    sourcesContent: isDev,
    treeShaking: true,
    target: ['es2020'],
    minify: !isDev,
    plugins: [{
        name: 'watchPlugin',
        setup(build) {
            build.onStart(() => {
                console.log(`Build started at ${new Date(Date.now()).toLocaleTimeString()}: ${build.initialOptions.outfile}`)
            })

            build.onEnd((result) => {
                if (result.errors.length > 0) {
                    console.log(`Build failed at ${new Date(Date.now()).toLocaleTimeString()}: ${build.initialOptions.outfile}`, result.errors)
                } else {
                    console.log(`Build finished at ${new Date(Date.now()).toLocaleTimeString()}: ${build.initialOptions.outfile}`)
                }
            })
        }
    }],
}

compile({
    ...defaultOptions,
    entryPoints: ['./resources/js/components/test-component.js'],
    outfile: './resources/js/dist/components/test-component.js',
})
```

As you can see at the bottom of the script, we are compiling a file called `resources/js/components/test-component.js` into `resources/js/dist/components/test-component.js`. You can change these paths to suit your needs. You can compile as many components as you want.

Now, create a new file called `resources/js/components/test-component.js`:

```js
// Import any external JavaScript libraries from NPM here.

export default function testComponent({
    state,
}) {
    return {
        state,
        
        // You can define any other Alpine.js properties here.

        init() {
            // Initialise the Alpine component here, if you need to.
        },
        
        // You can define any other Alpine.js functions here.
    }
}
```

Now, you can compile this file into `resources/js/dist/components/test-component.js` by running the following command:

```bash
node bin/build.js
```

If you want to watch for changes to this file instead of compiling once, try the following command:

```bash
node bin/build.js --dev
```

Now, you need to tell Filament to publish this compiled JavaScript file into the `/public` directory of the Laravel application, so it is accessible to the browser. To do this, you can use the `FilamentAsset::register()` method in the `boot()` method of a service provider, passing in an `AlpineComponent` object:

```php
use Filament\Support\Assets\AlpineComponent;
use Filament\Support\Facades\FilamentAsset;

FilamentAsset::register([
    AlpineComponent::make('test-component', __DIR__ . '/../../resources/js/dist/components/test-component.js'),
]);
```

When you run `php artisan filament:assets`, the compiled file will be copied into the `/public` directory.

Finally, you can load this asynchronous Alpine component in your view using `x-load` attributes and the `FilamentAsset::getAlpineComponentSrc()` method:

```blade
<div
    x-load
    x-load-src="{{ \Filament\Support\Facades\FilamentAsset::getAlpineComponentSrc('test-component') }}"
    x-data="testComponent({
        state: $wire.{{ $applyStateBindingModifiers("\$entangle('{$statePath}')") }},
    })"
>
    <input x-model="state" />
</div>
```

This example is for a [custom form field](../forms/custom). It passes the `state` in as a parameter to the `testComponent()` function, which is entangled with a Livewire component property. You can pass in any parameters you want, and access them in the `testComponent()` function. If you're not using a custom form field, you can ignore the `state` parameter in this example.

The `x-load` attributes come from the [Async Alpine](https://async-alpine.dev/docs/strategies) package, and any features of that package can be used here.

### Registering script data

Sometimes, you may wish to make data from the backend available to JavaScript files. To do this, you can use the `FilamentAsset::registerScriptData()` method in the `boot()` method of a service provider:

```php
use Filament\Support\Facades\FilamentAsset;

FilamentAsset::registerScriptData([
    'user' => [
        'name' => auth()->user()?->name,
    ],
]);
```

Now, you can access that data from any JavaScript file at runtime, using the `window.filamentData` object:

```js
window.filamentData.user.name // 'Dan Harrin'
```

### Registering JavaScript files from a URL

If you want to register a JavaScript file from a URL, you may do so. These assets will be loaded on every page as normal, but not copied into the `/public` directory when the `php artisan filament:assets` command is run. This is useful for registering external scripts from a CDN, or scripts that you are already compiling directly into the `/public` directory:

```php
use Filament\Support\Assets\Js;

FilamentAsset::register([
    Js::make('example-external-script', 'https://example.com/external.js'),
    Js::make('example-local-script', asset('js/local.js')),
]);
```

---

<!-- Source: 02-css-hooks.md -->

## Introduction

Filament uses CSS "hook" classes to allow various HTML elements to be customized using CSS.

## Discovering hook classes

We could document all the hook classes across the entire Filament UI, but that would be a lot of work, and probably not very useful to you. Instead, we recommend using your browser's developer tools to inspect the elements you want to customize, and then use the hook classes to target those elements.

All hook classes are prefixed with `fi-`, which is a great way to identify them. They are usually right at the start of the class list, so they are easy to find, but sometimes they may fall further down the list if we have to apply them conditionally with JavaScript or Blade.

If you don't find a hook class you're looking for, try not to hack around it, as it might expose your styling customizations to breaking changes in future releases. Instead, please open a pull request to add the hook class you need. We can help you maintain naming consistency. You probably don't even need to pull down the Filament repository locally for these pull requests, as you can just edit the Blade files directly on GitHub.

## Applying styles to hook classes

For example, if you want to customize the color of the sidebar, you can inspect the sidebar element in your browser's developer tools, see that it uses the `fi-sidebar`, and then add CSS to your app like this:

```css
.fi-sidebar {
    background-color: #fafafa;
}
```

Alternatively, since Filament is built upon Tailwind CSS, you can use their `@apply` directive to apply Tailwind classes to Filament elements:

```css
.fi-sidebar {
    @apply bg-gray-50 dark:bg-gray-950;
}
```

Occasionally, you may need to use the `!important` modifier to override existing styles, but please use this sparingly, as it can make your styles difficult to maintain:

```css
.fi-sidebar {
    @apply bg-gray-50 dark:bg-gray-950 !important;
}
```

You can even apply `!important` to only specific Tailwind classes, which is a little less intrusive, by prefixing the class name with `!`:

```css
.fi-sidebar {
    @apply !bg-gray-50 dark:!bg-gray-950;
}
```

## Common hook class abbreviations

We use a few common abbreviations in our hook classes to keep them short and readable:

- `fi` is short for "Filament"
- `fi-ac` is used to represent classes used in the Actions package
- `fi-fo` is used to represent classes used in the Forms package
- `fi-in` is used to represent classes used in the Infolists package
- `fi-no` is used to represent classes used in the Notifications package
- `fi-sc` is used to represent classes used in the Schema package
- `fi-ta` is used to represent classes used in the Tables package
- `fi-wi` is used to represent classes used in the Widgets package
- `btn` is short for "button"
- `col` is short for "column"
- `ctn` is short for "container"
- `wrp` is short for "wrapper"

## Publishing Blade views

You may be tempted to publish the internal Blade views to your application so that you can customize them. We don't recommend this, as it will introduce breaking changes into your application in future updates. Please use the [CSS hook classes](#applying-styles-to-hook-classes) wherever possible.

If you do decide to publish the Blade views, please lock all Filament packages to a specific version in your `composer.json` file, and then update Filament manually by bumping this number, testing your entire application after each update. This will help you identify breaking changes safely.

---

<!-- Source: 02-custom-pages.md -->

## Introduction

Filament allows you to create completely custom pages for the app.

## Creating a page

To create a new page, you can use:

```bash
php artisan make:filament-page Settings
```

This command will create two files - a page class in the `/Pages` directory of the Filament directory, and a view in the `/pages` directory of the Filament views directory.

Page classes are all full-page [Livewire](https://livewire.laravel.com) components with a few extra utilities you can use with the panel.

## Authorization

You can prevent pages from appearing in the menu by overriding the `canAccess()` method in your Page class. This is useful if you want to control which users can see the page in the navigation, and also which users can visit the page directly:

```php
public static function canAccess(): bool
{
    return auth()->user()->canManageSettings();
}
```

## Adding actions to pages

Actions are buttons that can perform tasks on the page, or visit a URL. You can read more about their capabilities [here](../actions).

Since all pages are Livewire components, you can [add actions](../components/action#adding-the-action) anywhere. Pages already have the `InteractsWithActions` trait, `HasActions` interface, and `<x-filament-actions::modals />` Blade component all set up for you.

### Header actions

You can also easily add actions to the header of any page, including [resource pages](resources). You don't need to worry about adding anything to the Blade template, we handle that for you. Just return your actions from the `getHeaderActions()` method of the page class:

```php
use Filament\Actions\Action;

protected function getHeaderActions(): array
{
    return [
        Action::make('edit')
            ->url(route('posts.edit', ['post' => $this->post])),
        Action::make('delete')
            ->requiresConfirmation()
            ->action(fn () => $this->post->delete()),
    ];
}
```

### Opening an action modal when a page loads

You can also open an action when a page loads by setting the `$defaultAction` property to the name of the action you want to open:

```php
use Filament\Actions\Action;

public $defaultAction = 'onboarding';

public function onboardingAction(): Action
{
    return Action::make('onboarding')
        ->modalHeading('Welcome')
        ->visible(fn (): bool => ! auth()->user()->isOnBoarded());
}
```

You can also pass an array of arguments to the default action using the `$defaultActionArguments` property:

```php
public $defaultActionArguments = ['step' => 2];
```

Alternatively, you can open an action modal when a page loads by specifying the `action` as a query string parameter to the page:

```
/admin/products/edit/932510?action=onboarding
```

### Refreshing form data

If you're using actions on an [Edit](resources/editing-records) or [View](resources/viewing-records) resource page, you can refresh data within the main form using the `refreshFormData()` method:

```php
use App\Models\Post;
use Filament\Actions\Action;

Action::make('approve')
    ->action(function (Post $record) {
        $record->approve();

        $this->refreshFormData([
            'status',
        ]);
    })
```

This method accepts an array of model attributes that you wish to refresh in the form.

## Adding widgets to pages

Filament allows you to display [widgets](dashboard) inside pages, below the header and above the footer.

To add a widget to a page, use the `getHeaderWidgets()` or `getFooterWidgets()` methods:

```php
use App\Filament\Widgets\StatsOverviewWidget;

protected function getHeaderWidgets(): array
{
    return [
        StatsOverviewWidget::class
    ];
}
```

`getHeaderWidgets()` returns an array of widgets to display above the page content, whereas `getFooterWidgets()` are displayed below.

If you'd like to learn how to build and customize widgets, check out the [Dashboard](dashboard) documentation section.

### Customizing the widgets' grid

You may change how many grid columns are used to display widgets.

You may override the `getHeaderWidgetsColumns()` or `getFooterWidgetsColumns()` methods to return a number of grid columns to use:

```php
public function getHeaderWidgetsColumns(): int | array
{
    return 3;
}
```

#### Responsive widgets grid

You may wish to change the number of widget grid columns based on the responsive [breakpoint](https://tailwindcss.com/docs/responsive-design#overview) of the browser. You can do this using an array that contains the number of columns that should be used at each breakpoint:

```php
public function getHeaderWidgetsColumns(): int | array
{
    return [
        'md' => 4,
        'xl' => 5,
    ];
}
```

This pairs well with [responsive widget widths](dashboard#responsive-widget-widths).

#### Passing data to widgets from the page

You may pass data to widgets from the page using the `getWidgetData()` method:

```php
public function getWidgetData(): array
{
    return [
        'stats' => [
            'total' => 100,
        ],
    ];
}
```

Now, you can define a corresponding public `$stats` array property on the widget class, which will be automatically filled:

```php
public $stats = [];
```

### Passing properties to widgets on pages

When registering a widget on a page, you can use the `make()` method to pass an array of [Livewire properties](https://livewire.laravel.com/docs/properties) to it:

```php
use App\Filament\Widgets\StatsOverviewWidget;

protected function getHeaderWidgets(): array
{
    return [
        StatsOverviewWidget::make([
            'status' => 'active',
        ]),
    ];
}
```

This array of properties gets mapped to [public Livewire properties](https://livewire.laravel.com/docs/properties) on the widget class:

```php
use Filament\Widgets\Widget;

class StatsOverviewWidget extends Widget
{
    public string $status;

    // ...
}
```

Now, you can access the `status` in the widget class using `$this->status`.

## Customizing the page title

By default, Filament will automatically generate a title for your page based on its name. You may override this by defining a `$title` property on your page class:

```php
protected static ?string $title = 'Custom Page Title';
```

Alternatively, you may return a string from the `getTitle()` method:

```php
use Illuminate\Contracts\Support\Htmlable;

public function getTitle(): string | Htmlable
{
    return __('Custom Page Title');
}
```

## Customizing the page navigation label

By default, Filament will use the page's [title](#customizing-the-page-title) as its [navigation](navigation) item label. You may override this by defining a `$navigationLabel` property on your page class:

```php
protected static ?string $navigationLabel = 'Custom Navigation Label';
```

Alternatively, you may return a string from the `getNavigationLabel()` method:

```php
public static function getNavigationLabel(): string
{
    return __('Custom Navigation Label');
}
```

## Customizing the page URL

By default, Filament will automatically generate a URL (slug) for your page based on its name. You may override this by defining a `$slug` property on your page class:

```php
protected static ?string $slug = 'custom-url-slug';
```

## Customizing the page heading

By default, Filament will use the page's [title](#customizing-the-page-title) as its heading. You may override this by defining a `$heading` property on your page class:

```php
protected ?string $heading = 'Custom Page Heading';
```

Alternatively, you may return a string from the `getHeading()` method:

```php
public function getHeading(): string
{
    return __('Custom Page Heading');
}
```

### Adding a page subheading

You may also add a subheading to your page by defining a `$subheading` property on your page class:

```php
protected ?string $subheading = 'Custom Page Subheading';
```

Alternatively, you may return a string from the `getSubheading()` method:

```php
public function getSubheading(): ?string
{
    return __('Custom Page Subheading');
}
```

## Replacing the page header with a custom view

You may replace the default [heading](#customizing-the-page-heading), [subheading](#adding-a-page-subheading) and [actions](#header-actions) with a custom header view for any page. You may return it from the `getHeader()` method:

```php
use Illuminate\Contracts\View\View;

public function getHeader(): ?View
{
    return view('filament.settings.custom-header');
}
```

This example assumes you have a Blade view at `resources/views/filament/settings/custom-header.blade.php`.

## Rendering a custom view in the footer of the page

You may also add a footer to any page, below its content. You may return it from the `getFooter()` method:

```php
use Illuminate\Contracts\View\View;

public function getFooter(): ?View
{
    return view('filament.settings.custom-footer');
}
```

This example assumes you have a Blade view at `resources/views/filament/settings/custom-footer.blade.php`.

## Customizing the maximum content width

By default, Filament will restrict the width of the content on the page, so it doesn't become too wide on large screens. To change this, you may override the `getMaxContentWidth()` method. Options correspond to [Tailwind's max-width scale](https://tailwindcss.com/docs/max-width). The options are `ExtraSmall`, `Small`, `Medium`, `Large`, `ExtraLarge`, `TwoExtraLarge`, `ThreeExtraLarge`, `FourExtraLarge`, `FiveExtraLarge`, `SixExtraLarge`, `SevenExtraLarge`, `Full`, `MinContent`, `MaxContent`, `FitContent`,  `Prose`, `ScreenSmall`, `ScreenMedium`, `ScreenLarge`, `ScreenExtraLarge` and `ScreenTwoExtraLarge`. The default is `SevenExtraLarge`:

```php
use Filament\Support\Enums\Width;

public function getMaxContentWidth(): Width
{
    return Width::Full;
}
```

## Generating URLs to pages

Filament provides `getUrl()` static method on page classes to generate URLs to them. Traditionally, you would need to construct the URL by hand or by using Laravel's `route()` helper, but these methods depend on knowledge of the page's slug or route naming conventions.

The `getUrl()` method, without any arguments, will generate a URL:

```php
use App\Filament\Pages\Settings;

Settings::getUrl(); // /admin/settings
```

If your page uses URL / query parameters, you should use the argument:

```php
use App\Filament\Pages\Settings;

Settings::getUrl(['section' => 'notifications']); // /admin/settings?section=notifications
```

### Generating URLs to pages in other panels

If you have multiple panels in your app, `getUrl()` will generate a URL within the current panel. You can also indicate which panel the page is associated with, by passing the panel ID to the `panel` argument:

```php
use App\Filament\Pages\Settings;

Settings::getUrl(panel: 'marketing');
```

## Adding sub-navigation between pages

You may want to add a common sub-navigation to multiple pages, to allow users to quickly navigate between them. You can do this by defining a [cluster](clusters). Clusters can also contain [resources](resources), and you can switch between multiple pages or resources within a cluster.

## Setting the sub-navigation position

The sub-navigation is rendered at the start of the page by default. You may change the position for a page by setting the `$subNavigationPosition` property on the page. The value may be `SubNavigationPosition::Start`, `SubNavigationPosition::End`, or `SubNavigationPosition::Top` to render the sub-navigation as tabs:

```php
use Filament\Pages\Enums\SubNavigationPosition;

protected static ?SubNavigationPosition $subNavigationPosition = SubNavigationPosition::End;
```

## Adding extra attributes to the body tag of a page

You may wish to add extra attributes to the `<body>` tag of a page. To do this, you can set an array of attributes in `$extraBodyAttributes`:

```php
protected array $extraBodyAttributes = [];
```

Or, you can return an array of attributes and their values from the `getExtraBodyAttributes()` method:

```php
public function getExtraBodyAttributes(): array
{
    return [
        'class' => 'settings-page',
    ];
}
```

---

<!-- Source: 02-form.md -->

import Aside from "@components/Aside.astro"

<Aside variant="warning">
    Before proceeding, make sure `filament/forms` is installed in your project. You can check by running:

    ```bash
    composer show filament/forms
    ```
    If it's not installed, consult the [installation guide](../introduction/installation#installing-the-individual-components) and configure the **individual components** according to the instructions.
</Aside>

## Setting up the Livewire component

First, generate a new Livewire component:

```bash
php artisan make:livewire CreatePost
```

Then, render your Livewire component on the page:

```blade
@livewire('create-post')
```

Alternatively, you can use a full-page Livewire component:

```php
use App\Livewire\CreatePost;
use Illuminate\Support\Facades\Route;

Route::get('posts/create', CreatePost::class);
```

## Adding the form

<Aside variant="warning">
    Before proceeding, please ensure that the **Forms package** is installed in your project. Consult the [installation guide](../introduction/installation#installing-the-individual-components) and configure the **individual components** according to the instructions.
</Aside>

There are 5 main tasks when adding a form to a Livewire component class. Each one is essential:

1) Implement the `HasSchemas` interface and use the `InteractsWithSchemas` trait.
2) Define a public Livewire property to store your form's data. In our example, we'll call this `$data`, but you can call it whatever you want.
3) Add a `form()` method, which is where you configure the form. [Add the form's schema](getting-started#form-schemas), and tell Filament to store the form data in the `$data` property (using `statePath('data')`).
4) Initialize the form with `$this->form->fill()` in `mount()`. This is imperative for every form that you build, even if it doesn't have any initial data.
5) Define a method to handle the form submission. In our example, we'll call this `create()`, but you can call it whatever you want. Inside that method, you can validate and get the form's data using `$this->form->getState()`. It's important that you use this method instead of accessing the `$this->data` property directly, because the form's data needs to be validated and transformed into a useful format before being returned.

```php
<?php

namespace App\Livewire;

use Filament\Forms\Components\MarkdownEditor;
use Filament\Forms\Components\TextInput;
use Filament\Schemas\Concerns\InteractsWithSchemas;
use Filament\Schemas\Contracts\HasSchemas;
use Illuminate\Contracts\View\View;
use Filament\Schemas\Schema;
use Livewire\Component;

class CreatePost extends Component implements HasSchemas
{
    use InteractsWithSchemas;
    
    public ?array $data = [];
    
    public function mount(): void
    {
        $this->form->fill();
    }
    
    public function form(Schema $schema): Schema
    {
        return $schema
            ->components([
                TextInput::make('title')
                    ->required(),
                MarkdownEditor::make('content'),
                // ...
            ])
            ->statePath('data');
    }
    
    public function create(): void
    {
        dd($this->form->getState());
    }
    
    public function render(): View
    {
        return view('livewire.create-post');
    }
}
```

Finally, in your Livewire component's view, render the form:

```blade
<div>
    <form wire:submit="create">
        {{ $this->form }}
        
        <button type="submit">
            Submit
        </button>
    </form>
    
    <x-filament-actions::modals />
</div>
```

<Aside variant="info">
    `<x-filament-actions::modals />` is used to render form component [action modals](../schemas/actions). The code can be put anywhere outside the `<form>` element, as long as it's within the Livewire component.
</Aside>

Visit your Livewire component in the browser, and you should see the form components from `components()`:

Submit the form with data, and you'll see the form's data dumped to the screen. You can save the data to a model instead of dumping it:

```php
use App\Models\Post;

public function create(): void
{
    Post::create($this->form->getState());
}
```

<Aside variant="info">
    `filament/forms` also includes the following packages:

    - `filament/actions`
    - `filament/schemas`
    - `filament/support`
    
    These packages allow you to use their components within Livewire components.
    For example, if your form uses [Actions](../actions), remember to implement the `HasActions` interface and use the `InteractsWithActions` trait on your Livewire component class.
    
    If you are using any other [Filament components](overview#package-components) in your form, make sure to install and integrate the corresponding package as well.
</Aside>

## Initializing the form with data

To fill the form with data, just pass that data to the `$this->form->fill()` method. For example, if you're editing an existing post, you might do something like this:

```php
use App\Models\Post;

public function mount(Post $post): void
{
    $this->form->fill($post->attributesToArray());
}
```

It's important that you use the `$this->form->fill()` method instead of assigning the data directly to the `$this->data` property. This is because the post's data needs to be internally transformed into a useful format before being stored.

## Setting a form model

Giving the `$form` access to a model is useful for a few reasons:

- It allows fields within that form to load information from that model. For example, select fields can [load their options from the database](fields/select#integrating-with-an-eloquent-relationship) automatically.
- The form can load and save the model's relationship data automatically. For example, you have an Edit Post form, with a [Repeater](fields/repeater#integrating-with-an-eloquent-relationship) which manages comments associated with that post. Filament will automatically load the comments for that post when you call `$this->form->fill([...])`, and save them back to the relationship when you call `$this->form->getState()`.
- Validation rules like `exists()` and `unique()` can automatically retrieve the database table name from the model.

It is advised to always pass the model to the form when there is one. As explained, it unlocks many new powers of Filament's form system.

To pass the model to the form, use the `$form->model()` method:

```php
use Filament\Schemas\Schema;

public Post $post;

public function form(Schema $schema): Schema
{
    return $schema
        ->components([
            // ...
        ])
        ->statePath('data')
        ->model($this->post);
}
```

### Passing the form model after the form has been submitted

In some cases, the form's model is not available until the form has been submitted. For example, in a Create Post form, the post does not exist until the form has been submitted. Therefore, you can't pass it in to `$form->model()`. However, you can pass a model class instead:

```php
use App\Models\Post;
use Filament\Schemas\Schema;

public function form(Schema $schema): Schema
{
    return $schema
        ->components([
            // ...
        ])
        ->statePath('data')
        ->model(Post::class);
}
```

On its own, this isn't as powerful as passing a model instance. For example, relationships won't be saved to the post after it is created. To do that, you'll need to pass the post to the form after it has been created, and call `saveRelationships()` to save the relationships to it:

```php
use App\Models\Post;

public function create(): void
{
    $post = Post::create($this->form->getState());
    
    // Save the relationships from the form to the post after it is created.
    $this->form->model($post)->saveRelationships();
}
```

## Saving form data to individual properties

In all of our previous examples, we've been saving the form's data to the public `$data` property on the Livewire component. However, you can save the data to individual properties instead. For example, if you have a form with a `title` field, you can save the form's data to the `$title` property instead. To do this, don't pass a `statePath()` to the form at all. Ensure that all of your fields have their own **public** properties on the class.

```php
use Filament\Forms\Components\MarkdownEditor;
use Filament\Forms\Components\TextInput;
use Filament\Schemas\Schema;

public ?string $title = null;

public ?string $content = null;

public function form(Schema $schema): Schema
{
    return $schema
        ->components([
            TextInput::make('title')
                ->required(),
            MarkdownEditor::make('content'),
            // ...
        ]);
}
```

## Using multiple forms

Many forms can be defined using the `InteractsWithSchemas` trait. Each of the forms should use a method with the same name:

```php
use Filament\Forms\Components\MarkdownEditor;
use Filament\Forms\Components\TextInput;
use Filament\Schemas\Schema;

public ?array $postData = [];

public ?array $commentData = [];

public function editPostForm(Schema $schema): Schema
{
    return $schema
        ->components([
            TextInput::make('title')
                ->required(),
            MarkdownEditor::make('content'),
            // ...
        ])
        ->statePath('postData')
        ->model($this->post);
}

public function createCommentForm(Schema $schema): Schema
{
    return $schema
        ->components([
            TextInput::make('name')
                ->required(),
            TextInput::make('email')
                ->email()
                ->required(),
            MarkdownEditor::make('content')
                ->required(),
            // ...
        ])
        ->statePath('commentData')
        ->model(Comment::class);
}
```

Now, each form is addressable by its name instead of `form`. For example, to fill the post form, you can use `$this->editPostForm->fill([...])`, or to get the data from the comment form you can use `$this->createCommentForm->getState()`.

You'll notice that each form has its own unique `statePath()`. Each form will write its state to a different array on your Livewire component, so it's important to define the public properties, `$postData` and `$commentData` in this example.

## Resetting a form's data

You can reset a form back to its default data at any time by calling `$this->form->fill()`. For example, you may wish to clear the contents of a form every time it's submitted:

```php
use App\Models\Comment;

public function createComment(): void
{
    Comment::create($this->form->getState());

    // Reinitialize the form to clear its data.
    $this->form->fill();
}
```

## Generating form Livewire components with the CLI

It's advised that you learn how to set up a Livewire component with forms manually, but once you are confident, you can use the CLI to generate a form for you.

```bash
php artisan make:filament-livewire-form RegistrationForm
```

This will generate a new `app/Livewire/RegistrationForm.php` component, which you can customize.

### Generating a form for an Eloquent model

Filament is also able to generate forms for a specific Eloquent model. These are more powerful, as they will automatically save the data in the form for you, and [ensure the form fields are properly configured](#setting-a-form-model) to access that model.

When generating a form with the `make:livewire-form` command, it will ask for the name of the model:

```bash
php artisan make:filament-livewire-form Products/CreateProduct
```

#### Generating an edit form for an Eloquent record

By default, passing a model to the `make:livewire-form` command will result in a form that creates a new record in your database. If you pass the `--edit` flag to the command, it will generate an edit form for a specific record. This will automatically fill the form with the data from the record, and save the data back to the model when the form is submitted.

```bash
php artisan make:filament-livewire-form Products/EditProduct --edit
```

### Automatically generating form schemas

Filament is also able to guess which form fields you want in the schema, based on the model's database columns. You can use the `--generate` flag when generating your form:

```bash
php artisan make:filament-livewire-form Products/CreateProduct --generate
```

---

<!-- Source: 02-getting-started.md -->

import Aside from "@components/Aside.astro"

Once you have [installed Filament](introduction/installation#installing-the-panel-builder), you can start building your application.

<Aside variant="info">
    This guide is for the Filament panel builder. If you are looking to use the Filament UI components outside of a panel, visit the [Components](components) documentation.
</Aside>

To start, visit `/admin` and sign in with a user account. You will be redirected to the default dashboard of the panel.

## Resources

Resources are the core of your Filament application. They are CRUD UIs for models that you want to manage in the panel.

Out of the box, Filament will generate three pages in each resource:
- **List**: A paginated table of all the records in the Eloquent model.
- **Create**: A form to create a new record.
- **Edit**: A form to edit an existing record.

You can also choose to generate a **View** page, which is a read-only display of a record.

Each resource usually has an item in the sidebar, which is automatically registered as soon as you create a resource.

To start your journey by creating a resource, visit the [Resources documentation](resources).

## Widgets

Widgets are components often used to build a dashboard, typically to display statistical data. Charts, numbers, tables, and completely custom widgets are supported.

Each widget has a PHP class and a Blade view. The PHP class is technically a [Livewire component](https://livewire.laravel.com/docs/components). As such, every widget has access to the full power of Livewire to build an interactive server-rendered UI.

Out of the box, Filament's dashboard contains a couple of widgets: one which greets the user and allows them to sign out, and another which displays information about Filament.

To start your journey by adding your own widget to the dashboard, visit the [Widgets documentation](widgets).

## Custom pages

Custom pages are a blank canvas for you to build whatever you want in a panel. They are often used for settings pages, documentation, or anything else you can think of.

Each custom page has a PHP class and a Blade view. The PHP class is technically a [full-page Livewire component](https://livewire.laravel.com/docs/components) (in fact, every page class in a Filament panel is a Livewire component). As such, every page has access to the full power of Livewire to build an interactive server-rendered UI.

To start your journey by creating a custom page, visit the [Custom pages documentation](navigation/custom-pages).

---

<!-- Source: 02-infolist.md -->

import Aside from "@components/Aside.astro"

<Aside variant="warning">
    Before proceeding, make sure `filament/infolists` is installed in your project. You can check by running:

    ```bash
    composer show filament/infolists
    ```
    If it's not installed, consult the [installation guide](../introduction/installation#installing-the-individual-components) and configure the **individual components** according to the instructions.
</Aside>

## Setting up the Livewire component

First, generate a new Livewire component:

```bash
php artisan make:livewire ViewProduct
```

Then, render your Livewire component on the page:

```blade
@livewire('view-product')
```

Alternatively, you can use a full-page Livewire component:

```php
use App\Livewire\ViewProduct;
use Illuminate\Support\Facades\Route;

Route::get('products/{product}', ViewProduct::class);
```

You must use the `InteractsWithSchemas` trait, and implement the `HasSchemas` interface on your Livewire component class:

```php
use Filament\Schemas\Concerns\InteractsWithSchemas;
use Filament\Schemas\Contracts\HasSchemas;
use Livewire\Component;

class ViewProduct extends Component implements HasSchemas
{
    use InteractsWithSchemas;

    // ...
}
```

## Adding the infolist

Next, add a method to the Livewire component which accepts an `$infolist` object, modifies it, and returns it:

```php
use Filament\Schemas\Schema;

public function productInfolist(Schema $schema): Schema
{
    return $schema
        ->record($this->product)
        ->components([
            // ...
        ]);
}
```

Finally, render the infolist in the Livewire component's view:

```blade
{{ $this->productInfolist }}
```

<Aside variant="info">
    `filament/infolists` also includes the following packages:

    - `filament/actions`
    - `filament/schemas`
    - `filament/support`
    
    These packages allow you to use their components within Livewire components.
    For example, if your infolist uses [Actions](../actions), remember to implement the `HasActions` interface and use the `InteractsWithActions` trait on your Livewire component class.
    
    If you are using any other [Filament components](overview#package-components) in your infolist, make sure to install and integrate the corresponding package as well.

</Aside>

## Passing data to the infolist

You can pass data to the infolist in two ways:

Either pass an Eloquent model instance to the `record()` method of the infolist, to automatically map all the model attributes and relationships to the entries in the infolist's schema:

```php
use Filament\Infolists\Components\TextEntry;
use Filament\Schemas\Schema;

public function productInfolist(Schema $schema): Schema
{
    return $schema
        ->record($this->product)
        ->components([
            TextEntry::make('name'),
            TextEntry::make('category.name'),
            // ...
        ]);
}
```

Alternatively, you can pass an array of data to the `state()` method of the infolist, to manually map the data to the entries in the infolist's schema:

```php
use Filament\Infolists\Components\TextEntry;
use Filament\Schemas\Schema;

public function productInfolist(Schema $schema): Schema
{
    return $schema
        ->constantState([
            'name' => 'MacBook Pro',
            'category' => [
                'name' => 'Laptops',
            ],
            // ...
        ])
        ->components([
            TextEntry::make('name'),
            TextEntry::make('category.name'),
            // ...
        ]);
}
```

---

<!-- Source: 02-installation.md -->

import Aside from "@components/Aside.astro"
import RadioGroup from "@components/RadioGroup.astro"
import RadioGroupOption from "@components/RadioGroupOption.astro"

Filament requires the following to run:

- PHP 8.2+
- Laravel v11.28+
- Tailwind CSS v4.0+

Installation comes in two flavors, depending on whether you want to build an app using our panel builder or use the components within your app's Blade views:

<div x-data="{ package: (window.location.hash === '#components') ? 'components' : 'panels' }">

<RadioGroup model="package">
    <RadioGroupOption value="panels">
        Panel builder

        <span slot="description">
            Most people choose this option to build a panel (e.g., admin panel) for their app. The panel builder combines all the individual components into a cohesive framework. You can create as many panels as you like within a Laravel installation, but you only need to install it once.
        </span>
    </RadioGroupOption>

    <RadioGroupOption value="components">
        Individual components

        <span slot="description">
            If you are using Blade to build your app from scratch, you can install individual components from Filament to enrich your UI.
        </span>
    </RadioGroupOption>
</RadioGroup>

<div x-show="package === 'panels'" x-cloak>

## Installing the panel builder

Install the Filament Panel Builder by running the following commands in your Laravel project directory:

```bash
composer require filament/filament:"^4.0"

php artisan filament:install --panels
```

<Aside variant="warning">
    When using Windows PowerShell to install Filament, you may need to run the command below, since it ignores `^` characters in version constraints:

    ```bash
    composer require filament/filament:"~4.0"

    php artisan filament:install --panels
    ```
</Aside>

This will create and register a new [Laravel service provider](https://laravel.com/docs/providers) called `app/Providers/Filament/AdminPanelProvider.php`.

<Aside variant="tip">
    If you get an error when accessing your panel, check that the service provider is registered in `bootstrap/providers.php`. If it's not registered, you'll need to [add it manually](https://laravel.com/docs/providers#registering-providers).
</Aside>

You can create a new user account using the following command:

```bash
php artisan make:filament-user
```

Open `/admin` in your web browser, sign in, and [start building your app](../getting-started)!

</div>

<div
    x-show="package === 'components'" 
    x-data="{ laravelProject: 'new' }"
    x-cloak
>

## Installing the individual components

Install the Filament components you want to use with Composer:

```bash
composer require
    filament/tables:"^4.0"
    filament/schemas:"^4.0"
    filament/forms:"^4.0"
    filament/infolists:"^4.0"
    filament/actions:"^4.0"
    filament/notifications:"^4.0"
    filament/widgets:"^4.0"
```

You can install additional packages later in your project without having to repeat these installation steps.

<Aside variant="warning">
    When using Windows PowerShell to install Filament, you may need to run the command below, since it ignores `^` characters in version constraints:

    ```bash
    composer require
        filament/tables:"~4.0"
        filament/schemas:"~4.0"
        filament/forms:"~4.0"
        filament/infolists:"~4.0"
        filament/actions:"~4.0"
        filament/notifications:"~4.0"
        filament/widgets:"~4.0"
    ```
</Aside>

If you only want to use the set of [Blade UI components](../components), you'll need to require `filament/support` at this stage.

<RadioGroup model="laravelProject">
    <RadioGroupOption value="new">
        New Laravel projects

        <span slot="description">
            Get started with Filament components quickly by running a simple command. Note that this will overwrite any modified files in your app, so it's only suitable for new Laravel projects.
        </span>
    </RadioGroupOption>

    <RadioGroupOption value="existing">
        Existing Laravel projects

        <span slot="description">
            If you have an existing Laravel project, you can still install Filament, but should do so manually to preserve your existing functionality.
        </span>
    </RadioGroupOption>
</RadioGroup>

<div x-show="laravelProject === 'new'" x-cloak>

To quickly set up Filament in a new Laravel project, run the following commands to install [Livewire](https://livewire.laravel.com), [Alpine.js](https://alpinejs.dev), and [Tailwind CSS](https://tailwindcss.com):

<Aside variant="warning">
    These commands will overwrite existing files in your application. Only run them in a new Laravel project!
</Aside>

Run the following command to install the Filament frontend assets:

```bash
php artisan filament:install --scaffold

npm install

npm run dev
```

During scaffolding, if you have the [Notifications](../notifications) package installed, Filament will ask if you want to install the required Livewire component into your default layout file. This component is required if you want to send flash notifications to users through Filament.

</div>

<div x-show="laravelProject === 'existing'" x-cloak>

Run the following command to install the Filament frontend assets:

```bash
php artisan filament:install
```

### Installing Tailwind CSS

Run the following command to install Tailwind CSS and its Vite plugin, if you don't have those installed already:

```bash
npm install tailwindcss @tailwindcss/vite --save-dev
```

### Configuring styles

To configure Filament's styles, you need to import the CSS files for the Filament packages you installed, usually in `resources/css/app.css`.

Depending on which combination of packages you installed, you can import only the necessary CSS files, to keep your app's CSS bundle size small:

```css
@import 'tailwindcss';

/* Required by all components */
@import '../../vendor/filament/support/resources/css/index.css';

/* Required by actions and tables */
@import '../../vendor/filament/actions/resources/css/index.css';

/* Required by actions, forms and tables */
@import '../../vendor/filament/forms/resources/css/index.css';

/* Required by actions and infolists */
@import '../../vendor/filament/infolists/resources/css/index.css';

/* Required by notifications */
@import '../../vendor/filament/notifications/resources/css/index.css';

/* Required by actions, infolists, forms, schemas and tables */
@import '../../vendor/filament/schemas/resources/css/index.css';

/* Required by tables */
@import '../../vendor/filament/tables/resources/css/index.css';

/* Required by widgets */
@import '../../vendor/filament/widgets/resources/css/index.css';

@variant dark (&:where(.dark, .dark *));
```

### Configure the Vite plugin

If it isn't already set up, add the `@tailwindcss/vite` plugin to your Vite configuration (`vite.config,js`):

```js
import { defineConfig } from 'vite'
import laravel from 'laravel-vite-plugin'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
        tailwindcss(),
    ],
})
```

### Compiling assets

Compile your new CSS and JavaScript assets using `npm run dev`.

### Configuring your layout 

If you don't have a Blade layout file yet, create it at `resources/views/components/layouts/app.blade.php` by running the following command:

```bash
php artisan livewire:layout
```

Add the following code to your new layout file:

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">

        <meta name="application-name" content="{{ config('app.name') }}">
        <meta name="csrf-token" content="{{ csrf_token() }}">
        <meta name="viewport" content="width=device-width, initial-scale=1">

        <title>{{ config('app.name') }}</title>

        <style>
            [x-cloak] {
                display: none !important;
            }
        </style>

        @filamentStyles
        @vite('resources/css/app.css')
    </head>

    <body class="antialiased">
        {{ $slot }}

        @livewire('notifications') {{-- Only required if you wish to send flash notifications --}}

        @filamentScripts
        @vite('resources/js/app.js')
    </body>
</html>
```

The important parts of this are the `@filamentStyles` in the `<head>` of the layout, and the `@filamentScripts` at the end of the `<body>`. Make sure to also include your app's compiled CSS and JavaScript files from Vite!

<Aside variant="info">
    The `@livewire('notifications')` line is required in the layout only if you have the [Notifications](../notifications) package installed and want to send flash notifications to users through Filament.
</Aside>

</div>

</div>

</div>

---

<!-- Source: 02-listing-records.md -->

## Using tabs to filter the records

You can add tabs above the table, which can be used to filter the records based on some predefined conditions. Each tab can scope the Eloquent query of the table in a different way. To register tabs, add a `getTabs()` method to the List page class, and return an array of `Tab` objects:

```php
use Filament\Schemas\Components\Tabs\Tab;
use Illuminate\Database\Eloquent\Builder;

public function getTabs(): array
{
    return [
        'all' => Tab::make(),
        'active' => Tab::make()
            ->modifyQueryUsing(fn (Builder $query) => $query->where('active', true)),
        'inactive' => Tab::make()
            ->modifyQueryUsing(fn (Builder $query) => $query->where('active', false)),
    ];
}
```

### Customizing the filter tab labels

The keys of the array will be used as identifiers for the tabs, so they can be persisted in the URL's query string. The label of each tab is also generated from the key, but you can override that by passing a label into the `make()` method of the tab:

```php
use Filament\Schemas\Components\Tabs\Tab;
use Illuminate\Database\Eloquent\Builder;

public function getTabs(): array
{
    return [
        'all' => Tab::make('All customers'),
        'active' => Tab::make('Active customers')
            ->modifyQueryUsing(fn (Builder $query) => $query->where('active', true)),
        'inactive' => Tab::make('Inactive customers')
            ->modifyQueryUsing(fn (Builder $query) => $query->where('active', false)),
    ];
}
```

### Adding icons to filter tabs

You can add icons to the tabs by passing an [icon](../styling/icons) into the `icon()` method of the tab:

```php
use use Filament\Schemas\Components\Tabs\Tab;

Tab::make()
    ->icon('heroicon-m-user-group')
```

You can also change the icon's position to be after the label instead of before it, using the `iconPosition()` method:

```php
use Filament\Support\Enums\IconPosition;

Tab::make()
    ->icon('heroicon-m-user-group')
    ->iconPosition(IconPosition::After)
```

### Adding badges to filter tabs

You can add badges to the tabs by passing a string into the `badge()` method of the tab:

```php
use Filament\Schemas\Components\Tabs\Tab;

Tab::make()
    ->badge(Customer::query()->where('active', true)->count())
```

#### Changing the color of filter tab badges

The color of a badge may be changed using the `badgeColor()` method:

```php
use Filament\Schemas\Components\Tabs\Tab;

Tab::make()
    ->badge(Customer::query()->where('active', true)->count())
    ->badgeColor('success')
```

### Adding extra attributes to filter tabs

You may also pass extra HTML attributes to filter tabs using `extraAttributes()`:

```php
use Filament\Schemas\Components\Tabs\Tab;

Tab::make()
    ->extraAttributes(['data-cy' => 'statement-confirmed-tab'])
```

### Customizing the default tab

To customize the default tab that is selected when the page is loaded, you can return the array key of the tab from the `getDefaultActiveTab()` method:

```php
use Filament\Schemas\Components\Tabs\Tab;

public function getTabs(): array
{
    return [
        'all' => Tab::make(),
        'active' => Tab::make(),
        'inactive' => Tab::make(),
    ];
}

public function getDefaultActiveTab(): string | int | null
{
    return 'active';
}
```

## Authorization

For authorization, Filament will observe any [model policies](https://laravel.com/docs/authorization#creating-policies) that are registered in your app.

Users may access the List page if the `viewAny()` method of the model policy returns `true`.

The `reorder()` method is used to control [reordering a record](#reordering-records).

## Customizing the table Eloquent query

Although you can [customize the Eloquent query for the entire resource](overview#customizing-the-resource-eloquent-query), you may also make specific modifications for the List page table. To do this, use the `modifyQueryUsing()` method in the `table()` method of the resource:

```php
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;

public static function table(Table $table): Table
{
    return $table
        ->modifyQueryUsing(fn (Builder $query) => $query->withoutGlobalScopes());
}
```

## Custom page content

Each page in Filament has its own [schema](../schemas), which defines the overall structure and content. You can override the schema for the page by defining a `content()` method on it. The `content()` method for the List page contains the following components by default:

```php
use Filament\Schemas\Components\EmbeddedTable;
use Filament\Schemas\Components\RenderHook;
use Filament\Schemas\Schema;

public function content(Schema $schema): Schema
{
    return $schema
        ->components([
            $this->getTabsContentComponent(), // This method returns a component to display the tabs above a table
            RenderHook::make(PanelsRenderHook::RESOURCE_PAGES_LIST_RECORDS_TABLE_BEFORE),
            EmbeddedTable::make(), // This is the component that renders the table that is defined in this resource
            RenderHook::make(PanelsRenderHook::RESOURCE_PAGES_LIST_RECORDS_TABLE_AFTER),
        ]);
}
```

Inside the `components()` array, you can insert any [schema component](../schemas). You can reorder the components by changing the order of the array or remove any of the components that are not needed.

### Using a custom Blade view

For further customization opportunities, you can override the static `$view` property on the page class to a custom view in your app:

```php
protected string $view = 'filament.resources.users.pages.list-users';
```

This assumes that you have created a view at `resources/views/filament/resources/users/pages/list-users.blade.php`:

```blade
<x-filament-panels::page>
    {{ $this->content }} {{-- This will render the content of the page defined in the `content()` method, which can be removed if you want to start from scratch --}}
</x-filament-panels::page>
```

---

<!-- Source: 02-multi-factor-authentication.md -->

import Aside from "@components/Aside.astro"

## Introduction

Users in Filament can sign in with their email address and password by default. However, you can enable multi-factor authentication (MFA) to add an extra layer of security to your users' accounts.

When MFA is enabled, users must perform an extra step before they are authenticated and have access to the application.

Filament includes two methods of MFA which you can enable out of the box:

- [App authentication](#app-authentication) uses a Google Authenticator-compatible app (such as the Google Authenticator, Authy, or Microsoft Authenticator apps) to generate a time-based one-time password (TOTP) that is used to verify the user.
- [Email authentication](#email-authentication) sends a one-time code to the user's email address, which they must enter to verify their identity.

In Filament, users set up multi-factor authentication from their [profile page](overview#authentication-features). If you use Filament's profile page feature, setting up multi-factor authentication will automatically add the correct UI elements to the profile page:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->profile();
}
```

## App authentication

To enable app authentication in a panel, you must first add a new column to your `users` table (or whichever table is being used for your "authenticatable" Eloquent model in this panel). The column needs to store the secret key used to generate and verify the time-based one-time passwords. It can be a normal `text()` column in a migration:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->text('app_authentication_secret')->nullable();
});
```

In the `User` model, you need to ensure that this column is encrypted and `$hidden`, since this is incredibly sensitive information that should be stored securely:

```php
use Filament\Models\Contracts\FilamentUser;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements FilamentUser, MustVerifyEmail
{
    // ...

    /**
     * @var array<string>
     */
    protected $hidden = [
        // ...
        'app_authentication_secret',
    ];

    /**
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            // ...
            'app_authentication_secret' => 'encrypted',
        ];
    }
    
    // ...
}
```

Next, you should implement the `HasAppAuthentication` interface on the `User` model. This provides Filament with the necessary methods to interact with the secret code and other information about the integration:

```php
use Filament\Auth\MultiFactor\App\Contracts\HasAppAuthentication;
use Filament\Models\Contracts\FilamentUser;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements FilamentUser, HasAppAuthentication, MustVerifyEmail
{
    // ...

    public function getAppAuthenticationSecret(): ?string
    {
        // This method should return the user's saved app authentication secret.
    
        return $this->app_authentication_secret;
    }

    public function saveAppAuthenticationSecret(?string $secret): void
    {
        // This method should save the user's app authentication secret.
    
        $this->app_authentication_secret = $secret;
        $this->save();
    }

    public function getAppAuthenticationHolderName(): string
    {
        // In a user's authentication app, each account can be represented by a "holder name".
        // If the user has multiple accounts in your app, it might be a good idea to use
        // their email address as then they are still uniquely identifiable.
    
        return $this->email;
    }
}
```

<Aside variant="tip">
    Since Filament uses an interface on your `User` model instead of assuming that the `app_authentication_secret` column exists, you can use any column name you want. You could even use a different model entirely if you want to store the secret in a different table.
</Aside>

Finally, you should activate the app authentication feature in your panel. To do this, use the `multiFactorAuthentication()` method in the [configuration](../panel-configuration), and pass a `AppAuthentication` instance to it:

```php
use Filament\Auth\MultiFactor\App\AppAuthentication;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->multiFactorAuthentication([
            AppAuthentication::make(),
        ]);
}
```

### Setting up app recovery codes

If your users lose access to their two-factor authentication app, they will be unable to sign in to your application. To prevent this, you can generate a set of recovery codes that users can use to sign in if they lose access to their two-factor authentication app.

In a similar way to the `app_authentication_secret` column, you should add a new column to your `users` table (or whichever table is being used for your "authenticatable" Eloquent model in this panel). The column needs to store the recovery codes. It can be a normal `text()` column in a migration:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->text('app_authentication_recovery_codes')->nullable();
});
```

In the `User` model, you need to ensure that this column is encrypted as an array and `$hidden`, since this is incredibly sensitive information that should be stored securely:

```php
use Filament\Auth\MultiFactor\App\Contracts\HasAppAuthentication;
use Filament\Models\Contracts\FilamentUser;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements FilamentUser, HasAppAuthentication, MustVerifyEmail
{
    // ...

    /**
     * @var array<string>
     */
    protected $hidden = [
        // ...
        'app_authentication_recovery_codes',
    ];

    /**
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            // ...
            'app_authentication_recovery_codes' => 'encrypted:array',
        ];
    }
    
    // ...
}
```

Next, you should implement the `HasAppAuthenticationRecovery` interface on the `User` model. This provides Filament with the necessary methods to interact with the recovery codes:

```php
use Filament\Auth\MultiFactor\App\Contracts\HasAppAuthentication;
use Filament\Auth\MultiFactor\App\Contracts\HasAppAuthenticationRecovery;
use Filament\Models\Contracts\FilamentUser;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements FilamentUser, HasAppAuthentication, HasAppAuthenticationRecovery, MustVerifyEmail
{
    // ...

    /**
     * @return ?array<string>
     */
    public function getAppAuthenticationRecoveryCodes(): ?array
    {
        // This method should return the user's saved app authentication recovery codes.
    
        return $this->app_authentication_recovery_codes;
    }

    /**
     * @param  array<string> | null  $codes
     */
    public function saveAppAuthenticationRecoveryCodes(?array $codes): void
    {
        // This method should save the user's app authentication recovery codes.
    
        $this->app_authentication_recovery_codes = $codes;
        $this->save();
    }
}
```

<Aside variant="tip">
    Since Filament uses an interface on your `User` model instead of assuming that the `app_authentication_recovery_codes` column exists, you can use any column name you want. You could even use a different model entirely if you want to store the recovery codes in a different table.
</Aside>

Finally, you should activate the app authentication recovery codes feature in your panel. To do this, pass the `recoverable()` method to the `AppAuthentication` instance in the `multiFactorAuthentication()` method in the [configuration](../panel-configuration):

```php
use Filament\Auth\MultiFactor\App\AppAuthentication;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->multiFactorAuthentication([
            AppAuthentication::make()
                ->recoverable(),
        ]);
}
```

#### Changing the number of recovery codes that are generated

By default, Filament generates 8 recovery codes for each user. If you want to change this, you can use the `recoveryCodeCount()` method on the `AppAuthentication` instance in the `multiFactorAuthentication()` method in the [configuration](../panel-configuration):

```php
use Filament\Auth\MultiFactor\App\AppAuthentication;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->multiFactorAuthentication([
            AppAuthentication::make()
                ->recoverable()
                ->recoveryCodeCount(10),
        ]);
}
```

#### Preventing users from regenerating their recovery codes

By default, users can visit their profile to regenerate their recovery codes. If you want to prevent this, you can use the `regenerableRecoveryCodes(false)` method on the `AppAuthentication` instance in the `multiFactorAuthentication()` method in the [configuration](../panel-configuration):

```php
use Filament\Auth\MultiFactor\App\AppAuthentication;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->multiFactorAuthentication([
            AppAuthentication::make()
                ->recoverable()
                ->regenerableRecoveryCodes(false),
        ]);
}
```

### Changing the app code expiration time

App codes are issued using a time-based one-time password (TOTP) algorithm, which means that they are only valid for a short period of time before and after the time they are generated. The time is defined in a "window" of time. By default, Filament uses an expiration window of `8`, which creates a 4-minute validity period on either side of the generation time (8 minutes in total).

To change the window, for example to only be valid for 2 minutes after it is generated, you can use the `codeWindow()` method on the `AppAuthentication` instance, set to `4`:

```php
use Filament\Auth\MultiFactor\App\AppAuthentication;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->multiFactorAuthentication([
            AppAuthentication::make()
                ->codeWindow(4),
        ]);
}
```

### Customizing the app authentication brand name

Each app authentication integration has a "brand name" that is displayed in the authentication app. By default, this is the name of your app. If you want to change this, you can use the `brandName()` method on the `AppAuthentication` instance in the `multiFactorAuthentication()` method in the [configuration](../panel-configuration):

```php
use Filament\Auth\MultiFactor\App\AppAuthentication;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->multiFactorAuthentication([
            AppAuthentication::make()
                ->brandName('Filament Demo'),
        ]);
}
```

## Email authentication

Email authentication sends the user one-time codes to their email address, which they must enter to verify their identity.

To enable email authentication in a panel, you must first add a new column to your `users` table (or whichever table is being used for your "authenticatable" Eloquent model in this panel). The column needs to store a boolean indicating whether or not email authentication is enabled:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->boolean('has_email_authentication')->default(false);
});
```

In the `User` model, you need to ensure that this column is cast to a boolean:

```php
use Filament\Models\Contracts\FilamentUser;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements FilamentUser, MustVerifyEmail
{
    /**
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            // ...
            'has_email_authentication' => 'boolean',
        ];
    }
    
    // ...
}
```

Next, you should implement the `HasEmailAuthentication` interface on the `User` model. This provides Filament with the necessary methods to interact with the column that indicates whether or not email authentication is enabled:

```php
use Filament\Auth\MultiFactor\Email\Contracts\HasEmailAuthentication;
use Filament\Models\Contracts\FilamentUser;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements FilamentUser, HasEmailAuthentication, MustVerifyEmail
{
    // ...

    public function hasEmailAuthentication(): bool
    {
        // This method should return true if the user has enabled email authentication.
        
        return $this->has_email_authentication;
    }

    public function toggleEmailAuthentication(bool $condition): void
    {
        // This method should save whether or not the user has enabled email authentication.
    
        $this->has_email_authentication = $condition;
        $this->save();
    }
}
```

<Aside variant="tip">
    Since Filament uses an interface on your `User` model instead of assuming that the `has_email_authentication` column exists, you can use any column name you want. You could even use a different model entirely if you want to store the setting in a different table.
</Aside>

Finally, you should activate the email authentication feature in your panel. To do this, use the `multiFactorAuthentication()` method in the [configuration](../panel-configuration), and pass an `EmailAuthentication` instance to it:

```php
use Filament\Auth\MultiFactor\Email\EmailAuthentication;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->multiFactorAuthentication([
            EmailAuthentication::make(),
        ]);
}
```

### Changing the email code expiration time

Email codes are issued with an lifetime of 4 minutes, after which they expire.

To change the expiration period, for example to only be valid for 2 minutes after codes are generated, you can use the `codeExpiryMinutes()` method on the `EmailAuthentication` instance, set to `2`:

```php
use Filament\Auth\MultiFactor\Email\EmailAuthentication;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->multiFactorAuthentication([
            EmailAuthentication::make()
                ->codeExpiryMinutes(2),
        ]);
}
```

## Requiring multi-factor authentication

By default, users are not required to set up multi-factor authentication. You can require users to configure it by passing `isRequired: true` as a parameter to the `multiFactorAuthentication()` method in the [configuration](../panel-configuration):

```php
use Filament\Auth\MultiFactor\App\AppAuthentication;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->multiFactorAuthentication([
            AppAuthentication::make(),
        ], isRequired: true);
}
```

When this is enabled, users will be prompted to set up multi-factor authentication after they sign in, if they have not already done so.

## Security notes about multi-factor authentication

In Filament, the multi-factor authentication process occurs before the user is actually authenticated into the app. This allows you to be sure that no users can authenticate and access the app without passing the multi-factor authentication step. You do not need to remember to add middleware to any of your authenticated routes to ensure that users completed the multi-factor authentication step.

However, if you have other parts of your Laravel app that authenticate users, please bear in mind that they will not be challenged for multi-factor authentication if they are already authenticated elsewhere and then visit the panel, unless [multi-factor authentication is required](#requiring-multi-factor-authentication) and they have not set it up yet.

---

<!-- Source: 02-notifications.md -->

import Aside from "@components/Aside.astro"

<Aside variant="warning">
    Before proceeding, make sure `filament/notifications` is installed in your project. You can check by running:

    ```bash
    composer show filament/notifications
    ```
    If it's not installed, consult the [installation guide](../introduction/installation#installing-the-individual-components) and configure the **individual components** according to the instructions.
</Aside>
## Introduction

To render notifications in your app, make sure the `notifications` Livewire component is rendered in your layout:

```blade
<div>
    @livewire('notifications')
</div>
```

Now, when [sending a notification](../notifications) from a Livewire request, it will appear for the user.

---

<!-- Source: 02-panel-plugins.md -->

## Introduction

The basis of Filament plugins are Laravel packages. They are installed into your Filament project via Composer, and follow all the standard techniques, like using service providers to register routes, views, and translations. If you're new to Laravel package development, here are some resources that can help you grasp the core concepts:

- [The Package Development section of the Laravel docs](https://laravel.com/docs/packages) serves as a great reference guide.
- [Spatie's Package Training course](https://spatie.be/products/laravel-package-training) is a good instructional video series to teach you the process step by step.
- [Spatie's Package Tools](https://github.com/spatie/laravel-package-tools) allows you to simplify your service provider classes using a fluent configuration object.

Filament plugins build on top of the concepts of Laravel packages and allow you to ship and consume reusable features for any Filament panel. They can be added to each panel one at a time, and are also configurable differently per-panel.

## Configuring the panel with a plugin class

A plugin class is used to allow your package to interact with a panel [configuration](../panel-configuration) file. It's a simple PHP class that implements the `Plugin` interface. 3 methods are required:

- The `getId()` method returns the unique identifier of the plugin amongst other plugins. Please ensure that it is specific enough to not clash with other plugins that might be used in the same project.
- The `register()` method allows you to use any [configuration](../panel-configuration) option that is available to the panel. This includes registering [resources](resources), [custom pages](../navigation/custom-pages), [themes](themes), [render hooks](../panel-configuration#render-hooks) and more.
- The `boot()` method is run only when the panel that the plugin is being registered to is actually in-use. It is executed by a middleware class.

```php
<?php

namespace DanHarrin\FilamentBlog;

use DanHarrin\FilamentBlog\Pages\Settings;
use DanHarrin\FilamentBlog\Resources\CategoryResource;
use DanHarrin\FilamentBlog\Resources\PostResource;
use Filament\Contracts\Plugin;
use Filament\Panel;

class BlogPlugin implements Plugin
{
    public function getId(): string
    {
        return 'blog';
    }

    public function register(Panel $panel): void
    {
        $panel
            ->resources([
                PostResource::class,
                CategoryResource::class,
            ])
            ->pages([
                Settings::class,
            ]);
    }

    public function boot(Panel $panel): void
    {
        //
    }
}
```

The users of your plugin can add it to a panel by instantiating the plugin class and passing it to the `plugin()` method of the [configuration](../panel-configuration):

```php
use DanHarrin\FilamentBlog\BlogPlugin;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->plugin(new BlogPlugin());
}
```

### Fluently instantiating the plugin class

You may want to add a `make()` method to your plugin class to provide a fluent interface for your users to instantiate it. In addition, by using the container (`app()`) to instantiate the plugin object, it can be replaced with a different implementation at runtime:

```php
use Filament\Contracts\Plugin;

class BlogPlugin implements Plugin
{
    public static function make(): static
    {
        return app(static::class);
    }
    
    // ...
}
```

Now, your users can use the `make()` method:

```php
use DanHarrin\FilamentBlog\BlogPlugin;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->plugin(BlogPlugin::make());
}
```

### Configuring plugins per-panel

You may add other methods to your plugin class, which allow your users to configure it. We suggest that you add both a setter and a getter method for each option you provide. You should use a property to store the preference in the setter and retrieve it again in the getter:

```php
use DanHarrin\FilamentBlog\Resources\AuthorResource;
use Filament\Contracts\Plugin;
use Filament\Panel;

class BlogPlugin implements Plugin
{
    protected bool $hasAuthorResource = false;
    
    public function authorResource(bool $condition = true): static
    {
        // This is the setter method, where the user's preference is
        // stored in a property on the plugin object.
        $this->hasAuthorResource = $condition;
    
        // The plugin object is returned from the setter method to
        // allow fluent chaining of configuration options.
        return $this;
    }
    
    public function hasAuthorResource(): bool
    {
        // This is the getter method, where the user's preference
        // is retrieved from the plugin property.
        return $this->hasAuthorResource;
    }
    
    public function register(Panel $panel): void
    {
        // Since the `register()` method is executed after the user
        // configures the plugin, you can access any of their
        // preferences inside it.
        if ($this->hasAuthorResource()) {
            // Here, we only register the author resource on the
            // panel if the user has requested it.
            $panel->resources([
                AuthorResource::class,
            ]);
        }
    }
    
    // ...
}
```

Additionally, you can use the unique ID of the plugin to access any of its configuration options from outside the plugin class. To do this, pass the ID to the `filament()` method:

```php
filament('blog')->hasAuthorResource()
```

You may wish to have better type safety and IDE autocompletion when accessing configuration. It's completely up to you how you choose to achieve this, but one idea could be adding a static method to the plugin class to retrieve it:

```php
use Filament\Contracts\Plugin;

class BlogPlugin implements Plugin
{
    public static function get(): static
    {
        return filament(app(static::class)->getId());
    }
    
    // ...
}
```

Now, you can access the plugin configuration using the new static method:

```php
BlogPlugin::get()->hasAuthorResource()
```

## Distributing a panel in a plugin

It's very easy to distribute an entire panel in a Laravel package. This way, a user can simply install your plugin and have an entirely new part of their app pre-built.

When [configuring](../panel-configuration) a panel, the configuration class extends the `PanelProvider` class, and that is a standard Laravel service provider. You can use it as a service provider in your package:

```php
<?php

namespace DanHarrin\FilamentBlog;

use Filament\Panel;
use Filament\PanelProvider;

class BlogPanelProvider extends PanelProvider
{
    public function panel(Panel $panel): Panel
    {
        return $panel
            ->id('blog')
            ->path('blog')
            ->resources([
                // ...
            ])
            ->pages([
                // ...
            ])
            ->widgets([
                // ...
            ])
            ->middleware([
                // ...
            ])
            ->authMiddleware([
                // ...
            ]);
    }
}
```

You should then register it as a service provider in the `composer.json` of your package:

```json
"extra": {
    "laravel": {
        "providers": [
            "DanHarrin\\FilamentBlog\\BlogPanelProvider"
        ]
    }
}
```

---

<!-- Source: 02-schema.md -->

import Aside from "@components/Aside.astro"

<Aside variant="warning">
    Before proceeding, make sure `filament/schemas` is installed in your project. You can check by running:

    ```bash
    composer show filament/schemas
    ```
    If it's not installed, consult the [installation guide](../introduction/installation#installing-the-individual-components) and configure the **individual components** according to the instructions.
</Aside>

## Setting up the Livewire component

First, generate a new Livewire component:

```bash
php artisan make:livewire ViewProduct
```

Then, render your Livewire component on the page:

```blade
@livewire('view-product')
```

Alternatively, you can use a full-page Livewire component:

```php
use App\Livewire\ViewProduct;
use Illuminate\Support\Facades\Route;

Route::get('products/{product}', ViewProduct::class);
```

You must use the `InteractsWithSchemas` trait, and implement the `HasSchemas` interface on your Livewire component class:

```php
use Filament\Schemas\Concerns\InteractsWithSchemas;
use Filament\Schemas\Contracts\HasSchemas;
use Livewire\Component;

class ViewProduct extends Component implements HasSchemas
{
    use InteractsWithSchemas;

    // ...
}
```

## Adding the schema

Next, add a method to the Livewire component which accepts a `$schema` object, modifies it, and returns it:

```php
use Filament\Schemas\Schema;

public function productSchema(Schema $schema): Schema
{
    return $schema
        ->components([
            // ...
        ]);
}
```

Finally, render the schema in the Livewire component's view:

```blade
{{ $this->productSchema }}
```

<Aside variant="info">
    `filament/schemas` also includes the following packages:

    - `filament/actions`
    - `filament/support`
    
    These packages allow you to use their components within Livewire components.
    For example, if your schema uses [Actions](../actions), remember to implement the `HasActions` interface and use the `InteractsWithActions` trait on your Livewire component class.
    
    If you are using any other [Filament components](overview#package-components) in your schema, make sure to install and integrate the corresponding package as well.
</Aside>

---

<!-- Source: 02-table.md -->

import Aside from "@components/Aside.astro"

<Aside variant="warning">
    Before proceeding, make sure `filament/tables` is installed in your project. You can check by running:

    ```bash
    composer show filament/tables
    ```
    If it's not installed, consult the [installation guide](../introduction/installation#installing-the-individual-components) and configure the **individual components** according to the instructions.
</Aside>

## Setting up the Livewire component

First, generate a new Livewire component:

```bash
php artisan make:livewire ListProducts
```

Then, render your Livewire component on the page:

```blade
@livewire('list-products')
```

Alternatively, you can use a full-page Livewire component:

```php
use App\Livewire\ListProducts;
use Illuminate\Support\Facades\Route;

Route::get('products', ListProducts::class);
```

## Adding the table

There are 3 tasks when adding a table to a Livewire component class:

1) Implement the `HasTable` and `HasSchemas` interfaces, and use the `InteractsWithTable` and `InteractsWithSchemas` traits.
2) Add a `table()` method, which is where you configure the table. [Add the table's columns, filters, and actions](getting-started#columns).
3) Make sure to define the base query that will be used to fetch rows in the table. For example, if you're listing products from your `Product` model, you will want to return `Product::query()`.

```php
<?php

namespace App\Livewire;

use App\Models\Shop\Product;
use Filament\Actions\Concerns\InteractsWithActions;  
use Filament\Actions\Contracts\HasActions;
use Filament\Schemas\Concerns\InteractsWithSchemas;
use Filament\Schemas\Contracts\HasSchemas;
use Filament\Tables\Columns\TextColumn;
use Filament\Tables\Concerns\InteractsWithTable;
use Filament\Tables\Contracts\HasTable;
use Filament\Tables\Table;
use Illuminate\Contracts\View\View;
use Livewire\Component;

class ListProducts extends Component implements HasActions, HasSchemas, HasTable
{
    use InteractsWithActions;
    use InteractsWithSchemas;
    use InteractsWithTable;
    
    public function table(Table $table): Table
    {
        return $table
            ->query(Product::query())
            ->columns([
                TextColumn::make('name'),
            ])
            ->filters([
                // ...
            ])
            ->recordActions([
                // ...
            ])
            ->toolbarActions([
                // ...
            ]);
    }
    
    public function render(): View
    {
        return view('livewire.list-products');
    }
}
```

Finally, in your Livewire component's view, render the table:

```blade
<div>
    {{ $this->table }}
</div>
```

Visit your Livewire component in the browser, and you should see the table.

<Aside variant="info">

    `filament/tables` also includes the following packages:
    
    - `filament/actions`
    - `filament/forms`
    - `filament/support`
    
    These packages allow you to use their components within Livewire components.
    For example, if your table uses [Actions](action#setting-up-the-livewire-component), remember to implement the `HasActions` interface and include the `InteractsWithActions` trait.
    
    If you are using any other [Filament components](overview#package-components) in your table, make sure to install and integrate the corresponding package as well.
</Aside>

## Building a table for an Eloquent relationship

If you want to build a table for an Eloquent relationship, you can use the `relationship()` and `inverseRelationship()` methods on the `$table` instead of passing a `query()`. `HasMany`, `HasManyThrough`, `BelongsToMany`, `MorphMany` and `MorphToMany` relationships are compatible:

```php
use App\Models\Category;
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

public Category $category;

public function table(Table $table): Table
{
    return $table
        ->relationship(fn (): BelongsToMany => $this->category->products())
        ->inverseRelationship('categories')
        ->columns([
            TextColumn::make('name'),
        ]);
}
```

In this example, we have a `$category` property which holds a `Category` model instance. The category has a relationship named `products`. We use a function to return the relationship instance. This is a many-to-many relationship, so the inverse relationship is called `categories`, and is defined on the `Product` model. We just need to pass the name of this relationship to the `inverseRelationship()` method, not the whole instance.

Now that the table is using a relationship instead of a plain Eloquent query, all actions will be performed on the relationship instead of the query. For example, if you use a [`CreateAction`](../actions/create), the new product will be automatically attached to the category.

If your relationship uses a pivot table, you can use all pivot columns as if they were normal columns on your table, as long as they are listed in the `withPivot()` method of the relationship *and* inverse relationship definition.

Relationship tables are used in the Panel Builder as ["relation managers"](../panels/resources/managing-relationships#creating-a-relation-manager). Most of the documented features for relation managers are also available for relationship tables. For instance, [attaching and detaching](../panels/resources/managing-relationships#attaching-and-detaching-records) and [associating and dissociating](../panels/resources/relation-managers#associating-and-dissociating-records) actions.

## Generating table Livewire components with the CLI

It's advised that you learn how to set up a Livewire component with the Table Builder manually, but once you are confident, you can use the CLI to generate a table for you.

```bash
php artisan make:livewire-table Products/ListProducts
```

This will ask you for the name of a prebuilt model, for example `Product`. Finally, it will generate a new `app/Livewire/Products/ListProducts.php` component, which you can customize.

### Automatically generating table columns

Filament is also able to guess which table columns you want in the table, based on the model's database columns. You can use the `--generate` flag when generating your table:

```bash
php artisan make:livewire-table Products/ListProducts --generate
```

---

<!-- Source: 02-testing-resources.md -->

## Authenticating as a user

Ensure that you are authenticated to access the app in your `TestCase`:

```php
use App\Models\User;

protected function setUp(): void
{
    parent::setUp();

    $this->actingAs(User::factory()->create());
}
```

Alternatively, if you are using Pest you can use a `beforeEach()` function at the top of your test file to authenticate:

```php
use App\Models\User;

beforeEach(function () {
    $user = User::factory()->create();

    actingAs($user);
});
```

## Testing a resource list page

To test if the list page is able to load, test the list page as a Livewire component, and call `assertOk()` to ensure that the HTTP response was 200 OK. You can also use the `assertCanSeeTableRecords()` method to check if records are being displayed in the table:

```php
use App\Filament\Resources\Users\Pages\ListUsers;
use App\Models\User;

it('can load the page', function () {
    $users = User::factory()->count(5)->create();

    livewire(ListUsers::class)
        ->assertOk()
        ->assertCanSeeTableRecords($users);
});
```

To test the table on the list page, you should visit the [Testing tables](testing-tables) section. To test any actions in the header of the page or actions in the table, you should visit the [Testing actions](testing-actions) section. Below are some common examples of other tests that you can run on the list page.

To [test that the table search is working](testing-tables#testing-that-a-column-can-be-searched), you can use the `searchTable()` method to search for a specific record. You can also use the `assertCanSeeTableRecords()` and `assertCanNotSeeTableRecords()` methods to check if the correct records are being displayed in the table:

```php
use App\Filament\Resources\Users\Pages\ListUsers;
use App\Models\User;

it('can search users by `name` or `email`', function () {
    $users = User::factory()->count(5)->create();

    livewire(ListUsers::class)
        ->assertCanSeeTableRecords($users)
        ->searchTable($users->first()->name)
        ->assertCanSeeTableRecords($users->take(1))
        ->assertCanNotSeeTableRecords($users->skip(1))
        ->searchTable($users->last()->email)
        ->assertCanSeeTableRecords($users->take(-1))
        ->assertCanNotSeeTableRecords($users->take($users->count() - 1));
});
```

To [test that the table sorting is working](testing-tables#testing-that-a-column-can-be-sorted), you can use the `sortTable()` method to sort the table by a specific column. You can also use the `assertCanSeeTableRecords()` method to check if the records are being displayed in the correct order:

```php
use App\Filament\Resources\Users\Pages\ListUsers;
use App\Models\User;

it('can sort users by `name`', function () {
    $users = User::factory()->count(5)->create();

    livewire(ListUsers::class)
        ->assertCanSeeTableRecords($users)
        ->sortTable('name')
        ->assertCanSeeTableRecords($users->sortBy('name'), inOrder: true)
        ->sortTable('name', 'desc')
        ->assertCanSeeTableRecords($users->sortByDesc('name'), inOrder: true);
});
```

To [test that the table filtering is working](testing-tables#testing-filters), you can use the `filterTable()` method to filter the table by a specific column. You can also use the `assertCanSeeTableRecords()` and `assertCanNotSeeTableRecords()` methods to check if the correct records are being displayed in the table:

```php
use App\Filament\Resources\Users\Pages\ListUsers;
use App\Models\User;

it('can filter users by `locale`', function () {
    $users = User::factory()->count(5)->create();

    livewire(ListUsers::class)
        ->assertCanSeeTableRecords($users)
        ->filterTable('locale', $users->first()->locale)
        ->assertCanSeeTableRecords($users->where('locale', $users->first()->locale))
        ->assertCanNotSeeTableRecords($users->where('locale', '!=', $users->first()->locale));
});
```

To [test that the table bulk actions are working](testing-actions#testing-table-bulk-actions), you can use the `selectTableRecords()` method to select multiple records in the table. You can also use the `callAction()` method to call a specific action on the selected records:

```php
use App\Filament\Resources\Users\Pages\ListUsers;
use App\Models\User;
use Filament\Actions\Testing\TestAction;
use function Pest\Laravel\assertDatabaseMissing;

it('can bulk delete users', function () {
    $users = User::factory()->count(5)->create();

    livewire(ListUsers::class)
        ->assertCanSeeTableRecords($users)
        ->selectTableRecords($users)
        ->callAction(TestAction::make(DeleteBulkAction::class)->table()->bulk())
        ->assertNotified()
        ->assertCanNotSeeTableRecords($users);

    $users->each(fn (User $user) => assertDatabaseMissing($user));
});
```

## Testing a resource create page

To test if the create page is able to load, test the create page as a Livewire component, and call `assertOk()` to ensure that the HTTP response was 200 OK:

```php
use App\Filament\Resources\Users\Pages\CreateUser;
use App\Models\User;

it('can load the page', function () {
    livewire(CreateUser::class)
        ->assertOk();
});
```

To test the form on the create page, you should visit the [Testing schemas](testing-schemas) section. To test any actions in the header of the page or in the form, you should visit the [Testing actions](testing-actions) section. Below are some common examples of other tests that you can run on the create page.

To test that the form is creating records correctly, you can use the `fillForm()` method to fill in the form fields, and then use the `call('create')` method to create the record. You can also use the `assertNotified()` method to check if a notification was displayed, and the `assertRedirect()` method to check if the user was redirected to another page:

```php
use App\Filament\Resources\Users\Pages\CreateUser;
use App\Models\User;
use function Pest\Laravel\assertDatabaseHas;

it('can create a user', function () {
    $newUserData = User::factory()->make();

    livewire(CreateUser::class)
        ->fillForm([
            'name' => $newUserData->name,
            'email' => $newUserData->email,
        ])
        ->call('create')
        ->assertNotified()
        ->assertRedirect();

    assertDatabaseHas(User::class, [
        'name' => $newUserData->name,
        'email' => $newUserData->email,
    ]);
});
```

To test that the form is validating properly, you can use the `fillForm()` method to fill in the form fields, and then use the `call('create')` method to create the record. You can also use the `assertHasFormErrors()` method to check if the form has any errors, and the `assertNotNotified()` method to check if no notification was displayed. You can also use the `assertNoRedirect()` method to check if the user was not redirected to another page. In this example, we use a [Pest dataset](https://pestphp.com/docs/datasets#content-bound-datasets) to test multiple rules without having to repeat the test code:

```php
use App\Filament\Resources\Users\Pages\CreateUser;
use App\Models\User;
use Illuminate\Support\Str;

it('validates the form data', function (array $data, array $errors) {
    $newUserData = User::factory()->make();

    livewire(CreateUser::class)
        ->fillForm([
            'name' => $newUserData->name,
            'email' => $newUserData->email,
            ...$data,
        ])
        ->call('create')
        ->assertHasFormErrors($errors)
        ->assertNotNotified()
        ->assertNoRedirect();
})->with([
    '`name` is required' => [['name' => null], ['name' => 'required']],
    '`name` is max 255 characters' => [['name' => Str::random(256)], ['name' => 'max']],
    '`email` is a valid email address' => [['email' => Str::random()], ['email' => 'email']],
    '`email` is required' => [['email' => null], ['email' => 'required']],
    '`email` is max 255 characters' => [['email' => Str::random(256)], ['email' => 'max']],
]);
```

## Testing a resource edit page

To test if the edit page is able to load, test the edit page as a Livewire component, and call `assertOk()` to ensure that the HTTP response was 200 OK. You can also use the `assertSchemaStateSet()` method to check if the form fields are set to the correct values:

```php
use App\Filament\Resources\Users\Pages\EditUser;
use App\Models\User;

it('can load the page', function () {
    $user = User::factory()->create();

    livewire(EditUser::class, [
        'record' => $user->id,
    ])
        ->assertOk()
        ->assertSchemaStateSet([
            'name' => $user->name,
            'email' => $user->email,
        ]);
});
```

To test the form on the edit page, you should visit the [Testing schemas](testing-schemas) section. To test any actions in the header of the page or in the form, you should visit the [Testing actions](testing-actions) section. Below are some common examples of other tests that you can run on the edit page.

```php
use App\Filament\Resources\Users\Pages\EditUser;
use App\Models\User;
use function Pest\Laravel\assertDatabaseHas;

it('can update a user', function () {
    $user = User::factory()->create();

    $newUserData = User::factory()->make();

    livewire(EditUser::class, [
        'record' => $user->id,
    ])
        ->fillForm([
            'name' => $newUserData->name,
            'email' => $newUserData->email,
        ])
        ->call('save')
        ->assertNotified();

    assertDatabaseHas(User::class, [
        'id' => $user->id,
        'name' => $newUserData->name,
        'email' => $newUserData->email,
    ]);
});
```

To test that the form is validating properly, you can use the `fillForm()` method to fill in the form fields, and then use the `call('save')` method to save the record. You can also use the `assertHasFormErrors()` method to check if the form has any errors, and the `assertNotNotified()` method to check if no notification was displayed. In this example, we use a [Pest dataset](https://pestphp.com/docs/datasets#content-bound-datasets) to test multiple rules without having to repeat the test code:

```php
use App\Filament\Resources\Users\Pages\EditUser;
use App\Models\User;
use Illuminate\Support\Str;

it('validates the form data', function (array $data, array $errors) {
    $user = User::factory()->create();

    $newUserData = User::factory()->make();

    livewire(EditUser::class, [
        'record' => $user->id,
    ])
        ->fillForm([
            'name' => $newUserData->name,
            'email' => $newUserData->email,
            ...$data,
        ])
        ->call('save')
        ->assertHasFormErrors($errors)
        ->assertNotNotified();
})->with([
    '`name` is required' => [['name' => null], ['name' => 'required']],
    '`name` is max 255 characters' => [['name' => Str::random(256)], ['name' => 'max']],
    '`email` is a valid email address' => [['email' => Str::random()], ['email' => 'email']],
    '`email` is required' => [['email' => null], ['email' => 'required']],
    '`email` is max 255 characters' => [['email' => Str::random(256)], ['email' => 'max']],
]);
```

To [test that an action is working](testing-actions), such as the `DeleteAction`, you can use the `callAction()` method to call the delete action. You can also use the `assertNotified()` method to check if a notification was displayed, and the `assertRedirect()` method to check if the user was redirected to another page:

```php
use App\Filament\Resources\Users\Pages\EditUser;
use App\Models\User;
use Filament\Actions\DeleteAction;
use function Pest\Laravel\assertDatabaseMissing;

it('can delete a user', function () {
    $user = User::factory()->create();

    livewire(EditUser::class, [
        'record' => $user->id,
    ])
        ->callAction(DeleteAction::class)
        ->assertNotified()
        ->assertRedirect();

    assertDatabaseMissing($user);
});
```

## Testing a resource view page

To test if the view page is able to load, test the view page as a Livewire component, and call `assertOk()` to ensure that the HTTP response was 200 OK. You can also use the `assertSchemaStateSet()` method to check if the infolist entries are set to the correct values:

```php
use App\Filament\Resources\Users\Pages\ViewUser;
use App\Models\User;

it('can load the page', function () {
    $user = User::factory()->create();

    livewire(ViewUser::class, [
        'record' => $user->id,
    ])
        ->assertOk()
        ->assertSchemaStateSet([
            'name' => $user->name,
            'email' => $user->email,
        ]);
});
```

To test the infolist on the view page, you should visit the [Testing schemas](testing-schemas) section. To test any actions in the header of the page or in the infolist, you should visit the [Testing actions](testing-actions) section.

## Testing relation managers

To test if a relation manager is rendered on a page, such as the edit page of a resource, you can use the `assertSeeLivewire()` method to check if the relation manager is being rendered:

```php
use App\Filament\Resources\Users\Pages\EditUser;
use App\Filament\Resources\Users\RelationManagers\PostsRelationManager;
use App\Models\User;

it('can load the relation manager', function () {
    $user = User::factory()->create();

    livewire(EditUser::class, [
        'record' => $user->id,
    ])
        ->assertSeeLivewire(PostsRelationManager::class);
});
```

Since relation managers are Livewire components, you can also test a relation manager's functionality itself, like its ability to load successfully with a 200 OK response, with the correct records in the table. When testing a relation manager, you need to pass in the `ownerRecord`, which is the record from the resource you are inside, and the `pageClass`, which is the class of the page you are on:

```php
use App\Filament\Resources\Users\Pages\EditUser;
use App\Filament\Resources\Users\RelationManagers\PostsRelationManager;
use App\Models\Post;
use App\Models\User;

it('can load the relation manager', function () {
    $user = User::factory()
        ->has(Post::factory()->count(5))
        ->create();

    livewire(PostsRelationManager::class, [
        'ownerRecord' => $user,
        'pageClass' => EditUser::class,
    ])
        ->assertOk()
        ->assertCanSeeTableRecords($user->posts);
});
```

You can [test searching](testing-tables#testing-that-a-column-can-be-searched), [sorting](testing-tables#testing-that-a-column-can-be-sorted), and [filtering](testing-tables#testing-filters) in the same way as you would on a resource list page.

You can also [test actions](testing-actions), for example, the `CreateAction` in the header of the table:

```php
use App\Filament\Resources\Users\Pages\EditUser;
use App\Filament\Resources\Users\RelationManagers\PostsRelationManager;
use App\Models\Post;
use App\Models\User;
use Filament\Actions\Testing\TestAction;
use function Pest\Laravel\assertDatabaseHas;

it('can create a post', function () {
    $user = User::factory()->create();

    $newPostData = Post::factory()->make();

    livewire(PostsRelationManager::class, [
        'ownerRecord' => $user,
        'pageClass' => EditUser::class,
    ])
        ->callAction(TestAction::make(CreateAction::class)->table(), [
            'title' => $newPostData->title,
            'content' => $newPostData->content,
        ])
        ->assertNotified();

    assertDatabaseHas(Post::class, [
        'title' => $newPostData->title,
        'content' => $newPostData->content,
        'user_id' => $user->id,
    ]);
});
```

## Testing multiple panels

If you have multiple panels and you would like to test a non-default panel, you will need to tell Filament which panel you are testing. This can be done in the `setUp()` method of the test case, or you can do it at the start of a particular test. Filament usually does this in a middleware when you access the panel through a request, so if you're not making a request in your test like when testing a Livewire component, you need to set the current panel manually:

```php
use Filament\Facades\Filament;

Filament::setCurrentPanel('app'); // Where `app` is the ID of the panel you want to test.
```

---

<!-- Source: 02-widget.md -->

import Aside from "@components/Aside.astro"

<Aside variant="warning">
    Before proceeding, make sure `filament/widgets` is installed in your project. You can check by running:

    ```bash
    composer show filament/widgets
    ```
    If it's not installed, consult the [installation guide](../introduction/installation#installing-the-individual-components) and configure the **individual components** according to the instructions.
</Aside>

## Creating a widget

Use the `make:filament-widget` command to generate a new widget. For details on customization and usage, see the [widgets section](../widgets).

## Adding the widget

Since widgets are Livewire components, you can easily render a widget in any Blade view using the `@livewire` directive:

```blade
<div>
    @livewire(\App\Livewire\Dashboard\PostsChart::class)
</div>
```

<Aside variant="info">
    If you're using a [table widget](../widgets/overview#table-widgets), make sure to install `filament/tables` as well.  
    Refer to the [installation guide](../introduction/installation#installing-the-individual-components) and follow the steps to configure the **individual components** properly.
</Aside>

---

<!-- Source: 03-avatar.md -->

## Introduction

The avatar component is used to render a circular or square image, often used to represent a user or entity as their "profile picture":

```blade
<x-filament::avatar
    src="https://filamentphp.com/dan.jpg"
    alt="Dan Harrin"
/>
```

## Setting the rounding of an avatar

Avatars are fully rounded by default, but you may make them square by setting the `circular` attribute to `false`:

```blade
<x-filament::avatar
    src="https://filamentphp.com/dan.jpg"
    alt="Dan Harrin"
    :circular="false"
/>
```

## Setting the size of an avatar

By default, the avatar will be "medium" size. You can set the size to either `sm`, `md`, or `lg` using the `size` attribute:

```blade
<x-filament::avatar
    src="https://filamentphp.com/dan.jpg"
    alt="Dan Harrin"
    size="lg"
/>
```

You can also pass your own custom size classes into the `size` attribute:

```blade
<x-filament::avatar
    src="https://filamentphp.com/dan.jpg"
    alt="Dan Harrin"
    size="w-12 h-12"
/>

---

<!-- Source: 03-badge.md -->

## Introduction

The badge component is used to render a small box with some text inside:

```blade
<x-filament::badge>
    New
</x-filament::badge>
```

## Setting the size of a badge

By default, the size of a badge is "medium". You can make it "extra small" or "small" by using the `size` attribute:

```blade
<x-filament::badge size="xs">
    New
</x-filament::badge>

<x-filament::badge size="sm">
    New
</x-filament::badge>
```

## Changing the color of the badge

By default, the color of a badge is "primary". You can change it to be `danger`, `gray`, `info`, `success` or `warning` by using the `color` attribute:

```blade
<x-filament::badge color="danger">
    New
</x-filament::badge>

<x-filament::badge color="gray">
    New
</x-filament::badge>

<x-filament::badge color="info">
    New
</x-filament::badge>

<x-filament::badge color="success">
    New
</x-filament::badge>

<x-filament::badge color="warning">
    New
</x-filament::badge>
```

## Adding an icon to a badge

You can add an [icon](../styling/icons) to a badge by using the `icon` attribute:

```blade
<x-filament::badge icon="heroicon-m-sparkles">
    New
</x-filament::badge>
```

You can also change the icon's position to be after the text instead of before it, using the `icon-position` attribute:

```blade
<x-filament::badge
    icon="heroicon-m-sparkles"
    icon-position="after"
>
    New
</x-filament::badge>
```

---

<!-- Source: 03-breadcrumbs.md -->

## Introduction

The breadcrumbs component is used to render a simple, linear navigation that informs the user of their current location within the application:

```blade
<x-filament::breadcrumbs :breadcrumbs="[
    '/' => 'Home',
    '/dashboard' => 'Dashboard',
    '/dashboard/users' => 'Users',
    '/dashboard/users/create' => 'Create User',
]" />
```

The keys of the array are URLs that the user is able to click on to navigate, and the values are the text that will be displayed for each link.

---

<!-- Source: 03-building-a-panel-plugin.md -->

import Aside from "@components/Aside.astro"

## Preface

Please read the docs on [panel plugin development](../panels/plugins) and the [getting started guide](getting-started) before continuing.

## Introduction

In this walkthrough, we'll build a simple plugin that adds a new form field that can be used in forms. This also means it will be available to users in their panels.

You can find the final code for this plugin at [https://github.com/awcodes/clock-widget](https://github.com/awcodes/clock-widget).

## Step 1: Create the plugin

First, we'll create the plugin using the steps outlined in the [getting started guide](getting-started#creating-a-plugin).

## Step 2: Clean up

Next, we'll clean up the plugin to remove the boilerplate code we don't need. This will seem like a lot, but since this is a simple plugin, we can remove a lot of the boilerplate code.

Remove the following directories and files:
1. `config`
1. `database`
1. `src/Commands`
1. `src/Facades`
1. `stubs`

Since our plugin doesn't have any settings or additional methods needed for functionality, we can also remove the `ClockWidgetPlugin.php` file.

1. `ClockWidgetPlugin.php`

Since Filament recommends that users style their plugins with a custom filament theme, we'll remove the files needed for using CSS in the plugin. This is optional, and you can still use CSS if you want, but it is not recommended.

1. `resources/css`
2. `postcss.config.js`

Now we can clean up our `composer.json` file to remove unneeded options.

```json
"autoload": {
    "psr-4": {
        // We can remove the database factories
        "Awcodes\\ClockWidget\\Database\\Factories\\": "database/factories/"
    }
},
"extra": {
    "laravel": {
        // We can remove the facade
        "aliases": {
            "ClockWidget": "Awcodes\\ClockWidget\\Facades\\ClockWidget"
        }
    }
},
```

The last step is to update the `package.json` file to remove unneeded options. Replace the contents of `package.json` with the following.

```json
{
    "private": true,
    "type": "module",
    "scripts": {
        "dev": "node bin/build.js --dev",
        "build": "node bin/build.js"
    },
    "devDependencies": {
        "esbuild": "^0.17.19"
    }
}
```

Then we need to install our dependencies.

```bash
npm install
```

You may also remove the Testing directories and files, but we'll leave them in for now, although we won't be using them for this example, and we highly recommend that you write tests for your plugins.

## Step 3: Setting up the provider

Now that we have our plugin cleaned up, we can start adding our code. The boilerplate in the `src/ClockWidgetServiceProvider.php` file has a lot going on so, let's delete everything and start from scratch.

<Aside variant="info">
    In this example, we will be registering an [async Alpine component](../assets#asynchronous-alpinejs-components). Since these assets are only loaded on request, we can register them as normal in the `packageBooted()` method. If you are registering assets, like CSS or JS files, that get loaded on every page regardless of if they are used or not, you should register them in the `register()` method of the `Plugin` configuration object, using [`$panel->assets()`](../panel-configuration#registering-assets-for-a-panel). Otherwise, if you register them in the `packageBooted()` method, they will be loaded in every panel, regardless of whether or not the plugin has been registered for that panel.
</Aside>

We need to be able to register our Widget with the panel and load our Alpine component when the widget is used. To do this, we'll need to add the following to the `packageBooted` method in our service provider. This will register our widget component with Livewire and our Alpine component with the Filament Asset Manager.

```php
use Filament\Support\Assets\AlpineComponent;
use Filament\Support\Facades\FilamentAsset;
use Livewire\Livewire;
use Spatie\LaravelPackageTools\Package;
use Spatie\LaravelPackageTools\PackageServiceProvider;

class ClockWidgetServiceProvider extends PackageServiceProvider
{
    public static string $name = 'clock-widget';

    public function configurePackage(Package $package): void
    {
        $package->name(static::$name)
            ->hasViews()
            ->hasTranslations();
    }

    public function packageBooted(): void
    {
        Livewire::component('clock-widget', ClockWidget::class);

        // Asset Registration
        FilamentAsset::register(
            assets:[
                 AlpineComponent::make('clock-widget', __DIR__ . '/../resources/dist/clock-widget.js'),
            ],
            package: 'awcodes/clock-widget'
        );
    }
}
```

## Step 4: Create the widget

Now we can create our widget. We'll first need to extend Filament's `Widget` class in our `ClockWidget.php` file and tell it where to find the view for the widget. Since we are using the PackageServiceProvider to register our views, we can use the `::` syntax to tell Filament where to find the view.

```php
use Filament\Widgets\Widget;

class ClockWidget extends Widget
{
    protected static string $view = 'clock-widget::widget';
}
```

Next, we'll need to create the view for our widget. Create a new file at `resources/views/widget.blade.php` and add the following code. We'll make use of Filament's blade components to save time on writing the html for the widget.

We are using async Alpine to load our Alpine component, so we'll need to add the `x-load` attribute to the div to tell Alpine to load our component. You can learn more about this in the [Core Concepts](../advanced/assets#asynchronous-alpinejs-components) section of the docs.

```blade
<x-filament-widgets::widget>
    <x-filament::section>
        <x-slot name="heading">
            {{ __('clock-widget::clock-widget.title') }}
        </x-slot>

        <div
            x-load
            x-load-src="{{ \Filament\Support\Facades\FilamentAsset::getAlpineComponentSrc('clock-widget', 'awcodes/clock-widget') }}"
            x-data="clockWidget()"
            class="text-center"
        >
            <p>{{ __('clock-widget::clock-widget.description') }}</p>
            <p class="text-xl" x-text="time"></p>
        </div>
    </x-filament::section>
</x-filament-widgets::widget>
```

Next, we need to write our Alpine component in `src/js/index.js`. And build our assets with `npm run build`.

```js
export default function clockWidget() {
    return {
        time: new Date().toLocaleTimeString(),
        init() {
            setInterval(() => {
                this.time = new Date().toLocaleTimeString();
            }, 1000);
        }
    }
}
```

We should also add translations for the text in the widget so users can translate the widget into their language. We'll add the translations to `resources/lang/en/widget.php`.

```php
return [
    'title' => 'Clock Widget',
    'description' => 'Your current time is:',
];
```

## Step 5: Update your README

You'll want to update your `README.md` file to include instructions on how to install your plugin and any other information you want to share with users, Like how to use it in their projects. For example:

```php
// Register the plugin and/or Widget in your Panel provider:

use Awcodes\ClockWidget\ClockWidgetWidget;

public function panel(Panel $panel): Panel
{
    return $panel
        ->widgets([
            ClockWidgetWidget::class,
        ]);
}
```

And, that's it, our users can now install our plugin and use it in their projects.

---

<!-- Source: 03-button.md -->

## Introduction

The button component is used to render a clickable button that can perform an action:

```blade
<x-filament::button wire:click="openNewUserModal">
    New user
</x-filament::button>
```

## Using a button as an anchor link

By default, a button's underlying HTML tag is `<button>`. You can change it to be an `<a>` tag by using the `tag` attribute:

```blade
<x-filament::button
    href="https://filamentphp.com"
    tag="a"
>
    Filament
</x-filament::button>
```

## Setting the size of a button

By default, the size of a button is "medium". You can make it "extra small", "small", "large" or "extra large" by using the `size` attribute:

```blade
<x-filament::button size="xs">
    New user
</x-filament::button>

<x-filament::button size="sm">
    New user
</x-filament::button>

<x-filament::button size="lg">
    New user
</x-filament::button>

<x-filament::button size="xl">
    New user
</x-filament::button>
```

## Changing the color of a button

By default, the color of a button is "primary". You can change it to be `danger`, `gray`, `info`, `success` or `warning` by using the `color` attribute:

```blade
<x-filament::button color="danger">
    New user
</x-filament::button>

<x-filament::button color="gray">
    New user
</x-filament::button>

<x-filament::button color="info">
    New user
</x-filament::button>

<x-filament::button color="success">
    New user
</x-filament::button>

<x-filament::button color="warning">
    New user
</x-filament::button>
```

## Adding an icon to a button

You can add an [icon](../styling/icons) to a button by using the `icon` attribute:

```blade
<x-filament::button icon="heroicon-m-sparkles">
    New user
</x-filament::button>
```

You can also change the icon's position to be after the text instead of before it, using the `icon-position` attribute:

```blade
<x-filament::button
    icon="heroicon-m-sparkles"
    icon-position="after"
>
    New user
</x-filament::button>
```

## Making a button outlined

You can make a button use an "outlined" design using the `outlined` attribute:

```blade
<x-filament::button outlined>
    New user
</x-filament::button>
```

## Adding a tooltip to a button

You can add a tooltip to a button by using the `tooltip` attribute:

```blade
<x-filament::button tooltip="Register a user">
    New user
</x-filament::button>
```

## Adding a badge to a button

You can render a [badge](badge) on top of a button by using the `badge` slot:

```blade
<x-filament::button>
    Mark notifications as read
    
    <x-slot name="badge">
        3
    </x-slot>
</x-filament::button>
```

You can [change the color](badge#changing-the-color-of-the-badge) of the badge using the `badge-color` attribute:

```blade
<x-filament::button badge-color="danger">
    Mark notifications as read
    
    <x-slot name="badge">
        3
    </x-slot>
</x-filament::button>
```

---

<!-- Source: 03-checkbox.md -->

## Introduction

You can use the checkbox component to render a checkbox input that can be used to toggle a boolean value:

```blade
<label>
    <x-filament::input.checkbox wire:model="isAdmin" />

    <span>
        Is Admin
    </span>
</label>
```

## Triggering the error state of the checkbox

The checkbox has special styling that you can use if it is invalid. To trigger this styling, you can use either Blade or Alpine.js.

To trigger the error state using Blade, you can pass the `valid` attribute to the component, which contains either true or false based on if the checkbox is valid or not:

```blade
<x-filament::input.checkbox
    wire:model="isAdmin"
    :valid="! $errors->has('isAdmin')"
/>
```

Alternatively, you can use an Alpine.js expression to trigger the error state, based on if it evaluates to `true` or `false`:

```blade
<div x-data="{ errors: ['isAdmin'] }">
    <x-filament::input.checkbox
        x-model="isAdmin"
        alpine-valid="! errors.includes('isAdmin')"
    />
</div>
```

---

<!-- Source: 03-colors.md -->

## Introduction

Filament uses CSS variables to define its color palette. These CSS variables are mapped to Tailwind classes in the preset file that you load when installing Filament. The reason why Filament uses CSS variables is that it allows the framework to pass a color palette from PHP via a `<style>` element that gets rendered as part of the `@filamentStyles` Blade directive.

By default, Filament's Tailwind preset file ships with 6 colors:

- `primary`, which is [Tailwind's `amber` color](https://tailwindcss.com/docs/customizing-colors) by default
- `success`, which is [Tailwind's `green` color](https://tailwindcss.com/docs/customizing-colors) by default
- `warning`, which is [Tailwind's `amber` color](https://tailwindcss.com/docs/customizing-colors) by default
- `danger`, which is [Tailwind's `red` color](https://tailwindcss.com/docs/customizing-colors) by default
- `info`, which is [Tailwind's `blue` color](https://tailwindcss.com/docs/customizing-colors) by default
- `gray`, which is [Tailwind's `zinc` color](https://tailwindcss.com/docs/customizing-colors) by default

You can [learn how to change these colors and register new ones](#customizing-the-default-colors).

## How to pass a color to Filament

A registered "color" in Filament is not just one shade. In fact, it is an entire color palette made of [11 shades](https://tailwindcss.com/docs/customizing-colors): `50`, `100`, `200`, `300`, `400`, `500`, `600`, `700`, `800`, `900`, and `950`. When you use a color in Filament, the framework will decide which shade to use based on the context. For example, it might use the `600` shade for a component's background, `500` when it is hovered, and `400` for its border. If the user has dark mode enabled, it might use `700`, `800`, or `900` instead.

On one hand, this means that you can specify a color in Filament without having to worry about the exact shade to use, or to specify a shade for each part of the component. Filament takes care of selecting a shade that should create an accessible contrast with other elements where possible.

To customize the color that something is in Filament, you can use its name. For example, if you wanted to use the `success` color, you could pass it to a color method of a PHP component like so:

```php
use Filament\Actions\Action;
use Filament\Forms\Components\Toggle;

Action::make('proceed')
    ->color('success')
    
Toggle::make('is_active')
    ->onColor('success')
```

If you would like to use a color in a [Blade component](../components), you can pass it as an attribute:

```blade
<x-filament::badge color="success">
    Active
</x-filament::badge>
```

## Customizing the default colors

From a service provider's `boot()` method, or middleware, you can call the `FilamentColor::register()` method, which you can use to customize which colors Filament uses for UI elements.

There are 6 default colors that are used throughout Filament that you are able to customize:

```php
use Filament\Support\Colors\Color;
use Filament\Support\Facades\FilamentColor;

FilamentColor::register([
    'danger' => Color::Red,
    'gray' => Color::Zinc,
    'info' => Color::Blue,
    'primary' => Color::Amber,
    'success' => Color::Green,
    'warning' => Color::Amber,
]);
```

The `Color` class contains every [Tailwind CSS color](https://tailwindcss.com/docs/customizing-colors#color-palette-reference) to choose from.

You can also pass in a function to `register()` which will only get called when the app is getting rendered. This is useful if you are calling `register()` from a service provider, and want to access objects like the currently authenticated user, which are initialized later in middleware.

### Registering extra colors

You may register a new color to use in any Filament component by passing it to the `FilamentColor::register()` method, with its name as the key in the array:

```php
use Filament\Support\Colors\Color;
use Filament\Support\Facades\FilamentColor;

FilamentColor::register([
    'secondary' => Color::Indigo,
]);
```

You will now be able to use `secondary` as a color in any Filament component.

## Using a non-Tailwind color

You can use custom colors that are not included in the [Tailwind CSS color](https://tailwindcss.com/docs/customizing-colors#color-palette-reference) palette by passing an array of color shades from `50` to `950` in OKLCH format:

```php
use Filament\Support\Facades\FilamentColor;

FilamentColor::register([
    'danger' => [
        50 => 'oklch(0.969 0.015 12.422)',
        100 => 'oklch(0.941 0.03 12.58)',
        200 => 'oklch(0.892 0.058 10.001)',
        300 => 'oklch(0.81 0.117 11.638)',
        400 => 'oklch(0.712 0.194 13.428)',
        500 => 'oklch(0.645 0.246 16.439)',
        600 => 'oklch(0.586 0.253 17.585)',
        700 => 'oklch(0.514 0.222 16.935)',
        800 => 'oklch(0.455 0.188 13.697)',
        900 => 'oklch(0.41 0.159 10.272)',
        950 => 'oklch(0.271 0.105 12.094)',
    ],
]);
```

### Generating a custom color palette

If you want us to attempt to generate a palette for you based on a singular hex or RGB value, you can pass that in:

```php
use Filament\Support\Facades\FilamentColor;

FilamentColor::register([
    'danger' => '#ff0000',
]);

FilamentColor::register([
    'danger' => 'rgb(255, 0, 0)',
]);
```

---

<!-- Source: 03-creating-records.md -->

## Customizing data before saving

Sometimes, you may wish to modify form data before it is finally saved to the database. To do this, you may define a `mutateFormDataBeforeCreate()` method on the Create page class, which accepts the `$data` as an array, and returns the modified version:

```php
protected function mutateFormDataBeforeCreate(array $data): array
{
    $data['user_id'] = auth()->id();

    return $data;
}
```

Alternatively, if you're creating records in a modal action, check out the [Actions documentation](../actions/create#customizing-data-before-saving).

## Customizing the creation process

You can tweak how the record is created using the `handleRecordCreation()` method on the Create page class:

```php
use Illuminate\Database\Eloquent\Model;

protected function handleRecordCreation(array $data): Model
{
    return static::getModel()::create($data);
}
```

Alternatively, if you're creating records in a modal action, check out the [Actions documentation](../actions/create#customizing-the-creation-process).

## Customizing redirects

By default, after saving the form, the user will be redirected to the [Edit page](editing-records) of the resource, or the [View page](viewing-records) if it is present.

You may set up a custom redirect when the form is saved by overriding the `getRedirectUrl()` method on the Create page class.

For example, the form can redirect back to the [List page](listing-records):

```php
protected function getRedirectUrl(): string
{
    return $this->getResource()::getUrl('index');
}
```

If you wish to be redirected to the previous page, else the index page:

```php
protected function getRedirectUrl(): string
{
    return $this->previousUrl ?? $this->getResource()::getUrl('index');
}
```

You can also use the [configuration](../panel-configuration) to customize the default redirect page for all resources at once:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->resourceCreatePageRedirect('index') // or
        ->resourceCreatePageRedirect('view') // or
        ->resourceCreatePageRedirect('edit');
}
```

## Customizing the save notification

When the record is successfully created, a notification is dispatched to the user, which indicates the success of their action.

To customize the title of this notification, define a `getCreatedNotificationTitle()` method on the create page class:

```php
protected function getCreatedNotificationTitle(): ?string
{
    return 'User registered';
}
```

Alternatively, if you're creating records in a modal action, check out the [Actions documentation](../actions/create#customizing-the-save-notification).

You may customize the entire notification by overriding the `getCreatedNotification()` method on the create page class:

```php
use Filament\Notifications\Notification;

protected function getCreatedNotification(): ?Notification
{
    return Notification::make()
        ->success()
        ->title('User registered')
        ->body('The user has been created successfully.');
}
```

To disable the notification altogether, return `null` from the `getCreatedNotification()` method on the create page class:

```php
use Filament\Notifications\Notification;

protected function getCreatedNotification(): ?Notification
{
    return null;
}
```

## Creating another record

### Disabling create another

To disable the "create and create another" feature, define the `$canCreateAnother` property as `false` on the Create page class:

```php
protected static bool $canCreateAnother = false;
```

Alternatively, if you'd like to specify a dynamic condition when the feature is disabled, you may override the `canCreateAnother()` method on the Create page class:

```php
public function canCreateAnother(): bool
{
    return false;
}
```

### Preserving data when creating another

By default, when the user uses the "create and create another" feature, all the form data is cleared so the user can start fresh. If you'd like to preserve some of the data in the form, you may override the `preserveFormDataWhenCreatingAnother()` method on the Create page class, and return the part of the `$data` array that you'd like to keep:

```php
use Illuminate\Support\Arr;

protected function preserveFormDataWhenCreatingAnother(array $data): array
{
    return Arr::only($data, ['is_admin', 'organization']);
}
```

To preserve all the data, return the entire `$data` array:

```php
protected function preserveFormDataWhenCreatingAnother(array $data): array
{
    return $data;
}
```

## Lifecycle hooks

Hooks may be used to execute code at various points within a page's lifecycle, like before a form is saved. To set up a hook, create a protected method on the Create page class with the name of the hook:

```php
protected function beforeCreate(): void
{
    // ...
}
```

In this example, the code in the `beforeCreate()` method will be called before the data in the form is saved to the database.

There are several available hooks for the Create page:

```php
use Filament\Resources\Pages\CreateRecord;

class CreateUser extends CreateRecord
{
    // ...

    protected function beforeFill(): void
    {
        // Runs before the form fields are populated with their default values.
    }

    protected function afterFill(): void
    {
        // Runs after the form fields are populated with their default values.
    }

    protected function beforeValidate(): void
    {
        // Runs before the form fields are validated when the form is submitted.
    }

    protected function afterValidate(): void
    {
        // Runs after the form fields are validated when the form is submitted.
    }

    protected function beforeCreate(): void
    {
        // Runs before the form fields are saved to the database.
    }

    protected function afterCreate(): void
    {
        // Runs after the form fields are saved to the database.
    }
}
```

Alternatively, if you're creating records in a modal action, check out the [Actions documentation](../actions/create#lifecycle-hooks).

## Halting the creation process

At any time, you may call `$this->halt()` from inside a lifecycle hook or mutation method, which will halt the entire creation process:

```php
use Filament\Actions\Action;
use Filament\Notifications\Notification;

protected function beforeCreate(): void
{
    if (! auth()->user()->team->subscribed()) {
        Notification::make()
            ->warning()
            ->title('You don\'t have an active subscription!')
            ->body('Choose a plan to continue.')
            ->persistent()
            ->actions([
                Action::make('subscribe')
                    ->button()
                    ->url(route('subscribe'), shouldOpenInNewTab: true),
            ])
            ->send();
    
        $this->halt();
    }
}
```

Alternatively, if you're creating records in a modal action, check out the [Actions documentation](../actions/create#halting-the-creation-process).

## Authorization

For authorization, Filament will observe any [model policies](https://laravel.com/docs/authorization#creating-policies) that are registered in your app.

Users may access the Create page if the `create()` method of the model policy returns `true`.

## Using a wizard

You may easily transform the creation process into a multistep wizard.

On the page class, add the corresponding `HasWizard` trait:

```php
use App\Filament\Resources\Categories\CategoryResource;
use Filament\Resources\Pages\CreateRecord;

class CreateCategory extends CreateRecord
{
    use CreateRecord\Concerns\HasWizard;
    
    protected static string $resource = CategoryResource::class;

    protected function getSteps(): array
    {
        return [
            // ...
        ];
    }
}
```

Inside the `getSteps()` array, return your [wizard steps](../schemas/wizards):

```php
use Filament\Forms\Components\MarkdownEditor;
use Filament\Forms\Components\TextInput;
use Filament\Forms\Components\Toggle;
use Filament\Schemas\Components\Wizard\Step;

protected function getSteps(): array
{
    return [
        Step::make('Name')
            ->description('Give the category a clear and unique name')
            ->schema([
                TextInput::make('name')
                    ->required()
                    ->live()
                    ->afterStateUpdated(fn ($state, callable $set) => $set('slug', Str::slug($state))),
                TextInput::make('slug')
                    ->disabled()
                    ->required()
                    ->unique(Category::class, 'slug', fn ($record) => $record),
            ]),
        Step::make('Description')
            ->description('Add some extra details')
            ->schema([
                MarkdownEditor::make('description')
                    ->columnSpan('full'),
            ]),
        Step::make('Visibility')
            ->description('Control who can view it')
            ->schema([
                Toggle::make('is_visible')
                    ->label('Visible to customers.')
                    ->default(true),
            ]),
    ];
}
```

Alternatively, if you're creating records in a modal action, check out the [Actions documentation](../actions/create#using-a-wizard).

Now, create a new record to see your wizard in action! Edit will still use the form defined within the resource class.

If you'd like to allow free navigation, so all the steps are skippable, override the `hasSkippableSteps()` method:

```php
public function hasSkippableSteps(): bool
{
    return true;
}
```

### Sharing fields between the form schema and wizards

If you'd like to reduce the amount of repetition between the resource form and wizard steps, it's a good idea to extract public static form functions for your fields, where you can easily retrieve an instance of a field from the form schema or the wizard:

```php
use Filament\Forms;
use Filament\Schemas\Schema;

class CategoryForm
{
    public static function configure(Schema $schema): Schema
    {
        return $schema
            ->components([
                static::getNameFormField(),
                static::getSlugFormField(),
                // ...
            ]);
    }
    
    public static function getNameFormField(): Forms\Components\TextInput
    {
        return TextInput::make('name')
            ->required()
            ->live()
            ->afterStateUpdated(fn ($state, callable $set) => $set('slug', Str::slug($state)));
    }
    
    public static function getSlugFormField(): Forms\Components\TextInput
    {
        return TextInput::make('slug')
            ->disabled()
            ->required()
            ->unique(Category::class, 'slug', fn ($record) => $record);
    }
}
```

```php
use App\Filament\Resources\Categories\Schemas\CategoryForm;
use Filament\Resources\Pages\CreateRecord;

class CreateCategory extends CreateRecord
{
    use CreateRecord\Concerns\HasWizard;
    
    protected static string $resource = CategoryResource::class;

    protected function getSteps(): array
    {
        return [
            Step::make('Name')
                ->description('Give the category a clear and unique name')
                ->schema([
                    CategoryForm::getNameFormField(),
                    CategoryForm::getSlugFormField(),
                ]),
            // ...
        ];
    }
}
```

## Importing resource records

Filament includes an `ImportAction` that you can add to the `getHeaderActions()` of the [List page](listing-records). It allows users to upload a CSV of data to import into the resource:

```php
use App\Filament\Imports\ProductImporter;
use Filament\Actions;

protected function getHeaderActions(): array
{
    return [
        Actions\ImportAction::make()
            ->importer(ProductImporter::class),
        Actions\CreateAction::make(),
    ];
}
```

The "importer" class [needs to be created](../actions/import#creating-an-importer) to tell Filament how to import each row of the CSV. You can learn everything about the `ImportAction` in the [Actions documentation](../actions/import).

## Custom actions

"Actions" are buttons that are displayed on pages, which allow the user to run a Livewire method on the page or visit a URL.

On resource pages, actions are usually in 2 places: in the top right of the page, and below the form.

For example, you may add a new button action in the header of the Create page:

```php
use App\Filament\Imports\UserImporter;
use Filament\Actions;
use Filament\Resources\Pages\CreateRecord;

class CreateUser extends CreateRecord
{
    // ...

    protected function getHeaderActions(): array
    {
        return [
            Actions\ImportAction::make()
                ->importer(UserImporter::class),
        ];
    }
}
```

Or, a new button next to "Create" below the form:

```php
use Filament\Actions\Action;
use Filament\Resources\Pages\CreateRecord;

class CreateUser extends CreateRecord
{
    // ...

    protected function getFormActions(): array
    {
        return [
            ...parent::getFormActions(),
            Action::make('close')->action('createAndClose'),
        ];
    }

    public function createAndClose(): void
    {
        // ...
    }
}
```

To view the entire actions API, please visit the [pages section](../navigation/custom-pages#adding-actions-to-pages).

### Adding a create action button to the header

The "Create" button can be moved to the header of the page by overriding the `getHeaderActions()` method and using `getCreateFormAction()`. You need to pass `formId()` to the action, to specify that the action should submit the form with the ID of `form`, which is the `<form>` ID used in the view of the page:

```php
protected function getHeaderActions(): array
{
    return [
        $this->getCreateFormAction()
            ->formId('form'),
    ];
}
```

You may remove all actions from the form by overriding the `getFormActions()` method to return an empty array:

```php
protected function getFormActions(): array
{
    return [];
}
```

## Custom page content

Each page in Filament has its own [schema](../schemas), which defines the overall structure and content. You can override the schema for the page by defining a `content()` method on it. The `content()` method for the Create page contains the following components by default:

```php
use Filament\Schemas\Schema;

public function content(Schema $schema): Schema
{
    return $schema
        ->components([
            $this->getFormContentComponent(), // This method returns a component to display the form that is defined in this resource
        ]);
}
```

Inside the `components()` array, you can insert any [schema component](../schemas). You can reorder the components by changing the order of the array or remove any of the components that are not needed.

### Using a custom Blade view

For further customization opportunities, you can override the static `$view` property on the page class to a custom view in your app:

```php
protected string $view = 'filament.resources.users.pages.create-user';
```

This assumes that you have created a view at `resources/views/filament/resources/users/pages/create-user.blade.php`:

```blade
<x-filament-panels::page>
    {{ $this->content }} {{-- This will render the content of the page defined in the `content()` method, which can be removed if you want to start from scratch --}}
</x-filament-panels::page>
```

---

<!-- Source: 03-dropdown.md -->

## Introduction

The dropdown component allows you to render a dropdown menu with a button that triggers it:

```blade
<x-filament::dropdown>
    <x-slot name="trigger">
        <x-filament::button>
            More actions
        </x-filament::button>
    </x-slot>
    
    <x-filament::dropdown.list>
        <x-filament::dropdown.list.item wire:click="openViewModal">
            View
        </x-filament::dropdown.list.item>
        
        <x-filament::dropdown.list.item wire:click="openEditModal">
            Edit
        </x-filament::dropdown.list.item>
        
        <x-filament::dropdown.list.item wire:click="openDeleteModal">
            Delete
        </x-filament::dropdown.list.item>
    </x-filament::dropdown.list>
</x-filament::dropdown>
```

## Using a dropdown item as an anchor link

By default, a dropdown item's underlying HTML tag is `<button>`. You can change it to be an `<a>` tag by using the `tag` attribute:

```blade
<x-filament::dropdown.list.item
    href="https://filamentphp.com"
    tag="a"
>
    Filament
</x-filament::dropdown.list.item>
```

## Changing the color of a dropdown item

By default, the color of a dropdown item is "gray". You can change it to be `danger`, `info`, `primary`, `success` or `warning` by using the `color` attribute:

```blade
<x-filament::dropdown.list.item color="danger">
    Edit
</x-filament::dropdown.list.item>

<x-filament::dropdown.list.item color="info">
    Edit
</x-filament::dropdown.list.item>

<x-filament::dropdown.list.item color="primary">
    Edit
</x-filament::dropdown.list.item>

<x-filament::dropdown.list.item color="success">
    Edit
</x-filament::dropdown.list.item>

<x-filament::dropdown.list.item color="warning">
    Edit
</x-filament::dropdown.list.item>
```

## Adding an icon to a dropdown item

You can add an [icon](../styling/icons) to a dropdown item by using the `icon` attribute:

```blade
<x-filament::dropdown.list.item icon="heroicon-m-pencil">
    Edit
</x-filament::dropdown.list.item>
```

### Changing the icon color of a dropdown item

By default, the icon color uses the [same color as the item itself](#changing-the-color-of-a-dropdown-item). You can override it to be `danger`, `info`, `primary`, `success` or `warning` by using the `icon-color` attribute:

```blade
<x-filament::dropdown.list.item icon="heroicon-m-pencil" icon-color="danger">
    Edit
</x-filament::dropdown.list.item>

<x-filament::dropdown.list.item icon="heroicon-m-pencil" icon-color="info">
    Edit
</x-filament::dropdown.list.item>

<x-filament::dropdown.list.item icon="heroicon-m-pencil" icon-color="primary">
    Edit
</x-filament::dropdown.list.item>

<x-filament::dropdown.list.item icon="heroicon-m-pencil" icon-color="success">
    Edit
</x-filament::dropdown.list.item>

<x-filament::dropdown.list.item icon="heroicon-m-pencil" icon-color="warning">
    Edit
</x-filament::dropdown.list.item>
```

## Adding an image to a dropdown item

You can add a circular image to a dropdown item by using the `image` attribute:

```blade
<x-filament::dropdown.list.item image="https://filamentphp.com/dan.jpg">
    Dan Harrin
</x-filament::dropdown.list.item>
```

## Adding a badge to a dropdown item

You can render a [badge](badge) on top of a dropdown item by using the `badge` slot:

```blade
<x-filament::dropdown.list.item>
    Mark notifications as read
    
    <x-slot name="badge">
        3
    </x-slot>
</x-filament::dropdown.list.item>
```

You can [change the color](badge#changing-the-color-of-the-badge) of the badge using the `badge-color` attribute:

```blade
<x-filament::dropdown.list.item badge-color="danger">
    Mark notifications as read
    
    <x-slot name="badge">
        3
    </x-slot>
</x-filament::dropdown.list.item>
```

## Setting the placement of a dropdown

The dropdown may be positioned relative to the trigger button by using the `placement` attribute:

```blade
<x-filament::dropdown placement="top-start">
    {{-- Dropdown items --}}
</x-filament::dropdown>
```

## Setting the width of a dropdown

The dropdown may be set to a width by using the `width` attribute. Options correspond to [Tailwind's max-width scale](https://tailwindcss.com/docs/max-width). The options are `xs`, `sm`, `md`, `lg`, `xl`, `2xl`, `3xl`, `4xl`, `5xl`, `6xl` and `7xl`:

```blade
<x-filament::dropdown width="xs">
    {{-- Dropdown items --}}
</x-filament::dropdown>
```

## Controlling the maximum height of a dropdown

The dropdown content can have a maximum height using the `max-height` attribute, so that it scrolls. You can pass a [CSS length](https://developer.mozilla.org/en-US/docs/Web/CSS/length):

```blade
<x-filament::dropdown max-height="400px">
    {{-- Dropdown items --}}
</x-filament::dropdown>
```

---

<!-- Source: 03-enums.md -->

import Aside from "@components/Aside.astro"

## Introduction

Enums are special PHP classes that represent a fixed set of constants. They are useful for modeling concepts that have a limited number of possible values, like days of the week, months in a year, or the suits in a deck of cards.

Since enum "cases" are instances of the enum class, adding interfaces to enums proves to be very useful. Filament provides a collection of interfaces that you can add to enums, which enhance your experience when working with them.

<Aside variant="warning">
    When using an enum with an attribute on your Eloquent model, please [ensure that it is cast correctly](https://laravel.com/docs/eloquent-mutators#enum-casting).
</Aside>

## Enum labels

The `HasLabel` interface transforms an enum instance into a textual label. This is useful for displaying human-readable enum values in your UI.

```php
use Filament\Support\Contracts\HasLabel;

enum Status: string implements HasLabel
{
    case Draft = 'draft';
    case Reviewing = 'reviewing';
    case Published = 'published';
    case Rejected = 'rejected';
    
    public function getLabel(): ?string
    {
        return $this->name;
        
        // or
    
        return match ($this) {
            self::Draft => 'Draft',
            self::Reviewing => 'Reviewing',
            self::Published => 'Published',
            self::Rejected => 'Rejected',
        };
    }
}
```

### Using the enum label with form field options

The `HasLabel` interface can be used to generate an array of options from an enum, where the enum's value is the key and the enum's label is the value. This applies to form fields like [`Select`](../forms/select) and [`CheckboxList`](../forms/checkbox-list), as well as the Table Builder's [`SelectColumn`](../tables/columns/select) and [`SelectFilter`](../tables/filters/select):

```php
use Filament\Forms\Components\CheckboxList;
use Filament\Forms\Components\Radio;
use Filament\Forms\Components\Select;
use Filament\Tables\Columns\SelectColumn;
use Filament\Tables\Filters\SelectFilter;

Select::make('status')
    ->options(Status::class)

CheckboxList::make('status')
    ->options(Status::class)

Radio::make('status')
    ->options(Status::class)

SelectColumn::make('status')
    ->options(Status::class)

SelectFilter::make('status')
    ->options(Status::class)
```

In these examples, `Status::class` is the enum class which implements `HasLabel`, and the options are generated from that:

```php
[
    'draft' => 'Draft',
    'reviewing' => 'Reviewing',
    'published' => 'Published',
    'rejected' => 'Rejected',
]
```

### Using the enum label with a text column in your table

If you use a [`TextColumn`](../tables/columns/text) with the Table Builder, and it is cast to an enum in your Eloquent model, Filament will automatically use the `HasLabel` interface to display the enum's label instead of its raw value.

### Using the enum label as a group title in your table

If you use a [grouping](../tables/grouping) with the Table Builder, and it is cast to an enum in your Eloquent model, Filament will automatically use the `HasLabel` interface to display the enum's label instead of its raw value. The label will be displayed as the [title of each group](../tables/grouping#setting-a-group-title).

### Using the enum label with a text entry in your infolist

If you use a [`TextEntry`](../infolists/text-entry) in an infolist, and it is cast to an enum in your Eloquent model, Filament will automatically use the `HasLabel` interface to display the enum's label instead of its raw value.

## Enum colors

The `HasColor` interface transforms an enum instance into a [color](colors). This is useful for displaying colored enum values in your UI.

```php
use Filament\Support\Contracts\HasColor;

enum Status: string implements HasColor
{
    case Draft = 'draft';
    case Reviewing = 'reviewing';
    case Published = 'published';
    case Rejected = 'rejected';
    
    public function getColor(): string | array | null
    {
        return match ($this) {
            self::Draft => 'gray',
            self::Reviewing => 'warning',
            self::Published => 'success',
            self::Rejected => 'danger',
        };
    }
}
```

### Using the enum color with a text column in your table

If you use a [`TextColumn`](../tables/columns/text) with the Table Builder, and it is cast to an enum in your Eloquent model, Filament will automatically use the `HasColor` interface to display the enum label in its color. This works best if you use the [`badge()`](../tables/columns/text#displaying-as-a-badge) method on the column.

### Using the enum color with a text entry in your infolist

If you use a [`TextEntry`](../infolists/text-entry) in an infolist, and it is cast to an enum in your Eloquent model, Filament will automatically use the `HasColor` interface to display the enum label in its color. This works best if you use the [`badge()`](../infolists/text#displaying-as-a-badge) method on the entry.

### Using the enum color with a toggle buttons field in your form

If you use a [`ToggleButtons`](../forms/toggle-buttons) form field, and it is set to use an enum for its options, Filament will automatically use the `HasColor` interface to display the enum label in its color.

## Enum icons

The `HasIcon` interface transforms an enum instance into an [icon](icons). This is useful for displaying icons alongside enum values in your UI.

```php
use Filament\Support\Contracts\HasIcon;

enum Status: string implements HasIcon
{
    case Draft = 'draft';
    case Reviewing = 'reviewing';
    case Published = 'published';
    case Rejected = 'rejected';
    
    public function getIcon(): ?string
    {
        return match ($this) {
            self::Draft => 'heroicon-m-pencil',
            self::Reviewing => 'heroicon-m-eye',
            self::Published => 'heroicon-m-check',
            self::Rejected => 'heroicon-m-x-mark',
        };
    }
}
```

### Using the enum icon with a text column in your table

If you use a [`TextColumn`](../tables/columns/text) with the Table Builder, and it is cast to an enum in your Eloquent model, Filament will automatically use the `HasIcon` interface to display the enum's icon aside its label. This works best if you use the [`badge()`](../tables/columns/text#displaying-as-a-badge) method on the column.

### Using the enum icon with a text entry in your infolist

If you use a [`TextEntry`](../infolists/text-entry) in an infolist, and it is cast to an enum in your Eloquent model, Filament will automatically use the `HasIcon` interface to display the enum's icon aside its label. This works best if you use the [`badge()`](../infolists/text#displaying-as-a-badge) method on the entry.

### Using the enum icon with a toggle buttons field in your form

If you use a [`ToggleButtons`](../forms/toggle-buttons) form field, and it is set to use an enum for its options, Filament will automatically use the `HasIcon` interface to display the enum's icon aside its label.

## Enum descriptions

The `HasDescription` interface transforms an enum instance into a textual description, often displayed under its [label](#enum-labels). This is useful for displaying human-friendly descriptions in your UI.

```php
use Filament\Support\Contracts\HasDescription;
use Filament\Support\Contracts\HasLabel;

enum Status: string implements HasLabel, HasDescription
{
    case Draft = 'draft';
    case Reviewing = 'reviewing';
    case Published = 'published';
    case Rejected = 'rejected';
    
    public function getLabel(): ?string
    {
        return $this->name;
    }
    
    public function getDescription(): ?string
    {
        return match ($this) {
            self::Draft => 'This has not finished being written yet.',
            self::Reviewing => 'This is ready for a staff member to read.',
            self::Published => 'This has been approved by a staff member and is public on the website.',
            self::Rejected => 'A staff member has decided this is not appropriate for the website.',
        };
    }
}
```

### Using the enum description with form field descriptions

The `HasDescription` interface can be used to generate an array of descriptions from an enum, where the enum's value is the key and the enum's description is the value. This applies to form fields like [`Radio`](../forms/radio#setting-option-descriptions) and [`CheckboxList`](../forms/checkbox-list#setting-option-descriptions):

```php
use Filament\Forms\Components\CheckboxList;
use Filament\Forms\Components\Radio;

Radio::make('status')
    ->options(Status::class)

CheckboxList::make('status')
    ->options(Status::class)
```

---

<!-- Source: 03-fieldset.md -->

## Introduction

You can use a fieldset to group multiple form fields together, optionally with a label:

```blade
<x-filament::fieldset>
    <x-slot name="label">
        Address
    </x-slot>
    
    {{-- Form fields --}}
</x-filament::fieldset>
```

---

<!-- Source: 03-icon-button.md -->

## Introduction

The button component is used to render a clickable button that can perform an action:

```blade
<x-filament::icon-button
    icon="heroicon-m-plus"
    wire:click="openNewUserModal"
    label="New label"
/>
```

## Using an icon button as an anchor link

By default, an icon button's underlying HTML tag is `<button>`. You can change it to be an `<a>` tag by using the `tag` attribute:

```blade
<x-filament::icon-button
    icon="heroicon-m-arrow-top-right-on-square"
    href="https://filamentphp.com"
    tag="a"
    label="Filament"
/>
```

## Setting the size of an icon button

By default, the size of an icon button is "medium". You can make it "extra small", "small", "large" or "extra large" by using the `size` attribute:

```blade
<x-filament::icon-button
    icon="heroicon-m-plus"
    size="xs"
    label="New label"
/>

<x-filament::icon-button
    icon="heroicon-m-plus"
    size="sm"
    label="New label"
/>

<x-filament::icon-button
    icon="heroicon-s-plus"
    size="lg"
    label="New label"
/>

<x-filament::icon-button
    icon="heroicon-s-plus"
    size="xl"
    label="New label"
/>
```

## Changing the color of an icon button

By default, the color of an icon button is "primary". You can change it to be `danger`, `gray`, `info`, `success` or `warning` by using the `color` attribute:

```blade
<x-filament::icon-button
    icon="heroicon-m-plus"
    color="danger"
    label="New label"
/>

<x-filament::icon-button
    icon="heroicon-m-plus"
    color="gray"
    label="New label"
/>

<x-filament::icon-button
    icon="heroicon-m-plus"
    color="info"
    label="New label"
/>

<x-filament::icon-button
    icon="heroicon-m-plus"
    color="success"
    label="New label"
/>

<x-filament::icon-button
    icon="heroicon-m-plus"
    color="warning"
    label="New label"
/>
```

## Adding a tooltip to an icon button

You can add a tooltip to an icon button by using the `tooltip` attribute:

```blade
<x-filament::icon-button
    icon="heroicon-m-plus"
    tooltip="Register a user"
    label="New label"
/>
```

## Adding a badge to an icon button

You can render a [badge](badge) on top of an icon button by using the `badge` slot:

```blade
<x-filament::icon-button
    icon="heroicon-m-x-mark"
    label="Mark notifications as read"
>
    <x-slot name="badge">
        3
    </x-slot>
</x-filament::icon-button>
```

You can [change the color](badge#changing-the-color-of-the-badge) of the badge using the `badge-color` attribute:

```blade
<x-filament::icon-button
    icon="heroicon-m-x-mark"
    label="Mark notifications as read"
    badge-color="danger"
>
    <x-slot name="badge">
        3
    </x-slot>
</x-filament::icon-button>
```

---

<!-- Source: 03-input-wrapper.md -->

## Introduction

The input wrapper component should be used as a wrapper around the [input](input) or [select](select) components. It provides a border and other elements such as a prefix or suffix.

```blade
<x-filament::input.wrapper>
    <x-filament::input
        type="text"
        wire:model="name"
    />
</x-filament::input.wrapper>

<x-filament::input.wrapper>
    <x-filament::input.select wire:model="status">
        <option value="draft">Draft</option>
        <option value="reviewing">Reviewing</option>
        <option value="published">Published</option>
    </x-filament::input.select>
</x-filament::input.wrapper>
```

## Triggering the error state of the input

The component has special styling that you can use if it is invalid. To trigger this styling, you can use either Blade or Alpine.js.

To trigger the error state using Blade, you can pass the `valid` attribute to the component, which contains either true or false based on if the input is valid or not:

```blade
<x-filament::input.wrapper :valid="! $errors->has('name')">
    <x-filament::input
        type="text"
        wire:model="name"
    />
</x-filament::input.wrapper>
```

Alternatively, you can use an Alpine.js expression to trigger the error state, based on if it evaluates to `true` or `false`:

```blade
<div x-data="{ errors: ['name'] }">
    <x-filament::input.wrapper alpine-valid="! errors.includes('name')">
        <x-filament::input
            type="text"
            wire:model="name"
        />
    </x-filament::input.wrapper>
</div>
```

## Disabling the input

To disable the input, you must also pass the `disabled` attribute to the wrapper component:

```blade
<x-filament::input.wrapper disabled>
    <x-filament::input
        type="text"
        wire:model="name"
        disabled
    />
</x-filament::input.wrapper>
```

## Adding affix text aside the input

You may place text before and after the input using the `prefix` and `suffix` slots:

```blade
<x-filament::input.wrapper>
    <x-slot name="prefix">
        https://
    </x-slot>

    <x-filament::input
        type="text"
        wire:model="domain"
    />

    <x-slot name="suffix">
        .com
    </x-slot>
</x-filament::input.wrapper>
```

### Using icons as affixes

You may place an [icon](../styling/icons) before and after the input using the `prefix-icon` and `suffix-icon` attributes:

```blade
<x-filament::input.wrapper suffix-icon="heroicon-m-globe-alt">
    <x-filament::input
        type="url"
        wire:model="domain"
    />
</x-filament::input.wrapper>
```

#### Setting the affix icon's color

Affix icons are gray by default, but you may set a different color using the `prefix-icon-color` and `affix-icon-color` attributes:

```blade
<x-filament::input.wrapper
    suffix-icon="heroicon-m-check-circle"
    suffix-icon-color="success"
>
    <x-filament::input
        type="url"
        wire:model="domain"
    />
</x-filament::input.wrapper>
```

---

<!-- Source: 03-input.md -->

## Introduction

The input component is a wrapper around the native `<input>` element. It provides a simple interface for entering a single line of text.

```blade
<x-filament::input.wrapper>
    <x-filament::input
        type="text"
        wire:model="name"
    />
</x-filament::input.wrapper>
```

To use the input component, you must wrap it in an "input wrapper" component, which provides a border and other elements such as a prefix or suffix. You can learn more about customizing the input wrapper component [here](input-wrapper).

---

<!-- Source: 03-link.md -->

## Introduction

The link component is used to render a clickable link that can perform an action:

```blade
<x-filament::link :href="route('users.create')">
    New user
</x-filament::link>
```

## Using a link as a button

By default, a link's underlying HTML tag is `<a>`. You can change it to be a `<button>` tag by using the `tag` attribute:

```blade
<x-filament::link
    wire:click="openNewUserModal"
    tag="button"
>
    New user
</x-filament::link>
```

## Setting the size of a link

By default, the size of a link is "medium". You can make it "small", "large", "extra large" or "extra extra large" by using the `size` attribute:

```blade
<x-filament::link size="sm">
    New user
</x-filament::link>

<x-filament::link size="lg">
    New user
</x-filament::link>

<x-filament::link size="xl">
    New user
</x-filament::link>

<x-filament::link size="2xl">
    New user
</x-filament::link>
```

## Setting the font weight of a link

By default, the font weight of links is `semibold`. You can make it `thin`, `extralight`, `light`, `normal`, `medium`, `bold`, `extrabold` or `black` by using the `weight` attribute:

```blade
<x-filament::link weight="thin">
    New user
</x-filament::link>

<x-filament::link weight="extralight">
    New user
</x-filament::link>

<x-filament::link weight="light">
    New user
</x-filament::link>

<x-filament::link weight="normal">
    New user
</x-filament::link>

<x-filament::link weight="medium">
    New user
</x-filament::link>

<x-filament::link weight="semibold">
    New user
</x-filament::link>
   
<x-filament::link weight="bold">
    New user
</x-filament::link>

<x-filament::link weight="black">
    New user
</x-filament::link> 
```

Alternatively, you can pass in a custom CSS class to define the weight:

```blade
<x-filament::link weight="md:font-[650]">
    New user
</x-filament::link>
```

## Changing the color of a link

By default, the color of a link is "primary". You can change it to be `danger`, `gray`, `info`, `success` or `warning` by using the `color` attribute:

```blade
<x-filament::link color="danger">
    New user
</x-filament::link>

<x-filament::link color="gray">
    New user
</x-filament::link>

<x-filament::link color="info">
    New user
</x-filament::link>

<x-filament::link color="success">
    New user
</x-filament::link>

<x-filament::link color="warning">
    New user
</x-filament::link>
```

## Adding an icon to a link

You can add an [icon](../styling/icons) to a link by using the `icon` attribute:

```blade
<x-filament::link icon="heroicon-m-sparkles">
    New user
</x-filament::link>
```

You can also change the icon's position to be after the text instead of before it, using the `icon-position` attribute:

```blade
<x-filament::link
    icon="heroicon-m-sparkles"
    icon-position="after"
>
    New user
</x-filament::link>
```

## Adding a tooltip to a link

You can add a tooltip to a link by using the `tooltip` attribute:

```blade
<x-filament::link tooltip="Register a user">
    New user
</x-filament::link>
```

## Adding a badge to a link

You can render a [badge](badge) on top of a link by using the `badge` slot:

```blade
<x-filament::link>
    Mark notifications as read

    <x-slot name="badge">
        3
    </x-slot>
</x-filament::link>
```

You can [change the color](badge#changing-the-color-of-the-badge) of the badge using the `badge-color` attribute:

```blade
<x-filament::link badge-color="danger">
    Mark notifications as read

    <x-slot name="badge">
        3
    </x-slot>
</x-filament::link>
```

---

<!-- Source: 03-loading-indicator.md -->

## Introduction

The loading indicator is an animated SVG that can be used to indicate that something is in progress:

```blade
<x-filament::loading-indicator class="h-5 w-5" />
```

---

<!-- Source: 03-modal.md -->

## Introduction

The modal component is able to open a dialog window or slide-over with any content:

```blade
<x-filament::modal>
    <x-slot name="trigger">
        <x-filament::button>
            Open modal
        </x-filament::button>
    </x-slot>

    {{-- Modal content --}}
</x-filament::modal>
```

## Controlling a modal from JavaScript

You can use the `trigger` slot to render a button that opens the modal. However, this is not required. You have complete control over when the modal opens and closes through JavaScript. First, give the modal an ID so that you can reference it:

```blade
<x-filament::modal id="edit-user">
    {{-- Modal content --}}
</x-filament::modal>
```

Now, you can dispatch an `open-modal` or `close-modal` browser event, passing the modal's ID, which will open or close the modal. For example, from a Livewire component:

```php
$this->dispatch('open-modal', id: 'edit-user');
```

Or from Alpine.js:

```php
$dispatch('open-modal', { id: 'edit-user' })
```

## Adding a heading to a modal

You can add a heading to a modal by using the `heading` slot:

```blade
<x-filament::modal>
    <x-slot name="heading">
        Modal heading
    </x-slot>

    {{-- Modal content --}}
</x-filament::modal>
```

## Adding a description to a modal

You can add a description, below the heading, to a modal by using the `description` slot:

```blade
<x-filament::modal>
    <x-slot name="heading">
        Modal heading
    </x-slot>

    <x-slot name="description">
        Modal description
    </x-slot>

    {{-- Modal content --}}
</x-filament::modal>
```

## Adding an icon to a modal

You can add an [icon](../styling/icons) to a modal by using the `icon` attribute:

```blade
<x-filament::modal icon="heroicon-o-information-circle">
    <x-slot name="heading">
        Modal heading
    </x-slot>

    {{-- Modal content --}}
</x-filament::modal>
```

By default, the color of an icon is "primary". You can change it to be `danger`, `gray`, `info`, `success` or `warning` by using the `icon-color` attribute:

```blade
<x-filament::modal
    icon="heroicon-o-exclamation-triangle"
    icon-color="danger"
>
    <x-slot name="heading">
        Modal heading
    </x-slot>

    {{-- Modal content --}}
</x-filament::modal>
```

## Adding a footer to a modal

You can add a footer to a modal by using the `footer` slot:

```blade
<x-filament::modal>
    {{-- Modal content --}}
    
    <x-slot name="footer">
        {{-- Modal footer content --}}
    </x-slot>
</x-filament::modal>
```

Alternatively, you can add actions into the footer by using the `footerActions` slot:

```blade
<x-filament::modal>
    {{-- Modal content --}}
    
    <x-slot name="footerActions">
        {{-- Modal footer actions --}}
    </x-slot>
</x-filament::modal>
```

## Changing the modal's alignment

By default, modal content will be aligned to the start, or centered if the modal is `xs` or `sm` in [width](#changing-the-modal-width). If you wish to change the alignment of content in a modal, you can use the `alignment` attribute and pass it `start` or `center`:

```blade
<x-filament::modal alignment="center">
    {{-- Modal content --}}
</x-filament::modal>
```

## Using a slide-over instead of a modal

You can open a "slide-over" dialog instead of a modal by using the `slide-over` attribute:

```blade
<x-filament::modal slide-over>
    {{-- Slide-over content --}}
</x-filament::modal>
```

## Making the modal header sticky

The header of a modal scrolls out of view with the modal content when it overflows the modal size. However, slide-overs have a sticky modal that's always visible. You may control this behavior using the `sticky-header` attribute:

```blade
<x-filament::modal sticky-header>
    <x-slot name="heading">
        Modal heading
    </x-slot>

    {{-- Modal content --}}
</x-filament::modal>
```

## Making the modal footer sticky

The footer of a modal is rendered inline after the content by default. Slide-overs, however, have a sticky footer that always shows when scrolling the content. You may enable this for a modal too using the `sticky-footer` attribute:

```blade
<x-filament::modal sticky-footer>
    {{-- Modal content --}}
    
    <x-slot name="footer">
        {{-- Modal footer content --}}
    </x-slot>
</x-filament::modal>
```

## Changing the modal width

You can change the width of the modal by using the `width` attribute. Options correspond to [Tailwind's max-width scale](https://tailwindcss.com/docs/max-width). The options are `xs`, `sm`, `md`, `lg`, `xl`, `2xl`, `3xl`, `4xl`, `5xl`, `6xl`, `7xl`, and `screen`:

```blade
<x-filament::modal width="5xl">
    {{-- Modal content --}}
</x-filament::modal>
```

## Closing the modal by clicking away

By default, when you click away from a modal, it will close itself. If you wish to disable this behavior for a specific action, you can use the `close-by-clicking-away` attribute:

```blade
<x-filament::modal :close-by-clicking-away="false">
    {{-- Modal content --}}
</x-filament::modal>
```

## Closing the modal by escaping

By default, when you press escape on a modal, it will close itself. If you wish to disable this behavior for a specific action, you can use the `close-by-escaping` attribute:

```blade
<x-filament::modal :close-by-escaping="false">
    {{-- Modal content --}}
</x-filament::modal>
```

## Hiding the modal close button

By default, modals with a header have a close button in the top right corner. You can remove the close button from the modal by using the `close-button` attribute:

```blade
<x-filament::modal :close-button="false">
    <x-slot name="heading">
        Modal heading
    </x-slot>

    {{-- Modal content --}}
</x-filament::modal>
```

## Preventing the modal from autofocusing

By default, modals will autofocus on the first focusable element when opened. If you wish to disable this behavior, you can use the `autofocus` attribute:

```blade
<x-filament::modal :autofocus="false">
    {{-- Modal content --}}
</x-filament::modal>
```

## Disabling the modal trigger button

By default, the trigger button will open the modal even if it is disabled, since the click event listener is registered on a wrapping element of the button itself. If you want to prevent the modal from opening, you should also use the `disabled` attribute on the trigger slot:

```blade
<x-filament::modal>
    <x-slot name="trigger" disabled>
        <x-filament::button :disabled="true">
            Open modal
        </x-filament::button>
    </x-slot>
    {{-- Modal content --}}
</x-filament::modal>
```

---

<!-- Source: 03-optimizing-local-development.md -->

import Aside from "@components/Aside.astro"

This section includes optional tips to optimize performance when running your Filament app locally.

If you're looking for production-specific optimizations, check out [Deploying to production](../deployment).

## Enabling OPcache

[OPcache](https://www.php.net/manual/en/book.opcache.php) improves PHP performance by storing precompiled script bytecode in shared memory, thereby removing the need for PHP to load and parse scripts on each request. This can significantly speed up your local development environment, especially for larger applications.

### Checking OPcache status

To check if [OPcache](https://www.php.net/manual/en/book.opcache.php) is enabled, run:

```bash
php -r "echo 'opcache.enable => ' . ini_get('opcache.enable') . PHP_EOL;"
```

You should see `opcache.enable => 1`. If not, enable it by adding the following line to your `php.ini`:

```ini
opcache.enable=1 # Enable OPcache
```

<Aside variant="tip">
    To locate your `php.ini` file, run: `php --ini`
</Aside>

### Configuring OPcache settings

If you're experiencing slow response times or suspect that OPcache is running out of space, you can adjust these parameters in your `php.ini` file:

```ini
opcache.memory_consumption=128
opcache.max_accelerated_files=10000
```

<Aside variant="tip">
    To locate your `php.ini` file, run: `php --ini`
</Aside>

- `opcache.memory_consumption`: defines how much memory (in megabytes) OPcache can use to store precompiled PHP code. You can try setting this to `128` and adjust based on your project's needs.
- `opcache.max_accelerated_files`: sets the maximum number of PHP files that OPcache can cache. You can try `10000` as a starting point and increase if your application includes a large number of files.

These settings are optional but useful if you're troubleshooting performance or working on a large Laravel app.

## Exclude your project folder from antivirus scanning

Issues with the performance of Filament, particularly on Windows, often involve [Microsoft Defender](https://www.microsoft.com/en-us/microsoft-365/microsoft-defender-for-individuals).

Security software, such as realtime file scanners or antivirus tools, can slow down your development environment by scanning files every time they're accessed. This can affect PHP execution, view rendering, and performance in general.

If you're noticing slowness, consider excluding your local project folder from realtime scanning.

Tools like [Microsoft Defender](https://www.microsoft.com/en-us/microsoft-365/microsoft-defender-for-individuals), or similar antivirus solutions, can be configured to skip specific directories. Check your antivirus or security software documentation for instructions on how to exclude specific folders from realtime scanning.

<Aside variant="warning">
    Only exclude folders from scanning if you fully trust the project and understand the risks.
</Aside>

## Disabling debugging tools

Debugging tools can be very useful for local development, but they can significantly slow down your application when you aren't actively using them. Temporarily disabling these tools when you need maximum performance can make a noticeable difference in your development experience.

### Disabling view debugging in Laravel Herd

[Laravel Herd](https://herd.laravel.com/) includes a view debugging tool for [macOS](https://herd.laravel.com/docs/macos/debugging/dumps#views) and [Windows](https://herd.laravel.com/docs/windows/debugging/dumps#views). It shows which views were rendered and what data was passed to them during a request.

While helpful for debugging, this feature can significantly slow down your app. If you're not actively using it, it's best to turn it off.

To disable view debugging in Herd:

- Open Herd > Dumps.
- Click Settings.
- Uncheck the "Views" option.

### Disabling Debugbar

While useful for debugging, [Laravel Debugbar](https://github.com/barryvdh/laravel-debugbar) can slow down your application, especially on complex pages, because it collects and renders a large amount of data on each request.

If you're noticing slowness, try disabling it by adding the following line to your `.env` file:

```dotenv
DEBUGBAR_ENABLED=false
```

If you still need Debugbar for development, consider disabling specific collectors you're not using.
Refer to the [Debugbar documentation](https://github.com/barryvdh/laravel-debugbar?tab=readme-ov-file#debugbar-for-laravel) for details.

### Disabling Xdebug

[Xdebug](https://xdebug.org) is a powerful tool for debugging, but it can significantly slow down performance. If you notice performance issues, check if `Xdebug` is installed and consider disabling it.

If `Xdebug` is installed but not disabled, it will still be enabled by default. If you have it installed, make sure it is explicitly disabled in your `php.ini` file:

```ini
xdebug.mode=off # Disable Xdebug
```

<Aside variant="tip">
    To locate your `php.ini` file, run: `php --ini`
</Aside>

## Caching Blade icons

Caching [Blade icons](https://blade-ui-kit.com/blade-icons) can help improve performance during local development, especially in views that render many icons.

To enable caching, run:

```bash
php artisan icons:cache
```

Make sure that when you install new Blade icon packages, you run the command again to discover the new icons.

---

<!-- Source: 03-pagination.md -->

## Introduction

The pagination component can be used in a Livewire Blade view only. It can render a list of paginated links:

```php
use App\Models\User;
use Illuminate\Contracts\View\View;
use Livewire\Component;

class ListUsers extends Component
{
    // ...
    
    public function render(): View
    {
        return view('livewire.list-users', [
            'users' => User::query()->paginate(10),
        ]);
    }
}
```

```blade
<x-filament::pagination :paginator="$users" />
```

Alternatively, you can use simple pagination or cursor pagination, which will just render a "previous" and "next" button:

```php
use App\Models\User;

User::query()->simplePaginate(10)
User::query()->cursorPaginate(10)
```

## Allowing the user to customize the number of items per page

You can allow the user to customize the number of items per page by passing an array of options to the `page-options` attribute. You must also define a Livewire property where the user's selection will be stored:

```php
use App\Models\User;
use Illuminate\Contracts\View\View;
use Livewire\Component;

class ListUsers extends Component
{
    public int | string $perPage = 10;
    
    // ...
    
    public function render(): View
    {
        return view('livewire.list-users', [
            'users' => User::query()->paginate($this->perPage),
        ]);
    }
}
```

```blade
<x-filament::pagination
    :paginator="$users"
    :page-options="[5, 10, 20, 50, 100, 'all']"
    :current-page-option-property="perPage"
/>
```

## Displaying links to the first and the last page

Extreme links are the first and last page links. You can add them by passing the `extreme-links` attribute to the component:

```blade
<x-filament::pagination
    :paginator="$users"
    extreme-links
/>
```

---

<!-- Source: 03-section.md -->

## Introduction

A section can be used to group content together, with an optional heading:

```blade
<x-filament::section>
    <x-slot name="heading">
        User details
    </x-slot>

    {{-- Content --}}
</x-filament::section>
```

## Adding a description to the section

You can add a description below the heading to the section by using the `description` slot:

```blade
<x-filament::section>
    <x-slot name="heading">
        User details
    </x-slot>

    <x-slot name="description">
        This is all the information we hold about the user.
    </x-slot>

    {{-- Content --}}
</x-filament::section>
```

## Adding an icon to the section header

You can add an [icon](../styling/icons) to a section by using the `icon` attribute:

```blade
<x-filament::section icon="heroicon-o-user">
    <x-slot name="heading">
        User details
    </x-slot>

    {{-- Content --}}
</x-filament::section>
```

### Changing the color of the section icon

By default, the color of the section icon is "gray". You can change it to be `danger`, `info`, `primary`, `success` or `warning` by using the `icon-color` attribute:

```blade
<x-filament::section
    icon="heroicon-o-user"
    icon-color="info"
>
    <x-slot name="heading">
        User details
    </x-slot>

    {{-- Content --}}
</x-filament::section>
```

### Changing the size of the section icon

By default, the size of the section icon is "large". You can change it to be "small" or "medium" by using the `icon-size` attribute:

```blade
<x-filament::section
    icon="heroicon-m-user"
    icon-size="sm"
>
    <x-slot name="heading">
        User details
    </x-slot>

    {{-- Content --}}
</x-filament::section>

<x-filament::section
    icon="heroicon-m-user"
    icon-size="md"
>
    <x-slot name="heading">
        User details
    </x-slot>

    {{-- Content --}}
</x-filament::section>
```

## Adding content to the end of the header

You may render additional content at the end of the header, next to the heading and description, using the `afterHeader` slot:

```blade
<x-filament::section>
    <x-slot name="heading">
        User details
    </x-slot>

    <x-slot name="afterHeader">
        {{-- Input to select the user's ID --}}
    </x-slot>

    {{-- Content --}}
</x-filament::section>
```

## Making a section collapsible

You can make the content of a section collapsible by using the `collapsible` attribute:

```blade
<x-filament::section collapsible>
    <x-slot name="heading">
        User details
    </x-slot>

    {{-- Content --}}
</x-filament::section>
```

### Making a section collapsed by default

You can make a section collapsed by default by using the `collapsed` attribute:

```blade
<x-filament::section
    collapsible
    collapsed
>
    <x-slot name="heading">
        User details
    </x-slot>

    {{-- Content --}}
</x-filament::section>
```

### Persisting collapsed sections

You can persist whether a section is collapsed in local storage using the `persist-collapsed` attribute, so it will remain collapsed when the user refreshes the page. You will also need a unique `id` attribute to identify the section from others, so that each section can persist its own collapse state:

```blade
<x-filament::section
    collapsible
    collapsed
    persist-collapsed
    id="user-details"
>
    <x-slot name="heading">
        User details
    </x-slot>

    {{-- Content --}}
</x-filament::section>
```

## Adding the section header aside the content instead of above it

You can change the position of the section header to be aside the content instead of above it by using the `aside` attribute:

```blade
<x-filament::section aside>
    <x-slot name="heading">
        User details
    </x-slot>

    {{-- Content --}}
</x-filament::section>
```

### Positioning the content before the header

You can change the position of the content to be before the header instead of after it by using the `content-before` attribute:

```blade
<x-filament::section
    aside
    content-before
>
    <x-slot name="heading">
        User details
    </x-slot>

    {{-- Content --}}
</x-filament::section>
```

---

<!-- Source: 03-select.md -->

## Introduction

The select component is a wrapper around the native `<select>` element. It provides a simple interface for selecting a single value from a list of options:

```blade
<x-filament::input.wrapper>
    <x-filament::input.select wire:model="status">
        <option value="draft">Draft</option>
        <option value="reviewing">Reviewing</option>
        <option value="published">Published</option>
    </x-filament::input.select>
</x-filament::input.wrapper>
```

To use the select component, you must wrap it in an "input wrapper" component, which provides a border and other elements such as a prefix or suffix. You can learn more about customizing the input wrapper component [here](input-wrapper).

---

<!-- Source: 03-tabs.md -->

## Introduction

The tabs component allows you to render a set of tabs, which can be used to toggle between multiple sections of content:

```blade
<x-filament::tabs label="Content tabs">
    <x-filament::tabs.item>
        Tab 1
    </x-filament::tabs.item>

    <x-filament::tabs.item>
        Tab 2
    </x-filament::tabs.item>

    <x-filament::tabs.item>
        Tab 3
    </x-filament::tabs.item>
</x-filament::tabs>
```

## Triggering the active state of the tab

By default, tabs do not appear "active". To make a tab appear active, you can use the `active` attribute:

```blade
<x-filament::tabs>
    <x-filament::tabs.item active>
        Tab 1
    </x-filament::tabs.item>

    {{-- Other tabs --}}
</x-filament::tabs>
```

You can also use the `active` attribute to make a tab appear active conditionally:

```blade
<x-filament::tabs>
    <x-filament::tabs.item
        :active="$activeTab === 'tab1'"
        wire:click="$set('activeTab', 'tab1')"
    >
        Tab 1
    </x-filament::tabs.item>

    {{-- Other tabs --}}
</x-filament::tabs>
```

Or you can use the `alpine-active` attribute to make a tab appear active conditionally using Alpine.js:

```blade
<x-filament::tabs x-data="{ activeTab: 'tab1' }">
    <x-filament::tabs.item
        alpine-active="activeTab === 'tab1'"
        x-on:click="activeTab = 'tab1'"
    >
        Tab 1
    </x-filament::tabs.item>

    {{-- Other tabs --}}
</x-filament::tabs>
```

## Setting a tab icon

Tabs may have an [icon](../styling/icons), which you can set using the `icon` attribute:

```blade
<x-filament::tabs>
    <x-filament::tabs.item icon="heroicon-m-bell">
        Notifications
    </x-filament::tabs.item>

    {{-- Other tabs --}}
</x-filament::tabs>
```

### Setting the tab icon position

The icon of the tab may be positioned before or after the label using the `icon-position` attribute:

```blade
<x-filament::tabs>
    <x-filament::tabs.item
        icon="heroicon-m-bell"
        icon-position="after"
    >
        Notifications
    </x-filament::tabs.item>

    {{-- Other tabs --}}
</x-filament::tabs>
```

## Setting a tab badge

Tabs may have a [badge](badge), which you can set using the `badge` slot:

```blade
<x-filament::tabs>
    <x-filament::tabs.item>
        Notifications

        <x-slot name="badge">
            5
        </x-slot>
    </x-filament::tabs.item>

    {{-- Other tabs --}}
</x-filament::tabs>
```

## Using a tab as an anchor link

By default, a tab's underlying HTML tag is `<button>`. You can change it to be an `<a>` tag by using the `tag` attribute:

```blade
<x-filament::tabs>
    <x-filament::tabs.item
        :href="route('notifications')"
        tag="a"
    >
        Notifications
    </x-filament::tabs.item>

    {{-- Other tabs --}}
</x-filament::tabs>
```

## Using vertical tabs

You can render the tabs vertically by using the `vertical` attribute:

```blade
<x-filament::tabs vertical>
    <x-filament::tabs.item>
        Tab 1
    </x-filament::tabs.item>

    <x-filament::tabs.item>
        Tab 2
    </x-filament::tabs.item>

    <x-filament::tabs.item>
        Tab 3
    </x-filament::tabs.item>
</x-filament::tabs>
```

---

<!-- Source: 03-tenancy.md -->

import Aside from "@components/Aside.astro"

## Introduction

Multi-tenancy is a concept where a single instance of an application serves multiple customers. Each customer has their own data and access rules that prevent them from viewing or modifying each other's data. This is a common pattern in SaaS applications. Users often belong to groups of users (often called teams or organizations). Records are owned by the group, and users can be members of multiple groups. This is suitable for applications where users need to collaborate on data.

Multi-tenancy is a very sensitive topic. It's important to understand the security implications of multi-tenancy and how to properly implement it. If implemented partially or incorrectly, data belonging to one tenant may be exposed to another tenant. Filament provides a set of tools to help you implement multi-tenancy in your application, but it is up to you to understand how to use them.

<Aside variant="warning">
    Filament does not provide any guarantees about the security of your application. It is your responsibility to ensure that your application is secure. Please see the [security](#tenancy-security) section for more information.
</Aside>

## Simple one-to-many tenancy

The term "multi-tenancy" is broad and may mean different things in different contexts. Filament's tenancy system implies that the user belongs to **many** tenants (*organizations, teams, companies, etc.*) and may switch between them.

If your case is simpler and you don't need a many-to-many relationship, then you don't need to set up the tenancy in Filament. You could use [observers](https://laravel.com/docs/eloquent#observers) and [global scopes](https://laravel.com/docs/eloquent#global-scopes) instead.

Let's say you have a database column `users.team_id`, you can scope all records to have the same `team_id` as the user using a [global scope](https://laravel.com/docs/eloquent#global-scopes):

```php
use Illuminate\Database\Eloquent\Builder;

class Post extends Model
{
    protected static function booted(): void
    {
        static::addGlobalScope('team', function (Builder $query) {
            if (auth()->hasUser()) {
                $query->where('team_id', auth()->user()->team_id);
                // or with a `team` relationship defined:
                $query->whereBelongsTo(auth()->user()->team);
            }
        });
    }
}
```

To automatically set the `team_id` on the record when it's created, you can create an [observer](https://laravel.com/docs/eloquent#observers):

```php
class PostObserver
{
    public function creating(Post $post): void
    {
        if (auth()->hasUser()) {
            $post->team_id = auth()->user()->team_id;
            // or with a `team` relationship defined:
            $post->team()->associate(auth()->user()->team);
        }
    }
}
```

## Setting up tenancy

To set up tenancy, you'll need to specify the "tenant" (like team or organization) model in the [configuration](../panel-configuration):

```php
use App\Models\Team;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenant(Team::class);
}
```

You'll also need to tell Filament which tenants a user belongs to. You can do this by implementing the `HasTenants` interface on the `App\Models\User` model:

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\FilamentUser;
use Filament\Models\Contracts\HasTenants;
use Filament\Panel;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Support\Collection;

class User extends Authenticatable implements FilamentUser, HasTenants
{
    // ...

    public function teams(): BelongsToMany
    {
        return $this->belongsToMany(Team::class);
    }

    public function getTenants(Panel $panel): Collection
    {
        return $this->teams;
    }

    public function canAccessTenant(Model $tenant): bool
    {
        return $this->teams()->whereKey($tenant)->exists();
    }
}
```

In this example, users belong to many teams, so there is a `teams()` relationship. The `getTenants()` method returns the teams that the user belongs to. Filament uses this to list the tenants that the user has access to.

For security, you also need to implement the `canAccessTenant()` method of the `HasTenants` interface to prevent users from accessing the data of other tenants by guessing their tenant ID and putting it into the URL.

You'll also want users to be able to [register new teams](#adding-a-tenant-registration-page).

## Adding a tenant registration page

A registration page will allow users to create a new tenant.

When visiting your app after logging in, users will be redirected to this page if they don't already have a tenant.

To set up a registration page, you'll need to create a new page class that extends `Filament\Pages\Tenancy\RegisterTenant`. This is a full-page Livewire component. You can put this anywhere you want, such as `app/Filament/Pages/Tenancy/RegisterTeam.php`:

```php
namespace App\Filament\Pages\Tenancy;

use App\Models\Team;
use Filament\Forms\Components\TextInput;
use Filament\Pages\Tenancy\RegisterTenant;
use Filament\Schemas\Schema;

class RegisterTeam extends RegisterTenant
{
    public static function getLabel(): string
    {
        return 'Register team';
    }

    public function form(Schema $schema): Schema
    {
        return $schema
            ->components([
                TextInput::make('name'),
                // ...
            ]);
    }

    protected function handleRegistration(array $data): Team
    {
        $team = Team::create($data);

        $team->members()->attach(auth()->user());

        return $team;
    }
}
```

You may add any [form components](../forms) to the `form()` method, and create the team inside the `handleRegistration()` method.

Now, we need to tell Filament to use this page. We can do this in the [configuration](../panel-configuration):

```php
use App\Filament\Pages\Tenancy\RegisterTeam;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenantRegistration(RegisterTeam::class);
}
```

### Customizing the tenant registration page

You can override any method you want on the base registration page class to make it act as you want. Even the `$view` property can be overridden to use a custom view of your choice.

## Adding a tenant profile page

A profile page will allow users to edit information about the tenant.

To set up a profile page, you'll need to create a new page class that extends `Filament\Pages\Tenancy\EditTenantProfile`. This is a full-page Livewire component. You can put this anywhere you want, such as `app/Filament/Pages/Tenancy/EditTeamProfile.php`:

```php
namespace App\Filament\Pages\Tenancy;

use Filament\Forms\Components\TextInput;
use Filament\Pages\Tenancy\EditTenantProfile;
use Filament\Schemas\Schema;

class EditTeamProfile extends EditTenantProfile
{
    public static function getLabel(): string
    {
        return 'Team profile';
    }

    public function form(Schema $schema): Schema
    {
        return $schema
            ->components([
                TextInput::make('name'),
                // ...
            ]);
    }
}
```

You may add any [form components](../forms) to the `form()` method. They will get saved directly to the tenant model.

Now, we need to tell Filament to use this page. We can do this in the [configuration](../panel-configuration):

```php
use App\Filament\Pages\Tenancy\EditTeamProfile;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenantProfile(EditTeamProfile::class);
}
```

### Customizing the tenant profile page

You can override any method you want on the base profile page class to make it act as you want. Even the `$view` property can be overridden to use a custom view of your choice.

## Accessing the current tenant

Anywhere in the app, you can access the tenant model for the current request using `Filament::getTenant()`:

```php
use Filament\Facades\Filament;

$tenant = Filament::getTenant();
```

## Billing

### Using Laravel Spark

Filament provides a billing integration with [Laravel Spark](https://spark.laravel.com). Your users can start subscriptions and manage their billing information.

To install the integration, first [install Spark](https://spark.laravel.com/docs/installation.html) and configure it for your tenant model.

Now, you can install the Filament billing provider for Spark using Composer:

```bash
composer require filament/spark-billing-provider
```

In the [configuration](../panel-configuration), set Spark as the `tenantBillingProvider()`:

```php
use Filament\Billing\Providers\SparkBillingProvider;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenantBillingProvider(new SparkBillingProvider());
}
```

Now, you're all good to go! Users can manage their billing by clicking a link in the tenant menu.

### Requiring a subscription

To require a subscription to use any part of the app, you can use the `requiresTenantSubscription()` configuration method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->requiresTenantSubscription();
}
```

Now, users will be redirected to the billing page if they don't have an active subscription.

#### Requiring a subscription for specific resources and pages

Sometimes, you may wish to only require a subscription for certain [resources](resources) and [custom pages](../navigation/custom-pages) in your app. You can do this by returning `true` from the `isTenantSubscriptionRequired()` method on the resource or page class:

```php
public static function isTenantSubscriptionRequired(Panel $panel): bool
{
    return true;
}
```

If you're using the `requiresTenantSubscription()` configuration method, then you can return `false` from this method to allow access to the resource or page as an exception.

### Writing a custom billing integration

Billing integrations are quite simple to write. You just need a class that implements the `Filament\Billing\Providers\Contracts\Provider` interface. This interface has two methods.

`getRouteAction()` is used to get the route action that should be run when the user visits the billing page. This could be a callback function, or the name of a controller, or a Livewire component - anything that works when using `Route::get()` in Laravel normally. For example, you could put in a simple redirect to your own billing page using a callback function.

`getSubscribedMiddleware()` returns the name of a middleware that should be used to check if the tenant has an active subscription. This middleware should redirect the user to the billing page if they don't have an active subscription.

Here's an example billing provider that uses a callback function for the route action and a middleware for the subscribed middleware:

```php
use App\Http\Middleware\RedirectIfUserNotSubscribed;
use Filament\Billing\Providers\Contracts\BillingProvider;
use Illuminate\Http\RedirectResponse;

class ExampleBillingProvider implements BillingProvider
{
    public function getRouteAction(): string
    {
        return function (): RedirectResponse {
            return redirect('https://billing.example.com');
        };
    }

    public function getSubscribedMiddleware(): string
    {
        return RedirectIfUserNotSubscribed::class;
    }
}
```

### Customizing the billing route slug

You can customize the URL slug used for the billing route using the `tenantBillingRouteSlug()` method in the [configuration](../panel-configuration):

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenantBillingRouteSlug('billing');
}
```

## Customizing the tenant menu

The tenant-switching menu is featured in the admin layout. It's fully customizable.

Each menu item is represented by an [action](../actions), and can be customized in the same way. To register new items, you can pass the actions to the `tenantMenuItems()` method of the [configuration](../panel-configuration):

```php
use App\Filament\Pages\Settings;
use Filament\Actions\Action;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenantMenuItems([
            Action::make('settings')
                ->url(fn (): string => Settings::getUrl())
                ->icon('heroicon-m-cog-8-tooth'),
            // ...
        ]);
}
```

### Customizing the registration link

To customize the [registration](#adding-a-tenant-registration-page) link in the tenant menu, register a new item with the `register` array key, and pass a function that [customizes the action](../actions) object:

```php
use Filament\Actions\Action;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenantMenuItems([
            'register' => fn (Action $action) => $action->label('Register new team'),
            // ...
        ]);
}
```

### Customizing the profile link

To customize the user profile link at the start of the tenant menu, register a new item with the `profile` array key, and pass a function that [customizes the action](../actions) object:

```php
use Filament\Actions\Action;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenantMenuItems([
            'profile' => fn (Action $action) => $action->label('Edit team profile'),
            // ...
        ]);
}
```

### Customizing the billing link

To customize the billing link in the tenant menu, register a new item with the `profile` array key, and pass a function that [customizes the action](../actions) object:

```php
use Filament\Actions\Action
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenantMenuItems([
            'billing' => fn (Action $action) => $action->label('Manage subscription'),
            // ...
        ]);
}
```

### Conditionally hiding tenant menu items

You can also conditionally hide a tenant menu item by using the `visible()` or `hidden()` methods, passing in a condition to check. Passing a function will defer condition evaluation until the menu is actually being rendered:

```php
use Filament\Actions\Action;

Action::make('settings')
    ->visible(fn (): bool => auth()->user()->can('manage-team'))
    // or
    ->hidden(fn (): bool => ! auth()->user()->can('manage-team'))
```

### Sending a `POST` HTTP request from a tenant menu item

You can send a `POST` HTTP request from a tenant menu item by passing a URL to the `url()` method, and also using `postToUrl()`:

```php
use Filament\Actions\Action;

Action::make('lockSession')
    ->url(fn (): string => route('lock-session'))
    ->postToUrl()
```

### Hiding the tenant menu

You can hide the tenant menu by using the `tenantMenu(false)`

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenantMenu(false);
}
```

However, this is a sign that Filament's tenancy feature is not suitable for your project. If each user only belongs to one tenant, you should stick to [simple one-to-many tenancy](#simple-one-to-many-tenancy).

## Setting up avatars

Out of the box, Filament uses [ui-avatars.com](https://ui-avatars.com) to generate avatars based on a user's name. However, if you user model has an `avatar_url` attribute, that will be used instead. To customize how Filament gets a user's avatar URL, you can implement the `HasAvatar` contract:

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\FilamentUser;
use Filament\Models\Contracts\HasAvatar;
use Illuminate\Database\Eloquent\Model;

class Team extends Model implements HasAvatar
{
    // ...

    public function getFilamentAvatarUrl(): ?string
    {
        return $this->avatar_url;
    }
}
```

The `getFilamentAvatarUrl()` method is used to retrieve the avatar of the current user. If `null` is returned from this method, Filament will fall back to [ui-avatars.com](https://ui-avatars.com).

You can easily swap out [ui-avatars.com](https://ui-avatars.com) for a different service, by creating a new avatar provider. [You can learn how to do this here.](overview#using-a-different-avatar-provider)

## Configuring the tenant relationships

When creating and listing records associated with a Tenant, Filament needs access to two Eloquent relationships for each resource - an "ownership" relationship that is defined on the resource model class, and a relationship on the tenant model class. By default, Filament will attempt to guess the names of these relationships based on standard Laravel conventions. For example, if the tenant model is `App\Models\Team`, it will look for a `team()` relationship on the resource model class. And if the resource model class is `App\Models\Post`, it will look for a `posts()` relationship on the tenant model class.

### Customizing the ownership relationship name

You can customize the name of the ownership relationship used across all resources at once, using the `ownershipRelationship` argument on the `tenant()` configuration method. In this example, resource model classes have an `owner` relationship defined:

```php
use App\Models\Team;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenant(Team::class, ownershipRelationship: 'owner');
}
```

Alternatively, you can set the `$tenantOwnershipRelationshipName` static property on the resource class, which can then be used to customize the ownership relationship name that is just used for that resource. In this example, the `Post` model class has an `owner` relationship defined:

```php
use Filament\Resources\Resource;

class PostResource extends Resource
{
    protected static ?string $tenantOwnershipRelationshipName = 'owner';

    // ...
}
```

### Customizing the resource relationship name

You can set the `$tenantRelationshipName` static property on the resource class, which can then be used to customize the relationship name that is used to fetch that resource. In this example, the tenant model class has an `blogPosts` relationship defined:

```php
use Filament\Resources\Resource;

class PostResource extends Resource
{
    protected static ?string $tenantRelationshipName = 'blogPosts';

    // ...
}
```

## Configuring the slug attribute

When using a tenant like a team, you might want to add a slug field to the URL rather than the team's ID. You can do that with the `slugAttribute` argument on the `tenant()` configuration method:

```php
use App\Models\Team;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenant(Team::class, slugAttribute: 'slug');
}
```

## Configuring the name attribute

By default, Filament will use the `name` attribute of the tenant to display its name in the app. To change this, you can implement the `HasName` contract:

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\HasName;
use Illuminate\Database\Eloquent\Model;

class Team extends Model implements HasName
{
    // ...

    public function getFilamentName(): string
    {
        return "{$this->name} {$this->subscription_plan}";
    }
}
```

The `getFilamentName()` method is used to retrieve the name of the current user.

## Setting the current tenant label

Inside the tenant switcher, you may wish to add a small label like "Active team" above the name of the current team. You can do this by implementing the `HasCurrentTenantLabel` method on the tenant model:

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\HasCurrentTenantLabel;
use Illuminate\Database\Eloquent\Model;

class Team extends Model implements HasCurrentTenantLabel
{
    // ...

    public function getCurrentTenantLabel(): string
    {
        return 'Active team';
    }
}
```

## Setting the default tenant

When signing in, Filament will redirect the user to the first tenant returned from the `getTenants()` method.

Sometimes, you might wish to change this. For example, you might store which team was last active, and redirect the user to that team instead.

To customize this, you can implement the `HasDefaultTenant` contract on the user:

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\FilamentUser;
use Filament\Models\Contracts\HasDefaultTenant;
use Filament\Models\Contracts\HasTenants;
use Filament\Panel;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class User extends Model implements FilamentUser, HasDefaultTenant, HasTenants
{
    // ...

    public function getDefaultTenant(Panel $panel): ?Model
    {
        return $this->latestTeam;
    }

    public function latestTeam(): BelongsTo
    {
        return $this->belongsTo(Team::class, 'latest_team_id');
    }
}
```

## Applying middleware to tenant-aware routes

You can apply extra middleware to all tenant-aware routes by passing an array of middleware classes to the `tenantMiddleware()` method in the [panel configuration file](../panel-configuration):

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenantMiddleware([
            // ...
        ]);
}
```

By default, middleware will be run when the page is first loaded, but not on subsequent Livewire AJAX requests. If you want to run middleware on every request, you can make it persistent by passing `true` as the second argument to the `tenantMiddleware()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenantMiddleware([
            // ...
        ], isPersistent: true);
}
```

## Adding a tenant route prefix

By default the URL structure will put the tenant ID or slug immediately after the panel path. If you wish to prefix it with another URL segment, use the `tenantRoutePrefix()` method:

```php
use App\Models\Team;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->path('admin')
        ->tenant(Team::class)
        ->tenantRoutePrefix('team');
}
```

Before, the URL structure was `/admin/1` for tenant 1. Now, it is `/admin/team/1`.

## Using a domain to identify the tenant

When using a tenant, you might want to use domain or subdomain routing like `team1.example.com/posts` instead of a route prefix like `/team1/posts` . You can do that with the `tenantDomain()` method, alongside the `tenant()` configuration method. The `tenant` argument corresponds to the slug attribute of the tenant model:

```php
use App\Models\Team;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenant(Team::class, slugAttribute: 'slug')
        ->tenantDomain('{tenant:slug}.example.com');
}
```

In the above examples, the tenants live on subdomains of the main app domain. You may also set the system up to resolve the entire domain from the tenant as well:

```php
use App\Models\Team;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenant(Team::class, slugAttribute: 'domain')
        ->tenantDomain('{tenant:domain}');
}
```

In this example, the `domain` attribute should contain a valid domain host, like `example.com` or `subdomain.example.com`.

<Aside variant="info">
    When using a parameter for the entire domain (`tenantDomain('{tenant:domain}')`), Filament will register a [global route parameter pattern](https://laravel.com/docs/routing#parameters-global-constraints) for all `tenant` parameters in the application to be `[a-z0-9.\-]+`. This is because Laravel does not allow the `.` character in route parameters by default. This might conflict with other panels using tenancy, or other parts of your application that use a `tenant` route parameter.
</Aside>

## Disabling tenancy for a resource

By default, all resources within a panel with tenancy will be scoped to the current tenant. If you have resources that are shared between tenants, you can disable tenancy for them by setting the `$isScopedToTenant` static property to `false` on the resource class:

```php
protected static bool $isScopedToTenant = false;
```

### Disabling tenancy for all resources

If you wish to opt-in to tenancy for each resource instead of opting-out, you can call `Resource::scopeToTenant(false)` inside a service provider's `boot()` method or a middleware:

```php
use Filament\Resources\Resource;

Resource::scopeToTenant(false);
```

Now, you can opt-in to tenancy for each resource by setting the `$isScopedToTenant` static property to `true` on a resource class:

```php
protected static bool $isScopedToTenant = true;
```

## Tenancy security

It's important to understand the security implications of multi-tenancy and how to properly implement it. If implemented partially or incorrectly, data belonging to one tenant may be exposed to another tenant. Filament provides a set of tools to help you implement multi-tenancy in your application, but it is up to you to understand how to use them. Filament does not provide any guarantees about the security of your application. It is your responsibility to ensure that your application is secure.

Below is a list of features that Filament provides to help you implement multi-tenancy in your application:

- Automatic global scoping of Eloquent model queries for [tenant-aware](#disabling-tenancy-for-a-resource) resources that belong to the panel with tenancy enabled. The query used to fetch records for a resource is automatically scoped to the current tenant. This query is used to render the resource's list table, and is also used to resolve records from the current URL when editing or viewing a record. This means that if a user attempts to view a record that does not belong to the current tenant, they will receive a 404 error.
    - A [tenant-aware](#disabling-tenancy-for-a-resource) resource has to exist in the panel with tenancy enabled for the resource's model to have the global scope applied. If you want to scope the queries for a model that does not have a corresponding resource, you must [use middleware to apply additional global scopes](#using-tenant-aware-middleware-to-apply-additional-global-scopes) for that model.
    - The global scopes are applied after the tenant has been identified from the request. This happens during the middleware stack of panel requests. If you make a query before the tenant has been identified, such as from early middleware in the stack or in a service provider, the query will not be scoped to the current tenant. To guarantee that middleware runs after the current tenant is identified, you should register it as [tenant middleware](#applying-middleware-to-tenant-aware-routes).
    - As per the point above, queries made outside the panel with tenancy enabled do not have access to the current tenant, so are not scoped. If in doubt, please check if your queries are properly scoped or not before deploying your application.
    - If you need to disable the tenancy global scope for a specific query, you can use the `withoutGlobalScope(filament()->getTenancyScopeName())` method on the query.
    - If any of your queries disable all global scopes, the tenancy global scope will be disabled as well. You should be careful when using this method, as it can lead to data leakage. If you need to disable all global scopes except the tenancy global scope, you can use the `withoutGlobalScopes()` method passing an array of the global scopes you want to disable.

- Automatic association of newly created Eloquent models with the current tenant. When a new record is created for a [tenant-aware](#disabling-tenancy-for-a-resource) resource, the tenant is automatically associated with the record. This means that the record will belong to the current tenant, as the foreign key column is automatically set to the tenant's ID. This is done by Filament registering an event listener for the `creating` and `created` events on the resource's Eloquent model. If you need to disable this feature for a specific model, you can use the `withoutTenancy()` method on the model.
    - A [tenant-aware](#disabling-tenancy-for-a-resource) resource has to exist in the panel with tenancy enabled for the resource's model to have the automatic association to happen. If you want automatic association for a model that does not have a corresponding resource, you must [register a listener for the `creating` event](https://laravel.com/docs/eloquent#events) for that model, and associate the `filament()->getTenant()` with it.
    - The events run after the tenant has been identified from the request. This happens during the middleware stack of panel requests. If you create a model before the tenant has been identified, such as from early middleware in the stack or in a service provider, it will not be associated with the current tenant. To guarantee that middleware runs after the current tenant is identified, you should register it as [tenant middleware](#applying-middleware-to-tenant-aware-routes).
    - As per the point above, models created outside the panel with tenancy enabled do not have access to the current tenant, so are not associated. If in doubt, please check if your models are properly associated or not before deploying your application.
    - If you need to disable the automatic association for a particular model, you can [mute the events](https://laravel.com/docs/eloquent#muting-events) temporarily while you create it. If any of your code currently does this or removes event listeners permanently, you should check this is not affecting the tenancy feature.

### `unique` and `exists` validation

Laravel's `unique` and `exists` validation rules do not use Eloquent models to query the database by default, so it will not use any global scopes defined on the model, including for multi-tenancy. As such, even if there is a soft-deleted record with the same value in a different tenant, the validation will fail.

If you would like two tenants to have complete data separation, you should use the `scopedUnique()` or `scopedExists()` methods instead, which replace Laravel's `unique` and `exists` implementations with ones that uses the model to query the database, applying any global scopes defined on the model, including for multi-tenancy:

```php
use Filament\Forms\Components\TextInput;

TextInput::make('email')
    ->scopedUnique()
    // or
    ->scopedExists()
```

For more information, see the [validation documentation](../forms/validation) for [`unique()`](../forms/validation#unique) and [`exists()`](../forms/validation#exists).

### Using tenant-aware middleware to apply additional global scopes

Since only models with resources that exist in the panel are automatically scoped to the current tenant, it might be useful to apply additional tenant scoping to other Eloquent models while they are being used in your panel. This would allow you to forget about scoping your queries to the current tenant, and instead have the scoping applied automatically. To do this, you can create a new middleware class like `ApplyTenantScopes`:

```bash
php artisan make:middleware ApplyTenantScopes
```

Inside the `handle()` method, you can apply any global scopes that you wish:

```php
use App\Models\Author;
use Closure;
use Filament\Facades\Filament;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Http\Request;

class ApplyTenantScopes
{
    public function handle(Request $request, Closure $next)
    {
        Author::addGlobalScope(
            'tenant',
            fn (Builder $query) => $query->whereBelongsTo(Filament::getTenant()),
        );

        return $next($request);
    }
}
```

You can now [register this middleware](#applying-middleware-to-tenant-aware-routes) for all tenant-aware routes, and ensure that it is used across all Livewire AJAX requests by making it persistent:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenantMiddleware([
            ApplyTenantScopes::class,
        ], isPersistent: true);
}
```

---

<!-- Source: 03-testing-tables.md -->

## Testing that a table can render

To ensure a table component renders, use the `assertSuccessful()` Livewire helper:

```php
use function Pest\Livewire\livewire;

it('can render page', function () {
    livewire(ListPosts::class)
        ->assertSuccessful();
});
```

To test which records are shown, you can use `assertCanSeeTableRecords()`, `assertCanNotSeeTableRecords()` and `assertCountTableRecords()`:

```php
use function Pest\Livewire\livewire;

it('cannot display trashed posts by default', function () {
    $posts = Post::factory()->count(4)->create();
    $trashedPosts = Post::factory()->trashed()->count(6)->create();

    livewire(PostResource\Pages\ListPosts::class)
        ->assertCanSeeTableRecords($posts)
        ->assertCanNotSeeTableRecords($trashedPosts)
        ->assertCountTableRecords(4);
});
```

> If your table uses pagination, `assertCanSeeTableRecords()` will only check for records on the first page. To switch page, call `call('gotoPage', 2)`.

> If your table uses `deferLoading()`, you should call `loadTable()` before `assertCanSeeTableRecords()`.

## Testing columns

To ensure that a certain column is rendered, pass the column name to `assertCanRenderTableColumn()`:

```php
use function Pest\Livewire\livewire;

it('can render post titles', function () {
    Post::factory()->count(10)->create();

    livewire(PostResource\Pages\ListPosts::class)
        ->assertCanRenderTableColumn('title');
});
```

This helper will get the HTML for this column, and check that it is present in the table.

For testing that a column is not rendered, you can use `assertCanNotRenderTableColumn()`:

```php
use function Pest\Livewire\livewire;

it('can not render post comments', function () {
    Post::factory()->count(10)->create()

    livewire(PostResource\Pages\ListPosts::class)
        ->assertCanNotRenderTableColumn('comments');
});
```

This helper will assert that the HTML for this column is not shown by default in the present table.

### Testing that a column can be searched

To search the table, call the `searchTable()` method with your search query.

You can then use `assertCanSeeTableRecords()` to check your filtered table records, and use `assertCanNotSeeTableRecords()` to assert that some records are no longer in the table:

```php
use function Pest\Livewire\livewire;

it('can search posts by title', function () {
    $posts = Post::factory()->count(10)->create();

    $title = $posts->first()->title;

    livewire(PostResource\Pages\ListPosts::class)
        ->searchTable($title)
        ->assertCanSeeTableRecords($posts->where('title', $title))
        ->assertCanNotSeeTableRecords($posts->where('title', '!=', $title));
});
```

To search individual columns, you can pass an array of searches to `searchTableColumns()`:

```php
use function Pest\Livewire\livewire;

it('can search posts by title column', function () {
    $posts = Post::factory()->count(10)->create();

    $title = $posts->first()->title;

    livewire(PostResource\Pages\ListPosts::class)
        ->searchTableColumns(['title' => $title])
        ->assertCanSeeTableRecords($posts->where('title', $title))
        ->assertCanNotSeeTableRecords($posts->where('title', '!=', $title));
});
```

### Testing that a column can be sorted

To sort table records, you can call `sortTable()`, passing the name of the column to sort by. You can use `'desc'` in the second parameter of `sortTable()` to reverse the sorting direction.

Once the table is sorted, you can ensure that the table records are rendered in order using `assertCanSeeTableRecords()` with the `inOrder` parameter:

```php
use function Pest\Livewire\livewire;

it('can sort posts by title', function () {
    $posts = Post::factory()->count(10)->create();

    livewire(PostResource\Pages\ListPosts::class)
        ->sortTable('title')
        ->assertCanSeeTableRecords($posts->sortBy('title'), inOrder: true)
        ->sortTable('title', 'desc')
        ->assertCanSeeTableRecords($posts->sortByDesc('title'), inOrder: true);
});
```

### Testing the state of a column

To assert that a certain column has a state or does not have a state for a record you can use `assertTableColumnStateSet()` and `assertTableColumnStateNotSet()`:

```php
use function Pest\Livewire\livewire;

it('can get post author names', function () {
    $posts = Post::factory()->count(10)->create();

    $post = $posts->first();

    livewire(PostResource\Pages\ListPosts::class)
        ->assertTableColumnStateSet('author.name', $post->author->name, record: $post)
        ->assertTableColumnStateNotSet('author.name', 'Anonymous', record: $post);
});
```

To assert that a certain column has a formatted state or does not have a formatted state for a record you can use `assertTableColumnFormattedStateSet()` and `assertTableColumnFormattedStateNotSet()`:

```php
use function Pest\Livewire\livewire;

it('can get post author names', function () {
    $post = Post::factory(['name' => 'John Smith'])->create();

    livewire(PostResource\Pages\ListPosts::class)
        ->assertTableColumnFormattedStateSet('author.name', 'Smith, John', record: $post)
        ->assertTableColumnFormattedStateNotSet('author.name', $post->author->name, record: $post);
});
```

### Testing the existence of a column

To ensure that a column exists, you can use the `assertTableColumnExists()` method:

```php
use function Pest\Livewire\livewire;

it('has an author column', function () {
    livewire(PostResource\Pages\ListPosts::class)
        ->assertTableColumnExists('author');
});
```

You may pass a function as an additional argument to assert that a column passes a given "truth test". This is useful for asserting that a column has a specific configuration. You can also pass in a record as the third parameter, which is useful if your check is dependant on which table row is being rendered:

```php
use function Pest\Livewire\livewire;
use Filament\Tables\Columns\TextColumn;

it('has an author column', function () {
    $post = Post::factory()->create();
    
    livewire(PostResource\Pages\ListPosts::class)
        ->assertTableColumnExists('author', function (TextColumn $column): bool {
            return $column->getDescriptionBelow() === $post->subtitle;
        }, $post);
});
```

### Testing the visibility of a column

To ensure that a particular user cannot see a column, you can use the `assertTableColumnVisible()` and `assertTableColumnHidden()` methods:

```php
use function Pest\Livewire\livewire;

it('shows the correct columns', function () {
    livewire(PostResource\Pages\ListPosts::class)
        ->assertTableColumnVisible('created_at')
        ->assertTableColumnHidden('author');
});
```

### Testing the description of a column

To ensure a column has the correct description above or below you can use the `assertTableColumnHasDescription()` and `assertTableColumnDoesNotHaveDescription()` methods:

```php
use function Pest\Livewire\livewire;

it('has the correct descriptions above and below author', function () {
    $post = Post::factory()->create();

    livewire(PostsTable::class)
        ->assertTableColumnHasDescription('author', 'Author! ↓↓↓', $post, 'above')
        ->assertTableColumnHasDescription('author', 'Author! ↑↑↑', $post)
        ->assertTableColumnDoesNotHaveDescription('author', 'Author! ↑↑↑', $post, 'above')
        ->assertTableColumnDoesNotHaveDescription('author', 'Author! ↓↓↓', $post);
});
```

### Testing the extra attributes of a column

To ensure that a column has the correct extra attributes, you can use the `assertTableColumnHasExtraAttributes()` and `assertTableColumnDoesNotHaveExtraAttributes()` methods:

```php
use function Pest\Livewire\livewire;

it('displays author in red', function () {
    $post = Post::factory()->create();

    livewire(PostsTable::class)
        ->assertTableColumnHasExtraAttributes('author', ['class' => 'text-danger-500'], $post)
        ->assertTableColumnDoesNotHaveExtraAttributes('author', ['class' => 'text-primary-500'], $post);
});
```

### Testing the options in a `SelectColumn`

If you have a select column, you can ensure it has the correct options with `assertTableSelectColumnHasOptions()` and `assertTableSelectColumnDoesNotHaveOptions()`:

```php
use function Pest\Livewire\livewire;

it('has the correct statuses', function () {
    $post = Post::factory()->create();

    livewire(PostsTable::class)
        ->assertTableSelectColumnHasOptions('status', ['unpublished' => 'Unpublished', 'published' => 'Published'], $post)
        ->assertTableSelectColumnDoesNotHaveOptions('status', ['archived' => 'Archived'], $post);
});
```

## Testing filters

To filter the table records, you can use the `filterTable()` method, along with `assertCanSeeTableRecords()` and `assertCanNotSeeTableRecords()`:

```php
use function Pest\Livewire\livewire;

it('can filter posts by `is_published`', function () {
    $posts = Post::factory()->count(10)->create();

    livewire(PostResource\Pages\ListPosts::class)
        ->assertCanSeeTableRecords($posts)
        ->filterTable('is_published')
        ->assertCanSeeTableRecords($posts->where('is_published', true))
        ->assertCanNotSeeTableRecords($posts->where('is_published', false));
});
```

For a simple filter, this will just enable the filter.

If you'd like to set the value of a `SelectFilter` or `TernaryFilter`, pass the value as a second argument:

```php
use function Pest\Livewire\livewire;

it('can filter posts by `author_id`', function () {
    $posts = Post::factory()->count(10)->create();

    $authorId = $posts->first()->author_id;

    livewire(PostResource\Pages\ListPosts::class)
        ->assertCanSeeTableRecords($posts)
        ->filterTable('author_id', $authorId)
        ->assertCanSeeTableRecords($posts->where('author_id', $authorId))
        ->assertCanNotSeeTableRecords($posts->where('author_id', '!=', $authorId));
});
```

### Resetting filters in a test

To reset all filters to their original state, call `resetTableFilters()`:

```php
use function Pest\Livewire\livewire;

it('can reset table filters', function () {
    $posts = Post::factory()->count(10)->create();

    livewire(PostResource\Pages\ListPosts::class)
        ->resetTableFilters();
});
```

### Removing filters in a test

To remove a single filter you can use `removeTableFilter()`:

```php
use function Pest\Livewire\livewire;

it('filters list by published', function () {
    $posts = Post::factory()->count(10)->create();

    $unpublishedPosts = $posts->where('is_published', false)->get();

    livewire(PostsTable::class)
        ->filterTable('is_published')
        ->assertCanNotSeeTableRecords($unpublishedPosts)
        ->removeTableFilter('is_published')
        ->assertCanSeeTableRecords($posts);
});
```

To remove all filters you can use `removeTableFilters()`:

```php
use function Pest\Livewire\livewire;

it('can remove all table filters', function () {
    $posts = Post::factory()->count(10)->forAuthor()->create();

    $unpublishedPosts = $posts
        ->where('is_published', false)
        ->where('author_id', $posts->first()->author->getKey());

    livewire(PostsTable::class)
        ->filterTable('is_published')
        ->filterTable('author', $author)
        ->assertCanNotSeeTableRecords($unpublishedPosts)
        ->removeTableFilters()
        ->assertCanSeeTableRecords($posts);
});
```

### Testing the visibility of a filter

To ensure that a particular user cannot see a filter, you can use the `assertTableFilterVisible()` and `assertTableFilterHidden()` methods:

```php
use function Pest\Livewire\livewire;

it('shows the correct filters', function () {
    livewire(PostsTable::class)
        ->assertTableFilterVisible('created_at')
        ->assertTableFilterHidden('author');
```

### Testing the existence of a filter

To ensure that a filter exists, you can use the `assertTableFilterExists()` method:

```php
use function Pest\Livewire\livewire;

it('has an author filter', function () {
    livewire(PostResource\Pages\ListPosts::class)
        ->assertTableFilterExists('author');
});
```

You may pass a function as an additional argument to assert that a filter passes a given "truth test". This is useful for asserting that a filter has a specific configuration:

```php
use function Pest\Livewire\livewire;
use Filament\Tables\Filters\SelectFilter;

it('has an author filter', function () {    
    livewire(PostResource\Pages\ListPosts::class)
        ->assertTableFilterExists('author', function (SelectFilter $column): bool {
            return $column->getLabel() === 'Select author';
        });
});
```

## Testing summaries

To test that a summary calculation is working, you may use the `assertTableColumnSummarySet()` method:

```php
use function Pest\Livewire\livewire;

it('can average values in a column', function () {
    $posts = Post::factory()->count(10)->create();

    livewire(PostResource\Pages\ListPosts::class)
        ->assertCanSeeTableRecords($posts)
        ->assertTableColumnSummarySet('rating', 'average', $posts->avg('rating'));
});
```

The first argument is the column name, the second is the summarizer ID, and the third is the expected value.

Note that the expected and actual values are normalized, such that `123.12` is considered the same as `"123.12"`, and `['Fred', 'Jim']` is the same as `['Jim', 'Fred']`.

You may set a summarizer ID by passing it to the `make()` method:

```php
use Filament\Tables\Columns\Summarizers\Average;
use Filament\Tables\Columns\TextColumn;

TextColumn::make('rating')
    ->summarize(Average::make('average'))
```

The ID should be unique between summarizers in that column.

### Testing summaries on only one pagination page

To calculate the average for only one pagination page, use the `isCurrentPaginationPageOnly` argument:

```php
use function Pest\Livewire\livewire;

it('can average values in a column', function () {
    $posts = Post::factory()->count(20)->create();

    livewire(PostResource\Pages\ListPosts::class)
        ->assertCanSeeTableRecords($posts->take(10))
        ->assertTableColumnSummarySet('rating', 'average', $posts->take(10)->avg('rating'), isCurrentPaginationPageOnly: true);
});
```

### Testing a range summarizer

To test a range, pass the minimum and maximum value into a tuple-style `[$minimum, $maximum]` array:

```php
use function Pest\Livewire\livewire;

it('can average values in a column', function () {
    $posts = Post::factory()->count(10)->create();

    livewire(PostResource\Pages\ListPosts::class)
        ->assertCanSeeTableRecords($posts)
        ->assertTableColumnSummarySet('rating', 'range', [$posts->min('rating'), $posts->max('rating')]);
});
```

---

<!-- Source: 03-user-menu.md -->

import AutoScreenshot from "@components/AutoScreenshot.astro"

## Introduction

The user menu is featured in the top right corner of the admin layout. It's fully customizable.

Each menu item is represented by an [action](../actions), and can be customized in the same way. To register new items, you can pass the actions to the `userMenuItems()` method of the [configuration](../panel-configuration):

```php
use App\Filament\Pages\Settings;
use Filament\Actions\Action;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->userMenuItems([
            Action::make('settings')
                ->url(fn (): string => Settings::getUrl())
                ->icon('heroicon-o-cog-6-tooth'),
            // ...
        ]);
}
```

<AutoScreenshot name="panels/navigation/user-menu" alt="User menu with custom menu item" version="3.x" />

## Customizing the profile link

To customize the user profile link at the start of the user menu, register a new item with the `profile` array key, and pass a function that [customizes the action](../actions) object:

```php
use Filament\Actions\Action;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->userMenuItems([
            'profile' => fn (Action $action) => $action->label('Edit profile'),
            // ...
        ]);
}
```

For more information on creating a profile page, check out the [authentication features documentation](../users#authentication-features).

## Customizing the logout link

To customize the user logout link at the end of the user menu, register a new item with the `logout` array key, and pass a function that [customizes the action](../actions) object:

```php
use Filament\Actions\Action;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->userMenuItems([
            'logout' => fn (Action $action) => $action->label('Log out'),
            // ...
        ]);
}
```

## Conditionally hiding user menu items

You can also conditionally hide a user menu item by using the `visible()` or `hidden()` methods, passing in a condition to check. Passing a function will defer condition evaluation until the menu is actually being rendered:

```php
use App\Models\Payment;
use Filament\Actions\Action;

Action::make('payments')
    ->visible(fn (): bool => auth()->user()->can('viewAny', Payment::class))
    // or
    ->hidden(fn (): bool => ! auth()->user()->can('viewAny', Payment::class))
```

## Sending a `POST` HTTP request from a user menu item

You can send a `POST` HTTP request from a user menu item by passing a URL to the `url()` method, and also using `postToUrl()`:

```php
use Filament\Actions\Action;

Action::make('lockSession')
    ->url(fn (): string => route('lock-session'))
    ->postToUrl()
```

## Disabling the user menu

You may disable the user menu entirely by passing `false` to the `userMenu()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->userMenu(false);
}
```

---

<!-- Source: 04-building-a-standalone-plugin.md -->

## Preface

Please read the docs on [panel plugin development](../panels/plugins/) and the [getting started guide](getting-started) before continuing.

## Introduction

In this walkthrough, we'll build a simple plugin that adds a new form component that can be used in forms. This also means it will be available to users in their panels.

You can find the final code for this plugin at [https://github.com/awcodes/headings](https://github.com/awcodes/headings).

## Step 1: Create the plugin

First, we'll create the plugin using the steps outlined in the [getting started guide](getting-started#creating-a-plugin).

## Step 2: Clean up

Next, we'll clean up the plugin to remove the boilerplate code we don't need. This will seem like a lot, but since this is a simple plugin, we can remove a lot of the boilerplate code.

Remove the following directories and files:
1. `bin`
2. `config`
3. `database`
4. `src/Commands`
5. `src/Facades`
6. `stubs`

Now we can clean up our `composer.json` file to remove unneeded options.

```json
"autoload": {
    "psr-4": {
        // We can remove the database factories
        "Awcodes\\Headings\\Database\\Factories\\": "database/factories/"
    }
},
"extra": {
    "laravel": {
        // We can remove the facade
        "aliases": {
            "Headings": "Awcodes\\Headings\\Facades\\ClockWidget"
        }
    }
},
```

Normally, Filament recommends that users style their plugins with a custom filament theme, but for the sake of example let's provide our own stylesheet that can be loaded asynchronously using the new `x-load` features in Filament v3. So, let's update our `package.json` file to include `cssnano`, `postcss`, `postcss-cli` and `postcss-nesting` to build our stylesheet.

```json
{
    "private": true,
    "scripts": {
        "build": "postcss resources/css/index.css -o resources/dist/headings.css"
    },
    "devDependencies": {
        "cssnano": "^6.0.1",
        "postcss": "^8.4.27",
        "postcss-cli": "^10.1.0",
        "postcss-nesting": "^13.0.0"
    }
}
```

Then we need to install our dependencies.

```bash
npm install
```

We will also need to update our `postcss.config.js` file to configure postcss.

```js
module.exports = {
    plugins: [
        require('postcss-nesting')(),
        require('cssnano')({
            preset: 'default',
        }),
    ],
};
```

You may also remove the testing directories and files, but we'll leave them in for now, although we won't be using them for this example, and we highly recommend that you write tests for your plugins.

## Step 3: Setting up the provider

Now that we have our plugin cleaned up, we can start adding our code. The boilerplate in the `src/HeadingsServiceProvider.php` file has a lot going on so, let's delete everything and start from scratch.

We need to be able to register our stylesheet with the Filament Asset Manager so that we can load it on demand in our blade view. To do this, we'll need to add the following to the `packageBooted` method in our service provider.

***Note the `loadedOnRequest()` method. This is important, because it tells Filament to only load the stylesheet when it's needed.***

```php
namespace Awcodes\Headings;

use Filament\Support\Assets\Css;
use Filament\Support\Facades\FilamentAsset;
use Spatie\LaravelPackageTools\Package;
use Spatie\LaravelPackageTools\PackageServiceProvider;

class HeadingsServiceProvider extends PackageServiceProvider
{
    public static string $name = 'headings';

    public function configurePackage(Package $package): void
    {
        $package->name(static::$name)
            ->hasViews();
    }

    public function packageBooted(): void
    {
        FilamentAsset::register([
            Css::make('headings', __DIR__ . '/../resources/dist/headings.css')->loadedOnRequest(),
        ], 'awcodes/headings');
    }
}
```

## Step 4: Creating our component

Next, we'll need to create our component. Create a new file at `src/Heading.php` and add the following code.

```php
namespace Awcodes\Headings;

use Closure;
use Filament\Schemas\Components\Component;
use Filament\Support\Colors\Color;
use Filament\Support\Concerns\HasColor;

class Heading extends Component
{
    use HasColor;

    protected string | int $level = 2;

    protected string | Closure $content = '';

    protected string $view = 'headings::heading';

    final public function __construct(string | int $level)
    {
        $this->level($level);
    }

    public static function make(string | int $level): static
    {
        return app(static::class, ['level' => $level]);
    }

    protected function setUp(): void
    {
        parent::setUp();

        $this->dehydrated(false);
    }

    public function content(string | Closure $content): static
    {
        $this->content = $content;

        return $this;
    }

    public function level(string | int $level): static
    {
        $this->level = $level;

        return $this;
    }

    public function getColor(): array
    {
        return $this->evaluate($this->color) ?? Color::Amber;
    }

    public function getContent(): string
    {
        return $this->evaluate($this->content);
    }

    public function getLevel(): string
    {
        return is_int($this->level) ? 'h' . $this->level : $this->level;
    }
}
```

## Step 5: Rendering our component

Next, we'll need to create the view for our component. Create a new file at `resources/views/heading.blade.php` and add the following code.

We are using x-load to asynchronously load stylesheet, so it's only loaded when necessary. You can learn more about this in the [Core Concepts](../advanced/assets#lazy-loading-css) section of the docs.

```blade
@php
    $level = $getLevel();
    $color = $getColor();
@endphp

<{{ $level }}
    x-data
    x-load-css="[@js(\Filament\Support\Facades\FilamentAsset::getStyleHref('headings', package: 'awcodes/headings'))]"
    {{
        $attributes
            ->class([
                'headings-component',
                match ($color) {
                    'gray' => 'text-gray-600 dark:text-gray-400',
                    default => 'text-custom-500',
                },
            ])
            ->style([
                \Filament\Support\get_color_css_variables($color, [500]) => $color !== 'gray',
            ])
    }}
>
    {{ $getContent() }}
</{{ $level }}>
```

## Step 6: Adding some styles

Next, let's provide some custom styling for our field. We'll add the following to `resources/css/index.css`. And run `npm run build` to compile our CSS.

```css
.headings-component {
    &:is(h1, h2, h3, h4, h5, h6) {
         font-weight: 700;
         letter-spacing: -.025em;
         line-height: 1.1;
     }

    &h1 {
         font-size: 2rem;
     }

    &h2 {
         font-size: 1.75rem;
     }

    &h3 {
         font-size: 1.5rem;
     }

    &h4 {
         font-size: 1.25rem;
     }

    &h5,
    &h6 {
         font-size: 1rem;
     }
}
```

Then we need to build our stylesheet.

```bash
npm run build
```

## Step 7: Update your README

You'll want to update your `README.md` file to include instructions on how to install your plugin and any other information you want to share with users, Like how to use it in their projects. For example:

```php
use Awcodes\Headings\Heading;

Heading::make(2)
    ->content('Product Information')
    ->color(Color::Lime),
```

And, that's it, our users can now install our plugin and use it in their projects.

---

<!-- Source: 04-clusters.md -->

## Introduction

Clusters are a hierarchical structure in panels that allow you to group [resources](../resources) and [custom pages](custom-pages) together. They are useful for organizing your panel into logical sections, and can help reduce the size of your panel's sidebar.

When using a cluster, a few things happen:

- A new navigation item is added to the navigation, which is a link to the first resource or page in the cluster.
- The individual navigation items for the resources or pages are no longer visible in the main navigation.
- A new sub-navigation UI is added to each resource or page in the cluster, which contains the navigation items for the resources or pages in the cluster.
- Resources and pages in the cluster get a new URL, prefixed with the name of the cluster. If you are generating URLs to [resources](../resources#generating-urls-to-resource-pages) and [pages](custom-pages#generating-urls-to-pages) correctly, then this change should be handled for you automatically.
- The cluster's name is in the breadcrumbs of all resources and pages in the cluster. When clicking it, you are taken to the first resource or page in the cluster.

## Creating a cluster

Before creating your first cluster, you must tell the panel where cluster classes should be located. Alongside methods like `discoverResources()` and `discoverPages()` in the [configuration](../panel-configuration), you can use `discoverClusters()`:

```php
public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->discoverResources(in: app_path('Filament/Resources'), for: 'App\\Filament\\Resources')
        ->discoverPages(in: app_path('Filament/Pages'), for: 'App\\Filament\\Pages')
        ->discoverClusters(in: app_path('Filament/Clusters'), for: 'App\\Filament\\Clusters');
}
```

Now, you can create a cluster with the `php artisan make:filament-cluster` command:

```bash
php artisan make:filament-cluster Settings
```

This will create a new cluster class in the `app/Filament/Clusters` directory:

```php
<?php

namespace App\Filament\Clusters\Settings;

use BackedEnum;
use Filament\Clusters\Cluster;
use Filament\Support\Icons\Heroicon;

class SettingsCluster extends Cluster
{
    protected static string | BackedEnum | null $navigationIcon = Heroicon::OutlinedSquares2x2;
}
```

The [`$navigationIcon`](../navigation#customizing-a-navigation-items-icon) property is defined by default since you will most likely want to customize this immediately. All other [navigation properties and methods](../navigation) are also available to use, including [`$navigationLabel`](../navigation#customizing-a-navigation-items-label), [`$navigationSort`](../navigation#sorting-navigation-items) and [`$navigationGroup`](../navigation#grouping-navigation-items). These are used to customize the cluster's main navigation item, in the same way you would customize the item for a resource or page.

## Adding resources and pages to a cluster

To add resources and pages to a cluster, you just need to define the `$cluster` property on the resource or page class, and set it to the cluster class [you created](#creating-a-cluster):

```php
use App\Filament\Clusters\Settings;

protected static ?string $cluster = SettingsCluster::class;
```

## Code structure recommendations for panels using clusters

When using clusters, it is recommended that you move all of your resources and pages into a directory with the same name as the cluster. For example, here is a directory structure for a panel that uses a cluster called `Settings`, containing a `ColorResource` and two custom pages:

```
.
+-- Clusters
|   +-- Settings
|   |   +-- SettingsCluster.php
|   |   +-- Pages
|   |   |   +-- ManageBranding.php
|   |   |   +-- ManageNotifications.php
|   |   +-- Resources
|   |   |   +-- ColorResource.php
|   |   |   +-- ColorResource
|   |   |   |   +-- Pages
|   |   |   |   |   +-- CreateColor.php
|   |   |   |   |   +-- EditColor.php
|   |   |   |   |   +-- ListColors.php
```

This is a recommendation, not a requirement. You can structure your panel however you like, as long as the resources and pages in your cluster use the [`$cluster`](#adding-resources-and-pages-to-a-cluster) property. This is just a suggestion to help you keep your panel organized.

When a cluster exists in your panel, and you generate new resources or pages with the `make:filament-resource` or `make:filament-page` commands, you will be asked if you want to create them inside a cluster directory, according to these guidelines. If you choose to, then Filament will also assign the correct `$cluster` property to the resource or page class for you. If you do not, you will need to [define the `$cluster` property](#adding-resources-and-pages-to-a-cluster) yourself.

## Setting the sub-navigation position for all pages in a cluster

The sub-navigation is rendered at the start of each page by default. It can be customized per-page, but you can also customize it for the entire cluster at once by setting the `$subNavigationPosition` property. The value may be `SubNavigationPosition::Start`, `SubNavigationPosition::End`, or `SubNavigationPosition::Top` to render the sub-navigation as tabs:

```php
use Filament\Pages\Enums\SubNavigationPosition;

protected static ?SubNavigationPosition $subNavigationPosition = SubNavigationPosition::End;
```

## Customizing the cluster breadcrumb

The cluster's name is in the breadcrumbs of all resources and pages in the cluster.

You may customize the breadcrumb name using the `$clusterBreadcrumb` property in the cluster class:

```php
protected static ?string $clusterBreadcrumb = 'cluster';
```

Alternatively, you may use the `getClusterBreadcrumb()` to define a dynamic breadcrumb name:

```php
public static function getClusterBreadcrumb(): string
{
    return __('filament/clusters/cluster.name');
}
```

---

<!-- Source: 04-editing-records.md -->

## Customizing data before filling the form

You may wish to modify the data from a record before it is filled into the form. To do this, you may define a `mutateFormDataBeforeFill()` method on the Edit page class to modify the `$data` array, and return the modified version before it is filled into the form:

```php
protected function mutateFormDataBeforeFill(array $data): array
{
    $data['user_id'] = auth()->id();

    return $data;
}
```

Alternatively, if you're editing records in a modal action, check out the [Actions documentation](../actions/edit#customizing-data-before-filling-the-form).

## Customizing data before saving

Sometimes, you may wish to modify form data before it is finally saved to the database. To do this, you may define a `mutateFormDataBeforeSave()` method on the Edit page class, which accepts the `$data` as an array, and returns it modified:

```php
protected function mutateFormDataBeforeSave(array $data): array
{
    $data['last_edited_by_id'] = auth()->id();

    return $data;
}
```

Alternatively, if you're editing records in a modal action, check out the [Actions documentation](../actions/edit#customizing-data-before-saving).

## Customizing the saving process

You can tweak how the record is updated using the `handleRecordUpdate()` method on the Edit page class:

```php
use Illuminate\Database\Eloquent\Model;

protected function handleRecordUpdate(Model $record, array $data): Model
{
    $record->update($data);

    return $record;
}
```

Alternatively, if you're editing records in a modal action, check out the [Actions documentation](../actions/edit#customizing-the-saving-process).

## Customizing redirects

By default, saving the form will not redirect the user to another page.

You may set up a custom redirect when the form is saved by overriding the `getRedirectUrl()` method on the Edit page class.

For example, the form can redirect back to the [List page](listing-records) of the resource:

```php
protected function getRedirectUrl(): string
{
    return $this->getResource()::getUrl('index');
}
```

Or the [View page](viewing-records):

```php
protected function getRedirectUrl(): string
{
    return $this->getResource()::getUrl('view', ['record' => $this->getRecord()]);
}
```

If you wish to be redirected to the previous page, else the index page:

```php
protected function getRedirectUrl(): string
{
    return $this->previousUrl ?? $this->getResource()::getUrl('index');
}
```

You can also use the [configuration](../panel-configuration) to customize the default redirect page for all resources at once:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->resourceEditPageRedirect('index') // or
        ->resourceEditPageRedirect('view');
}
```

## Customizing the save notification

When the record is successfully updated, a notification is dispatched to the user, which indicates the success of their action.

To customize the title of this notification, define a `getSavedNotificationTitle()` method on the edit page class:

```php
protected function getSavedNotificationTitle(): ?string
{
    return 'User updated';
}
```

Alternatively, if you're editing records in a modal action, check out the [Actions documentation](../actions/edit#customizing-the-save-notification).

You may customize the entire notification by overriding the `getSavedNotification()` method on the edit page class:

```php
use Filament\Notifications\Notification;

protected function getSavedNotification(): ?Notification
{
    return Notification::make()
        ->success()
        ->title('User updated')
        ->body('The user has been saved successfully.');
}
```

To disable the notification altogether, return `null` from the `getSavedNotification()` method on the edit page class:

```php
use Filament\Notifications\Notification;

protected function getSavedNotification(): ?Notification
{
    return null;
}
```

## Lifecycle hooks

Hooks may be used to execute code at various points within a page's lifecycle, like before a form is saved. To set up a hook, create a protected method on the Edit page class with the name of the hook:

```php
protected function beforeSave(): void
{
    // ...
}
```

In this example, the code in the `beforeSave()` method will be called before the data in the form is saved to the database.

There are several available hooks for the Edit pages:

```php
use Filament\Resources\Pages\EditRecord;

class EditUser extends EditRecord
{
    // ...

    protected function beforeFill(): void
    {
        // Runs before the form fields are populated from the database.
    }

    protected function afterFill(): void
    {
        // Runs after the form fields are populated from the database.
    }

    protected function beforeValidate(): void
    {
        // Runs before the form fields are validated when the form is saved.
    }

    protected function afterValidate(): void
    {
        // Runs after the form fields are validated when the form is saved.
    }

    protected function beforeSave(): void
    {
        // Runs before the form fields are saved to the database.
    }

    protected function afterSave(): void
    {
        // Runs after the form fields are saved to the database.
    }
}
```

Alternatively, if you're editing records in a modal action, check out the [Actions documentation](../actions/edit#lifecycle-hooks).

## Saving a part of the form independently

You may want to allow the user to save a part of the form independently of the rest of the form. One way to do this is with a [section action in the header or footer](../schemas/sections#adding-actions-to-the-sections-header-or-footer). From the `action()` method, you can call `saveFormComponentOnly()`, passing in the `Section` component that you want to save:

```php
use Filament\Actions\Action;
use Filament\Notifications\Notification;
use Filament\Resources\Pages\EditRecord;
use Filament\Schemas\Components\Section;

Section::make('Rate limiting')
    ->schema([
        // ...
    ])
    ->footerActions([
        fn (string $operation): Action => Action::make('save')
            ->action(function (Section $component, EditRecord $livewire) {
                $livewire->saveFormComponentOnly($component);
                
                Notification::make()
                    ->title('Rate limiting saved')
                    ->body('The rate limiting settings have been saved successfully.')
                    ->success()
                    ->send();
            })
            ->visible($operation === 'edit'),
    ])
```

The `$operation` helper is available, to ensure that the action is only visible when the form is being edited.

## Halting the saving process

At any time, you may call `$this->halt()` from inside a lifecycle hook or mutation method, which will halt the entire saving process:

```php
use Filament\Actions\Action;
use Filament\Notifications\Notification;

protected function beforeSave(): void
{
    if (! $this->getRecord()->team->subscribed()) {
        Notification::make()
            ->warning()
            ->title('You don\'t have an active subscription!')
            ->body('Choose a plan to continue.')
            ->persistent()
            ->actions([
                Action::make('subscribe')
                    ->button()
                    ->url(route('subscribe'), shouldOpenInNewTab: true),
            ])
            ->send();

        $this->halt();
    }
}
```

Alternatively, if you're editing records in a modal action, check out the [Actions documentation](../actions/edit#halting-the-saving-process).

## Authorization

For authorization, Filament will observe any [model policies](https://laravel.com/docs/authorization#creating-policies) that are registered in your app.

Users may access the Edit page if the `update()` method of the model policy returns `true`.

They also have the ability to delete the record if the `delete()` method of the policy returns `true`.

## Custom actions

"Actions" are buttons that are displayed on pages, which allow the user to run a Livewire method on the page or visit a URL.

On resource pages, actions are usually in 2 places: in the top right of the page, and below the form.

For example, you may add a new button action next to "Delete" on the Edit page:

```php
use Filament\Actions;
use Filament\Resources\Pages\EditRecord;

class EditUser extends EditRecord
{
    // ...

    protected function getHeaderActions(): array
    {
        return [
            Actions\Action::make('impersonate')
                ->action(function (): void {
                    // ...
                }),
            Actions\DeleteAction::make(),
        ];
    }
}
```

Or, a new button next to "Save" below the form:

```php
use Filament\Actions\Action;
use Filament\Resources\Pages\EditRecord;

class EditUser extends EditRecord
{
    // ...

    protected function getFormActions(): array
    {
        return [
            ...parent::getFormActions(),
            Action::make('close')->action('saveAndClose'),
        ];
    }

    public function saveAndClose(): void
    {
        // ...
    }
}
```

To view the entire actions API, please visit the [pages section](../navigation/custom-pages#adding-actions-to-pages).

### Adding a save action button to the header

The "Save" button can be added to the header of the page by overriding the `getHeaderActions()` method and using `getSaveFormAction()`. You need to pass `formId()` to the action, to specify that the action should submit the form with the ID of `form`, which is the `<form>` ID used in the view of the page:

```php
protected function getHeaderActions(): array
{
    return [
        $this->getSaveFormAction()
            ->formId('form'),
    ];
}
```

You may remove all actions from the form by overriding the `getFormActions()` method to return an empty array:

```php
protected function getFormActions(): array
{
    return [];
}
```

## Creating another Edit page

One Edit page may not be enough space to allow users to navigate many form fields. You can create as many Edit pages for a resource as you want. This is especially useful if you are using [resource sub-navigation](overview#resource-sub-navigation), as you are then easily able to switch between the different Edit pages.

To create an Edit page, you should use the `make:filament-page` command:

```bash
php artisan make:filament-page EditCustomerContact --resource=CustomerResource --type=EditRecord
```

You must register this new page in your resource's `getPages()` method:

```php
public static function getPages(): array
{
    return [
        'index' => Pages\ListCustomers::route('/'),
        'create' => Pages\CreateCustomer::route('/create'),
        'view' => Pages\ViewCustomer::route('/{record}'),
        'edit' => Pages\EditCustomer::route('/{record}/edit'),
        'edit-contact' => Pages\EditCustomerContact::route('/{record}/edit/contact'),
    ];
}
```

Now, you can define the `form()` for this page, which can contain other fields that are not present on the main Edit page:

```php
use Filament\Schemas\Schema;

public function form(Schema $schema): Schema
{
    return $schema
        ->components([
            // ...
        ]);
}
```

## Adding edit pages to resource sub-navigation

If you're using [resource sub-navigation](overview#resource-sub-navigation), you can register this page as normal in `getRecordSubNavigation()` of the resource:

```php
use App\Filament\Resources\Customers\Pages;
use Filament\Resources\Pages\Page;

public static function getRecordSubNavigation(Page $page): array
{
    return $page->generateNavigationItems([
        // ...
        Pages\EditCustomerContact::class,
    ]);
}
```

## Custom page content

Each page in Filament has its own [schema](../schemas), which defines the overall structure and content. You can override the schema for the page by defining a `content()` method on it. The `content()` method for the Edit page contains the following components by default:

```php
use Filament\Schemas\Schema;

public function content(Schema $schema): Schema
{
    return $schema
        ->components([
            $this->getFormContentComponent(), // This method returns a component to display the form that is defined in this resource
            $this->getRelationManagersContentComponent(), // This method returns a component to display the relation managers that are defined in this resource
        ]);
}
```

Inside the `components()` array, you can insert any [schema component](../schemas). You can reorder the components by changing the order of the array or remove any of the components that are not needed.

### Using a custom Blade view

For further customization opportunities, you can override the static `$view` property on the page class to a custom view in your app:

```php
protected string $view = 'filament.resources.users.pages.edit-user';
```

This assumes that you have created a view at `resources/views/filament/resources/users/pages/edit-user.blade.php`:

```blade
<x-filament-panels::page>
    {{-- `$this->getRecord()` will return the current Eloquent record for this page --}}
    
    {{ $this->content }} {{-- This will render the content of the page defined in the `content()` method, which can be removed if you want to start from scratch --}}
</x-filament-panels::page>
```

---

<!-- Source: 04-file-generation.md -->

## Introduction

Filament includes many CLI commands which generate files. This guide is to explain how you can customize the generated files.

The vast majority of files that Filament generates are PHP classes. Filament uses [`nette/php-generator`](https://github.com/nette/php-generator) to generate classes programmatically, instead of using template files. The advantage of this is that there is much more flexibility in the generated files, which is important when you need to support as many different configuration options as Filament has.

Each type of class is generated by a `ClassGenerator` class. Here are a list of `ClassGenerator` classes that Filament uses:

- `Filament\Actions\Commands\FileGenerators\ExporterClassGenerator` is used by the `make:filament-exporter` command.
- `Filament\Actions\Commands\FileGenerators\ImporterClassGenerator` is used by the `make:filament-importer` command.
- `Filament\Forms\Commands\FileGenerators\FieldClassGenerator` is used by the `make:filament-form-field` command.
- `Filament\Forms\Commands\FileGenerators\FormSchemaClassGenerator` is used by the `make:filament-form` command.
- `Filament\Forms\Commands\FileGenerators\LivewireFormComponentClassGenerator` is used by the `make:filament-livewire-form` command.
- `Filament\Infolists\Commands\FileGenerators\EntryClassGenerator` is used by the `make:filament-infolist-entry` command.
- `Filament\Commands\FileGenerators\Resources\Pages\ResourceCreateRecordPageClassGenerator` is used by the `make:filament-resource` and `make:filament-page` commands.
- `Filament\Commands\FileGenerators\Resources\Pages\ResourceCustomPageClassGenerator` is used by the `make:filament-resource` and `make:filament-page` commands.
- `Filament\Commands\FileGenerators\Resources\Pages\ResourceEditRecordPageClassGenerator` is used by the `make:filament-resource` and `make:filament-page` commands.
- `Filament\Commands\FileGenerators\Resources\Pages\ResourceListRecordsPageClassGenerator` is used by the `make:filament-resource` and `make:filament-page` commands.
- `Filament\Commands\FileGenerators\Resources\Pages\ResourceManageRecordsPageClassGenerator` is used by the `make:filament-resource` and `make:filament-page` commands.
- `Filament\Commands\FileGenerators\Resources\Pages\ResourceManageRelatedRecordsPageClassGenerator` is used by the `make:filament-resource` and `make:filament-page` commands.
- `Filament\Commands\FileGenerators\Resources\Pages\ResourceViewRecordPageClassGenerator` is used by the `make:filament-resource` and `make:filament-page` commands.
- `Filament\Commands\FileGenerators\Resources\SchemasResourceFormSchemaClassGenerator` is used by the `make:filament-resource` command.
- `Filament\Commands\FileGenerators\Resources\SchemasResourceInfolistSchemaClassGenerator` is used by the `make:filament-resource` command.
- `Filament\Commands\FileGenerators\Resources\SchemasResourceTableClassGenerator` is used by the `make:filament-resource` command.
- `Filament\Commands\FileGenerators\Resources\RelationManagerClassGenerator` is used by the `make:filament-relation-manager` command.
- `Filament\Commands\FileGenerators\Resources\ResourceClassGenerator` is used by the `make:filament-resource` command.
- `Filament\Commands\FileGenerators\ClusterClassGenerator` is used by the `make:filament-cluster` command.
- `Filament\Commands\FileGenerators\CustomPageClassGenerator` is used by the `make:filament-page` command.
- `Filament\Commands\FileGenerators\PanelProviderClassGenerator` is used by the `filament:install` and `make:filament-panel` commands.
- `Filament\Schemas\Commands\FileGenerators\ComponentClassGenerator` is used by the `make:filament-schema-component` command.
- `Filament\Schemas\Commands\FileGenerators\LivewireSchemaComponentClassGenerator` is used by the `make:filament-livewire-schema` command.
- `Filament\Schemas\Commands\FileGenerators\SchemaClassGenerator` is used by the `make:filament-schema` command.
- `Filament\Tables\Commands\FileGenerators\ColumnClassGenerator` is used by the `make:filament-table-column` command.
- `Filament\Tables\Commands\FileGenerators\LivewireTableComponentClassGenerator` is used by the `make:filament-livewire-table` command.
- `Filament\Tables\Commands\FileGenerators\TableClassGenerator` is used by the `make:filament-table` command.
- `Filament\Widgets\Commands\FileGenerators\ChartWidgetClassGenerator` is used by the `make:filament-widget` command.
- `Filament\Widgets\Commands\FileGenerators\CustomWidgetClassGenerator` is used by the `make:filament-widget` command.
- `Filament\Widgets\Commands\FileGenerators\StatsOverviewWidgetClassGenerator` is used by the `make:filament-widget` command.
- `Filament\Widgets\Commands\FileGenerators\TableWidgetClassGenerator` is used by the `make:filament-widget` command.

## The anatomy of a class generator

The best way to learn about class generators is to look at their source code. They all follow very similar patterns, and use the features from [`nette/php-generator`](https://github.com/nette/php-generator).

Here are some methods to look out for:

- `__construct()` accepts the parameters that are passed into the generator. This is all the information you have access to as context for generating the class.
- `getImports()` returns the imports that are used in the class being generated. This is not an exclusive list, and imports can be added while generating properties and methods as well, if that is easier than providing them in advance.
- `getExtends()` returns the fully qualified name of the class being extended.
- `addTraitsToClass()` is used to add traits to the class being generated. The `Class` object from `nette/php-generator` is passed in as a parameter.
- `addPropertiesToClass()` is used to add properties to the class being generated. The `Class` object from `nette/php-generator` is passed in as a parameter.
- `add*PropertyToClass()` methods are used to add a single property to the class being generated. The `Class` object from `nette/php-generator` is passed in as a parameter. They are usually called from `addPropertiesToClass()`.
- `addMethodsToClass()` is used to add methods to the class being generated. The `Class` object from `nette/php-generator` is passed in as a parameter.
- `add*MethodToClass()` methods are used to add a single method to the class being generated. The `Class` object from `nette/php-generator` is passed in as a parameter. They are usually called from `addMethodsToClass()`.

## Replacing a class generator

To be able to make changes to how a file is generated, you need to identify the correct class generator (see the list above) and replace it.

To replace it, create a new class that extends the class generator you want to replace. For example, if you want to replace the `ResourceClassGenerator`, create a new class like this:

```php
namespace App\Filament\Commands\FileGenerators\Resources;

use Filament\Commands\FileGenerators\Resources\ResourceClassGenerator as BaseResourceClassGenerator;

class ResourceClassGenerator extends BaseResourceClassGenerator
{
   // ...
}
```

You also need to register it as a binding in the service container. You can do this in a service provider such as `AppServiceProvider`:

```php
use App\Filament\Commands\FileGenerators\Resources\ResourceClassGenerator;
use Filament\Commands\FileGenerators\Resources\ResourceClassGenerator as BaseResourceClassGenerator;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register()
    {
        // ...
    
        $this->app->bind(BaseResourceClassGenerator::class, ResourceClassGenerator::class);
        
        // ...
    }
}
```

## Customizing an existing property or method in a class

While viewing the source code of the class generator, locate the property or method you want to customize. You can override it in your class generator by simply defining a new method with the same name. For example, here is a method you can find for adding the `$navigationIcon` property to a resource class:

```php
use BackedEnum;
use Filament\Support\Icons\Heroicon;
use Nette\PhpGenerator\ClassType;
use Nette\PhpGenerator\Literal;

protected function addNavigationIconPropertyToClass(ClassType $class): void
{
    $this->namespace->addUse(BackedEnum::class);
    $this->namespace->addUse(Heroicon::class);

    $property = $class->addProperty('navigationIcon', new Literal('Heroicon::OutlinedRectangleStack'))
        ->setProtected()
        ->setStatic()
        ->setType('string|BackedEnum|null');
    $this->configureNavigationIconProperty($property);
}
```

You could override that method to change its behavior, or you could just override the `configureNavigationIconProperty()` method to hook into how the property is configured. For example, make the property `public` instead of `protected`:

```php
use Nette\PhpGenerator\Property;

protected function configureNavigationIconProperty(Property $property): void
{
    $property->setPublic();
}
```

## Adding a new property or method to a class

To add a new property or method to a class, you can do this in the `addPropertiesToClass()` or `addMethodsToClass()` methods. To inherit the existing properties instead of replacing them, make sure you call `parent::addPropertiesToClass()` or `parent::addMethodsToClass()` at the start of your method. For example, here is how to add a new property to a resource class:

```php
use Nette\PhpGenerator\ClassType;
use Nette\PhpGenerator\Literal;

protected function addPropertiesToClass(ClassType $class): void
{
    parent::addPropertiesToClass($class);

    $class->addProperty('navigationSort', 10)
        ->setProtected()
        ->setStatic()
        ->setType('?int');
}
```

---

<!-- Source: 04-help.md -->

We offer a variety of support options, mostly free of charge. If you need help, the community is here for you!

## Discord

We are fortunate to have a growing community of Filament users that help each other out on our [Discord server](https://filamentphp.com/discord). Join now, it's free!
We also have many dedicated channels in different languages. Currently, we have channels for the following languages:

- [#ar](https://discord.com/channels/883083792112300104/961199444789973024) - Arabic 🇸🇦
- [#de](https://discord.com/channels/883083792112300104/998221767850070057) - German 🇩🇪
- [#es](https://discord.com/channels/883083792112300104/1049794522181275749) - Spanish 🇪🇸
- [#fa](https://discord.com/channels/883083792112300104/1042736860826443807) - Farsi 🇮🇷
- [#fr](https://discord.com/channels/883083792112300104/978572814317682688) - French 🇫🇷
- [#id](https://discord.com/channels/883083792112300104/1051444835254538271) - Indonesian 🇮🇩
- [#it](https://discord.com/channels/883083792112300104/979015654675996672) - Italian 🇮🇹
- [#ko](https://discord.com/channels/883083792112300104/1221712398017232926) - Korean 🇰🇷
- [#nl](https://discord.com/channels/883083792112300104/998685582031061102) - Dutch 🇳🇱
- [#pt-br](https://discord.com/channels/883083792112300104/966832715536162846) - Portuguese (Brazil) 🇧🇷
- [#tr](https://discord.com/channels/883083792112300104/988729996803702794) - Turkish 🇹🇷

If you are missing a channel for your language, please let us know and we will create one for you.

## GitHub

You can also reach out to us on our [GitHub discussions forum](https://github.com/filamentphp/filament/discussions), where community members and core team members are happy to help you out.

If you find a bug, you can open an [issue](https://github.com/filamentphp/filament/issues/new/choose), and even donate to the bug fix by using the link automatically added to the bottom of every new issue description.

If you have a feature request, you can create a discussion on GitHub [following these instructions](contributing#development-of-new-features). If you are not planning to contribute the feature yourself but the core team adds it to the roadmap, an issue will be created. You can sometimes fast-track the development of a new feature by donating money to it using the link added to the bottom of the issue description.

## One-on-one private support & consulting (paid)

If you're looking for dedicated help with your Filament project, we're here for you. Whether you're a solo developer or running a large company, we provide support and development services that fit your needs. More information can be found on our [consulting page](/consulting).

## Google

Since we make use of [Answer Overflow](https://www.answeroverflow.com/c/883083792112300104) on our [Discord](#discord) server, you are often one Google search away from finding an answer to your question or at least a hint on how to solve your problem. You may also find results from [GitHub](#github), the [Laracasts forum](#laracasts), or even Stack Overflow.

## Helping others

We encourage you to join any of the above platforms and help yourself and our community. Additionally, we encourage you to [contribute](contributing) to Filament itself. We are always looking for new contributors to help us improve Filament.

---

<!-- Source: 04-icons.md -->

## Introduction

Icons are used throughout the entire Filament UI to visually communicate core parts of the user experience. To render icons, we use the [Blade Icons](https://github.com/blade-ui-kit/blade-icons) package from Blade UI Kit.

They have a website where you can [search all the available icons](https://blade-ui-kit.com/blade-icons?set=1#search) from various Blade Icons packages. Each package contains a different icon set that you can choose from. Filament installs the "Heroicons" icon set by default, so if you are using icons from this set you do not need to install any additional packages.

## Using Heroicons in Filament

Filament includes the [Heroicons](https://heroicons.com) icon set by default. You can use any of the icons from this set in your Filament application without installing any additional packages. The `Heroicon` enum class allows you to leverage your IDE's autocompletion features to find the icon you want to use:

```php
use Filament\Actions\Action;
use Filament\Forms\Components\Toggle;
use Filament\Support\Icons\Heroicon;

Action::make('star')
    ->icon(Heroicon::OutlinedStar)
    
Toggle::make('is_starred')
    ->onIcon(Heroicon::Star)
```

Each icon comes with an "outlined" and "solid" variant, with the "outlined" variant's name being prefixed with `Outlined`. For example, the `Heroicon::Star` icon is the solid variant, while the `Heroicon::OutlinedStar` icon is the outlined variant.

The Heroicons set includes multiple sizes (16px, 20px and 24px) of solid icon, and when using the `Heroicon` enum class, Filament will automatically use the correct size for the context in which you are using it.

If you would like to use an icon in a [Blade component](../components), you can pass it as an attribute:

```blade
@php
    use Filament\Support\Icons\Heroicon;
@endphp

<x-filament::badge :icon="Heroicon::Star">
    Star
</x-filament::badge>
```

## Using other icon sets in Filament

Once you have [found an icon](https://blade-ui-kit.com/blade-icons?set=1#search), installed the icon set (if it's not a Heroicon) you would like to use in Filament, you need to use its name. For example, if you wanted to use the [`iconic-star`](https://blade-ui-kit.com/blade-icons/iconic-star) icon, you could pass it to an icon method of a PHP component like so:

```php
use Filament\Actions\Action;
use Filament\Forms\Components\Toggle;

Action::make('star')
    ->icon('iconic-star')
    
Toggle::make('is_starred')
    ->onIcon('iconic-check-circle')
```

If you would like to use an icon in a [Blade component](../components), you can pass it as an attribute:

```blade
<x-filament::badge icon="iconic-star">
    Star
</x-filament::badge>
```

## Using custom SVGs as icons

The [Blade Icons](https://github.com/blade-ui-kit/blade-icons) package allows you to register custom SVGs as icons. This is useful if you want to use your own custom icons in Filament.

To start with, publish the Blade Icons configuration file:

```bash
php artisan vendor:publish --tag=blade-icons
```

Now, open the `config/blade-icons.php` file, and uncomment the `default` set in the `sets` array.

Now that the default set exists in the config file, you can simply put any icons you want inside the `resources/svg` directory of your application. For example, if you put an SVG file named `star.svg` inside the `resources/svg` directory, you can reference it anywhere in Filament as `icon-star` (see below). The `icon-` prefix is configurable in the `config/blade-icons.php` file too. You can also render the custom icon in a Blade view using the [`@svg('icon-star')` directive](https://github.com/blade-ui-kit/blade-icons#directive).

```php
use Filament\Actions\Action;

Action::make('star')
    ->icon('icon-star')
```

## Replacing the default icons

Filament includes an icon management system that allows you to replace any icons that are used by default in the UI with your own. This happens in the `boot()` method of any service provider, like `AppServiceProvider`, or even a dedicated service provider for icons. If you wanted to build a plugin to replace Heroicons with a different set, you could absolutely do that by creating a Laravel package with a similar service provider.

To replace an icon, you can use the `FilamentIcon` facade. It has a `register()` method, which accepts an array of icons to replace. The key of the array is the unique [icon alias](#available-icon-aliases) that identifies the icon in the Filament UI, and the value is name of a Blade icon to replace it instead. Alternatively, you may use HTML instead of an icon name to render an icon from a Blade view for example:

```php
use Filament\Support\Facades\FilamentIcon;
use Filament\View\PanelsIconAlias;

FilamentIcon::register([
    PanelsIconAlias::GLOBAL_SEARCH_FIELD => 'fas-magnifying-glass',
    PanelsIconAlias::SIDEBAR_GROUP_COLLAPSE_BUTTON => view('icons.chevron-up'),
]);
```

## Available icon aliases

### Actions icon aliases

Using class `Filament\Actions\View\ActionsIconAlias`

- `ActionsIconAlias::ACTION_GROUP` - Trigger button of an action group
- `ActionsIconAlias::CREATE_ACTION_GROUPED` - Trigger button of a grouped create action
- `ActionsIconAlias::DELETE_ACTION` - Trigger button of a delete action
- `ActionsIconAlias::DELETE_ACTION_GROUPED` - Trigger button of a grouped delete action
- `ActionsIconAlias::DELETE_ACTION_MODAL` - Modal of a delete action
- `ActionsIconAlias::DETACH_ACTION` - Trigger button of a detach action
- `ActionsIconAlias::DETACH_ACTION_MODAL` - Modal of a detach action
- `ActionsIconAlias::DISSOCIATE_ACTION` - Trigger button of a dissociate action
- `ActionsIconAlias::DISSOCIATE_ACTION_MODAL` - Modal of a dissociate action
- `ActionsIconAlias::EDIT_ACTION` - Trigger button of an edit action
- `ActionsIconAlias::EDIT_ACTION_GROUPED` - Trigger button of a grouped edit action
- `ActionsIconAlias::EXPORT_ACTION_GROUPED` - Trigger button of a grouped export action
- `ActionsIconAlias::FORCE_DELETE_ACTION` - Trigger button of a force-delete action
- `ActionsIconAlias::FORCE_DELETE_ACTION_GROUPED` - Trigger button of a grouped force-delete action
- `ActionsIconAlias::FORCE_DELETE_ACTION_MODAL` - Modal of a force-delete action
- `ActionsIconAlias::IMPORT_ACTION_GROUPED` - Trigger button of a grouped import action
- `ActionsIconAlias::MODAL_CONFIRMATION` - Modal of an action that requires confirmation
- `ActionsIconAlias::REPLICATE_ACTION` - Trigger button of a replicate action
- `ActionsIconAlias::REPLICATE_ACTION_GROUPED` - Trigger button of a grouped replicate action
- `ActionsIconAlias::RESTORE_ACTION` - Trigger button of a restore action
- `ActionsIconAlias::RESTORE_ACTION_GROUPED` - Trigger button of a grouped restore action
- `ActionsIconAlias::RESTORE_ACTION_MODAL` - Modal of a restore action
- `ActionsIconAlias::VIEW_ACTION` - Trigger button of a view action
- `ActionsIconAlias::VIEW_ACTION_GROUPED` - Trigger button of a grouped view action

### Forms icon aliases

Using class `Filament\Forms\View\FormsIconAlias`

- `FormsIconAlias::COMPONENTS_BUILDER_ACTIONS_CLONE` - Trigger button of a clone action in a builder item
- `FormsIconAlias::COMPONENTS_BUILDER_ACTIONS_COLLAPSE` - Trigger button of a collapse action in a builder item
- `FormsIconAlias::COMPONENTS_BUILDER_ACTIONS_DELETE` - Trigger button of a delete action in a builder item
- `FormsIconAlias::COMPONENTS_BUILDER_ACTIONS_EXPAND` - Trigger button of an expand action in a builder item
- `FormsIconAlias::COMPONENTS_BUILDER_ACTIONS_MOVE_DOWN` - Trigger button of a move down action in a builder item
- `FormsIconAlias::COMPONENTS_BUILDER_ACTIONS_MOVE_UP` - Trigger button of a move up action in a builder item
- `FormsIconAlias::COMPONENTS_BUILDER_ACTIONS_REORDER` - Trigger button of a reorder action in a builder item
- `FormsIconAlias::COMPONENTS_CHECKBOX_LIST_SEARCH_FIELD` - Search input in a checkbox list
- `FormsIconAlias::COMPONENTS_FILE_UPLOAD_EDITOR_ACTIONS_DRAG_CROP` - Trigger button of a drag crop action in a file upload editor
- `FormsIconAlias::COMPONENTS_FILE_UPLOAD_EDITOR_ACTIONS_DRAG_MOVE` - Trigger button of a drag move action in a file upload editor
- `FormsIconAlias::COMPONENTS_FILE_UPLOAD_EDITOR_ACTIONS_FLIP_HORIZONTAL` - Trigger button of a flip horizontal action in a file upload editor
- `FormsIconAlias::COMPONENTS_FILE_UPLOAD_EDITOR_ACTIONS_FLIP_VERTICAL` - Trigger button of a flip vertical action in a file upload editor
- `FormsIconAlias::COMPONENTS_FILE_UPLOAD_EDITOR_ACTIONS_MOVE_DOWN` - Trigger button of a move down action in a file upload editor
- `FormsIconAlias::COMPONENTS_FILE_UPLOAD_EDITOR_ACTIONS_MOVE_LEFT` - Trigger button of a move left action in a file upload editor
- `FormsIconAlias::COMPONENTS_FILE_UPLOAD_EDITOR_ACTIONS_MOVE_RIGHT` - Trigger button of a move right action in a file upload editor
- `FormsIconAlias::COMPONENTS_FILE_UPLOAD_EDITOR_ACTIONS_MOVE_UP` - Trigger button of a move up action in a file upload editor
- `FormsIconAlias::COMPONENTS_FILE_UPLOAD_EDITOR_ACTIONS_ROTATE_LEFT` - Trigger button of a rotate left action in a file upload editor
- `FormsIconAlias::COMPONENTS_FILE_UPLOAD_EDITOR_ACTIONS_ROTATE_RIGHT` - Trigger button of a rotate right action in a file upload editor
- `FormsIconAlias::COMPONENTS_FILE_UPLOAD_EDITOR_ACTIONS_ZOOM_100` - Trigger button of a zoom 100 action in a file upload editor
- `FormsIconAlias::COMPONENTS_FILE_UPLOAD_EDITOR_ACTIONS_ZOOM_IN` - Trigger button of a zoom in action in a file upload editor
- `FormsIconAlias::COMPONENTS_FILE_UPLOAD_EDITOR_ACTIONS_ZOOM_OUT` - Trigger button of a zoom out action in a file upload editor
- `FormsIconAlias::COMPONENTS_KEY_VALUE_ACTIONS_DELETE` - Trigger button of a delete action in a key-value field item
- `FormsIconAlias::COMPONENTS_KEY_VALUE_ACTIONS_REORDER` - Trigger button of a reorder action in a key-value field item
- `FormsIconAlias::COMPONENTS_REPEATER_ACTIONS_CLONE` - Trigger button of a clone action in a repeater item
- `FormsIconAlias::COMPONENTS_REPEATER_ACTIONS_COLLAPSE` - Trigger button of a collapse action in a repeater item
- `FormsIconAlias::COMPONENTS_REPEATER_ACTIONS_DELETE` - Trigger button of a delete action in a repeater item
- `FormsIconAlias::COMPONENTS_REPEATER_ACTIONS_EXPAND` - Trigger button of an expand action in a repeater item
- `FormsIconAlias::COMPONENTS_REPEATER_ACTIONS_MOVE_DOWN` - Trigger button of a move down action in a repeater item
- `FormsIconAlias::COMPONENTS_REPEATER_ACTIONS_MOVE_UP` - Trigger button of a move up action in a repeater item
- `FormsIconAlias::COMPONENTS_REPEATER_ACTIONS_REORDER` - Trigger button of a reorder action in a repeater item
- `FormsIconAlias::COMPONENTS_RICH_EDITOR_PANELS_CUSTOM_BLOCKS_CLOSE_BUTTON` - Close button for custom blocks panel in a rich editor
- `FormsIconAlias::COMPONENTS_RICH_EDITOR_PANELS_CUSTOM_BLOCK_DELETE_BUTTON` - Delete button for a custom block in a rich editor
- `FormsIconAlias::COMPONENTS_RICH_EDITOR_PANELS_CUSTOM_BLOCK_EDIT_BUTTON` - Edit button for a custom block in a rich editor
- `FormsIconAlias::COMPONENTS_RICH_EDITOR_PANELS_MERGE_TAGS_CLOSE_BUTTON` - Close button for merge tags panel in a rich editor
- `FormsIconAlias::COMPONENTS_SELECT_ACTIONS_CREATE_OPTION` - Trigger button of a create option action in a select field
- `FormsIconAlias::COMPONENTS_SELECT_ACTIONS_EDIT_OPTION` - Trigger button of an edit option action in a select field
- `FormsIconAlias::COMPONENTS_TEXT_INPUT_ACTIONS_HIDE_PASSWORD` - Trigger button of a hide password action in a text input field
- `FormsIconAlias::COMPONENTS_TEXT_INPUT_ACTIONS_SHOW_PASSWORD` - Trigger button of a show password action in a text input field
- `FormsIconAlias::COMPONENTS_TOGGLE_BUTTONS_BOOLEAN_FALSE` - "False" option of a `boolean()` toggle buttons field
- `FormsIconAlias::COMPONENTS_TOGGLE_BUTTONS_BOOLEAN_TRUE` - "True" option of a `boolean()` toggle buttons field

### Infolists icon aliases

Using class `Filament\Infolists\View\InfolistsIconAlias`

- `InfolistsIconAlias::COMPONENTS_ICON_ENTRY_FALSE` - Falsy state of an icon entry
- `InfolistsIconAlias::COMPONENTS_ICON_ENTRY_TRUE` - Truthy state of an icon entry

### Notifications icon aliases

Using class `Filament\Notifications\View\NotificationsIconAlias`

- `NotificationsIconAlias::DATABASE_MODAL_EMPTY_STATE` - Empty state of the database notifications modal
- `NotificationsIconAlias::NOTIFICATION_CLOSE_BUTTON` - Button to close a notification
- `NotificationsIconAlias::NOTIFICATION_DANGER` - Danger notification
- `NotificationsIconAlias::NOTIFICATION_INFO` - Info notification
- `NotificationsIconAlias::NOTIFICATION_SUCCESS` - Success notification
- `NotificationsIconAlias::NOTIFICATION_WARNING` - Warning notification

### Panels icon aliases

Using class `Filament\View\PanelsIconAlias`

- `PanelsIconAlias::GLOBAL_SEARCH_FIELD` - Global search field
- `PanelsIconAlias::PAGES_DASHBOARD_ACTIONS_FILTER` - Trigger button of the dashboard filter action
- `PanelsIconAlias::PAGES_DASHBOARD_NAVIGATION_ITEM` - Dashboard page navigation item
- `PanelsIconAlias::PAGES_PASSWORD_RESET_REQUEST_PASSWORD_RESET_ACTIONS_LOGIN` - Trigger button of the login action on the request password reset page
- `PanelsIconAlias::PAGES_PASSWORD_RESET_REQUEST_PASSWORD_RESET_ACTIONS_LOGIN_RTL` - Trigger button of the login action on the request password reset page (right-to-left direction)
- `PanelsIconAlias::RESOURCES_PAGES_EDIT_RECORD_NAVIGATION_ITEM` - Resource edit record page navigation item
- `PanelsIconAlias::RESOURCES_PAGES_MANAGE_RELATED_RECORDS_NAVIGATION_ITEM` - Resource manage related records page navigation item
- `PanelsIconAlias::RESOURCES_PAGES_VIEW_RECORD_NAVIGATION_ITEM` - Resource view record page navigation item
- `PanelsIconAlias::SIDEBAR_COLLAPSE_BUTTON` - Button to collapse the sidebar
- `PanelsIconAlias::SIDEBAR_COLLAPSE_BUTTON_RTL` - Button to collapse the sidebar (right-to-left direction)
- `PanelsIconAlias::SIDEBAR_EXPAND_BUTTON` - Button to expand the sidebar
- `PanelsIconAlias::SIDEBAR_EXPAND_BUTTON_RTL` - Button to expand the sidebar (right-to-left direction)
- `PanelsIconAlias::SIDEBAR_GROUP_COLLAPSE_BUTTON` - Collapse button for a sidebar group
- `PanelsIconAlias::TENANT_MENU_BILLING_BUTTON` - Billing button in the tenant menu
- `PanelsIconAlias::TENANT_MENU_PROFILE_BUTTON` - Profile button in the tenant menu
- `PanelsIconAlias::TENANT_MENU_REGISTRATION_BUTTON` - Registration button in the tenant menu
- `PanelsIconAlias::TENANT_MENU_TOGGLE_BUTTON` - Button to toggle the tenant menu
- `PanelsIconAlias::THEME_SWITCHER_LIGHT_BUTTON` - Button to switch to the light theme from the theme switcher
- `PanelsIconAlias::THEME_SWITCHER_DARK_BUTTON` - Button to switch to the dark theme from the theme switcher
- `PanelsIconAlias::THEME_SWITCHER_SYSTEM_BUTTON` - Button to switch to the system theme from the theme switcher
- `PanelsIconAlias::TOPBAR_CLOSE_SIDEBAR_BUTTON` - Button to close the sidebar
- `PanelsIconAlias::TOPBAR_OPEN_SIDEBAR_BUTTON` - Button to open the sidebar
- `PanelsIconAlias::TOPBAR_GROUP_TOGGLE_BUTTON` - Toggle button for a topbar group
- `PanelsIconAlias::TOPBAR_OPEN_DATABASE_NOTIFICATIONS_BUTTON` - Button to open the database notifications modal
- `PanelsIconAlias::USER_MENU_PROFILE_ITEM` - Profile item in the user menu
- `PanelsIconAlias::USER_MENU_LOGOUT_BUTTON` - Button in the user menu to log out
- `PanelsIconAlias::WIDGETS_ACCOUNT_LOGOUT_BUTTON` - Button in the account widget to log out
- `PanelsIconAlias::WIDGETS_FILAMENT_INFO_OPEN_DOCUMENTATION_BUTTON` - Button to open the documentation from the Filament info widget
- `PanelsIconAlias::WIDGETS_FILAMENT_INFO_OPEN_GITHUB_BUTTON` - Button to open GitHub from the Filament info widget

### Schema icon aliases

Using class `Filament\Schemas\View\SchemaIconAlias`

- `SchemaIconAlias::COMPONENTS_WIZARD_COMPLETED_STEP` - Completed step in a wizard

### Tables icon aliases

Using class `Filament\Tables\View\TablesIconAlias`

- `TablesIconAlias::ACTIONS_DISABLE_REORDERING` - Trigger button of the disable reordering action
- `TablesIconAlias::ACTIONS_ENABLE_REORDERING` - Trigger button of the enable reordering action
- `TablesIconAlias::ACTIONS_FILTER` - Trigger button of the filter action
- `TablesIconAlias::ACTIONS_GROUP` - Trigger button of a group records action
- `TablesIconAlias::ACTIONS_OPEN_BULK_ACTIONS` - Trigger button of an open bulk actions action
- `TablesIconAlias::ACTIONS_COLUMN_MANAGER` - Trigger button of the column manager action
- `TablesIconAlias::COLUMNS_COLLAPSE_BUTTON` - Button to collapse a column
- `TablesIconAlias::COLUMNS_ICON_COLUMN_FALSE` - Falsy state of an icon column
- `TablesIconAlias::COLUMNS_ICON_COLUMN_TRUE` - Truthy state of an icon column
- `TablesIconAlias::EMPTY_STATE` - Empty state icon
- `TablesIconAlias::FILTERS_QUERY_BUILDER_CONSTRAINTS_BOOLEAN` - Default icon for a boolean constraint in the query builder
- `TablesIconAlias::FILTERS_QUERY_BUILDER_CONSTRAINTS_DATE` - Default icon for a date constraint in the query builder
- `TablesIconAlias::FILTERS_QUERY_BUILDER_CONSTRAINTS_NUMBER` - Default icon for a number constraint in the query builder
- `TablesIconAlias::FILTERS_QUERY_BUILDER_CONSTRAINTS_RELATIONSHIP` - Default icon for a relationship constraint in the query builder
- `TablesIconAlias::FILTERS_QUERY_BUILDER_CONSTRAINTS_SELECT` - Default icon for a select constraint in the query builder
- `TablesIconAlias::FILTERS_QUERY_BUILDER_CONSTRAINTS_TEXT` - Default icon for a text constraint in the query builder
- `TablesIconAlias::FILTERS_REMOVE_ALL_BUTTON` - Button to remove all filters
- `TablesIconAlias::GROUPING_COLLAPSE_BUTTON` - Button to collapse a group of records
- `TablesIconAlias::HEADER_CELL_SORT_ASC_BUTTON` - Sort button of a column sorted in ascending order
- `TablesIconAlias::HEADER_CELL_SORT_BUTTON` - Sort button of a column when it is currently not sorted
- `TablesIconAlias::HEADER_CELL_SORT_DESC_BUTTON` - Sort button of a column sorted in descending order
- `TablesIconAlias::REORDER_HANDLE` - Handle to grab in order to reorder a record with drag and drop
- `TablesIconAlias::SEARCH_FIELD` - Search input

### UI components icon aliases

Using class `Filament\Support\View\SupportIconAlias`

- `SupportIconAlias::BADGE_DELETE_BUTTON` - Button to delete a badge
- `SupportIconAlias::BREADCRUMBS_SEPARATOR` - Separator between breadcrumbs
- `SupportIconAlias::BREADCRUMBS_SEPARATOR_RTL` - Separator between breadcrumbs (right-to-left direction)
- `SupportIconAlias::MODAL_CLOSE_BUTTON` - Button to close a modal
- `SupportIconAlias::PAGINATION_FIRST_BUTTON` - Button to go to the first page
- `SupportIconAlias::PAGINATION_FIRST_BUTTON_RTL` - Button to go to the first page (right-to-left direction)
- `SupportIconAlias::PAGINATION_LAST_BUTTON` - Button to go to the last page
- `SupportIconAlias::PAGINATION_LAST_BUTTON_RTL` - Button to go to the last page (right-to-left direction)
- `SupportIconAlias::PAGINATION_NEXT_BUTTON` - Button to go to the next page
- `SupportIconAlias::PAGINATION_NEXT_BUTTON_RTL` - Button to go to the next page (right-to-left direction)
- `SupportIconAlias::PAGINATION_PREVIOUS_BUTTON` - Button to go to the previous page
- `SupportIconAlias::PAGINATION_PREVIOUS_BUTTON_RTL` - Button to go to the previous page (right-to-left direction)
- `SupportIconAlias::SECTION_COLLAPSE_BUTTON` - Button to collapse a section

### Widgets icon aliases

Using class `Filament\Widgets\View\WidgetsIconAlias`

- `WidgetsIconAlias::CHART_WIDGET_FILTER` - Button of the filter action

---

<!-- Source: 04-testing-schemas.md -->

## Filling a form in a test

To fill a form with data, pass the data to `fillForm()`:

```php
use function Pest\Livewire\livewire;

livewire(CreatePost::class)
    ->fillForm([
        'title' => fake()->sentence(),
        // ...
    ]);
```

> If you have multiple schemas on a Livewire component, you can specify which form you want to fill using `fillForm([...], 'createPostForm')`.

## Testing form field and infolist entry state

To check that a form has data, use `assertSchemaStateSet()`:

```php
use Illuminate\Support\Str;
use function Pest\Livewire\livewire;

it('can automatically generate a slug from the title', function () {
    $title = fake()->sentence();

    livewire(CreatePost::class)
        ->fillForm([
            'title' => $title,
        ])
        ->assertSchemaStateSet([
            'slug' => Str::slug($title),
        ]);
});
```

> If you have multiple schemas on a Livewire component, you can specify which schema you want to check using `assertSchemaStateSet([...], 'createPostForm')`.

You may also find it useful to pass a function to the `assertSchemaStateSet()` method, which allows you to access the form `$state` and perform additional assertions:

```php
use Illuminate\Support\Str;
use function Pest\Livewire\livewire;

it('can automatically generate a slug from the title without any spaces', function () {
    $title = fake()->sentence();

    livewire(CreatePost::class)
        ->fillForm([
            'title' => $title,
        ])
        ->assertSchemaStateSet(function (array $state): array {
            expect($state['slug'])
                ->not->toContain(' ');
                
            return [
                'slug' => Str::slug($title),
            ];
        });
});
```

You can return an array from the function if you want Filament to continue to assert the schema state after the function has been run.

## Testing form validation

Use `assertHasFormErrors()` to ensure that data is properly validated in a form:

```php
use function Pest\Livewire\livewire;

it('can validate input', function () {
    livewire(CreatePost::class)
        ->fillForm([
            'title' => null,
        ])
        ->call('create')
        ->assertHasFormErrors(['title' => 'required']);
});
```

And `assertHasNoFormErrors()` to ensure there are no validation errors:

```php
use function Pest\Livewire\livewire;

livewire(CreatePost::class)
    ->fillForm([
        'title' => fake()->sentence(),
        // ...
    ])
    ->call('create')
    ->assertHasNoFormErrors();
```

> If you have multiple schemas on a Livewire component, you can pass the name of a specific form as the second parameter like `assertHasFormErrors(['title' => 'required'], 'createPostForm')` or `assertHasNoFormErrors([], 'createPostForm')`.

## Testing the existence of a form

To check that a Livewire component has a form, use `assertFormExists()`:

```php
use function Pest\Livewire\livewire;

it('has a form', function () {
    livewire(CreatePost::class)
        ->assertFormExists();
});
```

> If you have multiple schemas on a Livewire component, you can pass the name of a specific form like `assertFormExists('createPostForm')`.

## Testing the existence of form fields

To ensure that a form has a given field, pass the field name to `assertFormFieldExists()`:

```php
use function Pest\Livewire\livewire;

it('has a title field', function () {
    livewire(CreatePost::class)
        ->assertFormFieldExists('title');
});
```

You may pass a function as an additional argument to assert that a field passes a given "truth test". This is useful for asserting that a field has a specific configuration:

```php
use function Pest\Livewire\livewire;

it('has a title field', function () {
    livewire(CreatePost::class)
        ->assertFormFieldExists('title', function (TextInput $field): bool {
            return $field->isDisabled();
        });
});
```

To assert that a form does not have a given field, pass the field name to `assertFormFieldDoesNotExist()`:

```php
use function Pest\Livewire\livewire;

it('does not have a conditional field', function () {
    livewire(CreatePost::class)
        ->assertFormFieldDoesNotExist('no-such-field');
});
```

> If you have multiple schemas on a Livewire component, you can specify which form you want to check for the existence of the field like `assertFormFieldExists('title', 'createPostForm')`.

## Testing the visibility of form fields

To ensure that a field is visible, pass the name to `assertFormFieldVisible()`:

```php
use function Pest\Livewire\livewire;

test('title is visible', function () {
    livewire(CreatePost::class)
        ->assertFormFieldVisible('title');
});
```

Or to ensure that a field is hidden you can pass the name to `assertFormFieldHidden()`:

```php
use function Pest\Livewire\livewire;

test('title is hidden', function () {
    livewire(CreatePost::class)
        ->assertFormFieldHidden('title');
});
```

> For both `assertFormFieldHidden()` and `assertFormFieldVisible()` you can pass the name of a specific form the field belongs to as the second argument like `assertFormFieldHidden('title', 'createPostForm')`.

## Testing disabled form fields

To ensure that a field is enabled, pass the name to `assertFormFieldEnabled()`:

```php
use function Pest\Livewire\livewire;

test('title is enabled', function () {
    livewire(CreatePost::class)
        ->assertFormFieldEnabled('title');
});
```

Or to ensure that a field is disabled you can pass the name to `assertFormFieldDisabled()`:

```php
use function Pest\Livewire\livewire;

test('title is disabled', function () {
    livewire(CreatePost::class)
        ->assertFormFieldDisabled('title');
});
```

> For both `assertFormFieldEnabled()` and `assertFormFieldDisabled()` you can pass the name of a specific form the field belongs to as the second argument like `assertFormFieldEnabled('title', 'createPostForm')`.

## Testing other schema components

If you need to check if a particular schema component exists rather than a field, you may use `assertSchemaComponentExists()`.  As components do not have names, this method uses the `key()` provided by the developer:

```php
use Filament\Schemas\Components\Section;

Section::make('Comments')
    ->key('comments-section')
    ->schema([
        //
    ])
```

```php
use function Pest\Livewire\livewire;

test('comments section exists', function () {
    livewire(EditPost::class)
        ->assertSchemaComponentExists('comments-section');
});
```

To assert that a schema does not have a given component, pass the component key to `assertSchemaComponentDoesNotExist()`:

```php
use function Pest\Livewire\livewire;

it('does not have a conditional component', function () {
    livewire(CreatePost::class)
        ->assertSchemaComponentDoesNotExist('no-such-section');
});
```

To check if the component exists and passes a given truth test, you can pass a function to the `checkComponentUsing` argument of `assertSchemaComponentExists()`, returning true or false if the component passes the test or not:

```php
use Filament\Forms\Components\Component;

use function Pest\Livewire\livewire;

test('comments section has heading', function () {
    livewire(EditPost::class)
        ->assertSchemaComponentExists(
            'comments-section',
            checkComponentUsing: function (Component $component): bool {
                return $component->getHeading() === 'Comments';
            },
        );
});
```

If you want more informative test results, you can embed an assertion within your truth test callback:

```php
use Filament\Forms\Components\Component;
use Illuminate\Testing\Assert;

use function Pest\Livewire\livewire;

test('comments section is enabled', function () {
    livewire(EditPost::class)
        ->assertSchemaComponentExists(
            'comments-section',
            checkComponentUsing: function (Component $component): bool {
                Assert::assertTrue(
                    $component->isEnabled(),
                    'Failed asserting that comments-section is enabled.',
                );
                
                return true;
            },
        );
});
```

## Testing repeaters

Internally, repeaters generate UUIDs for items to keep track of them in the Livewire HTML easier. This means that when you are testing a form with a repeater, you need to ensure that the UUIDs are consistent between the form and the test. This can be tricky, and if you don't do it correctly, your tests can fail as the tests are expecting a UUID, not a numeric key.

However, since Livewire doesn't need to keep track of the UUIDs in a test, you can disable the UUID generation and replace them with numeric keys, using the `Repeater::fake()` method at the start of your test:

```php
use Filament\Forms\Components\Repeater;
use function Pest\Livewire\livewire;

$undoRepeaterFake = Repeater::fake();

livewire(EditPost::class, ['record' => $post])
    ->assertSchemaStateSet([
        'quotes' => [
            [
                'content' => 'First quote',
            ],
            [
                'content' => 'Second quote',
            ],
        ],
        // ...
    ]);

$undoRepeaterFake();
```

You may also find it useful to test the number of items in a repeater by passing a function to the `assertSchemaStateSet()` method:

```php
use Filament\Forms\Components\Repeater;
use function Pest\Livewire\livewire;

$undoRepeaterFake = Repeater::fake();

livewire(EditPost::class, ['record' => $post])
    ->assertSchemaStateSet(function (array $state) {
        expect($state['quotes'])
            ->toHaveCount(2);
    });

$undoRepeaterFake();
```

### Testing repeater actions

In order to test that repeater actions are working as expected, you can utilize the `callFormComponentAction()` method to call your repeater actions and then [perform additional assertions](../testing#actions).

To interact with an action on a particular repeater item, you need to pass in the `item` argument with the key of that repeater item. If your repeater is reading from a relationship, you should prefix the ID (key) of the related record with `record-` to form the key of the repeater item:

```php
use App\Models\Quote;
use Filament\Forms\Components\Repeater;
use function Pest\Livewire\livewire;

$quote = Quote::first();

livewire(EditPost::class, ['record' => $post])
    ->callAction(TestAction::make('sendQuote')->schemaComponent('quotes')->arguments([
        'item' => "record-{$quote->getKey()}",
    ]))
    ->assertNotified('Quote sent!');
```

## Testing builders

Internally, builders generate UUIDs for items to keep track of them in the Livewire HTML easier. This means that when you are testing a form with a builder, you need to ensure that the UUIDs are consistent between the form and the test. This can be tricky, and if you don't do it correctly, your tests can fail as the tests are expecting a UUID, not a numeric key.

However, since Livewire doesn't need to keep track of the UUIDs in a test, you can disable the UUID generation and replace them with numeric keys, using the `Builder::fake()` method at the start of your test:

```php
use Filament\Forms\Components\Builder;
use function Pest\Livewire\livewire;

$undoBuilderFake = Builder::fake();

livewire(EditPost::class, ['record' => $post])
    ->assertSchemaStateSet([
        'content' => [
            [
                'type' => 'heading',
                'data' => [
                    'content' => 'Hello, world!',
                    'level' => 'h1',
                ],
            ],
            [
                'type' => 'paragraph',
                'data' => [
                    'content' => 'This is a test post.',
                ],
            ],
        ],
        // ...
    ]);

$undoBuilderFake();
```

You may also find it useful to access test the number of items in a repeater by passing a function to the `assertSchemaStateSet()` method:

```php
use Filament\Forms\Components\Builder;
use function Pest\Livewire\livewire;

$undoBuilderFake = Builder::fake();

livewire(EditPost::class, ['record' => $post])
    ->assertSchemaStateSet(function (array $state) {
        expect($state['content'])
            ->toHaveCount(2);
    });

$undoBuilderFake();
```

## Testing wizards

To go to a wizard's next step, use `goToNextWizardStep()`:

```php
use function Pest\Livewire\livewire;

it('moves to next wizard step', function () {
    livewire(CreatePost::class)
        ->goToNextWizardStep()
        ->assertHasFormErrors(['title']);
});
```

You can also go to the previous step by calling `goToPreviousWizardStep()`:

```php
use function Pest\Livewire\livewire;

it('moves to next wizard step', function () {
    livewire(CreatePost::class)
        ->goToPreviousWizardStep()
        ->assertHasFormErrors(['title']);
});
```

If you want to go to a specific step, use `goToWizardStep()`, then the `assertWizardCurrentStep` method which can ensure you are on the desired step without validation errors from the previous:

```php
use function Pest\Livewire\livewire;

it('moves to the wizards second step', function () {
    livewire(CreatePost::class)
        ->goToWizardStep(2)
        ->assertWizardCurrentStep(2);
});
```

If you have multiple schemas on a single Livewire component, any of the wizard test helpers can accept a `schema` parameter:

```php
use function Pest\Livewire\livewire;

it('moves to next wizard step only for fooForm', function () {
    livewire(CreatePost::class)
        ->goToNextWizardStep(schema: 'fooForm')
        ->assertHasFormErrors(['title'], schema: 'fooForm');
});
```

---

<!-- Source: 05-panel-configuration.md -->

import Aside from "@components/Aside.astro"

## Introduction

By default, the configuration file is located at `app/Providers/Filament/AdminPanelProvider.php`. Keep reading to learn more about [panels](#introducing-panels) and how each has [its own configuration file](#creating-a-new-panel).

## Introducing panels

By default, when you install the package, there is one panel that has been set up for you - and it lives on `/admin`. All the [resources](resources), [custom pages](navigation/custom-pages), and [dashboard widgets](dashboard) you create get registered to this panel.

However, you can create as many panels as you want, and each can have its own set of resources, pages and widgets.

For example, you could build a panel where users can log in at `/app` and access their dashboard, and admins can log in at `/admin` and manage the app. The `/app` panel and the `/admin` panel have their own resources, since each group of users has different requirements. Filament allows you to do that by providing you with the ability to create multiple panels.

### The default admin panel

When you run `filament:install`, a new file is created in `app/Providers/Filament` - `AdminPanelProvider.php`. This file contains the configuration for the `/admin` panel.

When this documentation refers to the "configuration", this is the file you need to edit. It allows you to completely customize the app.

### Creating a new panel

To create a new panel, you can use the `make:filament-panel` command, passing in the unique name of the new panel:

```bash
php artisan make:filament-panel app
```

This command will create a new panel called "app". A configuration file will be created at `app/Providers/Filament/AppPanelProvider.php`. You can access this panel at `/app`, but you can [customize the path](#changing-the-path) if you don't want that.

Since this configuration file is also a [Laravel service provider](https://laravel.com/docs/providers), it needs to be registered in `bootstrap/providers.php` (Laravel 11 and above) or `config/app.php` (Laravel 10 and below). Filament will attempt to do this for you, but if you get an error while trying to access your panel then this process has probably failed.

## Changing the path

In a panel configuration file, you can change the path that the app is accessible at using the `path()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->path('app');
}
```

If you want the app to be accessible without any prefix, you can set this to be an empty string:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->path('');
}
```

Make sure your `routes/web.php` file doesn't already define the `''` or `'/'` route, as it will take precedence.

## Render hooks

[Render hooks](advanced/render-hooks) allow you to render Blade content at various points in the framework views. You can [register global render hooks](advanced/render-hooks#registering-render-hooks) in a service provider or middleware, but it also allows you to register render hooks that are specific to a panel. To do that, you can use the `renderHook()` method on the panel configuration object. Here's an example, integrating [`wire-elements/modal`](https://github.com/wire-elements/modal) with Filament:

```php
use Filament\Panel;
use Filament\View\PanelsRenderHook;
use Illuminate\Support\Facades\Blade;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->renderHook(
            PanelsRenderHook::BODY_START,
            fn (): string => Blade::render('@livewire(\'livewire-ui-modal\')'),
        );
}
```

A full list of available render hooks can be found [here](advanced/render-hooks#available-render-hooks).

## Setting a domain

By default, Filament will respond to requests from all domains. If you'd like to scope it to a specific domain, you can use the `domain()` method, similar to [`Route::domain()` in Laravel](https://laravel.com/docs/routing#route-group-subdomain-routing):

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->domain('admin.example.com');
}
```

## Customizing the maximum content width

By default, Filament will restrict the width of the content on the page, so it doesn't become too wide on large screens. To change this, you may use the `maxContentWidth()` method. Options correspond to [Tailwind's max-width scale](https://tailwindcss.com/docs/max-width). The options are `ExtraSmall`, `Small`, `Medium`, `Large`, `ExtraLarge`, `TwoExtraLarge`, `ThreeExtraLarge`, `FourExtraLarge`, `FiveExtraLarge`, `SixExtraLarge`, `SevenExtraLarge`, `Full`, `MinContent`, `MaxContent`, `FitContent`,  `Prose`, `ScreenSmall`, `ScreenMedium`, `ScreenLarge`, `ScreenExtraLarge` and `ScreenTwoExtraLarge`. The default is `SevenExtraLarge`:

```php
use Filament\Panel;
use Filament\Support\Enums\Width;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->maxContentWidth(Width::Full);
}
```

If you'd like to set the max content width for pages of the type `SimplePage`, like login and registration pages, you may do so using the `simplePageMaxContentWidth()` method. The default is `Large`:

```php
use Filament\Panel;
use Filament\Support\Enums\Width;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->simplePageMaxContentWidth(Width::Small);
}
```

## Setting the default sub-navigation position

Sub-navigation is rendered at the start of each page by default. It can be customized per-page, per-resource and per-cluster, but you can also customize it for the entire panel at once using the `subNavigationPosition()` method. The value may be `SubNavigationPosition::Start`, `SubNavigationPosition::End`, or `SubNavigationPosition::Top` to render the sub-navigation as tabs:

```php
use Filament\Pages\Enums\SubNavigationPosition;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->subNavigationPosition(SubNavigationPosition::End);
}
```

## Lifecycle hooks

Hooks may be used to execute code during a panel's lifecycle. `bootUsing()` is a hook that gets run on every request that takes place within that panel. If you have multiple panels, only the current panel's `bootUsing()` will be run. The function gets run from middleware, after all service providers have been booted:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->bootUsing(function (Panel $panel) {
            // ...
        });
}
```

## SPA mode

SPA mode utilizes [Livewire's `wire:navigate` feature](https://livewire.laravel.com/docs/navigate) to make your server-rendered panel feel like a single-page-application, with less delay between page loads and a loading bar for longer requests. To enable SPA mode on a panel, you can use the `spa()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->spa();
}
```

### Disabling SPA navigation for specific URLs

By default, when enabling SPA mode, any URL that lives on the same domain as the current request will be navigated to using Livewire's [`wire:navigate`](https://livewire.laravel.com/docs/navigate) feature. If you want to disable this for specific URLs, you can use the `spaUrlExceptions()` method:

```php
use App\Filament\Resources\Posts\PostResource;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->spa()
        ->spaUrlExceptions(fn (): array => [
            url('/admin'),
            PostResource::getUrl(),
        ]);
}
```

<Aside variant="info">
    In this example, we are using [`getUrl()`](/resources#generating-urls-to-resource-pages) on a resource to get the URL to the resource's index page. This feature requires the panel to already be registered though, and the configuration is too early in the request lifecycle to do that. You can use a function to return the URLs instead, which will be resolved when the panel has been registered.
</Aside>

These URLs need to exactly match the URL that the user is navigating to, including the domain and protocol. If you'd like to use a pattern to match multiple URLs, you can use an asterisk (`*`) as a wildcard character:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->spa()
        ->spaUrlExceptions([
            '*/admin/posts/*',
        ]);
}
```

### Enabling SPA prefetching

SPA prefetching enhances the user experience by automatically prefetching pages when users hover over links, making navigation feel even more responsive. This feature utilizes [Livewire's `wire:navigate.hover` functionality](https://livewire.laravel.com/docs/navigate#prefetching-links).

To enable SPA mode with prefetching, you can pass the `hasPrefetching: true` parameter to the `spa()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->spa(hasPrefetching: true);
}
```

When prefetching is enabled, all links within your panel will automatically include `wire:navigate.hover`, which prefetches the page content when users hover over the link. This works seamlessly with [URL exceptions](#disabling-spa-navigation-for-specific-urls) - any URLs excluded from SPA mode will also be excluded from prefetching.

<Aside variant="info">
    Prefetching only works when SPA mode is enabled. If you disable SPA mode, prefetching will also be disabled automatically.
</Aside>

<Aside variant="warning">
    Prefetching heavy pages can lead to increased bandwidth usage and server load, especially if users hover over many links in quick succession. Use this feature judiciously, particularly if your app has pages with large amounts of data or complex queries.
</Aside>

## Unsaved changes alerts

You may alert users if they attempt to navigate away from a page without saving their changes. This is applied on [Create](resources/creating-records) and [Edit](resources/editing-records) pages of a resource, as well as any open action modals. To enable this feature, you can use the `unsavedChangesAlerts()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->unsavedChangesAlerts();
}
```

## Enabling database transactions

By default, Filament does not wrap operations in database transactions, and allows the user to enable this themselves when they have tested to ensure that their operations are safe to be wrapped in a transaction. However, you can enable database transactions at once for all operations by using the `databaseTransactions()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->databaseTransactions();
}
```

For any actions you do not want to be wrapped in a transaction, you can use the `databaseTransaction(false)` method:

```php
CreateAction::make()
    ->databaseTransaction(false)
```

And for any pages like [Create resource](resources/creating-records) and [Edit resource](resources/editing-records), you can define the `$hasDatabaseTransactions` property to `false` on the page class:

```php
use Filament\Resources\Pages\CreateRecord;

class CreatePost extends CreateRecord
{
    protected ?bool $hasDatabaseTransactions = false;

    // ...
}
```

### Opting in to database transactions for specific actions and pages

Instead of enabling database transactions everywhere and opting out of them for specific actions and pages, you can opt in to database transactions for specific actions and pages.

For actions, you can use the `databaseTransaction()` method:

```php
CreateAction::make()
    ->databaseTransaction()
```

For pages like [Create resource](resources/creating-records) and [Edit resource](resources/editing-records), you can define the `$hasDatabaseTransactions` property to `true` on the page class:

```php
use Filament\Resources\Pages\CreateRecord;

class CreatePost extends CreateRecord
{
    protected ?bool $hasDatabaseTransactions = true;

    // ...
}
```

## Registering assets for a panel

You can register [assets](../advanced/assets) that will only be loaded on pages within a specific panel, and not in the rest of the app. To do that, pass an array of assets to the `assets()` method:

```php
use Filament\Panel;
use Filament\Support\Assets\Css;
use Filament\Support\Assets\Js;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->assets([
            Css::make('custom-stylesheet', resource_path('css/custom.css')),
            Js::make('custom-script', resource_path('js/custom.js')),
        ]);
}
```

Before these [assets](../advanced/assets) can be used, you'll need to run `php artisan filament:assets`.

## Applying middleware

You can apply extra middleware to all routes by passing an array of middleware classes to the `middleware()` method in the configuration:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->middleware([
            // ...
        ]);
}
```

By default, middleware will be run when the page is first loaded, but not on subsequent Livewire AJAX requests. If you want to run middleware on every request, you can make it persistent by passing `true` as the second argument to the `middleware()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->middleware([
            // ...
        ], isPersistent: true);
}
```

### Applying middleware to authenticated routes

You can apply middleware to all authenticated routes by passing an array of middleware classes to the `authMiddleware()` method in the configuration:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->authMiddleware([
            // ...
        ]);
}
```

By default, middleware will be run when the page is first loaded, but not on subsequent Livewire AJAX requests. If you want to run middleware on every request, you can make it persistent by passing `true` as the second argument to the `authMiddleware()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->authMiddleware([
            // ...
        ], isPersistent: true);
}
```

## Disabling broadcasting

By default, Laravel Echo will automatically connect for every panel, if credentials have been set up in the [published `config/filament.php` configuration file](installation#publishing-configuration). To disable this automatic connection in a panel, you can use the `broadcasting(false)` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->broadcasting(false);
}
```

## Strict authorization mode

By default, when Filament authorizes the user access to a resource, it will first check if the policy exists for that model, and if it does, it will check if a method exists on the policy to perform the action. If the policy or policy method does not exist, it will grant the user access to the resource, as it assumes you have not set up authorization yet, or you do not require it.

If you would prefer Filament to throw an exception if the policy or policy method does not exist, you can enable strict authorization mode using the `strictAuthorization()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->strictAuthorization();
}
```

## Configuring error notifications

When Laravel's debug mode is disabled, Filament will replace Livewire's full-screen error modals with neater flash notifications. You can disable this behavior using the `errorNotifications(false)` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->errorNotifications(false);
}
```

You may customize the error notification text by passing strings to the `title` and `body` parameters of the `registerErrorNotification()` method:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->registerErrorNotification(
            title: 'An error occurred',
            body: 'Please try again later.',
        );
}
```

You may also register error notification text for a specific HTTP status code, such as `404`, by passing that status code in the `statusCode` parameter:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->registerErrorNotification(
            title: 'An error occurred',
            body: 'Please try again later.',
        )
        ->registerErrorNotification(
            title: 'Record not found',
            body: 'A record you are looking for does not exist.',
            statusCode: 404,
        );
}
```

You can also enable or disable error notifications for specific pages in a panel by setting the `$hasErrorNotifications` property on the page class:

```php
use Filament\Pages\Dashboard as BaseDashboard;

class Dashboard extends BaseDashboard
{
    protected ?bool $hasErrorNotifications = true;
    
    // or
    
    protected ?bool $hasErrorNotifications = false;

    // ...
}
```

If you would like to run custom code to check if error notifications should be shown, you can use the `hasErrorNotifications()` method on the page class:

```php
use Filament\Pages\Dashboard as BaseDashboard;

class Dashboard extends BaseDashboard
{
    public function hasErrorNotifications(): bool
    {
        return FeatureFlag::active();
    }

    // ...
}
```

You may also register error notification text by calling the `registerErrorNotification()` method on the page class from inside the `setUpErrorNotifications()` method:

```php
use Filament\Pages\Dashboard as BaseDashboard;

class Dashboard extends BaseDashboard
{
    protected function setUpErrorNotifications(): void
    {
        $this->registerErrorNotification(
            title: 'An error occurred',
            body: 'Please try again later.',
        );
    }

    // ...
}
```

You can also register error notification text for a specific HTTP status code, such as `404`, by passing that status code in the `statusCode` parameter:

```php
use Filament\Pages\Dashboard as BaseDashboard;

class Dashboard extends BaseDashboard
{
    protected function setUpErrorNotifications(): void
    {
        $this->registerErrorNotification(
            title: 'An error occurred',
            body: 'Please try again later.',
        );
    
        $this->registerErrorNotification(
            title: 'Record not found',
            body: 'A record you are looking for does not exist.',
            statusCode: 404,
        );
    }

    // ...
}
```

---

<!-- Source: 05-testing-actions.md -->

## Calling an action in a test

You can call an action by passing its name or class to `callAction()`:

```php
use function Pest\Livewire\livewire;

it('can send invoices', function () {
    $invoice = Invoice::factory()->create();

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->callAction('send');

    expect($invoice->refresh())
        ->isSent()->toBeTrue();
});
```

## Testing table actions

To test table actions, you can use a `TestAction` object with the `table()` method. This object receives the name of the action you want to test, and replaces the name of the action in any testing method you want to use. For example:

```php
use Filament\Actions\Testing\TestAction;
use function Pest\Livewire\livewire;

$invoice = Invoice::factory()->create();

livewire(ListInvoices::class)
    ->callAction(TestAction::make('send')->table($invoice));

livewire(ListInvoices::class)
    ->assertActionVisible(TestAction::make('send')->table($invoice))

livewire(ListInvoices::class)
    ->assertActionExists(TestAction::make('send')->table($invoice))
```

### Testing table header actions

To test a header action, you can use the `table()` method without passing in a specific record to test with:

```php
use Filament\Actions\Testing\TestAction;
use function Pest\Livewire\livewire;

livewire(ListInvoices::class)
    ->callAction(TestAction::make('create')->table());

livewire(ListInvoices::class)
    ->assertActionVisible(TestAction::make('create')->table())

livewire(ListInvoices::class)
    ->assertActionExists(TestAction::make('create')->table())
```

### Testing table bulk actions

To test a bulk action, first call `selectTableRecords()` and pass in any records you want to select. Then, use the `TestAction`'s `bulk()` method to specify the action you want to test. For example:

```php
use Filament\Actions\Testing\TestAction;
use function Pest\Livewire\livewire;

$invoices = Invoice::factory()->count(3)->create();

livewire(ListInvoices::class)
    ->selectTableRecords($invoices->pluck('id')->toArray())
    ->callAction(TestAction::make('send')->table()->bulk());

livewire(ListInvoices::class)
    ->assertActionVisible(TestAction::make('send')->table()->bulk())

livewire(ListInvoices::class)
    ->assertActionExists(TestAction::make('send')->table()->bulk())
```

## Testing actions in a schema

If an action belongs to a component in a resource's infolist, for example, if it is in the `belowContent()` method of an infolist entry, you can use the `TestAction` object with the `schemaComponent()` method. This object receives the name of the action you want to test and replaces the name of the action in any testing method you want to use. For example:

```php
use Filament\Actions\Testing\TestAction;
use function Pest\Livewire\livewire;

$invoice = Invoice::factory()->create();

livewire(EditInvoice::class)
    ->callAction(TestAction::make('send')->schemaComponent('customer_id'));

livewire(EditInvoice::class)
    ->assertActionVisible(TestAction::make('send')->schemaComponent('customer_id'))

livewire(EditInvoice::class)
    ->assertActionExists(TestAction::make('send')->schemaComponent('customer_id'))
```

## Testing actions inside another action's schema / form

If an action belongs to a component in another action's `schema()` (or `form()`), for example, if it is in the `belowContent()` method of a form field in an action modal, you can use the `TestAction` object with the `schemaComponent()` method. This object receives the name of the action you want to test and replaces the name of the action in any testing method you want to use. You should pass an array of `TestAction` objects in order, for example:

```php
use Filament\Actions\Testing\TestAction;
use function Pest\Livewire\livewire;

$invoice = Invoice::factory()->create();

livewire(ManageInvoices::class)
    ->callAction([
        TestAction::make('view')->table($invoice),
        TestAction::make('send')->schemaComponent('customer.name'),
    ]);
    
livewire(ManageInvoices::class)
    ->assertActionVisible([
        TestAction::make('view')->table($invoice),
        TestAction::make('send')->schemaComponent('customer.name'),
    ]);
    
livewire(ManageInvoices::class)
    ->assertActionExists([
        TestAction::make('view')->table($invoice),
        TestAction::make('send')->schemaComponent('customer.name'),
    ]);
```

## Testing forms in action modals

To pass an array of data into an action, use the `data` parameter:

```php
use function Pest\Livewire\livewire;

it('can send invoices', function () {
    $invoice = Invoice::factory()->create();

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->callAction('send', data: [
            'email' => $email = fake()->email(),
        ])
        ->assertHasNoFormErrors();

    expect($invoice->refresh())
        ->isSent()->toBeTrue()
        ->recipient_email->toBe($email);
});
```

If you ever need to only set an action's data without immediately calling it, you can use `fillForm()`:

```php
use function Pest\Livewire\livewire;

it('can send invoices', function () {
    $invoice = Invoice::factory()->create();

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->mountAction('send')
        ->fillForm([
            'email' => $email = fake()->email(),
        ])
});
```

### Testing validation errors in an action modal's form

`assertHasNoFormErrors()` is used to assert that no validation errors occurred when submitting the action form.

To check if a validation error has occurred with the data, use `assertHasFormErrors()`, similar to `assertHasErrors()` in Livewire:

```php
use function Pest\Livewire\livewire;

it('can validate invoice recipient email', function () {
    $invoice = Invoice::factory()->create();

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->callAction('send', data: [
            'email' => Str::random(),
        ])
        ->assertHasFormErrors(['email' => ['email']]);
});
```

To check if an action is pre-filled with data, you can use the `assertSchemaStateSet()` method:

```php
use function Pest\Livewire\livewire;

it('can send invoices to the primary contact by default', function () {
    $invoice = Invoice::factory()->create();
    $recipientEmail = $invoice->company->primaryContact->email;

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->mountAction('send')
        ->assertSchemaStateSet([
            'email' => $recipientEmail,
        ])
        ->callMountedAction()
        ->assertHasNoFormErrors();

    expect($invoice->refresh())
        ->isSent()->toBeTrue()
        ->recipient_email->toBe($recipientEmail);
});
```

## Testing the content of an action modal

To assert the content of a modal, you should first mount the action (rather than call it which closes the modal). You can then use [Livewire assertions](https://livewire.laravel.com/docs/testing#assertions) such as `assertSee()` to assert the modal contains the content that you expect it to:

```php
use function Pest\Livewire\livewire;

it('confirms the target address before sending', function () {
    $invoice = Invoice::factory()->create();
    $recipientEmail = $invoice->company->primaryContact->email;

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->mountAction('send')
        ->assertSee($recipientEmail);
});
```

## Testing the existence of an action

To ensure that an action exists or doesn't, you can use the `assertActionExists()` or  `assertActionDoesNotExist()` method:

```php
use function Pest\Livewire\livewire;

it('can send but not unsend invoices', function () {
    $invoice = Invoice::factory()->create();

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->assertActionExists('send')
        ->assertActionDoesNotExist('unsend');
});
```

You may pass a function as an additional argument to assert that an action passes a given "truth test". This is useful for asserting that an action has a specific configuration:

```php
use Filament\Actions\Action;
use function Pest\Livewire\livewire;

it('has the correct description', function () {
    $invoice = Invoice::factory()->create();

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->assertActionExists('send', function (Action $action): bool {
            return $action->getModalDescription() === 'This will send an email to the customer\'s primary address, with the invoice attached as a PDF';
        });
});
```

## Testing the visibility of an action

To ensure an action is hidden or visible for a user, you can use the `assertActionHidden()` or `assertActionVisible()` methods:

```php
use function Pest\Livewire\livewire;

it('can only print invoices', function () {
    $invoice = Invoice::factory()->create();

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->assertActionHidden('send')
        ->assertActionVisible('print');
});
```

## Testing disabled actions

To ensure an action is enabled or disabled for a user, you can use the `assertActionEnabled()` or `assertActionDisabled()` methods:

```php
use function Pest\Livewire\livewire;

it('can only print a sent invoice', function () {
    $invoice = Invoice::factory()->create();

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->assertActionDisabled('send')
        ->assertActionEnabled('print');
});
```

To ensure sets of actions exist in the correct order, you can use `assertActionsExistInOrder()`:

```php
use function Pest\Livewire\livewire;

it('can have actions in order', function () {
    $invoice = Invoice::factory()->create();

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->assertActionsExistInOrder(['send', 'export']);
});
```

To check if an action is hidden to a user, you can use the `assertActionHidden()` method:

```php
use function Pest\Livewire\livewire;

it('can not send invoices', function () {
    $invoice = Invoice::factory()->create();

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->assertActionHidden('send');
});
```

## Testing the label of an action

To ensure an action has the correct label, you can use `assertActionHasLabel()` and `assertActionDoesNotHaveLabel()`:

```php
use function Pest\Livewire\livewire;

it('send action has correct label', function () {
    $invoice = Invoice::factory()->create();

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->assertActionHasLabel('send', 'Email Invoice')
        ->assertActionDoesNotHaveLabel('send', 'Send');
});
```

## Testing the icon of an action

To ensure an action's button is showing the correct icon, you can use `assertActionHasIcon()` or `assertActionDoesNotHaveIcon()`:

```php
use function Pest\Livewire\livewire;

it('when enabled the send button has correct icon', function () {
    $invoice = Invoice::factory()->create();

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->assertActionEnabled('send')
        ->assertActionHasIcon('send', 'envelope-open')
        ->assertActionDoesNotHaveIcon('send', 'envelope');
});
```

## Testing the color of an action

To ensure that an action's button is displaying the right color, you can use `assertActionHasColor()` or `assertActionDoesNotHaveColor()`:

```php
use function Pest\Livewire\livewire;

it('actions display proper colors', function () {
    $invoice = Invoice::factory()->create();

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->assertActionHasColor('delete', 'danger')
        ->assertActionDoesNotHaveColor('print', 'danger');
});
```

## Testing the URL of an action

To ensure an action has the correct URL, you can use `assertActionHasUrl()`, `assertActionDoesNotHaveUrl()`, `assertActionShouldOpenUrlInNewTab()`, and `assertActionShouldNotOpenUrlInNewTab()`:

```php
use function Pest\Livewire\livewire;

it('links to the correct Filament sites', function () {
    $invoice = Invoice::factory()->create();

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->assertActionHasUrl('filament', 'https://filamentphp.com/')
        ->assertActionDoesNotHaveUrl('filament', 'https://github.com/filamentphp/filament')
        ->assertActionShouldOpenUrlInNewTab('filament')
        ->assertActionShouldNotOpenUrlInNewTab('github');
});
```

## Testing action arguments

To test action arguments, you can use a `TestAction` object with the `arguments()` method. This object receives the name of the action you want to test and replaces the name of the action in any testing method you want to use. For example:

```php
use Filament\Actions\Testing\TestAction;
use function Pest\Livewire\livewire;

$invoice = Invoice::factory()->create();

livewire(ManageInvoices::class)
    ->callAction(TestAction::make('send')->arguments(['invoice' => $invoice->getKey()]));

livewire(ManageInvoices::class)
    ->assertActionVisible(TestAction::make('send')->arguments(['invoice' => $invoice->getKey()]))

livewire(ManageInvoices::class)
    ->assertActionExists(TestAction::make('send')->arguments(['invoice' => $invoice->getKey()]))
```

## Testing if an action has been halted

To check if an action has been halted, you can use `assertActionHalted()`:

```php
use function Pest\Livewire\livewire;

it('stops sending if invoice has no email address', function () {
    $invoice = Invoice::factory(['email' => null])->create();

    livewire(EditInvoice::class, [
        'invoice' => $invoice,
    ])
        ->callAction('send')
        ->assertActionHalted('send');
});
```

## Using action class names in tests

Filament includes a host of prebuilt actions such as `CreateAction`, `EditAction` and `DeleteAction`, and you can use these class names in your tests instead of action names, for example:

```php
use Filament\Actions\CreateAction;
use function Pest\Livewire\livewire;

livewire(ManageInvoices::class)
    ->callAction(CreateAction::class)
```

If you have your own action classes in your app with a `make()` method, the name of your action is not discoverable by Filament unless it runs the `make()` method, which is not efficient. To use your own action class names in your tests, you can add an `#[ActionName]` attribute to your action class, which Filament can use to discover the name of your action. The name passed to the `#[ActionName]` attribute should be the same as the name of the action you would normally use in your tests. For example:

```php
use Filament\Actions\Action;
use Filament\Actions\ActionName;

#[ActionName('send')]
class SendInvoiceAction
{
    public static function make(): Action
    {
        return Action::make('send')
            ->requiresConfirmation()
            ->action(function () {
                // ...
            });
    }
}
```

Now, you can use the class name in your tests:

```php
use App\Filament\Resources\Invoices\Actions\SendInvoiceAction;
use Filament\Actions\Testing\TestAction;
use function Pest\Livewire\livewire;

$invoice = Invoice::factory()->create();

livewire(ManageInvoices::class)
    ->callAction(TestAction::make(SendInvoiceAction::class)->table($invoice);
```

If you have an action class that extends the `Action` class, you can add a `getDefaultName()` static method to the class, which will be used to discover the name of the action. It also allows users to omit the name of the action from the `make()` method when instantiating it. For example:

```php
use Filament\Actions\Action;

class SendInvoiceAction extends Action
{
    public static function getDefaultName(): string
    {
        return 'send';
    }

    protected function setUp(): void
    {
        parent::setUp();
        
        $this
            ->requiresConfirmation()
            ->action(function () {
                // ...
            });
    }
}
```

---

<!-- Source: 05-version-support-policy.md -->

## Introduction

| Version | New features               | Bug fixes                          | Security fixes                      |
|---------|----------------------------|------------------------------------|-------------------------------------|
| 1.x     | ❌ ended Jan 1 2022         | ❌ ended Jan 1 2025                 | ❌ ended Jul 1 2025                  |
| 2.x     | ❌ ended Jul 1 2023         | ❌ ended Jan 1 2025                 | ✅ until Jan 1 2026                  |
| 3.x     | ❌ ended Aug 1 2024         | ✅ until Aug 1 2026                 | ✅ until Jan 1 2028                  |
| 4.x     | ✅ until 5.x stable release | ✅ ~1 year after 5.x stable release | ✅ ~2 years after 5.x stable release |

## New features

Pull requests for new features are only accepted for the latest major version, except in special circumstances. Once a new major version is released, the Filament team will no longer accept pull requests for new features in previous versions. Any open pull requests will either be redirected to target the latest major version or closed, depending on conflicts with the new target branch.

## Bug fixes

After a major version is released, the Filament team will continue to merge pull requests for bug fixes in the previous major version for a year. After this period, pull requests for that version will no longer be accepted.

The Filament team processes bug reports for supported versions in chronological order, though critical bugs may be prioritized. Bug fixes are typically developed only for the latest major version. However, contributors can backport fixes to other supported versions by submitting pull requests.

## Security fixes

The Filament team currently plans to continue providing security fixes for major versions for at least two years.

If you discover a security vulnerability in Filament, please [report it through GitHub](https://github.com/filamentphp/filament/security/advisories). All security vulnerabilities will be addressed promptly.

Please note that while a Filament version may receive security fixes, its underlying dependencies (PHP, Laravel, and Livewire) may no longer be supported. Therefore, applications using older versions of Filament could still be vulnerable to security issues in these dependencies.

---

<!-- Source: 05-viewing-records.md -->

## Creating a resource with a View page

To create a new resource with a View page, you can use the `--view` flag:

```bash
php artisan make:filament-resource User --view
```

## Using an infolist instead of a disabled form

By default, the View page will display a disabled form with the record's data. If you preferred to display the record's data in an "infolist", you can define an `infolist()` method on the resource class:

```php
use Filament\Infolists;
use Filament\Schemas\Schema;

public static function infolist(Schema $schema): Schema
{
    return $schema
        ->components([
            Infolists\Components\TextEntry::make('name'),
            Infolists\Components\TextEntry::make('email'),
            Infolists\Components\TextEntry::make('notes')
                ->columnSpanFull(),
        ]);
}
```

The `components()` method is used to define the structure of your infolist. It is an array of [entries](../infolists#available-entries) and [layout components](../schemas/layout#available-layout-components), in the order they should appear in your infolist.

Check out the Infolists docs for a [guide](../infolists) on how to build infolists with Filament.

## Adding a View page to an existing resource

If you want to add a View page to an existing resource, create a new page in your resource's `Pages` directory:

```bash
php artisan make:filament-page ViewUser --resource=UserResource --type=ViewRecord
```

You must register this new page in your resource's `getPages()` method:

```php
public static function getPages(): array
{
    return [
        'index' => Pages\ListUsers::route('/'),
        'create' => Pages\CreateUser::route('/create'),
        'view' => Pages\ViewUser::route('/{record}'),
        'edit' => Pages\EditUser::route('/{record}/edit'),
    ];
}
```

## Viewing records in modals

If your resource is simple, you may wish to view records in modals rather than on the [View page](viewing-records). If this is the case, you can just [delete the view page](overview#deleting-resource-pages).

If your resource doesn't contain a `ViewAction`, you can add one to the `$table->recordActions()` array:

```php
use Filament\Actions\ViewAction;
use Filament\Tables\Table;

public static function table(Table $table): Table
{
    return $table
        ->columns([
            // ...
        ])
        ->recordActions([
            ViewAction::make(),
            // ...
        ]);
}
```

## Customizing data before filling the form

You may wish to modify the data from a record before it is filled into the form. To do this, you may define a `mutateFormDataBeforeFill()` method on the View page class to modify the `$data` array, and return the modified version before it is filled into the form:

```php
protected function mutateFormDataBeforeFill(array $data): array
{
    $data['user_id'] = auth()->id();

    return $data;
}
```

Alternatively, if you're viewing records in a modal action, check out the [Actions documentation](../actions/view#customizing-data-before-filling-the-form).

## Lifecycle hooks

Hooks may be used to execute code at various points within a page's lifecycle, like before a form is filled. To set up a hook, create a protected method on the View page class with the name of the hook:

```php
use Filament\Resources\Pages\ViewRecord;

class ViewUser extends ViewRecord
{
    // ...

    protected function beforeFill(): void
    {
        // Runs before the disabled form fields are populated from the database. Not run on pages using an infolist.
    }

    protected function afterFill(): void
    {
        // Runs after the disabled form fields are populated from the database. Not run on pages using an infolist.
    }
}
```

## Authorization

For authorization, Filament will observe any [model policies](https://laravel.com/docs/authorization#creating-policies) that are registered in your app.

Users may access the View page if the `view()` method of the model policy returns `true`.

## Creating another View page

One View page may not be enough space to allow users to navigate a lot of information. You can create as many View pages for a resource as you want. This is especially useful if you are using [resource sub-navigation](overview#resource-sub-navigation), as you are then easily able to switch between the different View pages.

To create a View page, you should use the `make:filament-page` command:

```bash
php artisan make:filament-page ViewCustomerContact --resource=CustomerResource --type=ViewRecord
```

You must register this new page in your resource's `getPages()` method:

```php
public static function getPages(): array
{
    return [
        'index' => Pages\ListCustomers::route('/'),
        'create' => Pages\CreateCustomer::route('/create'),
        'view' => Pages\ViewCustomer::route('/{record}'),
        'view-contact' => Pages\ViewCustomerContact::route('/{record}/contact'),
        'edit' => Pages\EditCustomer::route('/{record}/edit'),
    ];
}
```

Now, you can define the `infolist()` or `form()` for this page, which can contain other components that are not present on the main View page:

```php
use Filament\Schemas\Schema;

public function infolist(Schema $schema): Schema
{
    return $schema
        ->components([
            // ...
        ]);
}
```

## Customizing relation managers for a specific view page

You can specify which relation managers should appear on a view page by defining a `getAllRelationManagers()` method:

```php
protected function getAllRelationManagers(): array
{
    return [
        CustomerAddressesRelationManager::class,
        CustomerContactsRelationManager::class,
    ];
}
```

This is useful when you have [multiple view pages](#creating-another-view-page) and need different relation managers on
each page:

```php
// ViewCustomer.php
protected function getAllRelationManagers(): array
{
    return [
        RelationManagers\OrdersRelationManager::class,
        RelationManagers\SubscriptionsRelationManager::class,
    ];
}

// ViewCustomerContact.php 
protected function getAllRelationManagers(): array
{
    return [
        RelationManagers\ContactsRelationManager::class,
        RelationManagers\AddressesRelationManager::class,
    ];
}
```

If `getAllRelationManagers()` isn't defined, any relation managers defined in the resource will be used.

## Adding view pages to resource sub-navigation

If you're using [resource sub-navigation](overview#resource-sub-navigation), you can register this page as normal in `getRecordSubNavigation()` of the resource:

```php
use App\Filament\Resources\Customers\Pages;
use Filament\Resources\Pages\Page;

public static function getRecordSubNavigation(Page $page): array
{
    return $page->generateNavigationItems([
        // ...
        Pages\ViewCustomerContact::class,
    ]);
}
```

## Custom page content

Each page in Filament has its own [schema](../schemas), which defines the overall structure and content. You can override the schema for the page by defining a `content()` method on it. The `content()` method for the View page contains the following components by default:

```php
use Filament\Schemas\Schema;

public function content(Schema $schema): Schema
{
    return $schema
        ->components([
            $this->hasInfolist() // This method returns `true` if the page has an infolist defined
                ? $this->getInfolistContentComponent() // This method returns a component to display the infolist that is defined in this resource
                : $this->getFormContentComponent(), // This method returns a component to display the form that is defined in this resource
            $this->getRelationManagersContentComponent(), // This method returns a component to display the relation managers that are defined in this resource
        ]);
}
```

Inside the `components()` array, you can insert any [schema component](../schemas). You can reorder the components by changing the order of the array or remove any of the components that are not needed.

### Using a custom Blade view

For further customization opportunities, you can override the static `$view` property on the page class to a custom view in your app:

```php
protected string $view = 'filament.resources.users.pages.view-user';
```

This assumes that you have created a view at `resources/views/filament/resources/users/pages/view-user.blade.php`:

```blade
<x-filament-panels::page>
    {{-- `$this->getRecord()` will return the current Eloquent record for this page --}}
    
    {{ $this->content }} {{-- This will render the content of the page defined in the `content()` method, which can be removed if you want to start from scratch --}}
</x-filament-panels::page>
```

---

<!-- Source: 06-contributing.md -->

import Aside from "@components/Aside.astro"

<Aside variant="info">
    Parts of this guide are adapted from [Laravel's contribution guide](https://laravel.com/docs/contributions), which served as valuable inspiration.
</Aside>

## Reporting bugs

If you discover a bug in Filament, please report it by opening an issue on our [GitHub repository](https://github.com/filamentphp/filament/issues/new/choose). Before opening an issue, search through the [existing issues](https://github.com/filamentphp/filament/issues?q=is%3Aissue) to check if the bug has already been reported.

Please include as much information as possible, particularly the version numbers of packages in your application. You can use this Artisan command in your application to open a new issue with all the correct versions automatically pre-filled:

```bash
php artisan make:filament-issue
```

When creating an issue, we require a "reproduction repository". **Please do not link to your actual project**. Instead, what we need is a _minimal_ reproduction in a fresh project without any unnecessary code. This means it doesn't matter if your real project is private/confidential since we want a link to a separate, isolated reproduction. This allows us to fix the problem much quicker. **Issues will be automatically closed and not reviewed if this is missing, to preserve maintainer time and ensure the process is fair for those who put effort into reporting.** If you believe a reproduction repository is not suitable for the issue, which is a very rare case, please `@danharrin` and explain why. Saying "it's just a simple issue" is not an excuse for not creating a repository! [Need a head start? We have a template Filament project for you.](https://unitedbycode.com/filament-issue)

Remember, bug reports are created in the hope that others with the same problem will be able to collaborate with you on solving it. Do not expect that the bug report will automatically receive attention or that others will jump to fix it. Creating a bug report serves to help yourself and others start on the path of fixing the problem.

## Development of new features

If you would like to propose a new feature or improvement to Filament, you may use our [discussion forum](https://github.com/filamentphp/filament/discussions) hosted on GitHub. If you intend to implement the feature yourself in a pull request, we advise you to `@danharrin` (a core maintainer of Filament) in your feature discussion beforehand and ask if it is suitable for the framework to prevent wasting your time.

## Development of plugins

If you would like to develop a plugin for Filament, please refer to the [plugin development section](../plugins) in the documentation. Our [Discord](https://filamentphp.com/discord) server is also a great place to ask questions and get help with plugin development. You can start a conversation in the [`#plugin-developers-chat`](https://discord.com/channels/883083792112300104/970354547723730955) channel.

You can also [submit your plugin to be advertised on the Filament website](https://github.com/filamentphp/filamentphp.com/blob/main/README.md#contributing).

## Developing with a local copy of Filament

If you want to contribute to the Filament packages, then you may want to test it in a real Laravel project:

- Fork [the GitHub repository](https://github.com/filamentphp/filament) to your GitHub account.
- Create a Laravel app locally.
- Clone your fork into your Laravel app's root directory.
- In the `/filament` directory, create a branch for your fix, e.g. `fix/error-message`.

Install the packages in your app's `composer.json`:

```jsonc
{
    // ...
    "require": {
        "filament/filament": "*",
    },
    "minimum-stability": "dev",
    "repositories": [
        {
            "type": "path",
            "url": "filament/packages/*"
        }
    ],
    // ...
}
```

Now, run `composer update`.

Once you've finished making changes, you can commit them and submit a pull request to [the GitHub repository](https://github.com/filamentphp/filament).

## Checking for outdated translations

To check for outdated translations, you can use our Translation Tool. Clone the Filament repository, install the dependencies for the commands and then run the command.

```bash
# Clone
git clone git@github.com:filamentphp/filament.git

# Install dependencies
composer install

# Run the tool  
./bin/translation-tool.php
```

First select "List outdated translations" as the command and then choose the locale you want to check. This command will show you which translations are missing for the specified locale. You can then submit a pull request with the missing translations to [the GitHub repository](https://github.com/filamentphp/filament).

## Security vulnerabilities

If you discover a security vulnerability within Filament, please [report it through GitHub](https://github.com/filamentphp/filament/security/advisories). All security vulnerabilities will be promptly addressed. Please see our [version support policy](version-support-policy) to understand which versions are currently under maintenance.

## Code of Conduct

Please note that Filament is released with a [Contributor Code of Conduct](https://github.com/filamentphp/filament/blob/4.x/CODE_OF_CONDUCT.md). By participating in this project, you agree to abide by its terms.

---

<!-- Source: 06-deleting-records.md -->

## Handling soft-deletes

## Creating a resource with soft-delete

By default, you will not be able to interact with deleted records in the app. If you'd like to add functionality to restore, force-delete and filter trashed records in your resource, use the `--soft-deletes` flag when generating the resource:

```bash
php artisan make:filament-resource Customer --soft-deletes
```

## Adding soft-deletes to an existing resource

Alternatively, you may add soft-deleting functionality to an existing resource.

Firstly, you must update the resource:

```php
use Filament\Actions\BulkActionGroup;
use Filament\Actions\DeleteAction;
use Filament\Actions\DeleteBulkAction;
use Filament\Actions\ForceDeleteAction;
use Filament\Actions\ForceDeleteBulkAction;
use Filament\Actions\RestoreAction;
use Filament\Actions\RestoreBulkAction;
use Filament\Tables\Filters\TrashedFilter;
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\SoftDeletingScope;

public static function table(Table $table): Table
{
    return $table
        ->columns([
            // ...
        ])
        ->filters([
            TrashedFilter::make(),
            // ...
        ])
        ->recordActions([
            // You may add these actions to your table if you're using a simple
            // resource, or you just want to be able to delete records without
            // leaving the table.
            DeleteAction::make(),
            ForceDeleteAction::make(),
            RestoreAction::make(),
            // ...
        ])
        ->toolbarActions([
            BulkActionGroup::make([
                DeleteBulkAction::make(),
                ForceDeleteBulkAction::make(),
                RestoreBulkAction::make(),
                // ...
            ]),
        ]);
}

public static function getRecordRouteBindingEloquentQuery(): Builder
{
    return parent::getRecordRouteBindingEloquentQuery()
        ->withoutGlobalScopes([
            SoftDeletingScope::class,
        ]);
}
```

Now, update the Edit page class if you have one:

```php
use Filament\Actions;

protected function getHeaderActions(): array
{
    return [
        Actions\DeleteAction::make(),
        Actions\ForceDeleteAction::make(),
        Actions\RestoreAction::make(),
        // ...
    ];
}
```

## Deleting records on the List page

By default, you can bulk-delete records in your table. You may also wish to delete single records, using a `DeleteAction`:

```php
use Filament\Actions\DeleteAction;
use Filament\Tables\Table;

public static function table(Table $table): Table
{
    return $table
        ->columns([
            // ...
        ])
        ->recordActions([
            // ...
            DeleteAction::make(),
        ]);
}
```

## Authorization

For authorization, Filament will observe any [model policies](https://laravel.com/docs/authorization#creating-policies) that are registered in your app.

Users may delete records if the `delete()` method of the model policy returns `true`.

They also have the ability to bulk-delete records if the `deleteAny()` method of the policy returns `true`. Filament uses the `deleteAny()` method because iterating through multiple records and checking the `delete()` policy is not very performant.

You can use the `authorizeIndividualRecords()` method on the `BulkDeleteAction` to check the `delete()` policy for each record individually.

### Authorizing soft-deletes

The `forceDelete()` policy method is used to prevent a single soft-deleted record from being force-deleted. `forceDeleteAny()` is used to prevent records from being bulk force-deleted. Filament uses the `forceDeleteAny()` method because iterating through multiple records and checking the `forceDelete()` policy is not very performant.

The `restore()` policy method is used to prevent a single soft-deleted record from being restored. `restoreAny()` is used to prevent records from being bulk restored. Filament uses the `restoreAny()` method because iterating through multiple records and checking the `restore()` policy is not very performant.

---

<!-- Source: 06-testing-notifications.md -->

## Testing session notifications

To check if a notification was sent using the session, use the `assertNotified()` helper:

```php
use function Pest\Livewire\livewire;

it('sends a notification', function () {
    livewire(CreatePost::class)
        ->assertNotified();
});
```

```php
use Filament\Notifications\Notification;

it('sends a notification', function () {
    Notification::assertNotified();
});
```

```php
use function Filament\Notifications\Testing\assertNotified;

it('sends a notification', function () {
    assertNotified();
});
```

You may optionally pass a notification title to test for:

```php
use Filament\Notifications\Notification;
use function Pest\Livewire\livewire;

it('sends a notification', function () {
    livewire(CreatePost::class)
        ->assertNotified('Unable to create post');
});
```

Or test if the exact notification was sent:

```php
use Filament\Notifications\Notification;
use function Pest\Livewire\livewire;

it('sends a notification', function () {
    livewire(CreatePost::class)
        ->assertNotified(
            Notification::make()
                ->danger()
                ->title('Unable to create post')
                ->body('Something went wrong.'),
        );
});
```

Conversely, you can assert that a notification was not sent:

```php
use Filament\Notifications\Notification;
use function Pest\Livewire\livewire;

it('does not send a notification', function () {
    livewire(CreatePost::class)
        ->assertNotNotified()
        // or
        ->assertNotNotified('Unable to create post')
        // or
        ->assertNotNotified(
            Notification::make()
                ->danger()
                ->title('Unable to create post')
                ->body('Something went wrong.'),
        );
```

---

<!-- Source: 07-managing-relationships.md -->

import Aside from "@components/Aside.astro"

## Choosing the right tool for the job

Filament provides many ways to manage relationships in the app. Which feature you should use depends on the type of relationship you are managing, and which UI you are looking for.

### Relation managers - interactive tables underneath your resource forms

<Aside variant="info">
    These are compatible with `HasMany`, `HasManyThrough`, `BelongsToMany`, `MorphMany` and `MorphToMany` relationships.
</Aside>

[Relation managers](#creating-a-relation-manager) are interactive tables that allow administrators to list, create, attach, associate, edit, detach, dissociate and delete related records without leaving the resource's Edit or View page.

### Select & checkbox list - choose from existing records or create a new one

<Aside variant="info">
    These are compatible with `BelongsTo`, `MorphTo` and `BelongsToMany` relationships.
</Aside>

Using a [select](../forms/select#integrating-with-an-eloquent-relationship), users will be able to choose from a list of existing records. You may also [add a button that allows you to create a new record inside a modal](../forms/select#creating-new-records), without leaving the page.

When using a `BelongsToMany` relationship with a select, you'll be able to select multiple options, not just one. Records will be automatically added to your pivot table when you submit the form. If you wish, you can swap out the multi-select dropdown with a simple [checkbox list](../forms/checkbox-list#integrating-with-an-eloquent-relationship). Both components work in the same way.

### Repeaters - CRUD multiple related records inside the owner's form

<Aside variant="info">
    These are compatible with `HasMany` and `MorphMany` relationships.
</Aside>

[Repeaters](../forms/repeater#integrating-with-an-eloquent-relationship) are standard form components, which can render a repeatable set of fields infinitely. They can be hooked up to a relationship, so records are automatically read, created, updated, and deleted from the related table. They live inside the main form schema, and can be used inside resource pages, as well as nesting within action modals.

From a UX perspective, this solution is only suitable if your related model only has a few fields. Otherwise, the form can get very long.

### Layout form components - saving form fields to a single relationship

<Aside variant="info">
    These are compatible with `BelongsTo`, `HasOne` and `MorphOne` relationships.
</Aside>

All layout form components ([Grid](../schemas/layouts#grid-component), [Section](../schemas/sections), [Fieldset](../schemas/layouts#fieldset-component), etc.) have a [`relationship()` method](../forms/overview#saving-data-to-relationships). When you use this, all fields within that layout are saved to the related model instead of the owner's model:

```php
use Filament\Forms\Components\FileUpload;
use Filament\Forms\Components\Textarea;
use Filament\Forms\Components\TextInput;
use Filament\Schemas\Components\Fieldset;

Fieldset::make('Metadata')
    ->relationship('metadata')
    ->schema([
        TextInput::make('title'),
        Textarea::make('description'),
        FileUpload::make('image'),
    ])
```

In this example, the `title`, `description` and `image` are automatically loaded from the `metadata` relationship, and saved again when the form is submitted. If the `metadata` record does not exist, it is automatically created.

This feature is explained more in depth in the [Forms documentation](../forms/overview#saving-data-to-relationships). Please visit that page for more information about how to use it.

## Creating a relation manager

To create a relation manager, you can use the `make:filament-relation-manager` command:

```bash
php artisan make:filament-relation-manager CategoryResource posts title
```

- `CategoryResource` is the name of the resource class for the owner (parent) model.
- `posts` is the name of the relationship you want to manage.
- `title` is the name of the attribute that will be used to identify posts.

This will create a `CategoryResource/RelationManagers/PostsRelationManager.php` file. This contains a class where you are able to define a [form](overview#resource-forms) and [table](overview#resource-tables) for your relation manager:

```php
use Filament\Forms;
use Filament\Schemas\Schema;
use Filament\Tables;
use Filament\Tables\Table;

public function form(Schema $schema): Schema
{
    return $schema
        ->components([
            Forms\Components\TextInput::make('title')->required(),
            // ...
        ]);
}

public function table(Table $table): Table
{
    return $table
        ->columns([
            Tables\Columns\TextColumn::make('title'),
            // ...
        ]);
}
```

You must register the new relation manager in your resource's `getRelations()` method:

```php
public static function getRelations(): array
{
    return [
        RelationManagers\PostsRelationManager::class,
    ];
}
```

Once a table and form have been defined for the relation manager, visit the [Edit](editing-records) or [View](viewing-records) page of your resource to see it in action.

### Customizing the relation manager's URL parameter

If you pass a key to the array returned from `getRelations()`, it will be used in the URL for that relation manager when switching between multiple relation managers. For example, you can pass `posts` to use `?relation=posts` in the URL instead of a numeric array index:

```php
public static function getRelations(): array
{
    return [
        'posts' => RelationManagers\PostsRelationManager::class,
    ];
}
```

### Read-only mode

Relation managers are usually displayed on either the Edit or View page of a resource. On the View page, Filament will automatically hide all actions that modify the relationship, such as create, edit, and delete. We call this "read-only mode", and it is there by default to preserve the read-only behavior of the View page. However, you can disable this behavior, by overriding the `isReadOnly()` method on the relation manager class to return `false` all the time:

```php
public function isReadOnly(): bool
{
    return false;
}
```

Alternatively, if you hate this functionality, you can disable it for all relation managers at once in the panel [configuration](../panel-configuration):

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->readOnlyRelationManagersOnResourceViewPagesByDefault(false);
}
```

### Unconventional inverse relationship names

For inverse relationships that do not follow Laravel's naming guidelines, you may wish to use the `inverseRelationship()` method on the table:

```php
use Filament\Tables;
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->columns([
            Tables\Columns\TextColumn::make('title'),
            // ...
        ])
        ->inverseRelationship('section'); // Since the inverse related model is `Category`, this is normally `category`, not `section`.
}
```

### Handling soft-deletes

By default, you will not be able to interact with deleted records in the relation manager. If you'd like to add functionality to restore, force-delete and filter trashed records in your relation manager, use the `--soft-deletes` flag when generating the relation manager:

```bash
php artisan make:filament-relation-manager CategoryResource posts title --soft-deletes
```

You can find out more about soft-deleting [here](#deleting-records).

## Listing related records

Related records will be listed in a table. The entire relation manager is based around this table, which contains actions to [create](#creating-related-records), [edit](#editing-related-records), [attach / detach](#attaching-and-detaching-records), [associate / dissociate](#associating-and-dissociating-records), and delete records.

You may use any features of the [Table Builder](../tables) to customize relation managers.

### Listing with pivot attributes

For `BelongsToMany` and `MorphToMany` relationships, you may also add pivot table attributes. For example, if you have a `TeamsRelationManager` for your `UserResource`, and you want to add the `role` pivot attribute to the table, you can use:

```php
use Filament\Tables;

public function table(Table $table): Table
{
    return $table
        ->columns([
            Tables\Columns\TextColumn::make('name'),
            Tables\Columns\TextColumn::make('role'),
        ]);
}
```

Please ensure that any pivot attributes are listed in the `withPivot()` method of the relationship *and* inverse relationship.

## Creating related records

### Creating with pivot attributes

For `BelongsToMany` and `MorphToMany` relationships, you may also add pivot table attributes. For example, if you have a `TeamsRelationManager` for your `UserResource`, and you want to add the `role` pivot attribute to the create form, you can use:

```php
use Filament\Forms;
use Filament\Schemas\Schema;

public function form(Schema $schema): Schema
{
    return $schema
        ->components([
            Forms\Components\TextInput::make('name')->required(),
            Forms\Components\TextInput::make('role')->required(),
            // ...
        ]);
}
```

Please ensure that any pivot attributes are listed in the `withPivot()` method of the relationship *and* inverse relationship.

### Customizing the `CreateAction`

To learn how to customize the `CreateAction`, including mutating the form data, changing the notification, and adding lifecycle hooks, please see the [Actions documentation](../actions/create).

## Editing related records

### Editing with pivot attributes

For `BelongsToMany` and `MorphToMany` relationships, you may also edit pivot table attributes. For example, if you have a `TeamsRelationManager` for your `UserResource`, and you want to add the `role` pivot attribute to the edit form, you can use:

```php
use Filament\Forms;
use Filament\Schemas\Schema;

public function form(Schema $schema): Schema
{
    return $schema
        ->components([
            Forms\Components\TextInput::make('name')->required(),
            Forms\Components\TextInput::make('role')->required(),
            // ...
        ]);
}
```

Please ensure that any pivot attributes are listed in the `withPivot()` method of the relationship *and* inverse relationship.

### Customizing the `EditAction`

To learn how to customize the `EditAction`, including mutating the form data, changing the notification, and adding lifecycle hooks, please see the [Actions documentation](../actions/edit).

## Attaching and detaching records

Filament is able to attach and detach records for `BelongsToMany` and `MorphToMany` relationships.

When generating your relation manager, you may pass the `--attach` flag to also add `AttachAction`, `DetachAction` and `DetachBulkAction` to the table:

```bash
php artisan make:filament-relation-manager CategoryResource posts title --attach
```

Alternatively, if you've already generated your resource, you can just add the actions to the `$table` arrays:

```php
use Filament\Actions\AttachAction;
use Filament\Actions\BulkActionGroup;
use Filament\Actions\DetachAction;
use Filament\Actions\DetachBulkAction;
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->columns([
            // ...
        ])
        ->headerActions([
            // ...
            AttachAction::make(),
        ])
        ->recordActions([
            // ...
            DetachAction::make(),
        ])
        ->toolbarActions([
            BulkActionGroup::make([
                // ...
                DetachBulkAction::make(),
            ]),
        ]);
}
```

### Preloading the attachment modal select options

By default, as you search for a record to attach, options will load from the database via AJAX. If you wish to preload these options when the form is first loaded instead, you can use the `preloadRecordSelect()` method of `AttachAction`:

```php
use Filament\Actions\AttachAction;

AttachAction::make()
    ->preloadRecordSelect()
```

### Attaching with pivot attributes

When you attach record with the `Attach` button, you may wish to define a custom form to add pivot attributes to the relationship:

```php
use Filament\Actions\AttachAction;
use Filament\Forms;

AttachAction::make()
    ->schema(fn (AttachAction $action): array => [
        $action->getRecordSelect(),
        Forms\Components\TextInput::make('role')->required(),
    ])
```

In this example, `$action->getRecordSelect()` returns the select field to pick the record to attach. The `role` text input is then saved to the pivot table's `role` column.

Please ensure that any pivot attributes are listed in the `withPivot()` method of the relationship *and* inverse relationship.

### Scoping the options to attach

You may want to scope the options available to `AttachAction`:

```php
use Filament\Actions\AttachAction;
use Illuminate\Database\Eloquent\Builder;

AttachAction::make()
    ->recordSelectOptionsQuery(fn (Builder $query) => $query->whereBelongsTo(auth()->user()))
```

### Searching the options to attach across multiple columns

By default, the options available to `AttachAction` will be searched in the `recordTitleAttribute()` of the table. If you wish to search across multiple columns, you can use the `recordSelectSearchColumns()` method:

```php
use Filament\Actions\AttachAction;

AttachAction::make()
    ->recordSelectSearchColumns(['title', 'description'])
```

### Attaching multiple records

The `multiple()` method on the `AttachAction` component allows you to select multiple values:

```php
use Filament\Actions\AttachAction;

AttachAction::make()
    ->multiple()
```

### Customizing the select field in the attached modal

You may customize the select field object that is used during attachment by passing a function to the `recordSelect()` method:

```php
use Filament\Actions\AttachAction;
use Filament\Forms\Components\Select;

AttachAction::make()
    ->recordSelect(
        fn (Select $select) => $select->placeholder('Select a post'),
    )
```

### Handling duplicates

By default, you will not be allowed to attach a record more than once. This is because you must also set up a primary `id` column on the pivot table for this feature to work.

Please ensure that the `id` attribute is listed in the `withPivot()` method of the relationship *and* inverse relationship.

Finally, add the `allowDuplicates()` method to the table:

```php
public function table(Table $table): Table
{
    return $table
        ->allowDuplicates();
}
```

### Improving the performance of detach bulk actions

By default, the `DetachBulkAction` will load all Eloquent records into memory, before looping over them and detaching them one by one.

If you are detaching a large number of records, you may want to use the `chunkSelectedRecords()` method to fetch a smaller number of records at a time. This will reduce the memory usage of your application:

```php
use Filament\Actions\DetachBulkAction;

DetachBulkAction::make()
    ->chunkSelectedRecords(250)
```

Filament loads Eloquent records into memory before detaching them for two reasons:

- To allow individual records in the collection to be authorized with a model policy before detaching (using `authorizeIndividualRecords('delete')`, for example).
- To ensure that model events are run when detaching records, such as the `deleting` and `deleted` events in a model observer.

If you do not require individual record policy authorization and model events, you can use the `fetchSelectedRecords(false)` method, which will not fetch the records into memory before detaching them, and instead will detach them in a single query:

```php
use Filament\Actions\DetachBulkAction;

DetachBulkAction::make()
    ->fetchSelectedRecords(false)
```

## Associating and dissociating records

Filament is able to associate and dissociate records for `HasMany` and `MorphMany` relationships.

When generating your relation manager, you may pass the `--associate` flag to also add `AssociateAction`, `DissociateAction` and `DissociateBulkAction` to the table:

```bash
php artisan make:filament-relation-manager CategoryResource posts title --associate
```

Alternatively, if you've already generated your resource, you can just add the actions to the `$table` arrays:

```php
use Filament\Actions\AssociateAction;
use Filament\Actions\BulkActionGroup;
use Filament\Actions\DissociateAction;
use Filament\Actions\DissociateBulkAction;
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->columns([
            // ...
        ])
        ->headerActions([
            // ...
            AssociateAction::make(),
        ])
        ->recordActions([
            // ...
            DissociateAction::make(),
        ])
        ->toolbarActions([
            BulkActionGroup::make([
                // ...
                DissociateBulkAction::make(),
            ]),
        ]);
}
```

### Preloading the associate modal select options

By default, as you search for a record to associate, options will load from the database via AJAX. If you wish to preload these options when the form is first loaded instead, you can use the `preloadRecordSelect()` method of `AssociateAction`:

```php
use Filament\Actions\AssociateAction;

AssociateAction::make()
    ->preloadRecordSelect()
```

### Scoping the options to associate

You may want to scope the options available to `AssociateAction`:

```php
use Filament\Actions\AssociateAction;
use Illuminate\Database\Eloquent\Builder;

AssociateAction::make()
    ->recordSelectOptionsQuery(fn (Builder $query) => $query->whereBelongsTo(auth()->user()))
```

### Searching the options to associate across multiple columns

By default, the options available to `AssociateAction` will be searched in the `recordTitleAttribute()` of the table. If you wish to search across multiple columns, you can use the `recordSelectSearchColumns()` method:

```php
use Filament\Actions\AssociateAction;

AssociateAction::make()
    ->recordSelectSearchColumns(['title', 'description'])
```

### Associating multiple records

The `multiple()` method on the `AssociateAction` component allows you to select multiple values:

```php
use Filament\Actions\AssociateAction;

AssociateAction::make()
    ->multiple()
```

### Customizing the select field in the associate modal

You may customize the select field object that is used during association by passing a function to the `recordSelect()` method:

```php
use Filament\Actions\AssociateAction;
use Filament\Forms\Components\Select;

AssociateAction::make()
    ->recordSelect(
        fn (Select $select) => $select->placeholder('Select a post'),
    )
```

### Improving the performance of dissociate bulk actions

By default, the `DissociateBulkAction` will load all Eloquent records into memory, before looping over them and dissociating them one by one.

If you are dissociating a large number of records, you may want to use the `chunkSelectedRecords()` method to fetch a smaller number of records at a time. This will reduce the memory usage of your application:

```php
use Filament\Actions\DissociateBulkAction;

DissociateBulkAction::make()
    ->chunkSelectedRecords(250)
```

Filament loads Eloquent records into memory before dissociating them for two reasons:

- To allow individual records in the collection to be authorized with a model policy before dissociation (using `authorizeIndividualRecords('update')`, for example).
- To ensure that model events are run when dissociating records, such as the `updating` and `updated` events in a model observer.

If you do not require individual record policy authorization and model events, you can use the `fetchSelectedRecords(false)` method, which will not fetch the records into memory before dissociating them, and instead will dissociate them in a single query:

```php
use Filament\Actions\DissociateBulkAction;

DissociateBulkAction::make()
    ->fetchSelectedRecords(false)
```

## Viewing related records

When generating your relation manager, you may pass the `--view` flag to also add a `ViewAction` to the table:

```bash
php artisan make:filament-relation-manager CategoryResource posts title --view
```

Alternatively, if you've already generated your relation manager, you can just add the `ViewAction` to the `$table->recordActions()` array:

```php
use Filament\Actions\ViewAction;
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->columns([
            // ...
        ])
        ->recordActions([
            ViewAction::make(),
            // ...
        ]);
}
```

## Deleting related records

By default, you will not be able to interact with deleted records in the relation manager. If you'd like to add functionality to restore, force-delete and filter trashed records in your relation manager, use the `--soft-deletes` flag when generating the relation manager:

```bash
php artisan make:filament-relation-manager CategoryResource posts title --soft-deletes
```

Alternatively, you may add soft-deleting functionality to an existing relation manager:

```php
use Filament\Actions\DeleteAction;
use Filament\Actions\DeleteBulkAction;
use Filament\Actions\ForceDeleteAction;
use Filament\Actions\ForceDeleteBulkAction;
use Filament\Actions\RestoreAction;
use Filament\Actions\RestoreBulkAction;
use Filament\Tables\Filters\TrashedFilter;
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\SoftDeletingScope;

public function table(Table $table): Table
{
    return $table
        ->modifyQueryUsing(fn (Builder $query) => $query->withoutGlobalScopes([
            SoftDeletingScope::class,
        ]))
        ->columns([
            // ...
        ])
        ->filters([
            TrashedFilter::make(),
            // ...
        ])
        ->recordActions([
            DeleteAction::make(),
            ForceDeleteAction::make(),
            RestoreAction::make(),
            // ...
        ])
        ->toolbarActions([
            BulkActionGroup::make([
                DeleteBulkAction::make(),
                ForceDeleteBulkAction::make(),
                RestoreBulkAction::make(),
                // ...
            ]),
        ]);
}
```

### Customizing the `DeleteAction`

To learn how to customize the `DeleteAction`, including changing the notification and adding lifecycle hooks, please see the [Actions documentation](../actions/delete).

## Importing related records

The [`ImportAction`](../actions/import) can be added to the header of a relation manager to import records. In this case, you probably want to tell the importer which owner these new records belong to. You can use [import options](../actions/import#using-import-options) to pass through the ID of the owner record:

```php
ImportAction::make()
    ->importer(ProductImporter::class)
    ->options(['categoryId' => $this->getOwnerRecord()->getKey()])
```

Now, in the importer class, you can associate the owner in a one-to-many relationship with the imported record:

```php
public function resolveRecord(): ?Product
{
    $product = Product::firstOrNew([
        'sku' => $this->data['sku'],
    ]);
    
    $product->category()->associate($this->options['categoryId']);
    
    return $product;
}
```

Alternatively, you can attach the record in a many-to-many relationship using the `afterSave()` hook of the importer:

```php
protected function afterSave(): void
{
    $this->record->categories()->syncWithoutDetaching([$this->options['categoryId']]);
}
```

## Accessing the relationship's owner record

Relation managers are Livewire components. When they are first loaded, the owner record (the Eloquent record which serves as a parent - the main resource model) is saved into a property. You can read this property using:

```php
$this->getOwnerRecord()
```

However, if you're inside a `static` method like `form()` or `table()`, `$this` isn't accessible. So, you may [use a callback](../forms/overview#field-utility-injection) to access the `$livewire` instance:

```php
use Filament\Forms;
use Filament\Resources\RelationManagers\RelationManager;
use Filament\Schemas\Schema;

public function form(Schema $schema): Schema
{
    return $schema
        ->components([
            Forms\Components\Select::make('store_id')
                ->options(function (RelationManager $livewire): array {
                    return $livewire->getOwnerRecord()->stores()
                        ->pluck('name', 'id')
                        ->toArray();
                }),
            // ...
        ]);
}
```

All methods in Filament accept a callback which you can access `$livewire->ownerRecord` in.

## Grouping relation managers

You may choose to group relation managers together into one tab. To do this, you may wrap multiple managers in a `RelationGroup` object, with a label:

```php
use Filament\Resources\RelationManagers\RelationGroup;

public static function getRelations(): array
{
    return [
        // ...
        RelationGroup::make('Contacts', [
            RelationManagers\IndividualsRelationManager::class,
            RelationManagers\OrganizationsRelationManager::class,
        ]),
        // ...
    ];
}
```

## Conditionally showing relation managers

By default, relation managers will be visible if the `viewAny()` method for the related model policy returns `true`.

You may use the `canViewForRecord()` method to determine if the relation manager should be visible for a specific owner record and page:

```php
use Illuminate\Database\Eloquent\Model;

public static function canViewForRecord(Model $ownerRecord, string $pageClass): bool
{
    return $ownerRecord->status === Status::Draft;
}
```

## Combining the relation manager tabs with the form

On the Edit or View page class, override the `hasCombinedRelationManagerTabsWithContent()` method:

```php
public function hasCombinedRelationManagerTabsWithContent(): bool
{
    return true;
}
```

### Customizing the content tab

On the Edit or View page class, override the `getContentTabComponent()` method, and use any [Tab](../schemas/tabs) customization methods:

```php
use Filament\Schemas\Components\Tabs\Tab;

public function getContentTabComponent(): Tab
{
    return Tab::make('Settings')
        ->icon('heroicon-m-cog');
}
```

### Setting the position of the form tab

By default, the form tab is rendered before the relation tabs. To render it after, you can override the `getContentTabPosition()` method on the Edit or View page class:

```php
use Filament\Resources\Pages\Enums\ContentTabPosition;

public function getContentTabPosition(): ?ContentTabPosition
{
    return ContentTabPosition::After;
}
```

## Customizing relation manager tabs

To customize the tab for a relation manager, override the `getTabComponent()` method, and use any [Tab](../schemas/tabs) customization methods:

```php
use Filament\Schemas\Components\Tabs\Tab;
use Illuminate\Database\Eloquent\Model;

public static function getTabComponent(Model $ownerRecord, string $pageClass): Tab
{
    return Tab::make('Blog posts')
        ->badge($ownerRecord->posts()->count())
        ->badgeColor('info')
        ->badgeTooltip('The number of posts in this category')
        ->icon('heroicon-m-document-text');
}
```

If you are using a [relation group](#grouping-relation-managers), you can use the `tab()` method:

```php
use Filament\Resources\RelationManagers\RelationGroup;
use Filament\Schemas\Components\Tabs\Tab;
use Illuminate\Database\Eloquent\Model;

RelationGroup::make('Contacts', [
    // ...
])
    ->tab(fn (Model $ownerRecord): Tab => Tab::make('Blog posts')
        ->badge($ownerRecord->posts()->count())
        ->badgeColor('info')
        ->badgeTooltip('The number of posts in this category')
        ->icon('heroicon-m-document-text'));
```

## Sharing a resource's form and table with a relation manager

You may decide that you want a resource's form and table to be identical to a relation manager's, and subsequently want to reuse the code you previously wrote. This is easy, by calling the `form()` and `table()` methods of the resource from the relation manager:

```php
use App\Filament\Resources\Blog\Posts\PostResource;
use Filament\Schemas\Schema;
use Filament\Tables\Table;

public function form(Schema $schema): Schema
{
    return PostResource::form($schema);
}

public function table(Table $table): Table
{
    return PostResource::table($table);
}
```

### Hiding a shared form component on the relation manager

If you're sharing a form component from the resource with the relation manager, you may want to hide it on the relation manager. This is especially useful if you want to hide a `Select` field for the owner record in the relation manager, since Filament will handle this for you anyway. To do this, you may use the `hiddenOn()` method, passing the name of the relation manager:

```php
use App\Filament\Resources\Blog\Posts\PostResource\RelationManagers\CommentsRelationManager;
use Filament\Forms\Components\Select;

Select::make('post_id')
    ->relationship('post', 'title')
    ->hiddenOn(CommentsRelationManager::class)
```

### Hiding a shared table column on the relation manager

If you're sharing a table column from the resource with the relation manager, you may want to hide it on the relation manager. This is especially useful if you want to hide a column for the owner record in the relation manager, since this is not appropriate when the owner record is already listed above the relation manager. To do this, you may use the `hiddenOn()` method, passing the name of the relation manager:

```php
use App\Filament\Resources\Blog\Posts\PostResource\RelationManagers\CommentsRelationManager;
use Filament\Tables\Columns\TextColumn;

TextColumn::make('post.title')
    ->hiddenOn(CommentsRelationManager::class)
```

### Hiding a shared table filter on the relation manager

If you're sharing a table filter from the resource with the relation manager, you may want to hide it on the relation manager. This is especially useful if you want to hide a filter for the owner record in the relation manager, since this is not appropriate when the table is already filtered by the owner record. To do this, you may use the `hiddenOn()` method, passing the name of the relation manager:

```php
use App\Filament\Resources\Blog\Posts\PostResource\RelationManagers\CommentsRelationManager;
use Filament\Tables\Filters\SelectFilter;

SelectFilter::make('post')
    ->relationship('post', 'title')
    ->hiddenOn(CommentsRelationManager::class)
```

### Overriding shared configuration on the relation manager

Any configuration that you make inside the resource can be overwritten on the relation manager. For example, if you wanted to disable pagination on the relation manager's inherited table but not the resource itself:

```php
use App\Filament\Resources\Blog\Posts\PostResource;
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return PostResource::table($table)
        ->paginated(false);
}
```

It is probably also useful to provide extra configuration on the relation manager if you wanted to add a header action to [create](#creating-related-records), [attach](#attaching-and-detaching-records), or [associate](#associating-and-dissociating-records) records in the relation manager:

```php
use App\Filament\Resources\Blog\Posts\PostResource;
use Filament\Actions\AttachAction;
use Filament\Actions\CreateAction;
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return PostResource::table($table)
        ->headerActions([
            CreateAction::make(),
            AttachAction::make(),
        ]);
}
```

## Customizing the relation manager Eloquent query

You can apply your own query constraints or [model scopes](https://laravel.com/docs/eloquent#query-scopes) that affect the entire relation manager. To do this, you can pass a function to the `modifyQueryUsing()` method of the table, inside which you can customize the query:

```php
use Filament\Tables;
use Illuminate\Database\Eloquent\Builder;

public function table(Table $table): Table
{
    return $table
        ->modifyQueryUsing(fn (Builder $query) => $query->where('is_active', true))
        ->columns([
            // ...
        ]);
}
```

## Customizing the relation manager title

To set the title of the relation manager, you can use the `$title` property on the relation manager class:

```php
protected static ?string $title = 'Posts';
```

To set the title of the relation manager dynamically, you can override the `getTitle()` method on the relation manager class:

```php
use Illuminate\Database\Eloquent\Model;

public static function getTitle(Model $ownerRecord, string $pageClass): string
{
    return __('relation-managers.posts.title');
}
```

The title will be reflected in the [heading of the table](../tables/overview#customizing-the-table-header), as well as the relation manager tab if there is more than one. If you want to customize the table heading independently, you can still use the `$table->heading()` method:

```php
use Filament\Tables;

public function table(Table $table): Table
{
    return $table
        ->heading('Posts')
        ->columns([
            // ...
        ]);
}
```

## Customizing the relation manager record title

The relation manager uses the concept of a "record title attribute" to determine which attribute of the related model should be used to identify it. When creating a relation manager, this attribute is passed as the third argument to the `make:filament-relation-manager` command:

```bash
php artisan make:filament-relation-manager CategoryResource posts title
```

In this example, the `title` attribute of the `Post` model will be used to identify a post in the relation manager.

This is mainly used by the action classes. For instance, when you [attach](#attaching-and-detaching-records) or [associate](#associating-and-dissociating-records) a record, the titles will be listed in the select field. When you [edit](#editing-related-records), [view](#viewing-related-records) or [delete](#deleting-related-records) a record, the title will be used in the header of the modal.

In some cases, you may want to concatenate multiple attributes together to form a title. You can do this by replacing the `recordTitleAttribute()` configuration method with `recordTitle()`, passing a function that transforms a model into a title:

```php
use App\Models\Post;
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->recordTitle(fn (Post $record): string => "{$record->title} ({$record->id})")
        ->columns([
            // ...
        ]);
}
```

If you're using `recordTitle()`, and you have an [associate action](#associating-and-dissociating-records) or [attach action](#attaching-and-detaching-records), you will also want to specify search columns for those actions:

```php
use Filament\Actions\AssociateAction;
use Filament\Actions\AttachAction;

AssociateAction::make()
    ->recordSelectSearchColumns(['title', 'id']);

AttachAction::make()
    ->recordSelectSearchColumns(['title', 'id'])
```

## Relation pages

Using a `ManageRelatedRecords` page is an alternative to using a relation manager, if you want to keep the functionality of managing a relationship separate from editing or viewing the owner record.

This feature is ideal if you are using [resource sub-navigation](overview#resource-sub-navigation), as you are easily able to switch between the View or Edit page and the relation page.

To create a relation page, you should use the `make:filament-page` command:

```bash
php artisan make:filament-page ManageCustomerAddresses --resource=CustomerResource --type=ManageRelatedRecords
```

When you run this command, you will be asked a series of questions to customize the page, for example, the name of the relationship and its title attribute.

You must register this new page in your resource's `getPages()` method:

```php
public static function getPages(): array
{
    return [
        'index' => Pages\ListCustomers::route('/'),
        'create' => Pages\CreateCustomer::route('/create'),
        'view' => Pages\ViewCustomer::route('/{record}'),
        'edit' => Pages\EditCustomer::route('/{record}/edit'),
        'addresses' => Pages\ManageCustomerAddresses::route('/{record}/addresses'),
    ];
}
```

<Aside variant="warning">
    When using a relation page, you do not need to generate a relation manager with `make:filament-relation-manager`, and you do not need to register it in the `getRelations()` method of the resource.
</Aside>

Now, you can customize the page in exactly the same way as a relation manager, with the same `table()` and `form()`.

### Adding relation pages to resource sub-navigation

If you're using [resource sub-navigation](overview#resource-sub-navigation), you can register this page as normal in `getRecordSubNavigation()` of the resource:

```php
use App\Filament\Resources\Customers\Pages;
use Filament\Resources\Pages\Page;

public static function getRecordSubNavigation(Page $page): array
{
    return $page->generateNavigationItems([
        // ...
        Pages\ManageCustomerAddresses::class,
    ]);
}
```

## Passing properties to relation managers

When registering a relation manager in a resource, you can use the `make()` method to pass an array of [Livewire properties](https://livewire.laravel.com/docs/properties) to it:

```php
use App\Filament\Resources\Blog\Posts\PostResource\RelationManagers\CommentsRelationManager;

public static function getRelations(): array
{
    return [
        CommentsRelationManager::make([
            'status' => 'approved',
        ]),
    ];
}
```

This array of properties gets mapped to [public Livewire properties](https://livewire.laravel.com/docs/properties) on the relation manager class:

```php
use Filament\Resources\RelationManagers\RelationManager;

class CommentsRelationManager extends RelationManager
{
    public string $status;

    // ...
}
```

Now, you can access the `status` in the relation manager class using `$this->status`.

## Disabling lazy loading

By default, relation managers are lazy-loaded. This means that they will only be loaded when they are visible on the page.

To disable this behavior, you may override the `$isLazy` property on the relation manager class:

```php
protected static bool $isLazy = false;
```

---

<!-- Source: 08-nesting.md -->

## Overview

[Relation managers](managing-relationships#creating-a-relation-manager) and [relation pages](managing-relationships#relation-pages) provide you with an easy way to render a table of related records inside a resource.

For example, in a `CourseResource`, you may have a relation manager or page for `lessons` that belong to that course. You can create and edit lessons from the table, which opens modal dialogs.

However, lessons may be too complex to be created and edited in a modal. You may wish that lessons had their own resource, so that creating and editing them would be a full page experience. This is a nested resource.

## Creating a nested resource

To create a nested resource, you can use the `make:filament-resource` command with the `--nested` option:

```bash
php artisan make:filament-resource Lesson --nested
```

To access the nested resource, you will also need a [relation manager](managing-relationships#creating-a-relation-manager) or [relation page](managing-relationships#relation-pages). This is where the user can see the list of related records, and click links to the "create" and "edit" pages.

To create a relation manager or page, you can use the `make:filament-relation-manager` or `make:filament-page` command:

```bash
php artisan make:filament-relation-manager CourseResource lessons title

php artisan make:filament-page ManageCourseLessons --resource=CourseResource --type=ManageRelatedRecords
```

When creating a relation manager or page, Filament will ask if you want each table row to link to a resource instead of opening a modal, to which you should answer "yes" and select the nested resource that you just created.

After generating the relation manager or page, it will have a property pointing to the nested resource:

```php
use App\Filament\Resources\Courses\Resources\Lessons\LessonResource;

protected static ?string $relatedResource = LessonResource::class;
```

The nested resource class will have a property pointing to the parent resource:

```php
use App\Filament\Resources\Courses\CourseResource;

protected static ?string $parentResource = CourseResource::class;
```

## Customizing the relationship names

In the same way that relation managers and pages predict the name of relationships based on the models in those relationships, nested resources do the same. Sometimes, you may have a relationship that does not fit the traditional relationship naming convention, and you will need to inform Filament of the correct relationship names for the nested resource.

To customize the relationship names, first remove the `$parentResource` property from the nested resource class. Then define a `getParentResourceRegistration()` method:

```php
use App\Filament\Resources\Courses\CourseResource;
use Filament\Resources\ParentResourceRegistration;

public static function getParentResourceRegistration(): ?ParentResourceRegistration
{
    return CourseResource::asParent()
        ->relationship('lessons')
        ->inverseRelationship('course');
}
```

You can omit the calls to `relationship()` and `inverseRelationship()` if you want to use the default names.

## Registering a relation manager with the correct URL

When dealing with a nested resource that is listed by a relation manager, and the relation manager is amongst others on that page, you may notice that the URL to it is not correct when you redirect from the nested resource back to it. This is because each relation manager registered on a resource is assigned an integer, which is used to identify it in the URL when switching between multiple relation managers. For example, `?relation=0` might represent one relation manager in the URL, and `?relation=1` might represent another.

When redirecting from a nested resource back to a relation manager, Filament will assume that the relationship name is used to identify that relation manager in the URL. For example, if you have a nested `LessonResource` and a `LessonsRelationManager`, the relationship name is `lessons`, and should be used as the [URL parameter key](managing-relationships#customizing-the-relation-managers-url-parameter) for that relation manager when it is registered:

```php
public static function getRelations(): array
{
    return [
        'lessons' => LessonsRelationManager::class,
    ];
}
```

---

<!-- Source: 09-singular.md -->

## Overview

Resources aren't the only way to interact with Eloquent records in a Filament panel. Even though resources may solve many of your requirements, the "index" (root) page of a resource contains a table with a list of records in that resource.

Sometimes there is no need for a table that lists records in a resource. There is only a single record that the user interacts with. If it doesn't yet exist when the user visits the page, it gets created when the form is first submitted by the user to save it. If the record already exists, it is loaded into the form when the page is first loaded, and updated when the form is submitted.

For example, a CMS might have a `Page` Eloquent model and a `PageResource`, but you may also want to create a singular page outside the `PageResource` for editing the "homepage" of the website. This allows the user to directly edit the homepage without having to navigate to the `PageResource` and find the homepage record in the table.

Other examples of this include a "Settings" page, or a "Profile" page for the currently logged-in user. For these use cases, though, we recommend that you use the [Spatie Settings plugin](/plugins/filament-spatie-settings) and the [Profile](../users#authentication-features) features of Filament, which require less code to implement.

## Creating a singular resource

Although there is no specific "singular resource" feature in Filament, it is a highly-requested behavior and can be implemented quite simply using a [custom page](../navigation/custom-pages) with a [form](../forms). This guide will explain how to do this.

Firstly, create a [custom page](../navigation/custom-pages):

```bash
php artisan make:filament-page ManageHomepage
```

This command will create two files - a page class in the `/Filament/Pages` directory of your resource directory, and a Blade view in the `/filament/pages` directory of the resource views directory.

The page class should contain the following elements:
- A `$data` property, which will hold the current state of the form.
- A `mount()` method, which will load the current record from the database and fill the form with its data. If the record doesn't exist, `null` will be passed to the `fill()` method of the form, which will assign any default values to the form fields.
- A `form()` method, which will define the form schema. The form contains fields in the `components()` method. The `record()` method should be used to specify the record that the form should load relationship data from. The `statePath()` method should be used to specify the name of the property (`$data`) where the form's state should be stored.
- A `save()` method, which will save the form data to the database. The `getState()` method runs form validation and returns valid form data. This method should check if the record already exists, and if not, create a new one. The `wasRecentlyCreated` property of the model can be used to determine if the record was just created, and if so then any relationships should be saved as well. A notification is sent to the user to confirm that the record was saved.
- A `getRecord()` method, while not strictly necessary, is a good idea to have. This method will return the Eloquent record that the form is editing. It can be used across the other methods to avoid code duplication.

```php
namespace App\Filament\Pages;

use App\Models\WebsitePage;
use Filament\Forms\Components\RichEditor;
use Filament\Forms\Components\TextInput;
use Filament\Notifications\Notification;
use Filament\Pages\Page;
use Filament\Schemas\Schema;

/**
 * @property-read Schema $form
 */
class ManageHomepage extends Page
{
    /**
     * @var array<string, mixed> | null
     */
    public ?array $data = [];

    public function mount(): void
    {
        $this->form->fill($this->getRecord()?->attributesToArray());
    }

    public function form(Schema $schema): Schema
    {
        return $schema
            ->components([
                TextInput::make('title')
                    ->required()
                    ->maxLength(255),
                RichEditor::make('content'),
                // ...
            ])
            ->record($this->getRecord())
            ->statePath('data');
    }

    public function save(): void
    {
        $data = $this->form->getState();
        
        $record = $this->getRecord();
        
        if (! $record) {
            $record = new WebsitePage();
            $record->is_homepage = true;
        }
        
        $record->fill($data);
        $record->save();
        
        if ($record->wasRecentlyCreated) {
            $this->form->record($record)->saveRelationships();
        }

        Notification::make()
            ->success()
            ->title('Saved')
            ->send();
    }
    
    public function getRecord(): ?WebsitePage
    {
        return WebsitePage::query()
            ->where('is_homepage', true)
            ->first();
    }
}
```

The page Blade view should render the form and a Save button. The `wire:submit` directive should be used to call the `save()` method when the form is submitted:

```blade
<x-filament::page>
    <form wire:submit="save">
        {{ $this->form }}
    
        <div>
            <x-filament::button type="submit">
                Save
            </x-filament::button>
        </div>
    </form>
</x-filament::page>
```

---

<!-- Source: 10-global-search.md -->

import Aside from "@components/Aside.astro"

## Introduction

Global search allows you to search across all of your resource records, from anywhere in the app.

## Setting global search result titles

To enable global search on your model, you must [set a title attribute](overview#record-titles) for your resource:

```php
protected static ?string $recordTitleAttribute = 'title';
```

This attribute is used to retrieve the search result title for that record.

<Aside variant="warning">
    Your resource needs to have an Edit or View page to allow the global search results to link to a URL, otherwise no results will be returned for this resource.
</Aside>

You may customize the title further by overriding `getGlobalSearchResultTitle()` method. It may return a plain text string, or an instance of `Illuminate\Support\HtmlString` or `Illuminate\Contracts\Support\Htmlable`. This allows you to render HTML, or even Markdown, in the search result title:

```php
use Illuminate\Contracts\Support\Htmlable;

public static function getGlobalSearchResultTitle(Model $record): string | Htmlable
{
    return $record->name;
}
```

## Globally searching across multiple columns

If you would like to search across multiple columns of your resource, you may override the `getGloballySearchableAttributes()` method. "Dot notation" allows you to search inside relationships:

```php
public static function getGloballySearchableAttributes(): array
{
    return ['title', 'slug', 'author.name', 'category.name'];
}
```

## Adding extra details to global search results

Search results can display "details" below their title, which gives the user more information about the record. To enable this feature, you must override the `getGlobalSearchResultDetails()` method:

```php
public static function getGlobalSearchResultDetails(Model $record): array
{
    return [
        'Author' => $record->author->name,
        'Category' => $record->category->name,
    ];
}
```

In this example, the category and author of the record will be displayed below its title in the search result. However, the `category` and `author` relationships will be lazy-loaded, which will result in poor results performance. To [eager-load](https://laravel.com/docs/eloquent-relationships#eager-loading) these relationships, we must override the `getGlobalSearchEloquentQuery()` method:

```php
public static function getGlobalSearchEloquentQuery(): Builder
{
    return parent::getGlobalSearchEloquentQuery()->with(['author', 'category']);
}
```

## Customizing global search result URLs

Global search results will link to the [Edit page](editing-records) of your resource, or the [View page](viewing-page) if the user does not have [edit permissions](editing-records#authorization). To customize this, you may override the `getGlobalSearchResultUrl()` method and return a route of your choice:

```php
public static function getGlobalSearchResultUrl(Model $record): string
{
    return UserResource::getUrl('edit', ['record' => $record]);
}
```

## Adding actions to global search results

Global search supports actions, which are buttons that render below each search result. They can open a URL or dispatch a Livewire event.

Actions can be defined as follows:

```php
use Filament\Actions\Action;

public static function getGlobalSearchResultActions(Model $record): array
{
    return [
        Action::make('edit')
            ->url(static::getUrl('edit', ['record' => $record])),
    ];
}
```

You can learn more about how to style action buttons [here](../actions/overview).

### Opening URLs from global search actions

You can open a URL, optionally in a new tab, when clicking on an action:

```php
use Filament\Actions\Action;

Action::make('view')
    ->url(static::getUrl('view', ['record' => $record]), shouldOpenInNewTab: true)
```

### Dispatching Livewire events from global search actions

Sometimes you want to execute additional code when a global search result action is clicked. This can be achieved by setting a Livewire event which should be dispatched on clicking the action. You may optionally pass an array of data, which will be available as parameters in the event listener on your Livewire component:

```php
use Filament\Actions\Action;

Action::make('quickView')
    ->dispatch('quickView', [$record->id])
```

## Limiting the number of global search results

By default, global search will return up to 50 results per resource. You can customize this on the resource label by overriding the `$globalSearchResultsLimit` property:

```php
protected static int $globalSearchResultsLimit = 20;
```

## Disabling global search

As [explained above](#title), global search is automatically enabled once you set a title attribute for your resource. Sometimes you may want to specify the title attribute while not enabling global search.

This can be achieved by disabling global search in the [configuration](../panel-configuration):

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->globalSearch(false);
}
```

## Registering global search key bindings

The global search field can be opened using keyboard shortcuts. To configure these, pass the `globalSearchKeyBindings()` method to the [configuration](../panel-configuration):

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->globalSearchKeyBindings(['command+k', 'ctrl+k']);
}
```

## Configuring the global search debounce

Global search has a default debounce time of 500ms, to limit the number of requests that are made while the user is typing. You can alter this by using the `globalSearchDebounce()` method in the [configuration](../panel-configuration):

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->globalSearchDebounce('750ms');
}
```

## Configuring the global search field suffix

Global search field by default doesn't include any suffix. You may customize it using the `globalSearchFieldSuffix()` method in the [configuration](../panel-configuration).

If you want to display the currently configured [global search key bindings](#registering-global-search-key-bindings) in the suffix, you can use the `globalSearchFieldKeyBindingSuffix()` method, which will display the first registered key binding as the suffix of the global search field:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->globalSearchFieldKeyBindingSuffix();
}
```

To customize the suffix yourself, you can pass a string or function to the `globalSearchFieldSuffix()` method. For example, to provide a custom key binding suffix for each platform manually:

```php
use Filament\Panel;
use Filament\Support\Enums\Platform;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->globalSearchFieldSuffix(fn (): ?string => match (Platform::detect()) {
            Platform::Windows, Platform::Linux => 'CTRL+K',
            Platform::Mac => '⌘K',
            default => null,
        });
}
```

## Disabling search term splitting

By default, the global search will split the search term into individual words and search for each word separately. This allows for more flexible search queries. However, it can have a negative impact on performance when large datasets are involved. You can disable this behavior by setting the `$shouldSplitGlobalSearchTerms` property to `false` on the resource:

```php
protected static ?bool $shouldSplitGlobalSearchTerms = false;
```

---

<!-- Source: 11-widgets.md -->

## Introduction

Filament allows you to display widgets inside pages, below the header and above the footer.

You can use an existing [dashboard widget](../widgets), or create one specifically for the resource.

## Creating a resource widget

To get started building a resource widget:

```bash
php artisan make:filament-widget CustomerOverview --resource=CustomerResource
```

This command will create two files - a widget class in the `app/Filament/Resources/Customers/Widgets` directory, and a view in the `resources/views/filament/resources/customers/widgets` directory.

You must register the new widget in your resource's `getWidgets()` method:

```php
use App\Filament\Resources\Customers\Widgets\CustomerOverview;

public static function getWidgets(): array
{
    return [
        CustomerOverview::class,
    ];
}
```

If you'd like to learn how to build and customize widgets, check out the [Dashboard](../widgets) documentation section.

## Displaying a widget on a resource page

To display a widget on a resource page, use the `getHeaderWidgets()` or `getFooterWidgets()` methods for that page:

```php
<?php

namespace App\Filament\Resources\Customers\Pages;

use App\Filament\Resources\Customers\CustomerResource;

class ListCustomers extends ListRecords
{
    public static string $resource = CustomerResource::class;

    protected function getHeaderWidgets(): array
    {
        return [
            CustomerResource\Widgets\CustomerOverview::class,
        ];
    }
}
```

`getHeaderWidgets()` returns an array of widgets to display above the page content, whereas `getFooterWidgets()` are displayed below.

If you'd like to customize the number of grid columns used to arrange widgets, check out the [Pages documentation](../navigation/custom-pages#customizing-the-widgets-grid).

## Accessing the current record in the widget

If you're using a widget on an [Edit](editing-records) or [View](viewing-records) page, you may access the current record by defining a `$record` property on the widget class:

```php
use Illuminate\Database\Eloquent\Model;

public ?Model $record = null;
```

## Accessing page table data in the widget

If you're using a widget on a [List](listing-records) page, you may access the table data by first adding the `ExposesTableToWidgets` trait to the page class:

```php
use Filament\Pages\Concerns\ExposesTableToWidgets;
use Filament\Resources\Pages\ListRecords;

class ListProducts extends ListRecords
{
    use ExposesTableToWidgets;

    // ...
}
```

Now, on the widget class, you must add the `InteractsWithPageTable` trait, and return the name of the page class from the `getTablePage()` method:

```php
use App\Filament\Resources\Products\Pages\ListProducts;
use Filament\Widgets\Concerns\InteractsWithPageTable;
use Filament\Widgets\Widget;

class ProductStats extends Widget
{
    use InteractsWithPageTable;

    protected function getTablePage(): string
    {
        return ListProducts::class;
    }

    // ...
}
```

In the widget class, you can now access the Eloquent query builder instance for the table data using the `$this->getPageTableQuery()` method:

```php
use Filament\Widgets\StatsOverviewWidget\Stat;

Stat::make('Total Products', $this->getPageTableQuery()->count()),
```

Alternatively, you can access a collection of the records on the current page using the `$this->getPageTableRecords()` method:

```php
use Filament\Widgets\StatsOverviewWidget\Stat;

Stat::make('Total Products', $this->getPageTableRecords()->count()),
```

## Passing properties to widgets on resource pages

When registering a widget on a resource page, you can use the `make()` method to pass an array of [Livewire properties](https://livewire.laravel.com/docs/properties) to it:

```php
protected function getHeaderWidgets(): array
{
    return [
        CustomerResource\Widgets\CustomerOverview::make([
            'status' => 'active',
        ]),
    ];
}
```

This array of properties gets mapped to [public Livewire properties](https://livewire.laravel.com/docs/properties) on the widget class:

```php
use Filament\Widgets\Widget;

class CustomerOverview extends Widget
{
    public string $status;

    // ...
}
```

Now, you can access the `status` in the widget class using `$this->status`.

---

<!-- Source: 12-custom-pages.md -->

import Aside from "@components/Aside.astro"

## Introduction

Filament allows you to create completely custom pages for resources. To create a new page, you can use:

```bash
php artisan make:filament-page SortUsers --resource=UserResource --type=custom
```

This command will create two files - a page class in the `/Pages` directory of your resource directory, and a view in the `/pages` directory of the resource views directory.

You must register custom pages to a route in the static `getPages()` method of your resource:

```php
public static function getPages(): array
{
    return [
        // ...
        'sort' => Pages\SortUsers::route('/sort'),
    ];
}
```

<Aside variant="warning">
    The order of pages registered in this method matters - any wildcard route segments that are defined before hard-coded ones will be matched by Laravel's router first.
</Aside>

Any [parameters](https://laravel.com/docs/routing#route-parameters) defined in the route's path will be available to the page class, in an identical way to [Livewire](https://livewire.laravel.com/docs/components#accessing-route-parameters).

## Using a resource record

If you'd like to create a page that uses a record similar to the [Edit](editing-records) or [View](viewing-records) pages, you can use the `InteractsWithRecord` trait:

```php
use Filament\Resources\Pages\Page;
use Filament\Resources\Pages\Concerns\InteractsWithRecord;

class ManageUser extends Page
{
    use InteractsWithRecord;
    
    public function mount(int | string $record): void
    {
        $this->record = $this->resolveRecord($record);
    }

    // ...
}
```

The `mount()` method should resolve the record from the URL and store it in `$this->record`. You can access the record at any time using `$this->getRecord()` in the class or view.

To add the record to the route as a parameter, you must define `{record}` in `getPages()`:

```php
public static function getPages(): array
{
    return [
        // ...
        'manage' => Pages\ManageUser::route('/{record}/manage'),
    ];
}
```

---

<!-- Source: 13-code-quality-tips.md -->

## Using schema and table classes

Since many Filament methods define both the UI and the functionality of the app in just one method, it can be easy to end up with giant methods and files. These can be difficult to read, even if your code has a clean and consistent style.

Filament tries to mitigate some of this by providing dedicated schema and table classes when you generate a resource. These classes have a `configure()` method that accepts a `$schema` or `$table`. You can then call the `configure()` method from anywhere you want to define a schema or table.

For example, if you have the following `app/Filament/Resources/Customers/Schemas/CustomerForm.php` file:

```php
namespace App\Filament\Resources\Customers\Schemas;

use Filament\Forms\Components\TextInput;
use Filament\Schemas\Schema;

class CustomerForm
{
    public static function configure(Schema $schema): Schema
    {
        return $schema
            ->components([
                TextInput::make('name'),
                // ...
            ]);
    }
}
```

You can use this in the `form()` method of the resource:

```php
use App\Filament\Resources\Customers\Schemas\CustomerForm;
use Filament\Schemas\Schema;

public static function form(Schema $schema): Schema
{
    return CustomerForm::configure($schema);
}
```

You could do the same for the `table()`:

```php
use App\Filament\Resources\Customers\Schemas\CustomersTable;
use Filament\Tables\Table;

public static function table(Table $table): Table
{
    return CustomersTable::configure($table);
}
```

Or the `infolist()`:

```php
use App\Filament\Resources\Customers\Schemas\CustomerInfolist;
use Filament\Schemas\Schema;

public static function infolist(Schema $schema): Schema
{
    return CustomerInfolist::configure($schema);
}
```

These schema and table classes deliberately don't have a parent class or interface by default. If Filament were to enforce a method signature for the `configure()` method, you would not be able to pass your own configuration variables to the method, which could be useful if you wanted to reuse the same class in multiple places but with slight tweaks.

## Using component classes

Even if you are using [schema and table classes](#using-schema-and-table-classes) to keep the schema and table definitions in their own files, you can still end up with a very long `configure()` method. This is especially true if you are using a lot of components in your schema or table, or if the components require a lot of configuration.

You can mitigate this by creating dedicated classes for each component. For example, if you have a `TextInput` component that requires a lot of configuration, you can create a dedicated class for it:

```php
namespace App\Filament\Resources\Customers\Schemas\Components;

use Filament\Forms\Components\TextInput;

class CustomerNameInput
{
    public static function make(): TextInput
    {
        return TextInput::make('name')
            ->label('Full name')
            ->required()
            ->maxLength(255)
            ->placeholder('Enter your full name')
            ->belowContent('This is the name that will be displayed on your profile.');
    }
}
```

You can then use this class in your schema or table:

```php
use App\Filament\Resources\Customers\Schemas\Components\CustomerNameInput;
use Filament\Schemas\Schema;

public static function configure(Schema $schema): Schema
{
    return $schema
        ->components([
            CustomerNameInput::make(),
            // ...
        ]);
}
```

You could do this with a number of different types of component. There are no enforced rules as to how these components should be named or where they should be stored. However, here are some ideas:

- **Schema components**: These could live in the `Schemas/Components` directory of the resource. They could be named after the component they are wrapping, for example `CustomerNameInput` or `CustomerCountrySelect`.
- **Table columns**: These could live in the `Tables/Columns` directory of the resource. They could be named after the column followed by `Column`, for example `CustomerNameColumn` or `CustomerCountryColumn`.
- **Table filters**: These could live in the `Tables/Filters` directory of the resource. They could be named after the filter followed by `Filter`, for example `CustomerCountryFilter` or `CustomerStatusFilter`.
- **Actions**: These could live in the `Actions` directory of the resource. They could be named after the action followed by `Action` or `BulkAction`, for example `EmailCustomerAction` or `UpdateCustomerCountryBulkAction`.

As a further example, here is a potential `EmailCustomerAction` class:

```php
namespace App\Filament\Resources\Customers\Actions;

use App\Models\Customer;
use Filament\Actions\Action;
use Filament\Forms\Components\Textarea;
use Filament\Forms\Components\TextInput;
use Filament\Support\Icons\Heroicon;

class EmailCustomerAction
{
    public static function make(): Action
    {
        return Action::make('email')
            ->label('Send email')
            ->icon(Heroicon::Envelope)
            ->schema([
                TextInput::make('subject')
                    ->required()
                    ->maxLength(255),
                Textarea::make('body')
                    ->autosize()
                    ->required(),
            ])
            ->action(function (Customer $customer, array $data) {
                // ...
            });
    }
}
```

And you could use it in the `getHeaderActions()` of a page:

```php
use App\Filament\Resources\Customers\Actions\EmailCustomerAction;

protected function getHeaderActions(): array
{
    return [
        EmailCustomerAction::make(),
    ];
}
```

Or you could use it on a table row:

```php
use App\Filament\Resources\Customers\Actions\EmailCustomerAction;
use Filament\Tables\Table;

public static function configure(Table $table): Table
{
    return $table
        ->columns([
            // ...
        ])
        ->recordActions([
            EmailCustomerAction::make(),
        ]);
}
```

---

<!-- Source: 13-deployment.md -->

import Aside from "@components/Aside.astro"

## Introduction

Deploying a Laravel app using Filament to production is similar to deploying any other Laravel app. However, there are a few additional steps you should take to ensure that your Filament panel is optimized for performance and security.

For tips focused on local development performance, see [Optimizing local development](introduction/optimizing-local-development).

## Ensure that users are authorized to access panels

When Filament detects that your app's `APP_ENV` is not `local`, it will require you to set up authorization for your users. This is to ensure that only authorized users can access your Filament panel in production, while keeping the local environment easy to get started with.

To authorize users to access a panel, you should follow the [guide in the users section](users/overview#authorizing-access-to-the-panel).

<Aside variant="warning">
    If you do not follow these steps and your user model does not implement the `FilamentUser` interface, no users will be able to sign in to your panel in production.
</Aside>

## Improving Filament panel performance

### Optimizing Filament for production

To optimize Filament for production, you should run the following command in your deployment script:

```bash
php artisan filament:optimize
```

This command will [cache the Filament components](#caching-filament-components) and additionally the [Blade icons](#caching-blade-icons), which can significantly improve the performance of your Filament panels. This command is a shorthand for the commands `php artisan filament:cache-components` and `php artisan icons:cache`.

To clear the caches at once, you can run:

```bash
php artisan filament:optimize-clear
```

#### Caching Filament components

If you're not using the [`filament:optimize` command](#optimizing-filament-for-production), you may wish to consider running `php artisan filament:cache-components` in your deployment script, especially if you have large numbers of components (resources, pages, widgets, relation managers, custom Livewire components, etc.). This will create cache files in the `bootstrap/cache/filament` directory of your application, which contain indexes for each type of component. This can significantly improve the performance of Filament in some apps, as it reduces the number of files that need to be scanned and auto-discovered for components.

However, if you are actively developing your app locally, you should avoid using this command, as it will prevent any new components from being discovered until the cache is cleared or rebuilt.

You can clear the cache at any time without rebuilding it by running `php artisan filament:clear-cached-components`.

#### Caching Blade Icons

If you're not using the [`filament:optimize` command](#optimizing-filament-for-production), you may wish to consider running `php artisan icons:cache` locally, and also in your deployment script. This is because Filament uses the [Blade Icons](https://blade-ui-kit.com/blade-icons) package, which can be much more performant when cached.

### Enabling OPcache on your server

To check if [OPcache](https://www.php.net/manual/en/book.opcache.php) is enabled, run:

```bash
php -r "echo 'opcache.enable => ' . ini_get('opcache.enable') . PHP_EOL;"
```

You should see `opcache.enable => 1`. If not, enable it by adding the following line to your `php.ini`:

```ini
opcache.enable=1 # Enable OPcache
```

From the [Laravel Forge documentation](https://forge.laravel.com/docs/servers/php.html#opcache):

<Aside variant="tip">
    Optimizing the PHP OPcache for production will configure OPcache to store your compiled PHP code in memory to greatly improve performance.
</Aside>

Please use a search engine to find the relevant OPcache setup instructions for your environment.

### Optimizing your Laravel app

You should also consider optimizing your Laravel app for production by running `php artisan optimize` in your deployment script. This will cache the configuration files and routes.

## Ensuring assets are up to date

During the Filament installation process, Filament adds the `php artisan filament:upgrade` command to the `composer.json` file, in the `post-autoload-dump` script. This command will ensure that your assets are up to date whenever you download the package.

We strongly suggest that this script remains in your `composer.json` file, otherwise you may run into issues with missing or outdated assets in your production environment. However, if you must remove it, make sure that the command is run manually in your deployment process.

---

<!-- Source: 14-upgrade-guide.md -->

import Aside from "@components/Aside.astro"
import Checkbox from "@components/Checkbox.astro"
import Checkboxes from "@components/Checkboxes.astro"
import Disclosure from "@components/Disclosure.astro"

<Aside variant="info">
    If you see anything missing from this guide, please don’t hesitate to [make a pull request](https://github.com/filamentphp/filament/edit/4.x/docs/14-upgrade-guide.md) to our repository! Any help is appreciated!
</Aside>

## New requirements

- PHP 8.2+
- Laravel v11.28+
- Tailwind CSS v4.0+, if you’re currently using Tailwind CSS v3.0 with Filament. This doesn’t apply if you’re just using a Filament panel without a custom theme CSS file.
- Filament no longer requires `doctrine/dbal`, but if your application still does, and you don’t have it installed directly, you should add it to your `composer.json` file.

## Running the automated upgrade script

<Aside variant="info">
    The upgrade script is not a replacement for the upgrade guide. It handles many small changes that aren’t mentioned in the upgrade guide, but it doesn’t handle all breaking changes. You should still read the [manual upgrade steps](#breaking-changes-that-must-be-handled-manually) to see what changes you need to make to your code.
</Aside>

<Aside variant="info">
    Some plugins you're using may not be available in v4 just yet. You could temporarily remove them from your `composer.json` file until they've been upgraded, replace them with a similar plugins that are v4-compatible, wait for the plugins to be upgraded before upgrading your app, or even write PRs to help the authors upgrade them.
</Aside>

The first step to upgrade your Filament app is to run the automated upgrade script. This script will automatically upgrade your application to the latest version of Filament and make changes to your code, which handles most breaking changes:

```bash
composer require filament/upgrade:"^4.0" -W --dev

vendor/bin/filament-v4

# Run the commands output by the upgrade script, they are unique to your app
composer require filament/filament:"^4.0" -W --no-update
composer update
```

<Aside variant="warning">
    When using Windows PowerShell to install Filament, you may need to run the command below, since it ignores `^` characters in version constraints:

    ```bash
    composer require filament/upgrade:"~4.0" -W --dev

    vendor/bin/filament-v4

    # Run the commands output by the upgrade script, they are unique to your app
    composer require filament/filament:"~4.0" -W --no-update
    composer update
    ```
</Aside>

<Aside variant="warning">
    If installing the upgrade script fails, make sure that your PHPStan version is at least v2, or your Larastan version is at least v3. The script uses Rector v2, which requires PHPStan v2 or higher.
</Aside>

Make sure to carefully follow the instructions, and review the changes made by the script. You may need to make some manual changes to your code afterwards, but the script should handle most of the repetitive work for you.

Filament v4 introduces a new default directory structure for your Filament resources and clusters. If you’re using Filament panels with resources and clusters, you can choose to keep the old directory structure, or migrate to the new one. If you want to migrate to the new directory structure, you can run the following command:

```bash
php artisan filament:upgrade-directory-structure-to-v4 --dry-run
```

The `--dry-run` option will show you what the command would do without actually making any changes. If you’re happy with the changes, you can run the command without the `--dry-run` option to apply the changes:

```bash
php artisan filament:upgrade-directory-structure-to-v4
```

<Aside variant="warning">
    This directory upgrade script is not able to perfectly update any references to classes in the same namespace that were present in resource and cluster files, and those references will need to be updated manually after the script has run. You should use tools like [PHPStan](https://phpstan.org) to identify references to classes that are broken after the upgrade.
</Aside>

You can now `composer remove filament/upgrade --dev` as you don't need it anymore.

## Publishing the configuration file

Some changes in Filament v4 can be reverted using the configuration file. If you haven't published the configuration file yet, you can do so by running the following command:

```bash
php artisan vendor:publish --tag=filament-config
```

Firstly, the `default_filesystem_disk` in v4 is set to the `FILESYSTEM_DISK` variable instead of `FILAMENT_FILESYSTEM_DISK`. To preserve the v3 behavior, make sure you use this setting:

```php
return [

    // ...

    'default_filesystem_disk' => env('FILAMENT_FILESYSTEM_DISK', 'public'),

    // ...

]
```

v4 introduces some changes to how Filament generates files. A new `file_generation` section has been added to the v4 configuration file, so that you can revert back to the v3 style if you would like to keep new code consistent with how it looked before upgrading. If your configuration file doesn't already have a `file_generation` section, you should add it yourself, or re-publish the configuration file and tweak it to your liking:

```php
use Filament\Support\Commands\FileGenerators\FileGenerationFlag;

return [

    // ...

    'file_generation' => [
        'flags' => [
            FileGenerationFlag::EMBEDDED_PANEL_RESOURCE_SCHEMAS, // Define new forms and infolists inside the resource class instead of a separate schema class.
            FileGenerationFlag::EMBEDDED_PANEL_RESOURCE_TABLES, // Define new tables inside the resource class instead of a separate table class.
            FileGenerationFlag::PANEL_CLUSTER_CLASSES_OUTSIDE_DIRECTORIES, // Create new cluster classes outside of their directories. Not required if you run `php artisan filament:upgrade-directory-structure-to-v4`.
            FileGenerationFlag::PANEL_RESOURCE_CLASSES_OUTSIDE_DIRECTORIES, // Create new resource classes outside of their directories. Not required if you run `php artisan filament:upgrade-directory-structure-to-v4`.
            FileGenerationFlag::PARTIAL_IMPORTS, // Partially import components such as form fields and table columns instead of importing each component explicitly.
        ],
    ],

    // ...

]
```

<Aside variant="tip">
    The `filament/upgrade` package includes a command to help you move panel resources and clusters to the new directory structure, which is the default in v4:

    ```bash
    php artisan filament:upgrade-directory-structure-to-v4 --dry-run
    ```

    The `--dry-run` option will show you what the command would do without actually making any changes. If you are happy with the changes, you can run the command without the `--dry-run` option to apply the changes:

    ```bash
    php artisan filament:upgrade-directory-structure-to-v4
    ```

    This directory upgrade script is not able to perfectly update any references to classes in the same namespace that were present in resource and cluster files, and those references will need to be updated manually after the script has run. You should use tools like [PHPStan](https://phpstan.org) to identify references to classes that are broken after the upgrade.

    Once you have run the command, you do not need to keep the `FileGenerationFlag::PANEL_CLUSTER_CLASSES_OUTSIDE_DIRECTORIES` or `FileGenerationFlag::PANEL_RESOURCE_CLASSES_OUTSIDE_DIRECTORIES` flags in your configuration file, as the new directory structure is now the default. You can remove them from the `file_generation.flags` array.
</Aside>

## Breaking changes that must be handled manually

<div x-data="{ packages: ['panels', 'forms', 'infolists', 'tables', 'actions', 'notifications', 'widgets', 'support'] }">

To begin, filter the upgrade guide for your specific needs by selecting only the packages that you use in your project:

<Checkboxes>
    <Checkbox value="panels" model="packages">
        Panels
    </Checkbox>

    <Checkbox value="forms" model="packages">
        Forms

        <span slot="description">
            This package is also often used in a panel, or using the tables or actions package.
        </span>
    </Checkbox>

    <Checkbox value="infolists" model="packages">
        Infolists

        <span slot="description">
            This package is also often used in a panel, or using the tables or actions package.
        </span>
    </Checkbox>

    <Checkbox value="tables" model="packages">
        Tables

        <span slot="description">
            This package is also often used in a panel.
        </span>
    </Checkbox>

    <Checkbox value="actions" model="packages">
        Actions

        <span slot="description">
            This package is also often used in a panel.
        </span>
    </Checkbox>

    <Checkbox value="notifications" model="packages">
        Notifications

        <span slot="description">
            This package is also often used in a panel.
        </span>
    </Checkbox>

    <Checkbox value="widgets" model="packages">
        Widgets

        <span slot="description">
            This package is also often used in a panel.
        </span>
    </Checkbox>

    <Checkbox value="support" model="packages">
        Blade UI components
    </Checkbox>

    <Checkbox value="spatie-translatable-plugin" model="packages">
        Spatie Translatable Plugin
    </Checkbox>
</Checkboxes>

### High-impact changes

<Disclosure open x-show="packages.includes('forms') || packages.includes('infolists') || packages.includes('tables')">
<span slot="summary">File visibility is now private by default for non-local disks</span>

In addition to the [default disk being changed to `local`](#publishing-the-configuration-file), the file visibility settings for non-local disks (such as `s3`, but not `public` or `local`) across various components have been changed to `private` instead of `public` by default. This means that files aren't publicly accessible by default, and you need to generate a temporary signed URL to access them. This change affects the following components:

- `FileUpload` form field, including `SpatieMediaLibraryFileUpload`
- `ImageColumn` table column, including `SpatieMediaLibraryImageColumn`
- `ImageEntry` infolist entry, including `SpatieMediaLibraryImageEntry`

<Aside variant="tip">
    If you use a non-local disk such as `s3`, you can preserve the old default behavior across your entire app by adding the following code in the `boot()` method of a service provider like `AppServiceProvider`:

    ```php
    use Filament\Forms\Components\FileUpload;
    use Filament\Infolists\Components\ImageEntry;
    use Filament\Tables\Columns\ImageColumn;
    
    FileUpload::configureUsing(fn (FileUpload $fileUpload) => $fileUpload
        ->visibility('public'));
    
    ImageColumn::configureUsing(fn (ImageColumn $imageColumn) => $imageColumn
        ->visibility('public'));
    
    ImageEntry::configureUsing(fn (ImageEntry $imageEntry) => $imageEntry
        ->visibility('public'));
    ```
</Aside>
</Disclosure>

<Disclosure open x-show="packages.includes('panels')">
<span slot="summary">Custom themes need to be upgraded to Tailwind CSS v4</span>

Previously, custom theme CSS files contained this:

```css
@import '../../../../vendor/filament/filament/resources/css/theme.css';

@config 'tailwind.config.js';
```

Now, they should contain this:

```css
@import '../../../../vendor/filament/filament/resources/css/theme.css';

@source '../../../../app/Filament';
@source '../../../../resources/views/filament';
```

This will load Tailwind CSS. The `@source` entries tell Tailwind where to find the classes that are used in your app. You should check the `content` paths in your old `tailwind.config.js` file, and add them as `@source` entries like this. You **don't** need to include `vendor/filament` as a `@source`, but check plugins you have installed to see if they require `@source` entries.

Finally, you should use the [Tailwind upgrade tool](https://tailwindcss.com/docs/upgrade-guide#using-the-upgrade-tool) to automatically adjust your configuration files to use Tailwind v4, and install Tailwind v4 packages to replace Tailwind v3 ones:

```bash
npx @tailwindcss/upgrade
```

The `tailwind.config.js` file for your theme is no longer used, since Tailwind CSS v4 defines [configuration in CSS](https://tailwindcss.com/docs/adding-custom-styles). Any customizations you made to the `tailwind.config.js` file should be added to the CSS file.
</Disclosure>

<Disclosure open x-show="packages.includes('tables')">
<span slot="summary">Changes to table filters are deferred by default</span>

The `deferFilters()` method from Filament v3 is now the default behavior in Filament v4, so users must click a button before the filters are applied to the table. To disable this behavior, you can use the `deferFilters(false)` method.

```php
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->deferFilters(false);
}
```

<Aside variant="tip">
    You can preserve the old default behavior across your entire app by adding the following code in the `boot()` method of a service provider like `AppServiceProvider`:

    ```php
    use Filament\Tables\Table;

    Table::configureUsing(fn (Table $table) => $table
        ->deferFilters(false));
    ```
</Aside>
</Disclosure>

<Disclosure open x-show="packages.includes('forms') || packages.includes('infolists')">
<span slot="summary">The `Grid`, `Section` and `Fieldset` layout components now do not span all columns by default</span>

In v3, the `Grid`, `Section` and `Fieldset` layout components consumed the full width of their parent grid by default. This was inconsistent with the behavior of every other component in Filament, which only consumes one column of the grid by default. The intention was to make these components easier to integrate into the default Filament resource form and infolist, which uses a two column grid out of the box.

In v4, the `Grid`, `Section` and `Fieldset` layout components now only consume one column of the grid by default. If you want them to span all columns, you can use the `columnSpanFull()` method:

```php
use Filament\Schemas\Components\Fieldset;
use Filament\Schemas\Components\Grid;
use Filament\Schemas\Components\Section;

Fieldset::make()
    ->columnSpanFull()
    
Grid::make()
    ->columnSpanFull()

Section::make()
    ->columnSpanFull()
```

<Aside variant="tip">
    You can preserve the old default behavior across your entire app by adding the following code in the `boot()` method of a service provider like `AppServiceProvider`:

    ```php
    use Filament\Schemas\Components\Fieldset;
    use Filament\Schemas\Components\Grid;
    use Filament\Schemas\Components\Section;

    Fieldset::configureUsing(fn (Fieldset $fieldset) => $fieldset
        ->columnSpanFull());

    Grid::configureUsing(fn (Grid $grid) => $grid
        ->columnSpanFull());

    Section::configureUsing(fn (Section $section) => $section
        ->columnSpanFull());
    ```
</Aside>
</Disclosure>

<Disclosure open x-show="packages.includes('forms')">
<span slot="summary">The `unique()` validation rule behavior for ignoring Eloquent records</span>

In v3, the `unique()` method didn’t ignore the current form's Eloquent record when validating by default. This behavior was enabled by the `ignoreRecord: true` parameter, or by passing a custom `ignorable` record.

In v4, the `unique()` method's `ignoreRecord` parameter defaults to `true`.

If you were previously using `unique()` validation rule without the `ignoreRecord` or `ignorable` parameters, you should use `ignoreRecord: false` to disable the new behavior.

<Aside variant="tip">
    You can preserve the old default behavior across your entire app by adding the following code in the `boot()` method of a service provider like `AppServiceProvider`:

    ```php
    use Filament\Forms\Components\Field;

    Field::configureUsing(fn (Field $field) => $field
        ->uniqueValidationIgnoresRecordByDefault(false));
    ```
</Aside>
</Disclosure>

<Disclosure open x-show="packages.includes('tables')">
<span slot="summary">The `all` pagination page option is not available for tables by default</span>

The `all` pagination page method is now not available for tables by default. If you want to use it on a table, you can add it to the configuration:

```php
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->paginationPageOptions([5, 10, 25, 50, 'all']);
}
```

Be aware when using `all` as it will cause performance issues when dealing with a large number of records.

<Aside variant="tip">
    You can preserve the old default behavior across your entire app by adding the following code in the `boot()` method of a service provider like `AppServiceProvider`:

    ```php
    use Filament\Tables\Table;

    Table::configureUsing(fn (Table $table) => $table
        ->paginationPageOptions([5, 10, 25, 50, 'all']));
    ```
</Aside>
</Disclosure>

<Disclosure open x-show="packages.includes('spatie-translatable-plugin')">
<span slot="summary">The official Spatie Translatable Plugin is now deprecated</span>

Last year, the Filament team decided to hand over maintenance of the Spatie Translatable Plugin to the team at [Lara Zeus](https://larazeus.com), who are trusted developers of many Filament plugins. They’ve maintained a fork of the plugin ever since.

The official Spatie Translatable Plugin will not receive v4 support, and is now deprecated. You can use the [Lara Zeus Translatable Plugin](https://github.com/lara-zeus/spatie-translatable) as a direct replacement. The plugin is compatible with the same version of Spatie Translatable as the official plugin, and has been tested with Filament v4. It also fixes some long-standing bugs in the official plugin.

The [automated upgrade script](#running-the-automated-upgrade-script) suggests commands that uninstall the official plugin and install the Lara Zeus plugin, and replaces any references in your code to the official plugin with the Lara Zeus plugin.
</Disclosure>

### Medium-impact changes

<Disclosure x-show="packages.includes('forms')">
<span slot="summary">Enum field state</span>

In v3, fields that wrote to an enum attribute on a model, such as a `Select`, `CheckboxList` or `Radio` field using `options(Enum::class)`, would inconsistently return the value of the field as either the enum value or the enum instance, depending on whether or not the field was last modified by the server or by the user. This wasn’t useful, and you had to check the type of the value returned by the field to determine if it was an enum value or an enum instance.

In v4, the field state is always returned as the enum instance. This means that you can always use the enum methods on field state. If you weren’t handling the possibility of the field state being an enum instance in your code previously, you now need to do so.

The following code examples illustrate how field state may now return an enum instance:

```php
use App\Enums\Status;
use Filament\Forms\Components\Select;
use Filament\Forms\Components\TextInput;
use Filament\Schemas\Components\Utilities\Get;

Select::make('status')
    ->options(Status::class)
    ->afterStateUpdated(function (?Status $state) {
        // `$state` is now always an instance of `Status`, or `null` if the field is empty.
    });

TextInput::make('...')
    ->afterStateUpdated(function (Get $get) {
        // `$get('status')` is now always an instance of `Status`, or `null` if the field is empty.
    });

$data = $this->form->getState();
// `$data['status']` is now always an instance of `Status`, or `null` if the field is empty.
```
</Disclosure>

<Disclosure x-show="packages.includes('panels')">
<span slot="summary">URL parameter names have changed</span>

Filament v4 has renamed some of the URL parameters that are used on resource pages, to make them cleaner in the URL and easier to remember:

- `activeRelationManager` has been renamed to `relation` on Edit / View resource pages.
- `activeTab` has been renamed to `tab` on List / Manage Relation resource pages.
- `isTableReordering` has been renamed to `reordering` on List / Manage Relation resource pages.
- `tableFilters` has been renamed to `filters` on List / Manage Relation resource pages.
- `tableGrouping` has been renamed to `grouping` on List / Manage Relation resource pages.
- `tableGroupingDirection` has been renamed to `groupingDirection` on List / Manage Relation resource pages.
- `tableSearch` has been renamed to `search` on List / Manage Relation resource pages.
- `tableSort` has been renamed to `sort` on List / Manage Relation resource pages.

To find out if you’re using this parameter in your code, try searching for `'activeRelationManager' => ` (etc.) in your code, and looking for areas where you’re using `::getUrl()` or another method of generating a URL with a parameter.
</Disclosure>

<Disclosure x-show="packages.includes('panels')">
<span slot="summary">Automatic tenancy global scoping and association</span>

When using tenancy in v3, Filament only scoped resource queries to the current tenant: to render the resource table, resolve URL parameters, and fetch global search results. There were many situations where other queries in the panel weren’t scoped by default, and the developer had to manually scope them. While this was a documented feature, it created a lot of additional work for developers.

In v4, Filament automatically scopes all queries in a panel to the current tenant, and automatically associates new records with the current tenant using model events. This means that you no longer need to manually scope queries or associate new Eloquent records in most cases. There are still some important points to consider, so the [documentation](users/tenancy#tenancy-security) has been updated to reflect this.
</Disclosure>

<Disclosure x-show="packages.includes('forms')">
<span slot="summary">The `Radio` `inline()` method behavior</span>

In v3, the `inline()` method put the radio buttons inline with each other, and also inline with the label at the same time. This is inconsistent with other components.

In v4, the `inline()` method now only puts the radio buttons inline with each other, and not with the label. If you want the radio buttons to be inline with the label, you can use the `inlineLabel()` method as well.

If you were previously using `inline()->inlineLabel(false)` to achieve the v4 behavior, you can now simply use `inline()`.

<Aside variant="tip">
    You can preserve the old default behavior across your entire app by adding the following code in the `boot()` method of a service provider like `AppServiceProvider`:

    ```php
    use Filament\Forms\Components\Radio;
    
    Radio::configureUsing(fn (Radio $radio) => $radio
        ->inlineLabel(fn (): bool => $radio->isInline()));
    ```
</Aside>
</Disclosure>

<Disclosure x-show="packages.includes('actions')">
<span slot="summary">Import and export job retries</span>

In Filament v3, import and export jobs were retries continuously for 24 hours if they failed, with no backoff between tries by default. This caused issues for some users, as there was no backoff period and the jobs could be retried too quickly, causing the queue to be flooded with continuously failing jobs.

In v4, they’re retried 3 times with a 60 second backoff between each retry.

This behavior can be customized in the [importer](import#customizing-the-import-job-retries) and [exporter](export#customizing-the-export-job-retries) classes.
</Disclosure>

### Low-impact changes

<Disclosure x-show="packages.includes('tables') || packages.includes('infolists')">
<span slot="summary">The `isSeparate` parameter of `ImageColumn::limitedRemainingText()` and `ImageEntry::limitedRemainingText()` has been removed</span>

Previously, users were able to display the number of limited images separately to an image stack using the `isSeparate` parameter. Now the parameter has been removed, and if a stack exists, the text will always be stacked on top and not separate. If the images aren’t stacked, the text will be separate.
</Disclosure>

<Disclosure x-show="packages.includes('forms')">
<span slot="summary">The `RichEditor` component's `disableGrammarly()` method has been removed</span>

The `disableGrammarly()` method has been removed from the `RichEditor` component. This method was used to disable the Grammarly browser extension acting on the editor. Since moving the underlying implementation of the editor from Trix to TipTap, we haven’t found a way to disable Grammarly on the editor.
</Disclosure>

<Disclosure x-show="packages.includes('forms')">
<span slot="summary">Overriding the `Field::make()`, `MorphToSelect::make()`, `Placeholder::make()`, or `Builder\Block::make()` methods</span>

The signature for the `Field::make()`, `MorphToSelect::make()`, `Placeholder::make()`, and `Builder\Block::make()` methods has changed. Any classes that extend the `Field`, `MorphToSelect`, `Placeholder`, or `Builder\Block` class and override the `make()` method must update the method signature to match the new signature. The new signature is as follows:

```php
public static function make(?string $name = null): static
```

This is due to the introduction of the `getDefaultName()` method, that can be overridden to provide a default `$name` value if one is not specified (`null`). If you were previously overriding the `make()` method in order to provide a default `$name` value, it is advised that you now override the `getDefaultName()` method instead, to avoid further maintenance burden in the future:

```php
public static function getDefaultName(): ?string
{
    return 'default';
}
```

If you’re overriding the `make()` method to pass default configuration to the object once it is instantiated, please note that it is recommended to instead override the `setUp()` method, which is called immediately after the object is instantiated:

```php
protected function setUp(): void
{
    parent::setUp();

    $this->label('Default label');
}
```

Ideally, you should avoid overriding the `make()` method altogether as there are alternatives like `setUp()`, and doing so causes your code to be brittle if Filament decides to introduce new constructor parameters in the future.
</Disclosure>

<Disclosure x-show="packages.includes('infolists')">
<span slot="summary">Overriding the `Entry::make()` method</span>

The signature for the `Entry::make()` method has changed. Any classes that extend the `Entry` class and override the `make()` method must update the method signature to match the new signature. The new signature is as follows:

```php
public static function make(?string $name = null): static
```

This is due to the introduction of the `getDefaultName()` method, that can be overridden to provide a default `$name` value if one is not specified (`null`). If you were previously overriding the `make()` method in order to provide a default `$name` value, it is advised that you now override the `getDefaultName()` method instead, to avoid further maintenance burden in the future:

```php
public static function getDefaultName(): ?string
{
    return 'default';
}
```

If you’re overriding the `make()` method to pass default configuration to the object once it is instantiated, please note that it is recommended to instead override the `setUp()` method, which is called immediately after the object is instantiated:

```php
protected function setUp(): void
{
    parent::setUp();

    $this->label('Default label');
}
```

Ideally, you should avoid overriding the `make()` method altogether as there are alternatives like `setUp()`, and doing so causes your code to be brittle if Filament decides to introduce new constructor parameters in the future.
</Disclosure>

<Disclosure x-show="packages.includes('tables')">
<span slot="summary">Overriding the `Column::make()` or `Constraint::make()` methods</span>

The signature for the `Column::make()` and `Constraint::make()` methods has changed. Any classes that extend the `Column` or `Constraint` class and override the `make()` method must update the method signature to match the new signature. The new signature is as follows:

```php
public static function make(?string $name = null): static
```

This is due to the introduction of the `getDefaultName()` method, that can be overridden to provide a default `$name` value if one is not specified (`null`). If you were previously overriding the `make()` method in order to provide a default `$name` value, it is advised that you now override the `getDefaultName()` method instead, to avoid further maintenance burden in the future:

```php
public static function getDefaultName(): ?string
{
    return 'default';
}
```

If you’re overriding the `make()` method to pass default configuration to the object once it is instantiated, please note that it is recommended to instead override the `setUp()` method, which is called immediately after the object is instantiated:

```php
protected function setUp(): void
{
    parent::setUp();

    $this->label('Default label');
}
```

Ideally, you should avoid overriding the `make()` method altogether as there are alternatives like `setUp()`, and doing so causes your code to be brittle if Filament decides to introduce new constructor parameters in the future.
</Disclosure>

<Disclosure x-show="packages.includes('actions')">
<span slot="summary">Overriding the `ExportColumn::make()` or `ImportColumn::make()` methods</span>

The signature for the `ExportColumn::make()` and `ImportColumn::make()` methods has changed. Any classes that extend the `ExportColumn` or `ImportColumn` class and override the `make()` method must update the method signature to match the new signature. The new signature is as follows:

```php
public static function make(?string $name = null): static
```

This is due to the introduction of the `getDefaultName()` method, that can be overridden to provide a default `$name` value if one is not specified (`null`). If you were previously overriding the `make()` method in order to provide a default `$name` value, it is advised that you now override the `getDefaultName()` method instead, to avoid further maintenance burden in the future:

```php
public static function getDefaultName(): ?string
{
    return 'default';
}
```

If you’re overriding the `make()` method to pass default configuration to the object once it is instantiated, please note that it is recommended to instead override the `setUp()` method, which is called immediately after the object is instantiated:

```php
protected function setUp(): void
{
    parent::setUp();

    $this->label('Default label');
}
```

Ideally, you should avoid overriding the `make()` method altogether as there are alternatives like `setUp()`, and doing so causes your code to be brittle if Filament decides to introduce new constructor parameters in the future.
</Disclosure>

<Disclosure x-show="packages.includes('actions')">
<span slot="summary">Authenticating the user inside the import and export jobs</span>

In v3, the `Illuminate\Auth\Events\Login` event was fired from the import and export jobs, to set the current user. This is no longer the case in v4: the user is authenticated, but that event is not fired, to avoid running any listeners that should only run for actual user logins.
</Disclosure>

<Disclosure x-show="packages.includes('tables')">
<span slot="summary">Tables now have default primary key sorting</span>

Filament v4 introduces a new default behavior for tables: they will now automatically have a primary key sort applied to their queries to ensure that records are always returned in a consistent order.

If your table doesn't have a primary key, or you want to disable this behavior, you can do so by using the `defaultKeySort(false)` method:

```php
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->defaultKeySort(false);
}
```

<Aside variant="tip">
    You can preserve the old default behavior across your entire app by adding the following code in the `boot()` method of a service provider like `AppServiceProvider`:

    ```php
    use Filament\Tables\Table;

    Table::configureUsing(fn (Table $table) => $table
        ->defaultKeySort(false));
    ```
</Aside>
</Disclosure>

<Disclosure x-show="packages.includes('panels')">
<span slot="summary">Overriding the `can*()` authorization methods on a `Resource`, `RelationManager` or `ManageRelatedRecords` class</span>

Although these methods, such as `canCreate()`, `canViewAny()` and `canDelete()`weren’t documented, if you’re overriding those to provide custom authorization logic in v3, you should be aware that they aren’t always called in v4. The authorization logic has been improved to properly support [policy response objects](https://laravel.com/docs/authorization#policy-responses), and these methods were too simple as they are just able to return booleans.

If you can make the authorization customization inside the policy of the model instead, you should do that. If you need to customize the authorization logic in the resource or relation manager class, you should override the `get*AuthorizationResponse()` methods instead, such as `getCreateAuthorizationResponse()`, `getViewAnyAuthorizationResponse()` and `getDeleteAuthorizationResponse()`. These methods are called when the authorization logic is executed, and they return a [policy response object](https://laravel.com/docs/authorization#policy-responses). If you remove the override for the `can*()` methods, the `get*AuthorizationResponse()` methods will be used to determine the authorization response boolean, so you don't have to maintain the logic twice.
</Disclosure>

<Disclosure>
<span slot="summary">European Portuguese translations</span>

The European Portuguese translations have been moved from `pt_PT` to `pt`, which appears to be the more commonly used language code for the language within the Laravel community.
</Disclosure>

<Disclosure>
<span slot="summary">Nepalese translations</span>

The Nepalese translations have been moved from `np` to `ne`, which appears to be the more commonly used language code for the language within the Laravel community.
</Disclosure>

<Disclosure>
<span slot="summary">Norwegian translations</span>

The Norwegian translations have been moved from `no` to `nb`, which appears to be the more commonly used language code for the language within the Laravel community.
</Disclosure>

<Disclosure x-show="packages.includes('actions')">
<span slot="summary">Khmer translations</span>

The Khmer translations have been moved from `kh` to `km`, which appears to be the more commonly used language code for the language within the Laravel community.
</Disclosure>

<Disclosure x-show="packages.includes('tables')">
<span slot="summary">Some deprecated table configuration methods have been removed</span>

Before Filament v3, tables could be configured by overriding methods on the Livewire component class, instead of modifying the `$table` configuration object. This was deprecated in v3, and removed in v4. If you were using any of the following methods, you should remove them from your Livewire component class, and use their corresponding `$table` configuration methods instead:

- `getTableRecordUrlUsing()` should be replaced with `$table->recordUrl()`
- `getTableRecordClassesUsing()` should be replaced with `$table->recordClasses()`
- `getTableRecordActionUsing()` should be replaced with `$table->recordAction()`
- `isTableRecordSelectable()` should be replaced with `$table->checkIfRecordIsSelectableUsing()`
</Disclosure>

</div>

---

<!-- Source: filamentphp4doc.md -->

import Aside from "@components/Aside.astro"
import Disclosure from "@components/Disclosure.astro"

## Introduction

**Filament is a Server-Driven UI (SDUI) framework for Laravel.** It allows you to define user interfaces entirely in PHP using structured configuration objects, rather than traditional templating. Built on top of Livewire, Alpine.js, and Tailwind CSS, Filament empowers you to build full-featured interfaces like admin panels, dashboards, and form-based apps, all without writing custom JavaScript or frontend code.

<Disclosure>
    <span slot="summary">What is Server-Driven UI?</span>

    SDUI is a proven architecture used by companies like Meta, Airbnb, and Shopify. It moves control of the UI to the server, allowing for faster iteration, greater consistency, and centralized logic. Filament embraces this pattern for web development, letting you define interfaces declaratively using PHP classes that are rendered into HTML by the server.
    
    One key distinction to note is the difference between Server-Driven UI (SDUI) and Server-Rendered UI. While both approaches involve rendering content on the server, Server-Rendered UI relies on static templates (like traditional Blade views), where the structure and behavior of the UI are defined upfront in HTML or PHP files. In contrast, SDUI gives the server the power to dynamically generate the UI based on real-time configurations and business logic, allowing for more flexibility and reactivity without needing to modify frontend templates directly.
</Disclosure>

Thousands of developers use Filament to add admin panels to their Laravel applications, but it goes far beyond that. You can use Filament to build custom dashboards, user portals, CRMs, or even full applications with multiple panels. It integrates seamlessly with any frontend stack and works especially well alongside tools like Inertia.js, Livewire, and Blade.

If you're already using Blade views in your Laravel app, Filament components can enhance them too. You can drop Livewire components powered by Filament into any Blade view or route, and use the same schema-based builders for forms, tables, and more, all without switching your entire stack.

## Packages

The core of Filament comprises several packages:

- `filament/filament` - The core package for building panels (e.g., admin panels). This requires all other packages since panels often use many of their features.
- `filament/tables` - A data table builder that allows you to render an interactive table with filtering, sorting, pagination, and more.
- `filament/schemas` - A package that allows you to build UIs using an array of "component" PHP objects as configuration. This is used by many features in Filament to render UI. The package includes a base set of components that allow you to render content.
- `filament/forms` - A set of `filament/schemas` components for a large variety of form inputs (fields), complete with integrated validation.
- `filament/infolists` - A set of `filament/schemas` components for rendering "description lists". An infolist consists of "entries", which are key-value UI elements that can present read-only information like text, icons, and images. The data for an infolist can be sourced from anywhere but commonly comes from an individual Eloquent record.
- `filament/actions` - Action objects encapsulate the UI for a button, an interactive modal window that can be opened by the button, and the logic that should be executed when the modal window is submitted. They can be used anywhere in the UI and are commonly used to perform one-time actions like deleting a record, sending an email, or updating data in the database based on modal form input.
- `filament/notifications` - An easy way to send notifications to users in your app's UI. You can send a "flash" notification that appears immediately after a request to the server, a "database" notification which is stored in the database and rendered in a slide-over modal the user can open on demand, or a "broadcast" notification that is sent to the user in real-time over a websocket connection.
- `filament/widgets` - A set of dashboard "widgets" that can render anything, often statistical data. Charts, numbers, tables, and completely custom widgets can be rendered in a dashboard using this package.
- `filament/support` - This package contains a set of shared UI components and utilities used by all other packages. Users don't commonly install this directly, as it is a dependency of all other packages.

## Plugins

Filament is designed to be highly extensible, allowing you to add your own UI components and features to the framework. These extensions can live inside your codebase if they're specific to your application, or be distributed as Composer packages if they're general-purpose. In the Filament ecosystem, these Composer packages are called "plugins", and hundreds are available from the community. The Filament team also officially maintains several plugins that provide integration with popular third-party packages in the Laravel ecosystem.

The vast majority of plugins in the ecosystem are open-source and free to use. Some premium plugins are available for purchase, often offering enhanced customer support and quality.

<Aside variant="warning">
    Plugins not maintained by Filament team are created and managed by independent authors. While these plugins can enhance your experience, Filament cannot guarantee their quality, security, compatibility, or maintenance. We recommend reviewing the plugin's code, documentation, and user feedback before installation.
</Aside>

You can browse an extensive list of official and community plugins on the [Filament website](/plugins).

## Customizing the appearance

Tailwind CSS is a utility-based CSS framework that Filament uses as a token-based design system. Although Filament does not use Tailwind CSS utility classes directly in the HTML rendered by its components, it compiles Tailwind utilities into semantic CSS. These semantic classes can be targeted by Filament users with their own CSS to modify the appearance of components, creating a thin layer of overrides on top of the default Filament design.

A simple example demonstrating the power of this system is changing the border radius of all button components in Filament. By default, the following CSS code is used in the Filament codebase to style buttons using Tailwind utility classes:

```css
.fi-btn {
    @apply rounded-lg px-3 py-2 text-sm font-medium outline-none;
}
```

To decrease the [border radius in Tailwind CSS](https://tailwindcss.com/docs/border-radius), you can apply the `rounded-sm` (small) utility class to `.fi-btn` in your own CSS file:

```css
.fi-btn {
    @apply rounded-sm;
}
```

This overrides the default `rounded-lg` class with `rounded-sm` for all buttons in Filament while preserving the other styling properties of the button. This system provides a high level of flexibility to customize the appearance of Filament components without needing to write a complete custom stylesheet or maintain copies of HTML for each component.

For more information about customizing the appearance of Filament, visit the [Customizing styling documentation](../styling).

## Testing

The core packages in Filament undergo unit testing to ensure stability across releases. As a Filament user, you can write tests for applications built with the framework. Filament provides utilities for testing both functionality and UI components, compatible with either Pest or PHPUnit test suites. Testing is particularly crucial when customizing the framework or implementing custom functionality, though it's also valuable for verifying basic functionality works as intended.

For more information about testing Filament applications, visit the [Testing documentation](../testing).

## Alternatives to Filament

If you're reading this and concluding that Filament might not be the right choice for your project, that's okay! There are many other excellent projects in the Laravel ecosystem that might be a better fit for your needs. Here are a few we really like:

### Filament sounds too complicated

[Laravel Nova](https://nova.laravel.com) is an easy way to build an admin panel for your Laravel application. It's an official project maintained by the Laravel team, and purchasing it helps support the development of the Laravel framework.

### I do not want to use Livewire to customize anything

Many parts of Filament do not require you to touch Livewire at all, but building custom components might.

[Laravel Nova](https://nova.laravel.com) is built with Vue.js and Inertia.js, which might be a better fit for your project if it requires extensive customization and you have experience with these tools.

### I need an out-of-the-box CMS

[Statamic](https://statamic.com) is a CMS built on Laravel. It's a great choice if you need a CMS that's easy to set up and use, and you don't need to build a custom admin panel.

### I just want to write Blade views and handle the backend myself

[Flux](https://fluxui.dev) is the official Livewire UI kit and ships as a set of pre-built and pre-styled Blade components. It is maintained by the same team that maintains Livewire and Alpine.js, and purchasing it helps support the development of these projects.

---
title: Installation
contents: false
---
import Aside from "@components/Aside.astro"
import RadioGroup from "@components/RadioGroup.astro"
import RadioGroupOption from "@components/RadioGroupOption.astro"

Filament requires the following to run:

- PHP 8.2+
- Laravel v11.28+
- Tailwind CSS v4.0+

Installation comes in two flavors, depending on whether you want to build an app using our panel builder or use the components within your app's Blade views:

<div x-data="{ package: (window.location.hash === '#components') ? 'components' : 'panels' }">

<RadioGroup model="package">
    <RadioGroupOption value="panels">
        Panel builder

        <span slot="description">
            Most people choose this option to build a panel (e.g., admin panel) for their app. The panel builder combines all the individual components into a cohesive framework. You can create as many panels as you like within a Laravel installation, but you only need to install it once.
        </span>
    </RadioGroupOption>

    <RadioGroupOption value="components">
        Individual components

        <span slot="description">
            If you are using Blade to build your app from scratch, you can install individual components from Filament to enrich your UI.
        </span>
    </RadioGroupOption>
</RadioGroup>

<div x-show="package === 'panels'" x-cloak>

## Installing the panel builder

Install the Filament Panel Builder by running the following commands in your Laravel project directory:

```bash
composer require filament/filament:"^4.0"

php artisan filament:install --panels
```

<Aside variant="warning">
    When using Windows PowerShell to install Filament, you may need to run the command below, since it ignores `^` characters in version constraints:

    ```bash
    composer require filament/filament:"~4.0"

    php artisan filament:install --panels
    ```
</Aside>

This will create and register a new [Laravel service provider](https://laravel.com/docs/providers) called `app/Providers/Filament/AdminPanelProvider.php`.

<Aside variant="tip">
    If you get an error when accessing your panel, check that the service provider is registered in `bootstrap/providers.php`. If it's not registered, you'll need to [add it manually](https://laravel.com/docs/providers#registering-providers).
</Aside>

You can create a new user account using the following command:

```bash
php artisan make:filament-user
```

Open `/admin` in your web browser, sign in, and [start building your app](../getting-started)!

</div>

<div
    x-show="package === 'components'" 
    x-data="{ laravelProject: 'new' }"
    x-cloak
>

## Installing the individual components

Install the Filament components you want to use with Composer:

```bash
composer require
    filament/tables:"^4.0"
    filament/schemas:"^4.0"
    filament/forms:"^4.0"
    filament/infolists:"^4.0"
    filament/actions:"^4.0"
    filament/notifications:"^4.0"
    filament/widgets:"^4.0"
```

You can install additional packages later in your project without having to repeat these installation steps.

<Aside variant="warning">
    When using Windows PowerShell to install Filament, you may need to run the command below, since it ignores `^` characters in version constraints:

    ```bash
    composer require
        filament/tables:"~4.0"
        filament/schemas:"~4.0"
        filament/forms:"~4.0"
        filament/infolists:"~4.0"
        filament/actions:"~4.0"
        filament/notifications:"~4.0"
        filament/widgets:"~4.0"
    ```
</Aside>

If you only want to use the set of [Blade UI components](../components), you'll need to require `filament/support` at this stage.

<RadioGroup model="laravelProject">
    <RadioGroupOption value="new">
        New Laravel projects

        <span slot="description">
            Get started with Filament components quickly by running a simple command. Note that this will overwrite any modified files in your app, so it's only suitable for new Laravel projects.
        </span>
    </RadioGroupOption>

    <RadioGroupOption value="existing">
        Existing Laravel projects

        <span slot="description">
            If you have an existing Laravel project, you can still install Filament, but should do so manually to preserve your existing functionality.
        </span>
    </RadioGroupOption>
</RadioGroup>

<div x-show="laravelProject === 'new'" x-cloak>

To quickly set up Filament in a new Laravel project, run the following commands to install [Livewire](https://livewire.laravel.com), [Alpine.js](https://alpinejs.dev), and [Tailwind CSS](https://tailwindcss.com):

<Aside variant="warning">
    These commands will overwrite existing files in your application. Only run them in a new Laravel project!
</Aside>

Run the following command to install the Filament frontend assets:

```bash
php artisan filament:install --scaffold

npm install

npm run dev
```

During scaffolding, if you have the [Notifications](../notifications) package installed, Filament will ask if you want to install the required Livewire component into your default layout file. This component is required if you want to send flash notifications to users through Filament.

</div>

<div x-show="laravelProject === 'existing'" x-cloak>

Run the following command to install the Filament frontend assets:

```bash
php artisan filament:install
```

### Installing Tailwind CSS

Run the following command to install Tailwind CSS and its Vite plugin, if you don't have those installed already:

```bash
npm install tailwindcss @tailwindcss/vite --save-dev
```

### Configuring styles

To configure Filament's styles, you need to import the CSS files for the Filament packages you installed, usually in `resources/css/app.css`.

Depending on which combination of packages you installed, you can import only the necessary CSS files, to keep your app's CSS bundle size small:

```css
@import 'tailwindcss';

/* Required by all components */
@import '../../vendor/filament/support/resources/css/index.css';

/* Required by actions and tables */
@import '../../vendor/filament/actions/resources/css/index.css';

/* Required by actions, forms and tables */
@import '../../vendor/filament/forms/resources/css/index.css';

/* Required by actions and infolists */
@import '../../vendor/filament/infolists/resources/css/index.css';

/* Required by notifications */
@import '../../vendor/filament/notifications/resources/css/index.css';

/* Required by actions, infolists, forms, schemas and tables */
@import '../../vendor/filament/schemas/resources/css/index.css';

/* Required by tables */
@import '../../vendor/filament/tables/resources/css/index.css';

/* Required by widgets */
@import '../../vendor/filament/widgets/resources/css/index.css';

@variant dark (&:where(.dark, .dark *));
```

### Configure the Vite plugin

If it isn't already set up, add the `@tailwindcss/vite` plugin to your Vite configuration (`vite.config,js`):

```js
import { defineConfig } from 'vite'
import laravel from 'laravel-vite-plugin'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
        tailwindcss(),
    ],
})
```

### Compiling assets

Compile your new CSS and JavaScript assets using `npm run dev`.

### Configuring your layout 

If you don't have a Blade layout file yet, create it at `resources/views/components/layouts/app.blade.php` by running the following command:

```bash
php artisan livewire:layout
```

Add the following code to your new layout file:

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">

        <meta name="application-name" content="{{ config('app.name') }}">
        <meta name="csrf-token" content="{{ csrf_token() }}">
        <meta name="viewport" content="width=device-width, initial-scale=1">

        <title>{{ config('app.name') }}</title>

        <style>
            [x-cloak] {
                display: none !important;
            }
        </style>

        @filamentStyles
        @vite('resources/css/app.css')
    </head>

    <body class="antialiased">
        {{ $slot }}

        @livewire('notifications') {{-- Only required if you wish to send flash notifications --}}

        @filamentScripts
        @vite('resources/js/app.js')
    </body>
</html>
```

The important parts of this are the `@filamentStyles` in the `<head>` of the layout, and the `@filamentScripts` at the end of the `<body>`. Make sure to also include your app's compiled CSS and JavaScript files from Vite!

<Aside variant="info">
    The `@livewire('notifications')` line is required in the layout only if you have the [Notifications](../notifications) package installed and want to send flash notifications to users through Filament.
</Aside>

</div>

</div>

</div>
---
title: Optimizing local development
---
import Aside from "@components/Aside.astro"

This section includes optional tips to optimize performance when running your Filament app locally.

If you're looking for production-specific optimizations, check out [Deploying to production](../deployment).

## Enabling OPcache

[OPcache](https://www.php.net/manual/en/book.opcache.php) improves PHP performance by storing precompiled script bytecode in shared memory, thereby removing the need for PHP to load and parse scripts on each request. This can significantly speed up your local development environment, especially for larger applications.

### Checking OPcache status

To check if [OPcache](https://www.php.net/manual/en/book.opcache.php) is enabled, run:

```bash
php -r "echo 'opcache.enable => ' . ini_get('opcache.enable') . PHP_EOL;"
```

You should see `opcache.enable => 1`. If not, enable it by adding the following line to your `php.ini`:

```ini
opcache.enable=1 # Enable OPcache
```

<Aside variant="tip">
    To locate your `php.ini` file, run: `php --ini`
</Aside>

### Configuring OPcache settings

If you're experiencing slow response times or suspect that OPcache is running out of space, you can adjust these parameters in your `php.ini` file:

```ini
opcache.memory_consumption=128
opcache.max_accelerated_files=10000
```

<Aside variant="tip">
    To locate your `php.ini` file, run: `php --ini`
</Aside>

- `opcache.memory_consumption`: defines how much memory (in megabytes) OPcache can use to store precompiled PHP code. You can try setting this to `128` and adjust based on your project's needs.
- `opcache.max_accelerated_files`: sets the maximum number of PHP files that OPcache can cache. You can try `10000` as a starting point and increase if your application includes a large number of files.

These settings are optional but useful if you're troubleshooting performance or working on a large Laravel app.

## Exclude your project folder from antivirus scanning

Issues with the performance of Filament, particularly on Windows, often involve [Microsoft Defender](https://www.microsoft.com/en-us/microsoft-365/microsoft-defender-for-individuals).

Security software, such as realtime file scanners or antivirus tools, can slow down your development environment by scanning files every time they're accessed. This can affect PHP execution, view rendering, and performance in general.

If you're noticing slowness, consider excluding your local project folder from realtime scanning.

Tools like [Microsoft Defender](https://www.microsoft.com/en-us/microsoft-365/microsoft-defender-for-individuals), or similar antivirus solutions, can be configured to skip specific directories. Check your antivirus or security software documentation for instructions on how to exclude specific folders from realtime scanning.

<Aside variant="warning">
    Only exclude folders from scanning if you fully trust the project and understand the risks.
</Aside>

## Disabling debugging tools

Debugging tools can be very useful for local development, but they can significantly slow down your application when you aren't actively using them. Temporarily disabling these tools when you need maximum performance can make a noticeable difference in your development experience.

### Disabling view debugging in Laravel Herd

[Laravel Herd](https://herd.laravel.com/) includes a view debugging tool for [macOS](https://herd.laravel.com/docs/macos/debugging/dumps#views) and [Windows](https://herd.laravel.com/docs/windows/debugging/dumps#views). It shows which views were rendered and what data was passed to them during a request.

While helpful for debugging, this feature can significantly slow down your app. If you're not actively using it, it's best to turn it off.

To disable view debugging in Herd:

- Open Herd > Dumps.
- Click Settings.
- Uncheck the "Views" option.

### Disabling Debugbar

While useful for debugging, [Laravel Debugbar](https://github.com/barryvdh/laravel-debugbar) can slow down your application, especially on complex pages, because it collects and renders a large amount of data on each request.

If you're noticing slowness, try disabling it by adding the following line to your `.env` file:

```dotenv
DEBUGBAR_ENABLED=false
```

If you still need Debugbar for development, consider disabling specific collectors you're not using.
Refer to the [Debugbar documentation](https://github.com/barryvdh/laravel-debugbar?tab=readme-ov-file#debugbar-for-laravel) for details.

### Disabling Xdebug

[Xdebug](https://xdebug.org) is a powerful tool for debugging, but it can significantly slow down performance. If you notice performance issues, check if `Xdebug` is installed and consider disabling it.

If `Xdebug` is installed but not disabled, it will still be enabled by default. If you have it installed, make sure it is explicitly disabled in your `php.ini` file:

```ini
xdebug.mode=off # Disable Xdebug
```

<Aside variant="tip">
    To locate your `php.ini` file, run: `php --ini`
</Aside>

## Caching Blade icons

Caching [Blade icons](https://blade-ui-kit.com/blade-icons) can help improve performance during local development, especially in views that render many icons.

To enable caching, run:

```bash
php artisan icons:cache
```

Make sure that when you install new Blade icon packages, you run the command again to discover the new icons.
---
title: Help
contents: false
---

We offer a variety of support options, mostly free of charge. If you need help, the community is here for you!

## Discord

We are fortunate to have a growing community of Filament users that help each other out on our [Discord server](https://filamentphp.com/discord). Join now, it's free!
We also have many dedicated channels in different languages. Currently, we have channels for the following languages:

- [#ar](https://discord.com/channels/883083792112300104/961199444789973024) - Arabic 🇸🇦
- [#de](https://discord.com/channels/883083792112300104/998221767850070057) - German 🇩🇪
- [#es](https://discord.com/channels/883083792112300104/1049794522181275749) - Spanish 🇪🇸
- [#fa](https://discord.com/channels/883083792112300104/1042736860826443807) - Farsi 🇮🇷
- [#fr](https://discord.com/channels/883083792112300104/978572814317682688) - French 🇫🇷
- [#id](https://discord.com/channels/883083792112300104/1051444835254538271) - Indonesian 🇮🇩
- [#it](https://discord.com/channels/883083792112300104/979015654675996672) - Italian 🇮🇹
- [#ko](https://discord.com/channels/883083792112300104/1221712398017232926) - Korean 🇰🇷
- [#nl](https://discord.com/channels/883083792112300104/998685582031061102) - Dutch 🇳🇱
- [#pt-br](https://discord.com/channels/883083792112300104/966832715536162846) - Portuguese (Brazil) 🇧🇷
- [#tr](https://discord.com/channels/883083792112300104/988729996803702794) - Turkish 🇹🇷

If you are missing a channel for your language, please let us know and we will create one for you.

## GitHub

You can also reach out to us on our [GitHub discussions forum](https://github.com/filamentphp/filament/discussions), where community members and core team members are happy to help you out.

If you find a bug, you can open an [issue](https://github.com/filamentphp/filament/issues/new/choose), and even donate to the bug fix by using the link automatically added to the bottom of every new issue description.

If you have a feature request, you can create a discussion on GitHub [following these instructions](contributing#development-of-new-features). If you are not planning to contribute the feature yourself but the core team adds it to the roadmap, an issue will be created. You can sometimes fast-track the development of a new feature by donating money to it using the link added to the bottom of the issue description.

## One-on-one private support & consulting (paid)

If you're looking for dedicated help with your Filament project, we're here for you. Whether you're a solo developer or running a large company, we provide support and development services that fit your needs. More information can be found on our [consulting page](/consulting).

## Google

Since we make use of [Answer Overflow](https://www.answeroverflow.com/c/883083792112300104) on our [Discord](#discord) server, you are often one Google search away from finding an answer to your question or at least a hint on how to solve your problem. You may also find results from [GitHub](#github), the [Laracasts forum](#laracasts), or even Stack Overflow.

## Helping others

We encourage you to join any of the above platforms and help yourself and our community. Additionally, we encourage you to [contribute](contributing) to Filament itself. We are always looking for new contributors to help us improve Filament.
---
title: Help
contents: false
---

We offer a variety of support options, mostly free of charge. If you need help, the community is here for you!

## Discord

We are fortunate to have a growing community of Filament users that help each other out on our [Discord server](https://filamentphp.com/discord). Join now, it's free!
We also have many dedicated channels in different languages. Currently, we have channels for the following languages:

- [#ar](https://discord.com/channels/883083792112300104/961199444789973024) - Arabic 🇸🇦
- [#de](https://discord.com/channels/883083792112300104/998221767850070057) - German 🇩🇪
- [#es](https://discord.com/channels/883083792112300104/1049794522181275749) - Spanish 🇪🇸
- [#fa](https://discord.com/channels/883083792112300104/1042736860826443807) - Farsi 🇮🇷
- [#fr](https://discord.com/channels/883083792112300104/978572814317682688) - French 🇫🇷
- [#id](https://discord.com/channels/883083792112300104/1051444835254538271) - Indonesian 🇮🇩
- [#it](https://discord.com/channels/883083792112300104/979015654675996672) - Italian 🇮🇹
- [#ko](https://discord.com/channels/883083792112300104/1221712398017232926) - Korean 🇰🇷
- [#nl](https://discord.com/channels/883083792112300104/998685582031061102) - Dutch 🇳🇱
- [#pt-br](https://discord.com/channels/883083792112300104/966832715536162846) - Portuguese (Brazil) 🇧🇷
- [#tr](https://discord.com/channels/883083792112300104/988729996803702794) - Turkish 🇹🇷

If you are missing a channel for your language, please let us know and we will create one for you.

## GitHub

You can also reach out to us on our [GitHub discussions forum](https://github.com/filamentphp/filament/discussions), where community members and core team members are happy to help you out.

If you find a bug, you can open an [issue](https://github.com/filamentphp/filament/issues/new/choose), and even donate to the bug fix by using the link automatically added to the bottom of every new issue description.

If you have a feature request, you can create a discussion on GitHub [following these instructions](contributing#development-of-new-features). If you are not planning to contribute the feature yourself but the core team adds it to the roadmap, an issue will be created. You can sometimes fast-track the development of a new feature by donating money to it using the link added to the bottom of the issue description.

## One-on-one private support & consulting (paid)

If you're looking for dedicated help with your Filament project, we're here for you. Whether you're a solo developer or running a large company, we provide support and development services that fit your needs. More information can be found on our [consulting page](/consulting).

## Google

Since we make use of [Answer Overflow](https://www.answeroverflow.com/c/883083792112300104) on our [Discord](#discord) server, you are often one Google search away from finding an answer to your question or at least a hint on how to solve your problem. You may also find results from [GitHub](#github), the [Laracasts forum](#laracasts), or even Stack Overflow.

## Helping others

We encourage you to join any of the above platforms and help yourself and our community. Additionally, we encourage you to [contribute](contributing) to Filament itself. We are always looking for new contributors to help us improve Filament.
---
title: Version support policy
contents: false
---

## Introduction

| Version | New features               | Bug fixes                          | Security fixes                      |
|---------|----------------------------|------------------------------------|-------------------------------------|
| 1.x     | ❌ ended Jan 1 2022         | ❌ ended Jan 1 2025                 | ❌ ended Jul 1 2025                  |
| 2.x     | ❌ ended Jul 1 2023         | ❌ ended Jan 1 2025                 | ✅ until Jan 1 2026                  |
| 3.x     | ❌ ended Aug 1 2024         | ✅ until Aug 1 2026                 | ✅ until Jan 1 2028                  |
| 4.x     | ✅ until 5.x stable release | ✅ ~1 year after 5.x stable release | ✅ ~2 years after 5.x stable release |

## New features

Pull requests for new features are only accepted for the latest major version, except in special circumstances. Once a new major version is released, the Filament team will no longer accept pull requests for new features in previous versions. Any open pull requests will either be redirected to target the latest major version or closed, depending on conflicts with the new target branch.

## Bug fixes

After a major version is released, the Filament team will continue to merge pull requests for bug fixes in the previous major version for a year. After this period, pull requests for that version will no longer be accepted.

The Filament team processes bug reports for supported versions in chronological order, though critical bugs may be prioritized. Bug fixes are typically developed only for the latest major version. However, contributors can backport fixes to other supported versions by submitting pull requests.

## Security fixes

The Filament team currently plans to continue providing security fixes for major versions for at least two years.

If you discover a security vulnerability in Filament, please [report it through GitHub](https://github.com/filamentphp/filament/security/advisories). All security vulnerabilities will be addressed promptly.

Please note that while a Filament version may receive security fixes, its underlying dependencies (PHP, Laravel, and Livewire) may no longer be supported. Therefore, applications using older versions of Filament could still be vulnerable to security issues in these dependencies.
---
title: Contributing
contents: false
---
import Aside from "@components/Aside.astro"

<Aside variant="info">
    Parts of this guide are adapted from [Laravel's contribution guide](https://laravel.com/docs/contributions), which served as valuable inspiration.
</Aside>

## Reporting bugs

If you discover a bug in Filament, please report it by opening an issue on our [GitHub repository](https://github.com/filamentphp/filament/issues/new/choose). Before opening an issue, search through the [existing issues](https://github.com/filamentphp/filament/issues?q=is%3Aissue) to check if the bug has already been reported.

Please include as much information as possible, particularly the version numbers of packages in your application. You can use this Artisan command in your application to open a new issue with all the correct versions automatically pre-filled:

```bash
php artisan make:filament-issue
```

When creating an issue, we require a "reproduction repository". **Please do not link to your actual project**. Instead, what we need is a _minimal_ reproduction in a fresh project without any unnecessary code. This means it doesn't matter if your real project is private/confidential since we want a link to a separate, isolated reproduction. This allows us to fix the problem much quicker. **Issues will be automatically closed and not reviewed if this is missing, to preserve maintainer time and ensure the process is fair for those who put effort into reporting.** If you believe a reproduction repository is not suitable for the issue, which is a very rare case, please `@danharrin` and explain why. Saying "it's just a simple issue" is not an excuse for not creating a repository! [Need a head start? We have a template Filament project for you.](https://unitedbycode.com/filament-issue)

Remember, bug reports are created in the hope that others with the same problem will be able to collaborate with you on solving it. Do not expect that the bug report will automatically receive attention or that others will jump to fix it. Creating a bug report serves to help yourself and others start on the path of fixing the problem.

## Development of new features

If you would like to propose a new feature or improvement to Filament, you may use our [discussion forum](https://github.com/filamentphp/filament/discussions) hosted on GitHub. If you intend to implement the feature yourself in a pull request, we advise you to `@danharrin` (a core maintainer of Filament) in your feature discussion beforehand and ask if it is suitable for the framework to prevent wasting your time.

## Development of plugins

If you would like to develop a plugin for Filament, please refer to the [plugin development section](../plugins) in the documentation. Our [Discord](https://filamentphp.com/discord) server is also a great place to ask questions and get help with plugin development. You can start a conversation in the [`#plugin-developers-chat`](https://discord.com/channels/883083792112300104/970354547723730955) channel.

You can also [submit your plugin to be advertised on the Filament website](https://github.com/filamentphp/filamentphp.com/blob/main/README.md#contributing).

## Developing with a local copy of Filament

If you want to contribute to the Filament packages, then you may want to test it in a real Laravel project:

- Fork [the GitHub repository](https://github.com/filamentphp/filament) to your GitHub account.
- Create a Laravel app locally.
- Clone your fork into your Laravel app's root directory.
- In the `/filament` directory, create a branch for your fix, e.g. `fix/error-message`.

Install the packages in your app's `composer.json`:

```jsonc
{
    // ...
    "require": {
        "filament/filament": "*",
    },
    "minimum-stability": "dev",
    "repositories": [
        {
            "type": "path",
            "url": "filament/packages/*"
        }
    ],
    // ...
}
```

Now, run `composer update`.

Once you've finished making changes, you can commit them and submit a pull request to [the GitHub repository](https://github.com/filamentphp/filament).

## Checking for outdated translations

To check for outdated translations, you can use our Translation Tool. Clone the Filament repository, install the dependencies for the commands and then run the command.

```bash
# Clone
git clone git@github.com:filamentphp/filament.git

# Install dependencies
composer install

# Run the tool  
./bin/translation-tool.php
```

First select "List outdated translations" as the command and then choose the locale you want to check. This command will show you which translations are missing for the specified locale. You can then submit a pull request with the missing translations to [the GitHub repository](https://github.com/filamentphp/filament).

## Security vulnerabilities

If you discover a security vulnerability within Filament, please [report it through GitHub](https://github.com/filamentphp/filament/security/advisories). All security vulnerabilities will be promptly addressed. Please see our [version support policy](version-support-policy) to understand which versions are currently under maintenance.

## Code of Conduct

Please note that Filament is released with a [Contributor Code of Conduct](https://github.com/filamentphp/filament/blob/4.x/CODE_OF_CONDUCT.md). By participating in this project, you agree to abide by its terms.
---
title: Overview
---
import Aside from "@components/Aside.astro"

## Introduction

Resources are static classes that are used to build CRUD interfaces for your Eloquent models. They describe how administrators should be able to interact with data from your app using tables and forms.

## Creating a resource

To create a resource for the `App\Models\Customer` model:

```bash
php artisan make:filament-resource Customer
```

This will create several files in the `app/Filament/Resources` directory:

```
.
+-- Customers
|   +-- CustomerResource.php
|   +-- Pages
|   |   +-- CreateCustomer.php
|   |   +-- EditCustomer.php
|   |   +-- ListCustomers.php
|   +-- Schemas
|   |   +-- CustomerForm.php
|   +-- Tables
|   |   +-- CustomersTable.php
```

Your new resource class lives in `CustomerResource.php`.

The classes in the `Pages` directory are used to customize the pages in the app that interact with your resource. They're all full-page [Livewire](https://livewire.laravel.com) components that you can customize in any way you wish.

The classes in the `Schemas` directory are used to define the content of the [forms](../forms) and [infolists](../infolists) for your resource. The classes in the `Tables` directory are used to build the table for your resource.

<Aside variant="tip">
    Have you created a resource, but it's not appearing in the navigation menu? If you have a [model policy](#authorization), make sure you return `true` from the `viewAny()` method.
</Aside>

### Simple (modal) resources

Sometimes, your models are simple enough that you only want to manage records on one page, using modals to create, edit and delete records. To generate a simple resource with modals:

```bash
php artisan make:filament-resource Customer --simple
```

Your resource will have a "Manage" page, which is a List page with modals added.

Additionally, your simple resource will have no `getRelations()` method, as [relation managers](managing-relationships) are only displayed on the Edit and View pages, which are not present in simple resources. Everything else is the same.

### Automatically generating forms and tables

If you'd like to save time, Filament can automatically generate the [form](#resource-forms) and [table](#resource-tables) for you, based on your model's database columns, using `--generate`:

```bash
php artisan make:filament-resource Customer --generate
```

### Handling soft-deletes

By default, you will not be able to interact with deleted records in the app. If you'd like to add functionality to restore, force-delete and filter trashed records in your resource, use the `--soft-deletes` flag when generating the resource:

```bash
php artisan make:filament-resource Customer --soft-deletes
```

You can find out more about soft-deleting [here](deleting-records#handling-soft-deletes).

### Generating a View page

By default, only List, Create and Edit pages are generated for your resource. If you'd also like a [View page](viewing-records), use the `--view` flag:

```bash
php artisan make:filament-resource Customer --view
```

### Specifying a custom model namespace

By default, Filament will assume that your model exists in the `App\Models` directory. You can pass a different namespace for the model using the `--model-namespace` flag:

```bash
php artisan make:filament-resource Customer --model-namespace=Custom\\Path\\Models
```

In this example, the model should exist at `Custom\Path\Models\Customer`. Please note the double backslashes `\\` in the command that are required.

Now when [generating the resource](#automatically-generating-forms-and-tables), Filament will be able to locate the model and read the database schema.

### Generating the model, migration and factory at the same name

If you'd like to save time when scaffolding your resources, Filament can also generate the model, migration and factory for the new resource at the same time using the `--model`, `--migration` and `--factory` flags in any combination:

```bash
php artisan make:filament-resource Customer --model --migration --factory
```

## Record titles

A `$recordTitleAttribute` may be set for your resource, which is the name of the column on your model that can be used to identify it from others.

For example, this could be a blog post's `title` or a customer's `name`:

```php
protected static ?string $recordTitleAttribute = 'name';
```

This is required for features like [global search](global-search) to work.

<Aside variant="tip">
    You may specify the name of an [Eloquent accessor](https://laravel.com/docs/eloquent-mutators#defining-an-accessor) if just one column is inadequate at identifying a record.
</Aside>

## Resource forms

Resource classes contain a `form()` method that is used to build the forms on the [Create](creating-records) and [Edit](editing-records) pages.

By default, Filament creates a form schema file for you, which is referenced in the `form()` method. This is to keep your resource class clean and organized, otherwise it can get quite large:

```php
use App\Filament\Resources\Customers\Schemas\CustomerForm;
use Filament\Schemas\Schema;

public static function form(Schema $schema): Schema
{
    return CustomerForm::configure($schema);
}
```

In the `CustomerForm` class, you can define the fields and layout of your form:

```php
use Filament\Forms\Components\TextInput;
use Filament\Schemas\Schema;

public static function configure(Schema $schema): Schema
{
    return $schema
        ->components([
            TextInput::make('name')->required(),
            TextInput::make('email')->email()->required(),
            // ...
        ]);
}
```

The `components()` method is used to define the structure of your form. It is an array of [fields](../forms#available-fields) and [layout components](../schemas/layout#available-layout-components), in the order they should appear in your form.

Check out the Forms docs for a [guide](../forms) on how to build forms with Filament.

<Aside variant="tip">
    If you would prefer to define the form directly in the resource class, you can do so and delete the form schema class altogether:

    ```php
    use Filament\Forms\Components\TextInput;
    use Filament\Schemas\Schema;

    public static function form(Schema $schema): Schema
    {
        return $schema
            ->components([
                TextInput::make('name')->required(),
                TextInput::make('email')->email()->required(),
                // ...
            ]);
    }
    ```
</Aside>

### Hiding components based on the current operation

The `hiddenOn()` method of form components allows you to dynamically hide fields based on the current page or action.

In this example, we hide the `password` field on the `edit` page:

```php
use Filament\Forms\Components\TextInput;
use Filament\Support\Enums\Operation;

TextInput::make('password')
    ->password()
    ->required()
    ->hiddenOn(Operation::Edit),
```

Alternatively, we have a `visibleOn()` shortcut method for only showing a field on one page or action:

```php
use Filament\Forms\Components\TextInput;
use Filament\Support\Enums\Operation;

TextInput::make('password')
    ->password()
    ->required()
    ->visibleOn(Operation::Create),
```

## Resource tables

Resource classes contain a `table()` method that is used to build the table on the [List page](listing-records).

By default, Filament creates a table file for you, which is referenced in the `table()` method. This is to keep your resource class clean and organized, otherwise it can get quite large:

```php
use App\Filament\Resources\Customers\Schemas\CustomersTable;
use Filament\Tables\Table;

public static function table(Table $table): Table
{
    return CustomersTable::configure($table);
}
```

In the `CustomersTable` class, you can define the columns, filters and actions of the table:

```php
use Filament\Actions\BulkActionGroup;
use Filament\Actions\DeleteBulkAction;
use Filament\Actions\EditAction;
use Filament\Tables\Columns\TextColumn;
use Filament\Tables\Filters\Filter;
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;

public static function configure(Table $table): Table
{
    return $table
        ->columns([
            TextColumn::make('name'),
            TextColumn::make('email'),
            // ...
        ])
        ->filters([
            Filter::make('verified')
                ->query(fn (Builder $query): Builder => $query->whereNotNull('email_verified_at')),
            // ...
        ])
        ->recordActions([
            EditAction::make(),
        ])
        ->toolbarActions([
            BulkActionGroup::make([
                DeleteBulkAction::make(),
            ]),
        ]);
}
```

Check out the [tables](../tables) docs to find out how to add table columns, filters, actions and more.

<Aside variant="tip">
    If you would prefer to define the table directly in the resource class, you can do so and delete the table class altogether:

    ```php
    use Filament\Actions\BulkActionGroup;
    use Filament\Actions\DeleteBulkAction;
    use Filament\Actions\EditAction;
    use Filament\Tables\Columns\TextColumn;
    use Filament\Tables\Filters\Filter;
    use Filament\Tables\Table;
    use Illuminate\Database\Eloquent\Builder;
    
    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                TextColumn::make('name'),
                TextColumn::make('email'),
                // ...
            ])
            ->filters([
                Filter::make('verified')
                    ->query(fn (Builder $query): Builder => $query->whereNotNull('email_verified_at')),
                // ...
            ])
            ->recordActions([
                EditAction::make(),
            ])
            ->toolbarActions([
                BulkActionGroup::make([
                    DeleteBulkAction::make(),
                ]),
            ]);
    }
    ```
</Aside>

## Customizing the model label

Each resource has a "model label" which is automatically generated from the model name. For example, an `App\Models\Customer` model will have a `customer` label.

The label is used in several parts of the UI, and you may customize it using the `$modelLabel` property:

```php
protected static ?string $modelLabel = 'cliente';
```

Alternatively, you may use the `getModelLabel()` to define a dynamic label:

```php
public static function getModelLabel(): string
{
    return __('filament/resources/customer.label');
}
```

### Customizing the plural model label

Resources also have a "plural model label" which is automatically generated from the model label. For example, a `customer` label will be pluralized into `customers`.

You may customize the plural version of the label using the `$pluralModelLabel` property:

```php
protected static ?string $pluralModelLabel = 'clientes';
```

Alternatively, you may set a dynamic plural label in the `getPluralModelLabel()` method:

```php
public static function getPluralModelLabel(): string
{
    return __('filament/resources/customer.plural_label');
}
```

### Automatic model label capitalization

By default, Filament will automatically capitalize each word in the model label, for some parts of the UI. For example, in page titles, the navigation menu, and the breadcrumbs.

If you want to disable this behavior for a resource, you can set `$hasTitleCaseModelLabel` in the resource:

```php
protected static bool $hasTitleCaseModelLabel = false;
```

## Resource navigation items

Filament will automatically generate a navigation menu item for your resource using the [plural label](#plural-label).

If you'd like to customize the navigation item label, you may use the `$navigationLabel` property:

```php
protected static ?string $navigationLabel = 'Mis Clientes';
```

Alternatively, you may set a dynamic navigation label in the `getNavigationLabel()` method:

```php
public static function getNavigationLabel(): string
{
    return __('filament/resources/customer.navigation_label');
}
```

### Setting a resource navigation icon

The `$navigationIcon` property supports the name of any Blade component. By default, [Heroicons](https://heroicons.com) are installed. However, you may create your own custom icon components or install an alternative library if you wish.

```php
use BackedEnum;

protected static string | BackedEnum | null $navigationIcon = 'heroicon-o-user-group';
```

Alternatively, you may set a dynamic navigation icon in the `getNavigationIcon()` method:

```php
use BackedEnum;
use Illuminate\Contracts\Support\Htmlable;

public static function getNavigationIcon(): string | BackedEnum | Htmlable | null
{
    return 'heroicon-o-user-group';
}
```

### Sorting resource navigation items

The `$navigationSort` property allows you to specify the order in which navigation items are listed:

```php
protected static ?int $navigationSort = 2;
```

Alternatively, you may set a dynamic navigation item order in the `getNavigationSort()` method:

```php
public static function getNavigationSort(): ?int
{
    return 2;
}
```

### Grouping resource navigation items

You may group navigation items by specifying a `$navigationGroup` property:

```php
use UnitEnum;

protected static string | UnitEnum | null $navigationGroup = 'Shop';
```

Alternatively, you may use the `getNavigationGroup()` method to set a dynamic group label:

```php
public static function getNavigationGroup(): ?string
{
    return __('filament/navigation.groups.shop');
}
```

#### Grouping resource navigation items under other items

You may group navigation items as children of other items, by passing the label of the parent item as the `$navigationParentItem`:

```php
use UnitEnum;

protected static ?string $navigationParentItem = 'Products';

protected static string | UnitEnum | null $navigationGroup = 'Shop';
```

As seen above, if the parent item has a navigation group, that navigation group must also be defined, so the correct parent item can be identified.

You may also use the `getNavigationParentItem()` method to set a dynamic parent item label:

```php
public static function getNavigationParentItem(): ?string
{
    return __('filament/navigation.groups.shop.items.products');
}
```

<Aside variant="tip">
    If you're reaching for a third level of navigation like this, you should consider using [clusters](../navigation/clusters) instead, which are a logical grouping of resources and [custom pages](../navigation/custom-pages), which can share their own separate navigation.
</Aside>

## Generating URLs to resource pages

Filament provides `getUrl()` static method on resource classes to generate URLs to resources and specific pages within them. Traditionally, you would need to construct the URL by hand or by using Laravel's `route()` helper, but these methods depend on knowledge of the resource's slug or route naming conventions.

The `getUrl()` method, without any arguments, will generate a URL to the resource's [List page](listing-records):

```php
use App\Filament\Resources\Customers\CustomerResource;

CustomerResource::getUrl(); // /admin/customers
```

You may also generate URLs to specific pages within the resource. The name of each page is the array key in the `getPages()` array of the resource. For example, to generate a URL to the [Create page](creating-records):

```php
use App\Filament\Resources\Customers\CustomerResource;

CustomerResource::getUrl('create'); // /admin/customers/create
```

Some pages in the `getPages()` method use URL parameters like `record`. To generate a URL to these pages and pass in a record, you should use the second argument:

```php
use App\Filament\Resources\Customers\CustomerResource;

CustomerResource::getUrl('edit', ['record' => $customer]); // /admin/customers/edit/1
```

In this example, `$customer` can be an Eloquent model object, or an ID.

### Generating URLs to resource modals

This can be especially useful if you are using [simple resources](#simple-modal-resources) with only one page.

To generate a URL for an action in the resource's table, you should pass the `tableAction` and `tableActionRecord` as URL parameters:

```php
use App\Filament\Resources\Customers\CustomerResource;
use Filament\Actions\EditAction;

CustomerResource::getUrl(parameters: [
    'tableAction' => EditAction::getDefaultName(),
    'tableActionRecord' => $customer,
]); // /admin/customers?tableAction=edit&tableActionRecord=1
```

Or if you want to generate a URL for an action on the page like a `CreateAction` in the header, you can pass it in to the `action` parameter:

```php
use App\Filament\Resources\Customers\CustomerResource;
use Filament\Actions\CreateAction;

CustomerResource::getUrl(parameters: [
    'action' => CreateAction::getDefaultName(),
]); // /admin/customers?action=create
```

### Generating URLs to resources in other panels

If you have multiple panels in your app, `getUrl()` will generate a URL within the current panel. You can also indicate which panel the resource is associated with, by passing the panel ID to the `panel` argument:

```php
use App\Filament\Resources\Customers\CustomerResource;

CustomerResource::getUrl(panel: 'marketing');
```

## Customizing the resource Eloquent query

Within Filament, every query to your resource model will start with the `getEloquentQuery()` method.

Because of this, it's very easy to apply your own query constraints or [model scopes](https://laravel.com/docs/eloquent#query-scopes) that affect the entire resource:

```php
public static function getEloquentQuery(): Builder
{
    return parent::getEloquentQuery()->where('is_active', true);
}
```

### Disabling global scopes

By default, Filament will observe all global scopes that are registered to your model. However, this may not be ideal if you wish to access, for example, soft-deleted records.

To overcome this, you may override the `getEloquentQuery()` method that Filament uses:

```php
public static function getEloquentQuery(): Builder
{
    return parent::getEloquentQuery()->withoutGlobalScopes();
}
```

Alternatively, you may remove specific global scopes:

```php
public static function getEloquentQuery(): Builder
{
    return parent::getEloquentQuery()->withoutGlobalScopes([ActiveScope::class]);
}
```

More information about removing global scopes may be found in the [Laravel documentation](https://laravel.com/docs/eloquent#removing-global-scopes).

## Customizing the resource URL

By default, Filament will generate a URL based on the name of the resource. You can customize this by setting the `$slug` property on the resource:

```php
protected static ?string $slug = 'pending-orders';
```

## Resource sub-navigation

Sub-navigation allows the user to navigate between different pages within a resource. Typically, all pages in the sub-navigation will be related to the same record in the resource. For example, in a Customer resource, you may have a sub-navigation with the following pages:

- View customer, a [`ViewRecord` page](viewing-records) that provides a read-only view of the customer's details.
- Edit customer, an [`EditRecord` page](editing-records) that allows the user to edit the customer's details.
- Edit customer contact, an [`EditRecord` page](editing-records) that allows the user to edit the customer's contact details. You can [learn how to create more than one Edit page](editing-records#creating-another-edit-page).
- Manage addresses, a [`ManageRelatedRecords` page](managing-relationships#relation-pages) that allows the user to manage the customer's addresses.
- Manage payments, a [`ManageRelatedRecords` page](managing-relationships#relation-pages) that allows the user to manage the customer's payments.

To add a sub-navigation to each "singular record" page in the resource, you can add the `getRecordSubNavigation()` method to the resource class:

```php
use Filament\Resources\Pages\Page;

public static function getRecordSubNavigation(Page $page): array
{
    return $page->generateNavigationItems([
        ViewCustomer::class,
        EditCustomer::class,
        EditCustomerContact::class,
        ManageCustomerAddresses::class,
        ManageCustomerPayments::class,
    ]);
}
```

Each item in the sub-navigation can be customized using the [same navigation methods as normal pages](../navigation).

<Aside variant="tip">
    If you're looking to add sub-navigation to switch *between* entire resources and [custom pages](../navigation/custom-pages), you might be looking for [clusters](../navigation/clusters), which are used to group these together. The `getRecordSubNavigation()` method is intended to construct a navigation between pages that relate to a particular record *inside* a resource.
</Aside>

### Setting the sub-navigation position for a resource

The sub-navigation is rendered at the start of the page by default. You may change the position for all pages in a resource by setting the `$subNavigationPosition` property on the resource. The value may be `SubNavigationPosition::Start`, `SubNavigationPosition::End`, or `SubNavigationPosition::Top` to render the sub-navigation as tabs:

```php
use Filament\Pages\Enums\SubNavigationPosition;

protected static ?SubNavigationPosition $subNavigationPosition = SubNavigationPosition::End;
```

## Deleting resource pages

If you'd like to delete a page from your resource, you can just delete the page file from the `Pages` directory of your resource, and its entry in the `getPages()` method.

For example, you may have a resource with records that may not be created by anyone. Delete the `Create` page file, and then remove it from `getPages()`:

```php
public static function getPages(): array
{
    return [
        'index' => ListCustomers::route('/'),
        'edit' => EditCustomer::route('/{record}/edit'),
    ];
}
```

Deleting a page will not delete any actions that link to that page. Any actions will open a modal instead of sending the user to the non-existent page. For instance, the `CreateAction` on the List page, the `EditAction` on the table or View page, or the `ViewAction` on the table or Edit page. If you want to remove those buttons, you must delete the actions as well.

## Security

## Authorization

For authorization, Filament will observe any [model policies](https://laravel.com/docs/authorization#creating-policies) that are registered in your app. The following methods are used:

- `viewAny()` is used to completely hide resources from the navigation menu, and prevents the user from accessing any pages.
- `create()` is used to control [creating new records](creating-records).
- `update()` is used to control [editing a record](editing-records).
- `view()` is used to control [viewing a record](viewing-records).
- `delete()` is used to prevent a single record from being deleted. `deleteAny()` is used to prevent records from being bulk deleted. Filament uses the `deleteAny()` method because iterating through multiple records and checking the `delete()` policy is not very performant. When using a `DeleteBulkAction`, if you want to call the `delete()` method for each record anyway, you should use the `DeleteBulkAction::make()->authorizeIndividualRecords()` method. Any records that fail the authorization check will not be processed.
- `forceDelete()` is used to prevent a single soft-deleted record from being force-deleted. `forceDeleteAny()` is used to prevent records from being bulk force-deleted. Filament uses the `forceDeleteAny()` method because iterating through multiple records and checking the `forceDelete()` policy is not very performant. When using a `ForceDeleteBulkAction`, if you want to call the `forceDelete()` method for each record anyway, you should use the `ForceDeleteBulkAction::make()->authorizeIndividualRecords()` method. Any records that fail the authorization check will not be processed.
- `restore()` is used to prevent a single soft-deleted record from being restored. `restoreAny()` is used to prevent records from being bulk restored. Filament uses the `restoreAny()` method because iterating through multiple records and checking the `restore()` policy is not very performant. When using a `RestoreBulkAction`, if you want to call the `restore()` method for each record anyway, you should use the `RestoreBulkAction::make()->authorizeIndividualRecords()` method. Any records that fail the authorization check will not be processed.
- `reorder()` is used to control [reordering records in a table](listing-records#reordering-records).

### Skipping authorization

If you'd like to skip authorization for a resource, you may set the `$shouldSkipAuthorization` property to `true`:

```php
protected static bool $shouldSkipAuthorization = true;
```

### Protecting model attributes

Filament will expose all model attributes to JavaScript, except if they are `$hidden` on your model. This is Livewire's behavior for model binding. We preserve this functionality to facilitate the dynamic addition and removal of form fields after they are initially loaded, while preserving the data they may need.

<Aside variant="danger">
    While attributes may be visible in JavaScript, only those with a form field are actually editable by the user. This is not an issue with mass assignment.
</Aside>

To remove certain attributes from JavaScript on the Edit and View pages, you may override [the `mutateFormDataBeforeFill()` method](editing-records#customizing-data-before-filling-the-form):

```php
protected function mutateFormDataBeforeFill(array $data): array
{
    unset($data['is_admin']);

    return $data;
}
```

In this example, we remove the `is_admin` attribute from JavaScript, as it's not being used by the form.
---
title: Listing records
---

## Using tabs to filter the records

You can add tabs above the table, which can be used to filter the records based on some predefined conditions. Each tab can scope the Eloquent query of the table in a different way. To register tabs, add a `getTabs()` method to the List page class, and return an array of `Tab` objects:

```php
use Filament\Schemas\Components\Tabs\Tab;
use Illuminate\Database\Eloquent\Builder;

public function getTabs(): array
{
    return [
        'all' => Tab::make(),
        'active' => Tab::make()
            ->modifyQueryUsing(fn (Builder $query) => $query->where('active', true)),
        'inactive' => Tab::make()
            ->modifyQueryUsing(fn (Builder $query) => $query->where('active', false)),
    ];
}
```

### Customizing the filter tab labels

The keys of the array will be used as identifiers for the tabs, so they can be persisted in the URL's query string. The label of each tab is also generated from the key, but you can override that by passing a label into the `make()` method of the tab:

```php
use Filament\Schemas\Components\Tabs\Tab;
use Illuminate\Database\Eloquent\Builder;

public function getTabs(): array
{
    return [
        'all' => Tab::make('All customers'),
        'active' => Tab::make('Active customers')
            ->modifyQueryUsing(fn (Builder $query) => $query->where('active', true)),
        'inactive' => Tab::make('Inactive customers')
            ->modifyQueryUsing(fn (Builder $query) => $query->where('active', false)),
    ];
}
```

### Adding icons to filter tabs

You can add icons to the tabs by passing an [icon](../styling/icons) into the `icon()` method of the tab:

```php
use use Filament\Schemas\Components\Tabs\Tab;

Tab::make()
    ->icon('heroicon-m-user-group')
```

You can also change the icon's position to be after the label instead of before it, using the `iconPosition()` method:

```php
use Filament\Support\Enums\IconPosition;

Tab::make()
    ->icon('heroicon-m-user-group')
    ->iconPosition(IconPosition::After)
```

### Adding badges to filter tabs

You can add badges to the tabs by passing a string into the `badge()` method of the tab:

```php
use Filament\Schemas\Components\Tabs\Tab;

Tab::make()
    ->badge(Customer::query()->where('active', true)->count())
```

#### Changing the color of filter tab badges

The color of a badge may be changed using the `badgeColor()` method:

```php
use Filament\Schemas\Components\Tabs\Tab;

Tab::make()
    ->badge(Customer::query()->where('active', true)->count())
    ->badgeColor('success')
```

### Adding extra attributes to filter tabs

You may also pass extra HTML attributes to filter tabs using `extraAttributes()`:

```php
use Filament\Schemas\Components\Tabs\Tab;

Tab::make()
    ->extraAttributes(['data-cy' => 'statement-confirmed-tab'])
```

### Customizing the default tab

To customize the default tab that is selected when the page is loaded, you can return the array key of the tab from the `getDefaultActiveTab()` method:

```php
use Filament\Schemas\Components\Tabs\Tab;

public function getTabs(): array
{
    return [
        'all' => Tab::make(),
        'active' => Tab::make(),
        'inactive' => Tab::make(),
    ];
}

public function getDefaultActiveTab(): string | int | null
{
    return 'active';
}
```

## Authorization

For authorization, Filament will observe any [model policies](https://laravel.com/docs/authorization#creating-policies) that are registered in your app.

Users may access the List page if the `viewAny()` method of the model policy returns `true`.

The `reorder()` method is used to control [reordering a record](#reordering-records).

## Customizing the table Eloquent query

Although you can [customize the Eloquent query for the entire resource](overview#customizing-the-resource-eloquent-query), you may also make specific modifications for the List page table. To do this, use the `modifyQueryUsing()` method in the `table()` method of the resource:

```php
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;

public static function table(Table $table): Table
{
    return $table
        ->modifyQueryUsing(fn (Builder $query) => $query->withoutGlobalScopes());
}
```

## Custom page content

Each page in Filament has its own [schema](../schemas), which defines the overall structure and content. You can override the schema for the page by defining a `content()` method on it. The `content()` method for the List page contains the following components by default:

```php
use Filament\Schemas\Components\EmbeddedTable;
use Filament\Schemas\Components\RenderHook;
use Filament\Schemas\Schema;

public function content(Schema $schema): Schema
{
    return $schema
        ->components([
            $this->getTabsContentComponent(), // This method returns a component to display the tabs above a table
            RenderHook::make(PanelsRenderHook::RESOURCE_PAGES_LIST_RECORDS_TABLE_BEFORE),
            EmbeddedTable::make(), // This is the component that renders the table that is defined in this resource
            RenderHook::make(PanelsRenderHook::RESOURCE_PAGES_LIST_RECORDS_TABLE_AFTER),
        ]);
}
```

Inside the `components()` array, you can insert any [schema component](../schemas). You can reorder the components by changing the order of the array or remove any of the components that are not needed.

### Using a custom Blade view

For further customization opportunities, you can override the static `$view` property on the page class to a custom view in your app:

```php
protected string $view = 'filament.resources.users.pages.list-users';
```

This assumes that you have created a view at `resources/views/filament/resources/users/pages/list-users.blade.php`:

```blade
<x-filament-panels::page>
    {{ $this->content }} {{-- This will render the content of the page defined in the `content()` method, which can be removed if you want to start from scratch --}}
</x-filament-panels::page>
```
---
title: Creating records
---

## Customizing data before saving

Sometimes, you may wish to modify form data before it is finally saved to the database. To do this, you may define a `mutateFormDataBeforeCreate()` method on the Create page class, which accepts the `$data` as an array, and returns the modified version:

```php
protected function mutateFormDataBeforeCreate(array $data): array
{
    $data['user_id'] = auth()->id();

    return $data;
}
```

Alternatively, if you're creating records in a modal action, check out the [Actions documentation](../actions/create#customizing-data-before-saving).

## Customizing the creation process

You can tweak how the record is created using the `handleRecordCreation()` method on the Create page class:

```php
use Illuminate\Database\Eloquent\Model;

protected function handleRecordCreation(array $data): Model
{
    return static::getModel()::create($data);
}
```

Alternatively, if you're creating records in a modal action, check out the [Actions documentation](../actions/create#customizing-the-creation-process).

## Customizing redirects

By default, after saving the form, the user will be redirected to the [Edit page](editing-records) of the resource, or the [View page](viewing-records) if it is present.

You may set up a custom redirect when the form is saved by overriding the `getRedirectUrl()` method on the Create page class.

For example, the form can redirect back to the [List page](listing-records):

```php
protected function getRedirectUrl(): string
{
    return $this->getResource()::getUrl('index');
}
```

If you wish to be redirected to the previous page, else the index page:

```php
protected function getRedirectUrl(): string
{
    return $this->previousUrl ?? $this->getResource()::getUrl('index');
}
```

You can also use the [configuration](../panel-configuration) to customize the default redirect page for all resources at once:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->resourceCreatePageRedirect('index') // or
        ->resourceCreatePageRedirect('view') // or
        ->resourceCreatePageRedirect('edit');
}
```

## Customizing the save notification

When the record is successfully created, a notification is dispatched to the user, which indicates the success of their action.

To customize the title of this notification, define a `getCreatedNotificationTitle()` method on the create page class:

```php
protected function getCreatedNotificationTitle(): ?string
{
    return 'User registered';
}
```

Alternatively, if you're creating records in a modal action, check out the [Actions documentation](../actions/create#customizing-the-save-notification).

You may customize the entire notification by overriding the `getCreatedNotification()` method on the create page class:

```php
use Filament\Notifications\Notification;

protected function getCreatedNotification(): ?Notification
{
    return Notification::make()
        ->success()
        ->title('User registered')
        ->body('The user has been created successfully.');
}
```

To disable the notification altogether, return `null` from the `getCreatedNotification()` method on the create page class:

```php
use Filament\Notifications\Notification;

protected function getCreatedNotification(): ?Notification
{
    return null;
}
```

## Creating another record

### Disabling create another

To disable the "create and create another" feature, define the `$canCreateAnother` property as `false` on the Create page class:

```php
protected static bool $canCreateAnother = false;
```

Alternatively, if you'd like to specify a dynamic condition when the feature is disabled, you may override the `canCreateAnother()` method on the Create page class:

```php
public function canCreateAnother(): bool
{
    return false;
}
```

### Preserving data when creating another

By default, when the user uses the "create and create another" feature, all the form data is cleared so the user can start fresh. If you'd like to preserve some of the data in the form, you may override the `preserveFormDataWhenCreatingAnother()` method on the Create page class, and return the part of the `$data` array that you'd like to keep:

```php
use Illuminate\Support\Arr;

protected function preserveFormDataWhenCreatingAnother(array $data): array
{
    return Arr::only($data, ['is_admin', 'organization']);
}
```

To preserve all the data, return the entire `$data` array:

```php
protected function preserveFormDataWhenCreatingAnother(array $data): array
{
    return $data;
}
```

## Lifecycle hooks

Hooks may be used to execute code at various points within a page's lifecycle, like before a form is saved. To set up a hook, create a protected method on the Create page class with the name of the hook:

```php
protected function beforeCreate(): void
{
    // ...
}
```

In this example, the code in the `beforeCreate()` method will be called before the data in the form is saved to the database.

There are several available hooks for the Create page:

```php
use Filament\Resources\Pages\CreateRecord;

class CreateUser extends CreateRecord
{
    // ...

    protected function beforeFill(): void
    {
        // Runs before the form fields are populated with their default values.
    }

    protected function afterFill(): void
    {
        // Runs after the form fields are populated with their default values.
    }

    protected function beforeValidate(): void
    {
        // Runs before the form fields are validated when the form is submitted.
    }

    protected function afterValidate(): void
    {
        // Runs after the form fields are validated when the form is submitted.
    }

    protected function beforeCreate(): void
    {
        // Runs before the form fields are saved to the database.
    }

    protected function afterCreate(): void
    {
        // Runs after the form fields are saved to the database.
    }
}
```

Alternatively, if you're creating records in a modal action, check out the [Actions documentation](../actions/create#lifecycle-hooks).

## Halting the creation process

At any time, you may call `$this->halt()` from inside a lifecycle hook or mutation method, which will halt the entire creation process:

```php
use Filament\Actions\Action;
use Filament\Notifications\Notification;

protected function beforeCreate(): void
{
    if (! auth()->user()->team->subscribed()) {
        Notification::make()
            ->warning()
            ->title('You don\'t have an active subscription!')
            ->body('Choose a plan to continue.')
            ->persistent()
            ->actions([
                Action::make('subscribe')
                    ->button()
                    ->url(route('subscribe'), shouldOpenInNewTab: true),
            ])
            ->send();
    
        $this->halt();
    }
}
```

Alternatively, if you're creating records in a modal action, check out the [Actions documentation](../actions/create#halting-the-creation-process).

## Authorization

For authorization, Filament will observe any [model policies](https://laravel.com/docs/authorization#creating-policies) that are registered in your app.

Users may access the Create page if the `create()` method of the model policy returns `true`.

## Using a wizard

You may easily transform the creation process into a multistep wizard.

On the page class, add the corresponding `HasWizard` trait:

```php
use App\Filament\Resources\Categories\CategoryResource;
use Filament\Resources\Pages\CreateRecord;

class CreateCategory extends CreateRecord
{
    use CreateRecord\Concerns\HasWizard;
    
    protected static string $resource = CategoryResource::class;

    protected function getSteps(): array
    {
        return [
            // ...
        ];
    }
}
```

Inside the `getSteps()` array, return your [wizard steps](../schemas/wizards):

```php
use Filament\Forms\Components\MarkdownEditor;
use Filament\Forms\Components\TextInput;
use Filament\Forms\Components\Toggle;
use Filament\Schemas\Components\Wizard\Step;

protected function getSteps(): array
{
    return [
        Step::make('Name')
            ->description('Give the category a clear and unique name')
            ->schema([
                TextInput::make('name')
                    ->required()
                    ->live()
                    ->afterStateUpdated(fn ($state, callable $set) => $set('slug', Str::slug($state))),
                TextInput::make('slug')
                    ->disabled()
                    ->required()
                    ->unique(Category::class, 'slug', fn ($record) => $record),
            ]),
        Step::make('Description')
            ->description('Add some extra details')
            ->schema([
                MarkdownEditor::make('description')
                    ->columnSpan('full'),
            ]),
        Step::make('Visibility')
            ->description('Control who can view it')
            ->schema([
                Toggle::make('is_visible')
                    ->label('Visible to customers.')
                    ->default(true),
            ]),
    ];
}
```

Alternatively, if you're creating records in a modal action, check out the [Actions documentation](../actions/create#using-a-wizard).

Now, create a new record to see your wizard in action! Edit will still use the form defined within the resource class.

If you'd like to allow free navigation, so all the steps are skippable, override the `hasSkippableSteps()` method:

```php
public function hasSkippableSteps(): bool
{
    return true;
}
```

### Sharing fields between the form schema and wizards

If you'd like to reduce the amount of repetition between the resource form and wizard steps, it's a good idea to extract public static form functions for your fields, where you can easily retrieve an instance of a field from the form schema or the wizard:

```php
use Filament\Forms;
use Filament\Schemas\Schema;

class CategoryForm
{
    public static function configure(Schema $schema): Schema
    {
        return $schema
            ->components([
                static::getNameFormField(),
                static::getSlugFormField(),
                // ...
            ]);
    }
    
    public static function getNameFormField(): Forms\Components\TextInput
    {
        return TextInput::make('name')
            ->required()
            ->live()
            ->afterStateUpdated(fn ($state, callable $set) => $set('slug', Str::slug($state)));
    }
    
    public static function getSlugFormField(): Forms\Components\TextInput
    {
        return TextInput::make('slug')
            ->disabled()
            ->required()
            ->unique(Category::class, 'slug', fn ($record) => $record);
    }
}
```

```php
use App\Filament\Resources\Categories\Schemas\CategoryForm;
use Filament\Resources\Pages\CreateRecord;

class CreateCategory extends CreateRecord
{
    use CreateRecord\Concerns\HasWizard;
    
    protected static string $resource = CategoryResource::class;

    protected function getSteps(): array
    {
        return [
            Step::make('Name')
                ->description('Give the category a clear and unique name')
                ->schema([
                    CategoryForm::getNameFormField(),
                    CategoryForm::getSlugFormField(),
                ]),
            // ...
        ];
    }
}
```

## Importing resource records

Filament includes an `ImportAction` that you can add to the `getHeaderActions()` of the [List page](listing-records). It allows users to upload a CSV of data to import into the resource:

```php
use App\Filament\Imports\ProductImporter;
use Filament\Actions;

protected function getHeaderActions(): array
{
    return [
        Actions\ImportAction::make()
            ->importer(ProductImporter::class),
        Actions\CreateAction::make(),
    ];
}
```

The "importer" class [needs to be created](../actions/import#creating-an-importer) to tell Filament how to import each row of the CSV. You can learn everything about the `ImportAction` in the [Actions documentation](../actions/import).

## Custom actions

"Actions" are buttons that are displayed on pages, which allow the user to run a Livewire method on the page or visit a URL.

On resource pages, actions are usually in 2 places: in the top right of the page, and below the form.

For example, you may add a new button action in the header of the Create page:

```php
use App\Filament\Imports\UserImporter;
use Filament\Actions;
use Filament\Resources\Pages\CreateRecord;

class CreateUser extends CreateRecord
{
    // ...

    protected function getHeaderActions(): array
    {
        return [
            Actions\ImportAction::make()
                ->importer(UserImporter::class),
        ];
    }
}
```

Or, a new button next to "Create" below the form:

```php
use Filament\Actions\Action;
use Filament\Resources\Pages\CreateRecord;

class CreateUser extends CreateRecord
{
    // ...

    protected function getFormActions(): array
    {
        return [
            ...parent::getFormActions(),
            Action::make('close')->action('createAndClose'),
        ];
    }

    public function createAndClose(): void
    {
        // ...
    }
}
```

To view the entire actions API, please visit the [pages section](../navigation/custom-pages#adding-actions-to-pages).

### Adding a create action button to the header

The "Create" button can be moved to the header of the page by overriding the `getHeaderActions()` method and using `getCreateFormAction()`. You need to pass `formId()` to the action, to specify that the action should submit the form with the ID of `form`, which is the `<form>` ID used in the view of the page:

```php
protected function getHeaderActions(): array
{
    return [
        $this->getCreateFormAction()
            ->formId('form'),
    ];
}
```

You may remove all actions from the form by overriding the `getFormActions()` method to return an empty array:

```php
protected function getFormActions(): array
{
    return [];
}
```

## Custom page content

Each page in Filament has its own [schema](../schemas), which defines the overall structure and content. You can override the schema for the page by defining a `content()` method on it. The `content()` method for the Create page contains the following components by default:

```php
use Filament\Schemas\Schema;

public function content(Schema $schema): Schema
{
    return $schema
        ->components([
            $this->getFormContentComponent(), // This method returns a component to display the form that is defined in this resource
        ]);
}
```

Inside the `components()` array, you can insert any [schema component](../schemas). You can reorder the components by changing the order of the array or remove any of the components that are not needed.

### Using a custom Blade view

For further customization opportunities, you can override the static `$view` property on the page class to a custom view in your app:

```php
protected string $view = 'filament.resources.users.pages.create-user';
```

This assumes that you have created a view at `resources/views/filament/resources/users/pages/create-user.blade.php`:

```blade
<x-filament-panels::page>
    {{ $this->content }} {{-- This will render the content of the page defined in the `content()` method, which can be removed if you want to start from scratch --}}
</x-filament-panels::page>
```
---
title: Viewing records
---

## Creating a resource with a View page

To create a new resource with a View page, you can use the `--view` flag:

```bash
php artisan make:filament-resource User --view
```

## Using an infolist instead of a disabled form

By default, the View page will display a disabled form with the record's data. If you preferred to display the record's data in an "infolist", you can define an `infolist()` method on the resource class:

```php
use Filament\Infolists;
use Filament\Schemas\Schema;

public static function infolist(Schema $schema): Schema
{
    return $schema
        ->components([
            Infolists\Components\TextEntry::make('name'),
            Infolists\Components\TextEntry::make('email'),
            Infolists\Components\TextEntry::make('notes')
                ->columnSpanFull(),
        ]);
}
```

The `components()` method is used to define the structure of your infolist. It is an array of [entries](../infolists#available-entries) and [layout components](../schemas/layout#available-layout-components), in the order they should appear in your infolist.

Check out the Infolists docs for a [guide](../infolists) on how to build infolists with Filament.

## Adding a View page to an existing resource

If you want to add a View page to an existing resource, create a new page in your resource's `Pages` directory:

```bash
php artisan make:filament-page ViewUser --resource=UserResource --type=ViewRecord
```

You must register this new page in your resource's `getPages()` method:

```php
public static function getPages(): array
{
    return [
        'index' => Pages\ListUsers::route('/'),
        'create' => Pages\CreateUser::route('/create'),
        'view' => Pages\ViewUser::route('/{record}'),
        'edit' => Pages\EditUser::route('/{record}/edit'),
    ];
}
```

## Viewing records in modals

If your resource is simple, you may wish to view records in modals rather than on the [View page](viewing-records). If this is the case, you can just [delete the view page](overview#deleting-resource-pages).

If your resource doesn't contain a `ViewAction`, you can add one to the `$table->recordActions()` array:

```php
use Filament\Actions\ViewAction;
use Filament\Tables\Table;

public static function table(Table $table): Table
{
    return $table
        ->columns([
            // ...
        ])
        ->recordActions([
            ViewAction::make(),
            // ...
        ]);
}
```

## Customizing data before filling the form

You may wish to modify the data from a record before it is filled into the form. To do this, you may define a `mutateFormDataBeforeFill()` method on the View page class to modify the `$data` array, and return the modified version before it is filled into the form:

```php
protected function mutateFormDataBeforeFill(array $data): array
{
    $data['user_id'] = auth()->id();

    return $data;
}
```

Alternatively, if you're viewing records in a modal action, check out the [Actions documentation](../actions/view#customizing-data-before-filling-the-form).

## Lifecycle hooks

Hooks may be used to execute code at various points within a page's lifecycle, like before a form is filled. To set up a hook, create a protected method on the View page class with the name of the hook:

```php
use Filament\Resources\Pages\ViewRecord;

class ViewUser extends ViewRecord
{
    // ...

    protected function beforeFill(): void
    {
        // Runs before the disabled form fields are populated from the database. Not run on pages using an infolist.
    }

    protected function afterFill(): void
    {
        // Runs after the disabled form fields are populated from the database. Not run on pages using an infolist.
    }
}
```

## Authorization

For authorization, Filament will observe any [model policies](https://laravel.com/docs/authorization#creating-policies) that are registered in your app.

Users may access the View page if the `view()` method of the model policy returns `true`.

## Creating another View page

One View page may not be enough space to allow users to navigate a lot of information. You can create as many View pages for a resource as you want. This is especially useful if you are using [resource sub-navigation](overview#resource-sub-navigation), as you are then easily able to switch between the different View pages.

To create a View page, you should use the `make:filament-page` command:

```bash
php artisan make:filament-page ViewCustomerContact --resource=CustomerResource --type=ViewRecord
```

You must register this new page in your resource's `getPages()` method:

```php
public static function getPages(): array
{
    return [
        'index' => Pages\ListCustomers::route('/'),
        'create' => Pages\CreateCustomer::route('/create'),
        'view' => Pages\ViewCustomer::route('/{record}'),
        'view-contact' => Pages\ViewCustomerContact::route('/{record}/contact'),
        'edit' => Pages\EditCustomer::route('/{record}/edit'),
    ];
}
```

Now, you can define the `infolist()` or `form()` for this page, which can contain other components that are not present on the main View page:

```php
use Filament\Schemas\Schema;

public function infolist(Schema $schema): Schema
{
    return $schema
        ->components([
            // ...
        ]);
}
```

## Customizing relation managers for a specific view page

You can specify which relation managers should appear on a view page by defining a `getAllRelationManagers()` method:

```php
protected function getAllRelationManagers(): array
{
    return [
        CustomerAddressesRelationManager::class,
        CustomerContactsRelationManager::class,
    ];
}
```

This is useful when you have [multiple view pages](#creating-another-view-page) and need different relation managers on
each page:

```php
// ViewCustomer.php
protected function getAllRelationManagers(): array
{
    return [
        RelationManagers\OrdersRelationManager::class,
        RelationManagers\SubscriptionsRelationManager::class,
    ];
}

// ViewCustomerContact.php 
protected function getAllRelationManagers(): array
{
    return [
        RelationManagers\ContactsRelationManager::class,
        RelationManagers\AddressesRelationManager::class,
    ];
}
```

If `getAllRelationManagers()` isn't defined, any relation managers defined in the resource will be used.

## Adding view pages to resource sub-navigation

If you're using [resource sub-navigation](overview#resource-sub-navigation), you can register this page as normal in `getRecordSubNavigation()` of the resource:

```php
use App\Filament\Resources\Customers\Pages;
use Filament\Resources\Pages\Page;

public static function getRecordSubNavigation(Page $page): array
{
    return $page->generateNavigationItems([
        // ...
        Pages\ViewCustomerContact::class,
    ]);
}
```

## Custom page content

Each page in Filament has its own [schema](../schemas), which defines the overall structure and content. You can override the schema for the page by defining a `content()` method on it. The `content()` method for the View page contains the following components by default:

```php
use Filament\Schemas\Schema;

public function content(Schema $schema): Schema
{
    return $schema
        ->components([
            $this->hasInfolist() // This method returns `true` if the page has an infolist defined
                ? $this->getInfolistContentComponent() // This method returns a component to display the infolist that is defined in this resource
                : $this->getFormContentComponent(), // This method returns a component to display the form that is defined in this resource
            $this->getRelationManagersContentComponent(), // This method returns a component to display the relation managers that are defined in this resource
        ]);
}
```

Inside the `components()` array, you can insert any [schema component](../schemas). You can reorder the components by changing the order of the array or remove any of the components that are not needed.

### Using a custom Blade view

For further customization opportunities, you can override the static `$view` property on the page class to a custom view in your app:

```php
protected string $view = 'filament.resources.users.pages.view-user';
```

This assumes that you have created a view at `resources/views/filament/resources/users/pages/view-user.blade.php`:

```blade
<x-filament-panels::page>
    {{-- `$this->getRecord()` will return the current Eloquent record for this page --}}
    
    {{ $this->content }} {{-- This will render the content of the page defined in the `content()` method, which can be removed if you want to start from scratch --}}
</x-filament-panels::page>
```
---
title: Editing records
---

## Customizing data before filling the form

You may wish to modify the data from a record before it is filled into the form. To do this, you may define a `mutateFormDataBeforeFill()` method on the Edit page class to modify the `$data` array, and return the modified version before it is filled into the form:

```php
protected function mutateFormDataBeforeFill(array $data): array
{
    $data['user_id'] = auth()->id();

    return $data;
}
```

Alternatively, if you're editing records in a modal action, check out the [Actions documentation](../actions/edit#customizing-data-before-filling-the-form).

## Customizing data before saving

Sometimes, you may wish to modify form data before it is finally saved to the database. To do this, you may define a `mutateFormDataBeforeSave()` method on the Edit page class, which accepts the `$data` as an array, and returns it modified:

```php
protected function mutateFormDataBeforeSave(array $data): array
{
    $data['last_edited_by_id'] = auth()->id();

    return $data;
}
```

Alternatively, if you're editing records in a modal action, check out the [Actions documentation](../actions/edit#customizing-data-before-saving).

## Customizing the saving process

You can tweak how the record is updated using the `handleRecordUpdate()` method on the Edit page class:

```php
use Illuminate\Database\Eloquent\Model;

protected function handleRecordUpdate(Model $record, array $data): Model
{
    $record->update($data);

    return $record;
}
```

Alternatively, if you're editing records in a modal action, check out the [Actions documentation](../actions/edit#customizing-the-saving-process).

## Customizing redirects

By default, saving the form will not redirect the user to another page.

You may set up a custom redirect when the form is saved by overriding the `getRedirectUrl()` method on the Edit page class.

For example, the form can redirect back to the [List page](listing-records) of the resource:

```php
protected function getRedirectUrl(): string
{
    return $this->getResource()::getUrl('index');
}
```

Or the [View page](viewing-records):

```php
protected function getRedirectUrl(): string
{
    return $this->getResource()::getUrl('view', ['record' => $this->getRecord()]);
}
```

If you wish to be redirected to the previous page, else the index page:

```php
protected function getRedirectUrl(): string
{
    return $this->previousUrl ?? $this->getResource()::getUrl('index');
}
```

You can also use the [configuration](../panel-configuration) to customize the default redirect page for all resources at once:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->resourceEditPageRedirect('index') // or
        ->resourceEditPageRedirect('view');
}
```

## Customizing the save notification

When the record is successfully updated, a notification is dispatched to the user, which indicates the success of their action.

To customize the title of this notification, define a `getSavedNotificationTitle()` method on the edit page class:

```php
protected function getSavedNotificationTitle(): ?string
{
    return 'User updated';
}
```

Alternatively, if you're editing records in a modal action, check out the [Actions documentation](../actions/edit#customizing-the-save-notification).

You may customize the entire notification by overriding the `getSavedNotification()` method on the edit page class:

```php
use Filament\Notifications\Notification;

protected function getSavedNotification(): ?Notification
{
    return Notification::make()
        ->success()
        ->title('User updated')
        ->body('The user has been saved successfully.');
}
```

To disable the notification altogether, return `null` from the `getSavedNotification()` method on the edit page class:

```php
use Filament\Notifications\Notification;

protected function getSavedNotification(): ?Notification
{
    return null;
}
```

## Lifecycle hooks

Hooks may be used to execute code at various points within a page's lifecycle, like before a form is saved. To set up a hook, create a protected method on the Edit page class with the name of the hook:

```php
protected function beforeSave(): void
{
    // ...
}
```

In this example, the code in the `beforeSave()` method will be called before the data in the form is saved to the database.

There are several available hooks for the Edit pages:

```php
use Filament\Resources\Pages\EditRecord;

class EditUser extends EditRecord
{
    // ...

    protected function beforeFill(): void
    {
        // Runs before the form fields are populated from the database.
    }

    protected function afterFill(): void
    {
        // Runs after the form fields are populated from the database.
    }

    protected function beforeValidate(): void
    {
        // Runs before the form fields are validated when the form is saved.
    }

    protected function afterValidate(): void
    {
        // Runs after the form fields are validated when the form is saved.
    }

    protected function beforeSave(): void
    {
        // Runs before the form fields are saved to the database.
    }

    protected function afterSave(): void
    {
        // Runs after the form fields are saved to the database.
    }
}
```

Alternatively, if you're editing records in a modal action, check out the [Actions documentation](../actions/edit#lifecycle-hooks).

## Saving a part of the form independently

You may want to allow the user to save a part of the form independently of the rest of the form. One way to do this is with a [section action in the header or footer](../schemas/sections#adding-actions-to-the-sections-header-or-footer). From the `action()` method, you can call `saveFormComponentOnly()`, passing in the `Section` component that you want to save:

```php
use Filament\Actions\Action;
use Filament\Notifications\Notification;
use Filament\Resources\Pages\EditRecord;
use Filament\Schemas\Components\Section;

Section::make('Rate limiting')
    ->schema([
        // ...
    ])
    ->footerActions([
        fn (string $operation): Action => Action::make('save')
            ->action(function (Section $component, EditRecord $livewire) {
                $livewire->saveFormComponentOnly($component);
                
                Notification::make()
                    ->title('Rate limiting saved')
                    ->body('The rate limiting settings have been saved successfully.')
                    ->success()
                    ->send();
            })
            ->visible($operation === 'edit'),
    ])
```

The `$operation` helper is available, to ensure that the action is only visible when the form is being edited.

## Halting the saving process

At any time, you may call `$this->halt()` from inside a lifecycle hook or mutation method, which will halt the entire saving process:

```php
use Filament\Actions\Action;
use Filament\Notifications\Notification;

protected function beforeSave(): void
{
    if (! $this->getRecord()->team->subscribed()) {
        Notification::make()
            ->warning()
            ->title('You don\'t have an active subscription!')
            ->body('Choose a plan to continue.')
            ->persistent()
            ->actions([
                Action::make('subscribe')
                    ->button()
                    ->url(route('subscribe'), shouldOpenInNewTab: true),
            ])
            ->send();

        $this->halt();
    }
}
```

Alternatively, if you're editing records in a modal action, check out the [Actions documentation](../actions/edit#halting-the-saving-process).

## Authorization

For authorization, Filament will observe any [model policies](https://laravel.com/docs/authorization#creating-policies) that are registered in your app.

Users may access the Edit page if the `update()` method of the model policy returns `true`.

They also have the ability to delete the record if the `delete()` method of the policy returns `true`.

## Custom actions

"Actions" are buttons that are displayed on pages, which allow the user to run a Livewire method on the page or visit a URL.

On resource pages, actions are usually in 2 places: in the top right of the page, and below the form.

For example, you may add a new button action next to "Delete" on the Edit page:

```php
use Filament\Actions;
use Filament\Resources\Pages\EditRecord;

class EditUser extends EditRecord
{
    // ...

    protected function getHeaderActions(): array
    {
        return [
            Actions\Action::make('impersonate')
                ->action(function (): void {
                    // ...
                }),
            Actions\DeleteAction::make(),
        ];
    }
}
```

Or, a new button next to "Save" below the form:

```php
use Filament\Actions\Action;
use Filament\Resources\Pages\EditRecord;

class EditUser extends EditRecord
{
    // ...

    protected function getFormActions(): array
    {
        return [
            ...parent::getFormActions(),
            Action::make('close')->action('saveAndClose'),
        ];
    }

    public function saveAndClose(): void
    {
        // ...
    }
}
```

To view the entire actions API, please visit the [pages section](../navigation/custom-pages#adding-actions-to-pages).

### Adding a save action button to the header

The "Save" button can be added to the header of the page by overriding the `getHeaderActions()` method and using `getSaveFormAction()`. You need to pass `formId()` to the action, to specify that the action should submit the form with the ID of `form`, which is the `<form>` ID used in the view of the page:

```php
protected function getHeaderActions(): array
{
    return [
        $this->getSaveFormAction()
            ->formId('form'),
    ];
}
```

You may remove all actions from the form by overriding the `getFormActions()` method to return an empty array:

```php
protected function getFormActions(): array
{
    return [];
}
```

## Creating another Edit page

One Edit page may not be enough space to allow users to navigate many form fields. You can create as many Edit pages for a resource as you want. This is especially useful if you are using [resource sub-navigation](overview#resource-sub-navigation), as you are then easily able to switch between the different Edit pages.

To create an Edit page, you should use the `make:filament-page` command:

```bash
php artisan make:filament-page EditCustomerContact --resource=CustomerResource --type=EditRecord
```

You must register this new page in your resource's `getPages()` method:

```php
public static function getPages(): array
{
    return [
        'index' => Pages\ListCustomers::route('/'),
        'create' => Pages\CreateCustomer::route('/create'),
        'view' => Pages\ViewCustomer::route('/{record}'),
        'edit' => Pages\EditCustomer::route('/{record}/edit'),
        'edit-contact' => Pages\EditCustomerContact::route('/{record}/edit/contact'),
    ];
}
```

Now, you can define the `form()` for this page, which can contain other fields that are not present on the main Edit page:

```php
use Filament\Schemas\Schema;

public function form(Schema $schema): Schema
{
    return $schema
        ->components([
            // ...
        ]);
}
```

## Adding edit pages to resource sub-navigation

If you're using [resource sub-navigation](overview#resource-sub-navigation), you can register this page as normal in `getRecordSubNavigation()` of the resource:

```php
use App\Filament\Resources\Customers\Pages;
use Filament\Resources\Pages\Page;

public static function getRecordSubNavigation(Page $page): array
{
    return $page->generateNavigationItems([
        // ...
        Pages\EditCustomerContact::class,
    ]);
}
```

## Custom page content

Each page in Filament has its own [schema](../schemas), which defines the overall structure and content. You can override the schema for the page by defining a `content()` method on it. The `content()` method for the Edit page contains the following components by default:

```php
use Filament\Schemas\Schema;

public function content(Schema $schema): Schema
{
    return $schema
        ->components([
            $this->getFormContentComponent(), // This method returns a component to display the form that is defined in this resource
            $this->getRelationManagersContentComponent(), // This method returns a component to display the relation managers that are defined in this resource
        ]);
}
```

Inside the `components()` array, you can insert any [schema component](../schemas). You can reorder the components by changing the order of the array or remove any of the components that are not needed.

### Using a custom Blade view

For further customization opportunities, you can override the static `$view` property on the page class to a custom view in your app:

```php
protected string $view = 'filament.resources.users.pages.edit-user';
```

This assumes that you have created a view at `resources/views/filament/resources/users/pages/edit-user.blade.php`:

```blade
<x-filament-panels::page>
    {{-- `$this->getRecord()` will return the current Eloquent record for this page --}}
    
    {{ $this->content }} {{-- This will render the content of the page defined in the `content()` method, which can be removed if you want to start from scratch --}}
</x-filament-panels::page>
```
---
title: Deleting records
---

## Handling soft-deletes

## Creating a resource with soft-delete

By default, you will not be able to interact with deleted records in the app. If you'd like to add functionality to restore, force-delete and filter trashed records in your resource, use the `--soft-deletes` flag when generating the resource:

```bash
php artisan make:filament-resource Customer --soft-deletes
```

## Adding soft-deletes to an existing resource

Alternatively, you may add soft-deleting functionality to an existing resource.

Firstly, you must update the resource:

```php
use Filament\Actions\BulkActionGroup;
use Filament\Actions\DeleteAction;
use Filament\Actions\DeleteBulkAction;
use Filament\Actions\ForceDeleteAction;
use Filament\Actions\ForceDeleteBulkAction;
use Filament\Actions\RestoreAction;
use Filament\Actions\RestoreBulkAction;
use Filament\Tables\Filters\TrashedFilter;
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\SoftDeletingScope;

public static function table(Table $table): Table
{
    return $table
        ->columns([
            // ...
        ])
        ->filters([
            TrashedFilter::make(),
            // ...
        ])
        ->recordActions([
            // You may add these actions to your table if you're using a simple
            // resource, or you just want to be able to delete records without
            // leaving the table.
            DeleteAction::make(),
            ForceDeleteAction::make(),
            RestoreAction::make(),
            // ...
        ])
        ->toolbarActions([
            BulkActionGroup::make([
                DeleteBulkAction::make(),
                ForceDeleteBulkAction::make(),
                RestoreBulkAction::make(),
                // ...
            ]),
        ]);
}

public static function getRecordRouteBindingEloquentQuery(): Builder
{
    return parent::getRecordRouteBindingEloquentQuery()
        ->withoutGlobalScopes([
            SoftDeletingScope::class,
        ]);
}
```

Now, update the Edit page class if you have one:

```php
use Filament\Actions;

protected function getHeaderActions(): array
{
    return [
        Actions\DeleteAction::make(),
        Actions\ForceDeleteAction::make(),
        Actions\RestoreAction::make(),
        // ...
    ];
}
```

## Deleting records on the List page

By default, you can bulk-delete records in your table. You may also wish to delete single records, using a `DeleteAction`:

```php
use Filament\Actions\DeleteAction;
use Filament\Tables\Table;

public static function table(Table $table): Table
{
    return $table
        ->columns([
            // ...
        ])
        ->recordActions([
            // ...
            DeleteAction::make(),
        ]);
}
```

## Authorization

For authorization, Filament will observe any [model policies](https://laravel.com/docs/authorization#creating-policies) that are registered in your app.

Users may delete records if the `delete()` method of the model policy returns `true`.

They also have the ability to bulk-delete records if the `deleteAny()` method of the policy returns `true`. Filament uses the `deleteAny()` method because iterating through multiple records and checking the `delete()` policy is not very performant.

You can use the `authorizeIndividualRecords()` method on the `BulkDeleteAction` to check the `delete()` policy for each record individually.

### Authorizing soft-deletes

The `forceDelete()` policy method is used to prevent a single soft-deleted record from being force-deleted. `forceDeleteAny()` is used to prevent records from being bulk force-deleted. Filament uses the `forceDeleteAny()` method because iterating through multiple records and checking the `forceDelete()` policy is not very performant.

The `restore()` policy method is used to prevent a single soft-deleted record from being restored. `restoreAny()` is used to prevent records from being bulk restored. Filament uses the `restoreAny()` method because iterating through multiple records and checking the `restore()` policy is not very performant.
---
title: Managing relationships
---
import Aside from "@components/Aside.astro"

## Choosing the right tool for the job

Filament provides many ways to manage relationships in the app. Which feature you should use depends on the type of relationship you are managing, and which UI you are looking for.

### Relation managers - interactive tables underneath your resource forms

<Aside variant="info">
    These are compatible with `HasMany`, `HasManyThrough`, `BelongsToMany`, `MorphMany` and `MorphToMany` relationships.
</Aside>

[Relation managers](#creating-a-relation-manager) are interactive tables that allow administrators to list, create, attach, associate, edit, detach, dissociate and delete related records without leaving the resource's Edit or View page.

### Select & checkbox list - choose from existing records or create a new one

<Aside variant="info">
    These are compatible with `BelongsTo`, `MorphTo` and `BelongsToMany` relationships.
</Aside>

Using a [select](../forms/select#integrating-with-an-eloquent-relationship), users will be able to choose from a list of existing records. You may also [add a button that allows you to create a new record inside a modal](../forms/select#creating-new-records), without leaving the page.

When using a `BelongsToMany` relationship with a select, you'll be able to select multiple options, not just one. Records will be automatically added to your pivot table when you submit the form. If you wish, you can swap out the multi-select dropdown with a simple [checkbox list](../forms/checkbox-list#integrating-with-an-eloquent-relationship). Both components work in the same way.

### Repeaters - CRUD multiple related records inside the owner's form

<Aside variant="info">
    These are compatible with `HasMany` and `MorphMany` relationships.
</Aside>

[Repeaters](../forms/repeater#integrating-with-an-eloquent-relationship) are standard form components, which can render a repeatable set of fields infinitely. They can be hooked up to a relationship, so records are automatically read, created, updated, and deleted from the related table. They live inside the main form schema, and can be used inside resource pages, as well as nesting within action modals.

From a UX perspective, this solution is only suitable if your related model only has a few fields. Otherwise, the form can get very long.

### Layout form components - saving form fields to a single relationship

<Aside variant="info">
    These are compatible with `BelongsTo`, `HasOne` and `MorphOne` relationships.
</Aside>

All layout form components ([Grid](../schemas/layouts#grid-component), [Section](../schemas/sections), [Fieldset](../schemas/layouts#fieldset-component), etc.) have a [`relationship()` method](../forms/overview#saving-data-to-relationships). When you use this, all fields within that layout are saved to the related model instead of the owner's model:

```php
use Filament\Forms\Components\FileUpload;
use Filament\Forms\Components\Textarea;
use Filament\Forms\Components\TextInput;
use Filament\Schemas\Components\Fieldset;

Fieldset::make('Metadata')
    ->relationship('metadata')
    ->schema([
        TextInput::make('title'),
        Textarea::make('description'),
        FileUpload::make('image'),
    ])
```

In this example, the `title`, `description` and `image` are automatically loaded from the `metadata` relationship, and saved again when the form is submitted. If the `metadata` record does not exist, it is automatically created.

This feature is explained more in depth in the [Forms documentation](../forms/overview#saving-data-to-relationships). Please visit that page for more information about how to use it.

## Creating a relation manager

To create a relation manager, you can use the `make:filament-relation-manager` command:

```bash
php artisan make:filament-relation-manager CategoryResource posts title
```

- `CategoryResource` is the name of the resource class for the owner (parent) model.
- `posts` is the name of the relationship you want to manage.
- `title` is the name of the attribute that will be used to identify posts.

This will create a `CategoryResource/RelationManagers/PostsRelationManager.php` file. This contains a class where you are able to define a [form](overview#resource-forms) and [table](overview#resource-tables) for your relation manager:

```php
use Filament\Forms;
use Filament\Schemas\Schema;
use Filament\Tables;
use Filament\Tables\Table;

public function form(Schema $schema): Schema
{
    return $schema
        ->components([
            Forms\Components\TextInput::make('title')->required(),
            // ...
        ]);
}

public function table(Table $table): Table
{
    return $table
        ->columns([
            Tables\Columns\TextColumn::make('title'),
            // ...
        ]);
}
```

You must register the new relation manager in your resource's `getRelations()` method:

```php
public static function getRelations(): array
{
    return [
        RelationManagers\PostsRelationManager::class,
    ];
}
```

Once a table and form have been defined for the relation manager, visit the [Edit](editing-records) or [View](viewing-records) page of your resource to see it in action.

### Customizing the relation manager's URL parameter

If you pass a key to the array returned from `getRelations()`, it will be used in the URL for that relation manager when switching between multiple relation managers. For example, you can pass `posts` to use `?relation=posts` in the URL instead of a numeric array index:

```php
public static function getRelations(): array
{
    return [
        'posts' => RelationManagers\PostsRelationManager::class,
    ];
}
```

### Read-only mode

Relation managers are usually displayed on either the Edit or View page of a resource. On the View page, Filament will automatically hide all actions that modify the relationship, such as create, edit, and delete. We call this "read-only mode", and it is there by default to preserve the read-only behavior of the View page. However, you can disable this behavior, by overriding the `isReadOnly()` method on the relation manager class to return `false` all the time:

```php
public function isReadOnly(): bool
{
    return false;
}
```

Alternatively, if you hate this functionality, you can disable it for all relation managers at once in the panel [configuration](../panel-configuration):

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->readOnlyRelationManagersOnResourceViewPagesByDefault(false);
}
```

### Unconventional inverse relationship names

For inverse relationships that do not follow Laravel's naming guidelines, you may wish to use the `inverseRelationship()` method on the table:

```php
use Filament\Tables;
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->columns([
            Tables\Columns\TextColumn::make('title'),
            // ...
        ])
        ->inverseRelationship('section'); // Since the inverse related model is `Category`, this is normally `category`, not `section`.
}
```

### Handling soft-deletes

By default, you will not be able to interact with deleted records in the relation manager. If you'd like to add functionality to restore, force-delete and filter trashed records in your relation manager, use the `--soft-deletes` flag when generating the relation manager:

```bash
php artisan make:filament-relation-manager CategoryResource posts title --soft-deletes
```

You can find out more about soft-deleting [here](#deleting-records).

## Listing related records

Related records will be listed in a table. The entire relation manager is based around this table, which contains actions to [create](#creating-related-records), [edit](#editing-related-records), [attach / detach](#attaching-and-detaching-records), [associate / dissociate](#associating-and-dissociating-records), and delete records.

You may use any features of the [Table Builder](../tables) to customize relation managers.

### Listing with pivot attributes

For `BelongsToMany` and `MorphToMany` relationships, you may also add pivot table attributes. For example, if you have a `TeamsRelationManager` for your `UserResource`, and you want to add the `role` pivot attribute to the table, you can use:

```php
use Filament\Tables;

public function table(Table $table): Table
{
    return $table
        ->columns([
            Tables\Columns\TextColumn::make('name'),
            Tables\Columns\TextColumn::make('role'),
        ]);
}
```

Please ensure that any pivot attributes are listed in the `withPivot()` method of the relationship *and* inverse relationship.

## Creating related records

### Creating with pivot attributes

For `BelongsToMany` and `MorphToMany` relationships, you may also add pivot table attributes. For example, if you have a `TeamsRelationManager` for your `UserResource`, and you want to add the `role` pivot attribute to the create form, you can use:

```php
use Filament\Forms;
use Filament\Schemas\Schema;

public function form(Schema $schema): Schema
{
    return $schema
        ->components([
            Forms\Components\TextInput::make('name')->required(),
            Forms\Components\TextInput::make('role')->required(),
            // ...
        ]);
}
```

Please ensure that any pivot attributes are listed in the `withPivot()` method of the relationship *and* inverse relationship.

### Customizing the `CreateAction`

To learn how to customize the `CreateAction`, including mutating the form data, changing the notification, and adding lifecycle hooks, please see the [Actions documentation](../actions/create).

## Editing related records

### Editing with pivot attributes

For `BelongsToMany` and `MorphToMany` relationships, you may also edit pivot table attributes. For example, if you have a `TeamsRelationManager` for your `UserResource`, and you want to add the `role` pivot attribute to the edit form, you can use:

```php
use Filament\Forms;
use Filament\Schemas\Schema;

public function form(Schema $schema): Schema
{
    return $schema
        ->components([
            Forms\Components\TextInput::make('name')->required(),
            Forms\Components\TextInput::make('role')->required(),
            // ...
        ]);
}
```

Please ensure that any pivot attributes are listed in the `withPivot()` method of the relationship *and* inverse relationship.

### Customizing the `EditAction`

To learn how to customize the `EditAction`, including mutating the form data, changing the notification, and adding lifecycle hooks, please see the [Actions documentation](../actions/edit).

## Attaching and detaching records

Filament is able to attach and detach records for `BelongsToMany` and `MorphToMany` relationships.

When generating your relation manager, you may pass the `--attach` flag to also add `AttachAction`, `DetachAction` and `DetachBulkAction` to the table:

```bash
php artisan make:filament-relation-manager CategoryResource posts title --attach
```

Alternatively, if you've already generated your resource, you can just add the actions to the `$table` arrays:

```php
use Filament\Actions\AttachAction;
use Filament\Actions\BulkActionGroup;
use Filament\Actions\DetachAction;
use Filament\Actions\DetachBulkAction;
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->columns([
            // ...
        ])
        ->headerActions([
            // ...
            AttachAction::make(),
        ])
        ->recordActions([
            // ...
            DetachAction::make(),
        ])
        ->toolbarActions([
            BulkActionGroup::make([
                // ...
                DetachBulkAction::make(),
            ]),
        ]);
}
```

### Preloading the attachment modal select options

By default, as you search for a record to attach, options will load from the database via AJAX. If you wish to preload these options when the form is first loaded instead, you can use the `preloadRecordSelect()` method of `AttachAction`:

```php
use Filament\Actions\AttachAction;

AttachAction::make()
    ->preloadRecordSelect()
```

### Attaching with pivot attributes

When you attach record with the `Attach` button, you may wish to define a custom form to add pivot attributes to the relationship:

```php
use Filament\Actions\AttachAction;
use Filament\Forms;

AttachAction::make()
    ->schema(fn (AttachAction $action): array => [
        $action->getRecordSelect(),
        Forms\Components\TextInput::make('role')->required(),
    ])
```

In this example, `$action->getRecordSelect()` returns the select field to pick the record to attach. The `role` text input is then saved to the pivot table's `role` column.

Please ensure that any pivot attributes are listed in the `withPivot()` method of the relationship *and* inverse relationship.

### Scoping the options to attach

You may want to scope the options available to `AttachAction`:

```php
use Filament\Actions\AttachAction;
use Illuminate\Database\Eloquent\Builder;

AttachAction::make()
    ->recordSelectOptionsQuery(fn (Builder $query) => $query->whereBelongsTo(auth()->user()))
```

### Searching the options to attach across multiple columns

By default, the options available to `AttachAction` will be searched in the `recordTitleAttribute()` of the table. If you wish to search across multiple columns, you can use the `recordSelectSearchColumns()` method:

```php
use Filament\Actions\AttachAction;

AttachAction::make()
    ->recordSelectSearchColumns(['title', 'description'])
```

### Attaching multiple records

The `multiple()` method on the `AttachAction` component allows you to select multiple values:

```php
use Filament\Actions\AttachAction;

AttachAction::make()
    ->multiple()
```

### Customizing the select field in the attached modal

You may customize the select field object that is used during attachment by passing a function to the `recordSelect()` method:

```php
use Filament\Actions\AttachAction;
use Filament\Forms\Components\Select;

AttachAction::make()
    ->recordSelect(
        fn (Select $select) => $select->placeholder('Select a post'),
    )
```

### Handling duplicates

By default, you will not be allowed to attach a record more than once. This is because you must also set up a primary `id` column on the pivot table for this feature to work.

Please ensure that the `id` attribute is listed in the `withPivot()` method of the relationship *and* inverse relationship.

Finally, add the `allowDuplicates()` method to the table:

```php
public function table(Table $table): Table
{
    return $table
        ->allowDuplicates();
}
```

### Improving the performance of detach bulk actions

By default, the `DetachBulkAction` will load all Eloquent records into memory, before looping over them and detaching them one by one.

If you are detaching a large number of records, you may want to use the `chunkSelectedRecords()` method to fetch a smaller number of records at a time. This will reduce the memory usage of your application:

```php
use Filament\Actions\DetachBulkAction;

DetachBulkAction::make()
    ->chunkSelectedRecords(250)
```

Filament loads Eloquent records into memory before detaching them for two reasons:

- To allow individual records in the collection to be authorized with a model policy before detaching (using `authorizeIndividualRecords('delete')`, for example).
- To ensure that model events are run when detaching records, such as the `deleting` and `deleted` events in a model observer.

If you do not require individual record policy authorization and model events, you can use the `fetchSelectedRecords(false)` method, which will not fetch the records into memory before detaching them, and instead will detach them in a single query:

```php
use Filament\Actions\DetachBulkAction;

DetachBulkAction::make()
    ->fetchSelectedRecords(false)
```

## Associating and dissociating records

Filament is able to associate and dissociate records for `HasMany` and `MorphMany` relationships.

When generating your relation manager, you may pass the `--associate` flag to also add `AssociateAction`, `DissociateAction` and `DissociateBulkAction` to the table:

```bash
php artisan make:filament-relation-manager CategoryResource posts title --associate
```

Alternatively, if you've already generated your resource, you can just add the actions to the `$table` arrays:

```php
use Filament\Actions\AssociateAction;
use Filament\Actions\BulkActionGroup;
use Filament\Actions\DissociateAction;
use Filament\Actions\DissociateBulkAction;
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->columns([
            // ...
        ])
        ->headerActions([
            // ...
            AssociateAction::make(),
        ])
        ->recordActions([
            // ...
            DissociateAction::make(),
        ])
        ->toolbarActions([
            BulkActionGroup::make([
                // ...
                DissociateBulkAction::make(),
            ]),
        ]);
}
```

### Preloading the associate modal select options

By default, as you search for a record to associate, options will load from the database via AJAX. If you wish to preload these options when the form is first loaded instead, you can use the `preloadRecordSelect()` method of `AssociateAction`:

```php
use Filament\Actions\AssociateAction;

AssociateAction::make()
    ->preloadRecordSelect()
```

### Scoping the options to associate

You may want to scope the options available to `AssociateAction`:

```php
use Filament\Actions\AssociateAction;
use Illuminate\Database\Eloquent\Builder;

AssociateAction::make()
    ->recordSelectOptionsQuery(fn (Builder $query) => $query->whereBelongsTo(auth()->user()))
```

### Searching the options to associate across multiple columns

By default, the options available to `AssociateAction` will be searched in the `recordTitleAttribute()` of the table. If you wish to search across multiple columns, you can use the `recordSelectSearchColumns()` method:

```php
use Filament\Actions\AssociateAction;

AssociateAction::make()
    ->recordSelectSearchColumns(['title', 'description'])
```

### Associating multiple records

The `multiple()` method on the `AssociateAction` component allows you to select multiple values:

```php
use Filament\Actions\AssociateAction;

AssociateAction::make()
    ->multiple()
```

### Customizing the select field in the associate modal

You may customize the select field object that is used during association by passing a function to the `recordSelect()` method:

```php
use Filament\Actions\AssociateAction;
use Filament\Forms\Components\Select;

AssociateAction::make()
    ->recordSelect(
        fn (Select $select) => $select->placeholder('Select a post'),
    )
```

### Improving the performance of dissociate bulk actions

By default, the `DissociateBulkAction` will load all Eloquent records into memory, before looping over them and dissociating them one by one.

If you are dissociating a large number of records, you may want to use the `chunkSelectedRecords()` method to fetch a smaller number of records at a time. This will reduce the memory usage of your application:

```php
use Filament\Actions\DissociateBulkAction;

DissociateBulkAction::make()
    ->chunkSelectedRecords(250)
```

Filament loads Eloquent records into memory before dissociating them for two reasons:

- To allow individual records in the collection to be authorized with a model policy before dissociation (using `authorizeIndividualRecords('update')`, for example).
- To ensure that model events are run when dissociating records, such as the `updating` and `updated` events in a model observer.

If you do not require individual record policy authorization and model events, you can use the `fetchSelectedRecords(false)` method, which will not fetch the records into memory before dissociating them, and instead will dissociate them in a single query:

```php
use Filament\Actions\DissociateBulkAction;

DissociateBulkAction::make()
    ->fetchSelectedRecords(false)
```

## Viewing related records

When generating your relation manager, you may pass the `--view` flag to also add a `ViewAction` to the table:

```bash
php artisan make:filament-relation-manager CategoryResource posts title --view
```

Alternatively, if you've already generated your relation manager, you can just add the `ViewAction` to the `$table->recordActions()` array:

```php
use Filament\Actions\ViewAction;
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->columns([
            // ...
        ])
        ->recordActions([
            ViewAction::make(),
            // ...
        ]);
}
```

## Deleting related records

By default, you will not be able to interact with deleted records in the relation manager. If you'd like to add functionality to restore, force-delete and filter trashed records in your relation manager, use the `--soft-deletes` flag when generating the relation manager:

```bash
php artisan make:filament-relation-manager CategoryResource posts title --soft-deletes
```

Alternatively, you may add soft-deleting functionality to an existing relation manager:

```php
use Filament\Actions\DeleteAction;
use Filament\Actions\DeleteBulkAction;
use Filament\Actions\ForceDeleteAction;
use Filament\Actions\ForceDeleteBulkAction;
use Filament\Actions\RestoreAction;
use Filament\Actions\RestoreBulkAction;
use Filament\Tables\Filters\TrashedFilter;
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\SoftDeletingScope;

public function table(Table $table): Table
{
    return $table
        ->modifyQueryUsing(fn (Builder $query) => $query->withoutGlobalScopes([
            SoftDeletingScope::class,
        ]))
        ->columns([
            // ...
        ])
        ->filters([
            TrashedFilter::make(),
            // ...
        ])
        ->recordActions([
            DeleteAction::make(),
            ForceDeleteAction::make(),
            RestoreAction::make(),
            // ...
        ])
        ->toolbarActions([
            BulkActionGroup::make([
                DeleteBulkAction::make(),
                ForceDeleteBulkAction::make(),
                RestoreBulkAction::make(),
                // ...
            ]),
        ]);
}
```

### Customizing the `DeleteAction`

To learn how to customize the `DeleteAction`, including changing the notification and adding lifecycle hooks, please see the [Actions documentation](../actions/delete).

## Importing related records

The [`ImportAction`](../actions/import) can be added to the header of a relation manager to import records. In this case, you probably want to tell the importer which owner these new records belong to. You can use [import options](../actions/import#using-import-options) to pass through the ID of the owner record:

```php
ImportAction::make()
    ->importer(ProductImporter::class)
    ->options(['categoryId' => $this->getOwnerRecord()->getKey()])
```

Now, in the importer class, you can associate the owner in a one-to-many relationship with the imported record:

```php
public function resolveRecord(): ?Product
{
    $product = Product::firstOrNew([
        'sku' => $this->data['sku'],
    ]);
    
    $product->category()->associate($this->options['categoryId']);
    
    return $product;
}
```

Alternatively, you can attach the record in a many-to-many relationship using the `afterSave()` hook of the importer:

```php
protected function afterSave(): void
{
    $this->record->categories()->syncWithoutDetaching([$this->options['categoryId']]);
}
```

## Accessing the relationship's owner record

Relation managers are Livewire components. When they are first loaded, the owner record (the Eloquent record which serves as a parent - the main resource model) is saved into a property. You can read this property using:

```php
$this->getOwnerRecord()
```

However, if you're inside a `static` method like `form()` or `table()`, `$this` isn't accessible. So, you may [use a callback](../forms/overview#field-utility-injection) to access the `$livewire` instance:

```php
use Filament\Forms;
use Filament\Resources\RelationManagers\RelationManager;
use Filament\Schemas\Schema;

public function form(Schema $schema): Schema
{
    return $schema
        ->components([
            Forms\Components\Select::make('store_id')
                ->options(function (RelationManager $livewire): array {
                    return $livewire->getOwnerRecord()->stores()
                        ->pluck('name', 'id')
                        ->toArray();
                }),
            // ...
        ]);
}
```

All methods in Filament accept a callback which you can access `$livewire->ownerRecord` in.

## Grouping relation managers

You may choose to group relation managers together into one tab. To do this, you may wrap multiple managers in a `RelationGroup` object, with a label:

```php
use Filament\Resources\RelationManagers\RelationGroup;

public static function getRelations(): array
{
    return [
        // ...
        RelationGroup::make('Contacts', [
            RelationManagers\IndividualsRelationManager::class,
            RelationManagers\OrganizationsRelationManager::class,
        ]),
        // ...
    ];
}
```

## Conditionally showing relation managers

By default, relation managers will be visible if the `viewAny()` method for the related model policy returns `true`.

You may use the `canViewForRecord()` method to determine if the relation manager should be visible for a specific owner record and page:

```php
use Illuminate\Database\Eloquent\Model;

public static function canViewForRecord(Model $ownerRecord, string $pageClass): bool
{
    return $ownerRecord->status === Status::Draft;
}
```

## Combining the relation manager tabs with the form

On the Edit or View page class, override the `hasCombinedRelationManagerTabsWithContent()` method:

```php
public function hasCombinedRelationManagerTabsWithContent(): bool
{
    return true;
}
```

### Customizing the content tab

On the Edit or View page class, override the `getContentTabComponent()` method, and use any [Tab](../schemas/tabs) customization methods:

```php
use Filament\Schemas\Components\Tabs\Tab;

public function getContentTabComponent(): Tab
{
    return Tab::make('Settings')
        ->icon('heroicon-m-cog');
}
```

### Setting the position of the form tab

By default, the form tab is rendered before the relation tabs. To render it after, you can override the `getContentTabPosition()` method on the Edit or View page class:

```php
use Filament\Resources\Pages\Enums\ContentTabPosition;

public function getContentTabPosition(): ?ContentTabPosition
{
    return ContentTabPosition::After;
}
```

## Customizing relation manager tabs

To customize the tab for a relation manager, override the `getTabComponent()` method, and use any [Tab](../schemas/tabs) customization methods:

```php
use Filament\Schemas\Components\Tabs\Tab;
use Illuminate\Database\Eloquent\Model;

public static function getTabComponent(Model $ownerRecord, string $pageClass): Tab
{
    return Tab::make('Blog posts')
        ->badge($ownerRecord->posts()->count())
        ->badgeColor('info')
        ->badgeTooltip('The number of posts in this category')
        ->icon('heroicon-m-document-text');
}
```

If you are using a [relation group](#grouping-relation-managers), you can use the `tab()` method:

```php
use Filament\Resources\RelationManagers\RelationGroup;
use Filament\Schemas\Components\Tabs\Tab;
use Illuminate\Database\Eloquent\Model;

RelationGroup::make('Contacts', [
    // ...
])
    ->tab(fn (Model $ownerRecord): Tab => Tab::make('Blog posts')
        ->badge($ownerRecord->posts()->count())
        ->badgeColor('info')
        ->badgeTooltip('The number of posts in this category')
        ->icon('heroicon-m-document-text'));
```

## Sharing a resource's form and table with a relation manager

You may decide that you want a resource's form and table to be identical to a relation manager's, and subsequently want to reuse the code you previously wrote. This is easy, by calling the `form()` and `table()` methods of the resource from the relation manager:

```php
use App\Filament\Resources\Blog\Posts\PostResource;
use Filament\Schemas\Schema;
use Filament\Tables\Table;

public function form(Schema $schema): Schema
{
    return PostResource::form($schema);
}

public function table(Table $table): Table
{
    return PostResource::table($table);
}
```

### Hiding a shared form component on the relation manager

If you're sharing a form component from the resource with the relation manager, you may want to hide it on the relation manager. This is especially useful if you want to hide a `Select` field for the owner record in the relation manager, since Filament will handle this for you anyway. To do this, you may use the `hiddenOn()` method, passing the name of the relation manager:

```php
use App\Filament\Resources\Blog\Posts\PostResource\RelationManagers\CommentsRelationManager;
use Filament\Forms\Components\Select;

Select::make('post_id')
    ->relationship('post', 'title')
    ->hiddenOn(CommentsRelationManager::class)
```

### Hiding a shared table column on the relation manager

If you're sharing a table column from the resource with the relation manager, you may want to hide it on the relation manager. This is especially useful if you want to hide a column for the owner record in the relation manager, since this is not appropriate when the owner record is already listed above the relation manager. To do this, you may use the `hiddenOn()` method, passing the name of the relation manager:

```php
use App\Filament\Resources\Blog\Posts\PostResource\RelationManagers\CommentsRelationManager;
use Filament\Tables\Columns\TextColumn;

TextColumn::make('post.title')
    ->hiddenOn(CommentsRelationManager::class)
```

### Hiding a shared table filter on the relation manager

If you're sharing a table filter from the resource with the relation manager, you may want to hide it on the relation manager. This is especially useful if you want to hide a filter for the owner record in the relation manager, since this is not appropriate when the table is already filtered by the owner record. To do this, you may use the `hiddenOn()` method, passing the name of the relation manager:

```php
use App\Filament\Resources\Blog\Posts\PostResource\RelationManagers\CommentsRelationManager;
use Filament\Tables\Filters\SelectFilter;

SelectFilter::make('post')
    ->relationship('post', 'title')
    ->hiddenOn(CommentsRelationManager::class)
```

### Overriding shared configuration on the relation manager

Any configuration that you make inside the resource can be overwritten on the relation manager. For example, if you wanted to disable pagination on the relation manager's inherited table but not the resource itself:

```php
use App\Filament\Resources\Blog\Posts\PostResource;
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return PostResource::table($table)
        ->paginated(false);
}
```

It is probably also useful to provide extra configuration on the relation manager if you wanted to add a header action to [create](#creating-related-records), [attach](#attaching-and-detaching-records), or [associate](#associating-and-dissociating-records) records in the relation manager:

```php
use App\Filament\Resources\Blog\Posts\PostResource;
use Filament\Actions\AttachAction;
use Filament\Actions\CreateAction;
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return PostResource::table($table)
        ->headerActions([
            CreateAction::make(),
            AttachAction::make(),
        ]);
}
```

## Customizing the relation manager Eloquent query

You can apply your own query constraints or [model scopes](https://laravel.com/docs/eloquent#query-scopes) that affect the entire relation manager. To do this, you can pass a function to the `modifyQueryUsing()` method of the table, inside which you can customize the query:

```php
use Filament\Tables;
use Illuminate\Database\Eloquent\Builder;

public function table(Table $table): Table
{
    return $table
        ->modifyQueryUsing(fn (Builder $query) => $query->where('is_active', true))
        ->columns([
            // ...
        ]);
}
```

## Customizing the relation manager title

To set the title of the relation manager, you can use the `$title` property on the relation manager class:

```php
protected static ?string $title = 'Posts';
```

To set the title of the relation manager dynamically, you can override the `getTitle()` method on the relation manager class:

```php
use Illuminate\Database\Eloquent\Model;

public static function getTitle(Model $ownerRecord, string $pageClass): string
{
    return __('relation-managers.posts.title');
}
```

The title will be reflected in the [heading of the table](../tables/overview#customizing-the-table-header), as well as the relation manager tab if there is more than one. If you want to customize the table heading independently, you can still use the `$table->heading()` method:

```php
use Filament\Tables;

public function table(Table $table): Table
{
    return $table
        ->heading('Posts')
        ->columns([
            // ...
        ]);
}
```

## Customizing the relation manager record title

The relation manager uses the concept of a "record title attribute" to determine which attribute of the related model should be used to identify it. When creating a relation manager, this attribute is passed as the third argument to the `make:filament-relation-manager` command:

```bash
php artisan make:filament-relation-manager CategoryResource posts title
```

In this example, the `title` attribute of the `Post` model will be used to identify a post in the relation manager.

This is mainly used by the action classes. For instance, when you [attach](#attaching-and-detaching-records) or [associate](#associating-and-dissociating-records) a record, the titles will be listed in the select field. When you [edit](#editing-related-records), [view](#viewing-related-records) or [delete](#deleting-related-records) a record, the title will be used in the header of the modal.

In some cases, you may want to concatenate multiple attributes together to form a title. You can do this by replacing the `recordTitleAttribute()` configuration method with `recordTitle()`, passing a function that transforms a model into a title:

```php
use App\Models\Post;
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->recordTitle(fn (Post $record): string => "{$record->title} ({$record->id})")
        ->columns([
            // ...
        ]);
}
```

If you're using `recordTitle()`, and you have an [associate action](#associating-and-dissociating-records) or [attach action](#attaching-and-detaching-records), you will also want to specify search columns for those actions:

```php
use Filament\Actions\AssociateAction;
use Filament\Actions\AttachAction;

AssociateAction::make()
    ->recordSelectSearchColumns(['title', 'id']);

AttachAction::make()
    ->recordSelectSearchColumns(['title', 'id'])
```

## Relation pages

Using a `ManageRelatedRecords` page is an alternative to using a relation manager, if you want to keep the functionality of managing a relationship separate from editing or viewing the owner record.

This feature is ideal if you are using [resource sub-navigation](overview#resource-sub-navigation), as you are easily able to switch between the View or Edit page and the relation page.

To create a relation page, you should use the `make:filament-page` command:

```bash
php artisan make:filament-page ManageCustomerAddresses --resource=CustomerResource --type=ManageRelatedRecords
```

When you run this command, you will be asked a series of questions to customize the page, for example, the name of the relationship and its title attribute.

You must register this new page in your resource's `getPages()` method:

```php
public static function getPages(): array
{
    return [
        'index' => Pages\ListCustomers::route('/'),
        'create' => Pages\CreateCustomer::route('/create'),
        'view' => Pages\ViewCustomer::route('/{record}'),
        'edit' => Pages\EditCustomer::route('/{record}/edit'),
        'addresses' => Pages\ManageCustomerAddresses::route('/{record}/addresses'),
    ];
}
```

<Aside variant="warning">
    When using a relation page, you do not need to generate a relation manager with `make:filament-relation-manager`, and you do not need to register it in the `getRelations()` method of the resource.
</Aside>

Now, you can customize the page in exactly the same way as a relation manager, with the same `table()` and `form()`.

### Adding relation pages to resource sub-navigation

If you're using [resource sub-navigation](overview#resource-sub-navigation), you can register this page as normal in `getRecordSubNavigation()` of the resource:

```php
use App\Filament\Resources\Customers\Pages;
use Filament\Resources\Pages\Page;

public static function getRecordSubNavigation(Page $page): array
{
    return $page->generateNavigationItems([
        // ...
        Pages\ManageCustomerAddresses::class,
    ]);
}
```

## Passing properties to relation managers

When registering a relation manager in a resource, you can use the `make()` method to pass an array of [Livewire properties](https://livewire.laravel.com/docs/properties) to it:

```php
use App\Filament\Resources\Blog\Posts\PostResource\RelationManagers\CommentsRelationManager;

public static function getRelations(): array
{
    return [
        CommentsRelationManager::make([
            'status' => 'approved',
        ]),
    ];
}
```

This array of properties gets mapped to [public Livewire properties](https://livewire.laravel.com/docs/properties) on the relation manager class:

```php
use Filament\Resources\RelationManagers\RelationManager;

class CommentsRelationManager extends RelationManager
{
    public string $status;

    // ...
}
```

Now, you can access the `status` in the relation manager class using `$this->status`.

## Disabling lazy loading

By default, relation managers are lazy-loaded. This means that they will only be loaded when they are visible on the page.

To disable this behavior, you may override the `$isLazy` property on the relation manager class:

```php
protected static bool $isLazy = false;
```
---
title: Nested resources
---

## Overview

[Relation managers](managing-relationships#creating-a-relation-manager) and [relation pages](managing-relationships#relation-pages) provide you with an easy way to render a table of related records inside a resource.

For example, in a `CourseResource`, you may have a relation manager or page for `lessons` that belong to that course. You can create and edit lessons from the table, which opens modal dialogs.

However, lessons may be too complex to be created and edited in a modal. You may wish that lessons had their own resource, so that creating and editing them would be a full page experience. This is a nested resource.

## Creating a nested resource

To create a nested resource, you can use the `make:filament-resource` command with the `--nested` option:

```bash
php artisan make:filament-resource Lesson --nested
```

To access the nested resource, you will also need a [relation manager](managing-relationships#creating-a-relation-manager) or [relation page](managing-relationships#relation-pages). This is where the user can see the list of related records, and click links to the "create" and "edit" pages.

To create a relation manager or page, you can use the `make:filament-relation-manager` or `make:filament-page` command:

```bash
php artisan make:filament-relation-manager CourseResource lessons title

php artisan make:filament-page ManageCourseLessons --resource=CourseResource --type=ManageRelatedRecords
```

When creating a relation manager or page, Filament will ask if you want each table row to link to a resource instead of opening a modal, to which you should answer "yes" and select the nested resource that you just created.

After generating the relation manager or page, it will have a property pointing to the nested resource:

```php
use App\Filament\Resources\Courses\Resources\Lessons\LessonResource;

protected static ?string $relatedResource = LessonResource::class;
```

The nested resource class will have a property pointing to the parent resource:

```php
use App\Filament\Resources\Courses\CourseResource;

protected static ?string $parentResource = CourseResource::class;
```

## Customizing the relationship names

In the same way that relation managers and pages predict the name of relationships based on the models in those relationships, nested resources do the same. Sometimes, you may have a relationship that does not fit the traditional relationship naming convention, and you will need to inform Filament of the correct relationship names for the nested resource.

To customize the relationship names, first remove the `$parentResource` property from the nested resource class. Then define a `getParentResourceRegistration()` method:

```php
use App\Filament\Resources\Courses\CourseResource;
use Filament\Resources\ParentResourceRegistration;

public static function getParentResourceRegistration(): ?ParentResourceRegistration
{
    return CourseResource::asParent()
        ->relationship('lessons')
        ->inverseRelationship('course');
}
```

You can omit the calls to `relationship()` and `inverseRelationship()` if you want to use the default names.

## Registering a relation manager with the correct URL

When dealing with a nested resource that is listed by a relation manager, and the relation manager is amongst others on that page, you may notice that the URL to it is not correct when you redirect from the nested resource back to it. This is because each relation manager registered on a resource is assigned an integer, which is used to identify it in the URL when switching between multiple relation managers. For example, `?relation=0` might represent one relation manager in the URL, and `?relation=1` might represent another.

When redirecting from a nested resource back to a relation manager, Filament will assume that the relationship name is used to identify that relation manager in the URL. For example, if you have a nested `LessonResource` and a `LessonsRelationManager`, the relationship name is `lessons`, and should be used as the [URL parameter key](managing-relationships#customizing-the-relation-managers-url-parameter) for that relation manager when it is registered:

```php
public static function getRelations(): array
{
    return [
        'lessons' => LessonsRelationManager::class,
    ];
}
```
---
title: Singular resources
---

## Overview

Resources aren't the only way to interact with Eloquent records in a Filament panel. Even though resources may solve many of your requirements, the "index" (root) page of a resource contains a table with a list of records in that resource.

Sometimes there is no need for a table that lists records in a resource. There is only a single record that the user interacts with. If it doesn't yet exist when the user visits the page, it gets created when the form is first submitted by the user to save it. If the record already exists, it is loaded into the form when the page is first loaded, and updated when the form is submitted.

For example, a CMS might have a `Page` Eloquent model and a `PageResource`, but you may also want to create a singular page outside the `PageResource` for editing the "homepage" of the website. This allows the user to directly edit the homepage without having to navigate to the `PageResource` and find the homepage record in the table.

Other examples of this include a "Settings" page, or a "Profile" page for the currently logged-in user. For these use cases, though, we recommend that you use the [Spatie Settings plugin](/plugins/filament-spatie-settings) and the [Profile](../users#authentication-features) features of Filament, which require less code to implement.

## Creating a singular resource

Although there is no specific "singular resource" feature in Filament, it is a highly-requested behavior and can be implemented quite simply using a [custom page](../navigation/custom-pages) with a [form](../forms). This guide will explain how to do this.

Firstly, create a [custom page](../navigation/custom-pages):

```bash
php artisan make:filament-page ManageHomepage
```

This command will create two files - a page class in the `/Filament/Pages` directory of your resource directory, and a Blade view in the `/filament/pages` directory of the resource views directory.

The page class should contain the following elements:
- A `$data` property, which will hold the current state of the form.
- A `mount()` method, which will load the current record from the database and fill the form with its data. If the record doesn't exist, `null` will be passed to the `fill()` method of the form, which will assign any default values to the form fields.
- A `form()` method, which will define the form schema. The form contains fields in the `components()` method. The `record()` method should be used to specify the record that the form should load relationship data from. The `statePath()` method should be used to specify the name of the property (`$data`) where the form's state should be stored.
- A `save()` method, which will save the form data to the database. The `getState()` method runs form validation and returns valid form data. This method should check if the record already exists, and if not, create a new one. The `wasRecentlyCreated` property of the model can be used to determine if the record was just created, and if so then any relationships should be saved as well. A notification is sent to the user to confirm that the record was saved.
- A `getRecord()` method, while not strictly necessary, is a good idea to have. This method will return the Eloquent record that the form is editing. It can be used across the other methods to avoid code duplication.

```php
namespace App\Filament\Pages;

use App\Models\WebsitePage;
use Filament\Forms\Components\RichEditor;
use Filament\Forms\Components\TextInput;
use Filament\Notifications\Notification;
use Filament\Pages\Page;
use Filament\Schemas\Schema;

/**
 * @property-read Schema $form
 */
class ManageHomepage extends Page
{
    /**
     * @var array<string, mixed> | null
     */
    public ?array $data = [];

    public function mount(): void
    {
        $this->form->fill($this->getRecord()?->attributesToArray());
    }

    public function form(Schema $schema): Schema
    {
        return $schema
            ->components([
                TextInput::make('title')
                    ->required()
                    ->maxLength(255),
                RichEditor::make('content'),
                // ...
            ])
            ->record($this->getRecord())
            ->statePath('data');
    }

    public function save(): void
    {
        $data = $this->form->getState();
        
        $record = $this->getRecord();
        
        if (! $record) {
            $record = new WebsitePage();
            $record->is_homepage = true;
        }
        
        $record->fill($data);
        $record->save();
        
        if ($record->wasRecentlyCreated) {
            $this->form->record($record)->saveRelationships();
        }

        Notification::make()
            ->success()
            ->title('Saved')
            ->send();
    }
    
    public function getRecord(): ?WebsitePage
    {
        return WebsitePage::query()
            ->where('is_homepage', true)
            ->first();
    }
}
```

The page Blade view should render the form and a Save button. The `wire:submit` directive should be used to call the `save()` method when the form is submitted:

```blade
<x-filament::page>
    <form wire:submit="save">
        {{ $this->form }}
    
        <div>
            <x-filament::button type="submit">
                Save
            </x-filament::button>
        </div>
    </form>
</x-filament::page>
```