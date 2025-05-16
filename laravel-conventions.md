# Laravel - TALL Stack + FilamentPHP Conventions

This document outlines the conventions, best practices, and workflows for developing Laravel applications using the TALL stack (Tailwind CSS, Alpine.js, Laravel, Livewire) and FilamentPHP. These guidelines are designed to help AI-assisted development and ensure consistent code quality across the project.

## Table of Contents

1. [Introduction](#introduction)
2. [Development Environment](#development-environment)
3. [File Structure](#file-structure)
4. [Tailwind CSS Conventions](#tailwind-css-conventions)
5. [Alpine.js Conventions](#alpinejs-conventions)
6. [Laravel Conventions](#laravel-conventions)
7. [Livewire Conventions](#livewire-conventions)
8. [FilamentPHP Conventions](#filamentphp-conventions)
9. [Common Artisan Commands](#common-artisan-commands)
10. [Testing](#testing)
11. [AI-Assisted Development Guidelines](#ai-assisted-development-guidelines)

## Introduction

The TALL stack is a set of frameworks to build interactive apps using Laravel. It stands for:

- **T**: Tailwind CSS - A utility-first CSS framework
- **A**: Alpine.js - A lightweight JavaScript framework for adding interactivity
- **L**: Laravel - PHP framework for backend functionality
- **L**: Livewire - A full-stack framework for dynamic interfaces without writing JavaScript

Additionally, we use **FilamentPHP** for rapid admin panel and CRUD interface development.

This combination provides a powerful, cohesive development experience that enables building modern web applications with minimal context switching between technologies.

## Development Environment

We use Laravel Sail, a light-weight command-line interface for Laravel's Docker development environment.

### Starting and Stopping the Environment

```bash
# Start the development environment
./vendor/bin/sail up

# Start in detached mode
./vendor/bin/sail up -d

# Stop the environment
./vendor/bin/sail down
```


### Executing Commands

Always prefix commands with `sail` to execute them within the Docker container:

```bash
# Run composer commands
./vendor/bin/sail composer require laravel/sanctum

# Run artisan commands
./vendor/bin/sail artisan migrate

# Run npm commands
./vendor/bin/sail npm run dev
```


## File Structure

Maintain Laravel's standard file structure with these additional conventions:

```
app/
├── Http/
│   ├── Livewire/         # Livewire components
│   └── Controllers/      # Standard controllers
├── Filament/
│   ├── Resources/        # Filament resources
│   ├── Pages/            # Custom Filament pages
│   └── Widgets/          # Filament dashboard widgets
├── Models/               # Eloquent models
├── Providers/            # Service providers
resources/
├── views/
│   ├── livewire/        # Livewire component views
│   ├── components/      # Blade components
│   ├── layouts/         # Layout templates
├── css/                 # CSS/SCSS files
├── js/                  # JavaScript files
```


## Tailwind CSS Conventions

### Utility Class Order

Use a consistent order for utility classes based on the "Concentric CSS" approach:

1. Positioning/visibility (`relative`, `absolute`, `hidden`, etc.)
2. Box model (`w-`, `h-`, `m-`, `p-`, etc.)
3. Borders (`border-`, `rounded-`, etc.)
4. Backgrounds (`bg-`)
5. Typography (`text-`, `font-`, etc.)
6. Other visual adjustments (`shadow-`, `opacity-`, etc.)

### Best Practices

1. Use the most concise utility class combinations possible. For example, use `mx-2` instead of `ml-2 mr-2`.
2. Always prefix utilities that apply only at certain breakpoints:

```html
<div class="block lg:flex lg:flex-col lg:justify-center">...</div>
```

3. For custom components, use the `@apply` directive in a CSS file instead of repeating long utility chains.
4. Customize Tailwind through the configuration file rather than writing custom CSS.
5. Use arbitrary values sparingly (`w-[327px]`) and prefer the design system tokens.
6. Consider extracting common patterns to Blade components for reuse.

## Alpine.js Conventions

### Component Structure

1. Use the `x-data` attribute to define component state:

```html
<div x-data="{ open: false }">
  <!-- Component content -->
</div>
```

2. Keep Alpine.js components small and focused on specific interactions.

### Event Handling

1. For component communication, use the "window as event bus" pattern:

```html
<!-- Trigger event -->
<button @click="$dispatch('user-updated', { id: 1 })">Update</button>

<!-- Listen for event -->
<div x-on:user-updated.window="handleUpdate($event.detail)">...</div>
```

2. Use namespaced events to avoid collisions:

```html
<button @click="$dispatch('app:modal-opened')">Open Modal</button>
```


### Best Practices

1. Use `.away` modifier for closing dropdowns/modals when clicking outside.
2. Separate complex logic into dedicated JavaScript functions.
3. Avoid nesting Alpine components more than one level deep.
4. Use Alpine for simple interactive behaviors; for complex state management, use Livewire.

## Laravel Conventions

### Models

1. Always create models using Artisan:

```bash
./vendor/bin/sail artisan make:model Product --migration --factory --seed
```

2. Define relationships, casts, and fillable properties explicitly:

```php
protected $fillable = ['name', 'price', 'description'];
protected $casts = ['price' => 'float', 'published_at' => 'datetime'];
```


### Migrations

1. Create migrations with descriptive names:

```bash
./vendor/bin/sail artisan make:migration create_products_table
./vendor/bin/sail artisan make:migration add_category_id_to_products_table
```

2. Use method chaining for column definitions:

```php
$table->string('name')->index();
$table->decimal('price', 8, 2)->nullable();
```


### Routes

1. Group related routes by domain or functionality.
2. Use resource controllers for CRUD operations.
3. Name all routes for easier reference in Livewire and Blade.

## Livewire Conventions

### Creating Components

Always use Artisan commands to create Livewire components:

```bash
# Basic component
./vendor/bin/sail artisan make:livewire ProductList

# Component in a subdirectory
./vendor/bin/sail artisan make:livewire Admin/Dashboard

# Inline component (single file)
./vendor/bin/sail artisan make:livewire Counter --inline
```


### Component Structure

1. Place public properties at the top of the class.
2. Group lifecycle hooks together (`mount`, `hydrate`, etc.).
3. Place computed properties and actions below.

Example:

```php
class ProductForm extends Component
{
    // Public properties
    public $product;
    public $name;
    public $price;
    
    // Validation rules
    protected $rules = [
        'name' => 'required|min:3',
        'price' => 'required|numeric',
    ];
    
    // Lifecycle hooks
    public function mount(Product $product = null)
    {
        $this->product = $product ?? new Product();
        $this->name = $product->name ?? '';
        $this->price = $product->price ?? '';
    }
    
    // Actions
    public function save()
    {
        $this->validate();
        
        $this->product->update([
            'name' => $this->name,
            'price' => $this->price,
        ]);
        
        $this->dispatch('product-saved');
    }
}
```


### Best Practices

1. Don't pass large objects to Livewire components; use primitive types instead.
2. Keep component nesting level at 1 to avoid performance issues.
3. Always set up the root element in component templates:

```html
<div>
    <!-- Component content here -->
</div>
```

4. Use properties for simple state and computed properties for derived values.
5. Define validation rules using the `$rules` property.
6. Use `dispatch()` and `on()` for component communication instead of deeply nested components.

## FilamentPHP Conventions

### Creating Resources

Use Artisan to generate Filament resources:

```bash
# Basic resource
./vendor/bin/sail artisan make:filament-resource Product

# With soft deletes
./vendor/bin/sail artisan make:filament-resource Customer --soft-deletes

# Auto-generate forms and tables based on the model
./vendor/bin/sail artisan make:filament-resource Product --generate

# Add a View page
./vendor/bin/sail artisan make:filament-resource Product --view
```


### Resource Structure

1. Define the form schema in the `form()` method:

```php
public static function form(Form $form): Form
{
    return $form->schema([
        Forms\Components\TextInput::make('name')->required(),
        Forms\Components\TextInput::make('price')
            ->numeric()
            ->required(),
    ]);
}
```

2. Define the table structure in the `table()` method:

```php
public static function table(Table $table): Table
{
    return $table
        ->columns([
            Tables\Columns\TextColumn::make('name'),
            Tables\Columns\TextColumn::make('price')
                ->money('usd'),
        ])
        ->filters([
            Tables\Filters\SelectFilter::make('category_id')
                ->relationship('category', 'name'),
        ]);
}
```


### Best Practices

1. Customize the Eloquent query using `getEloquentQuery()` for resource-wide filters.
2. Use `$recordTitleAttribute` to identify records in searches and relations.
3. Leverage form component visibility methods (`hiddenOn()`, `visibleOn()`) to customize forms based on the context.
4. Create relationship managers to handle related models within a resource.
5. Set custom slugs with `protected static ?string $slug = 'custom-slug';` for better URLs.

## Common Artisan Commands

All commands should be prefixed with `./vendor/bin/sail artisan`:

### Laravel Core Commands

```bash
# Create a controller
make:controller ProductController --resource

# Create a model with associated files
make:model Product --migration --factory --controller --resource

# Create a migration
make:migration create_products_table

# Run migrations
migrate

# Rollback last migration
migrate:rollback

# Create a seeder
make:seeder ProductSeeder

# Run seeders
db:seed

# Create a policy
make:policy ProductPolicy --model=Product

# Create a middleware
make:middleware CheckSubscription
```


### Livewire Commands

```bash
# Create a Livewire component
make:livewire ProductList

# Create a Livewire component in a subdirectory
make:livewire Admin/Dashboard

# Create an inline Livewire component
make:livewire Counter --inline
```


### Filament Commands

```bash
# Create a resource
make:filament-resource Product

# Create a resource with automatic generation
make:filament-resource Product --generate

# Create a resource with soft deletes
make:filament-resource Product --soft-deletes

# Create a page
make:filament-page Settings

# Create a widget
make:filament-widget StatsOverview
```


## Testing

### Testing Commands

```bash
# Run all tests
./vendor/bin/sail artisan test

# Run a specific test file
./vendor/bin/sail artisan test --filter ProductTest

# Run tests with coverage report
./vendor/bin/sail artisan test --coverage
```


### Testing Livewire Components

Use Livewire's testing helpers for testing components:

```php
use App\Http\Livewire\ProductForm;
use App\Models\Product;
use Livewire\Livewire;

public function test_can_create_product()
{
    Livewire::test(ProductForm::class)
        ->set('name', 'New Product')
        ->set('price', 99.99)
        ->call('save')
        ->assertHasNoErrors();
        
    $this->assertDatabaseHas('products', [
        'name' => 'New Product',
        'price' => 99.99,
    ]);
}
```


## AI-Assisted Development Guidelines

When using AI tools for development in the TALL stack:

1. **Component Generation**: When asking AI to generate code, always specify which part of the TALL stack you're working with.
2. **Tailwind Styling**: Request AI to follow the utility class ordering convention when generating HTML with Tailwind classes.
3. **Livewire Components**: Always ask AI to structure code for creating Livewire components using Artisan commands first, then edit the files.
4. **Alpine Logic**: Request simple Alpine.js logic inside HTML elements for interactivity, and when more complex, suggest moving to Livewire components.
5. **FilamentPHP Resources**: When working with admin interfaces, prioritize using FilamentPHP resources over custom implementations.
6. **Context Awareness**: Provide the AI with context about the specific TALL stack version you're using, as APIs may differ.
7. **Command Execution**: Always remind AI to use `./vendor/bin/sail` prefix for any CLI commands to ensure they run in the proper Docker container.
8. **Template Structure**: When generating Blade templates, remind AI to follow the project's layout structure and component organization.
9. **Performance Considerations**: Ask AI to consider performance implications, especially with Livewire (avoiding large objects in properties) and Tailwind (minimizing class duplication).
10. **End-to-End Solutions**: For complex features, request the AI to provide all necessary parts - models, migrations, Livewire components, Blade templates, etc. - while respecting the conventions of each technology.

By following these guidelines, AI assistance can significantly enhance productivity while maintaining code quality and consistency in your TALL stack application.

[^1]: https://dev.to/jeroenvanrensen/the-tall-stack-explained-4oif

[^2]: https://github.com/michael-rubel/livewire-best-practices

[^3]: https://codewithhugo.com/alpinejs-component-communication-event-bus/

[^4]: https://www.uxpin.com/studio/blog/tailwind-best-practices/

[^5]: https://kombai.com/tailwind/order/

[^6]: https://pagedone.io/blocks/marketing/team

[^7]: https://filamentphp.com/docs/1.x/admin/resources/

[^8]: https://laravel.com/docs/12.x/sail

[^9]: https://app.studyraid.com/en/read/11454/358971/creating-livewire-components-using-artisan

[^10]: https://bluehouse.group/blog/the-tall-stack-experience-embracing-order-and-creativity/

[^11]: https://www.reddit.com/r/tailwindcss/comments/1evdkx8/best_practice_using_tailwind/

[^12]: https://filamentphp.com/docs/3.x/panels/resources/getting-started/

[^13]: https://www.curotec.com/insights/comparing-the-features-of-tall-and-vilt-stacks/

[^14]: https://tailwindcss.com/docs/styling-with-utility-classes

[^15]: https://github.com/livewire/awesome-tall-stack/blob/main/README.md

[^16]: https://benjamincrozat.com/tailwind-css

[^17]: https://digitalcode.sa/en/blog/unleashing-the-power-of-tall-stack-55091

[^18]: https://gist.github.com/sandren/0f22e116f01611beab2b1195ab731b63

[^19]: https://tallstackui.com/docs/form/date

[^20]: https://tailgrids.com/blog/tailwind-css-best-practices-and-performance-optimization

[^21]: https://www.youtube.com/watch?v=-3s-Yz14B0Q

[^22]: https://aggregata.de/en/blog/alpinejs/alpine-in-a-production-environment/

[^23]: https://github.com/tailwindlabs/prettier-plugin-tailwindcss

[^24]: https://www.material-tailwind.com/blocks/team-sections

[^25]: https://github.com/filamentphp/filament/discussions/11231

[^26]: https://filamentphp.com/docs/3.x/actions/installation/

[^27]: https://cylab.be/blog/339/get-started-with-laravel-sail

[^28]: https://laravel-livewire.com/docs/2.x/making-components

[^29]: https://github.com/filamentphp/filament/discussions/3509

[^30]: https://dev.to/biostate/how-filamenphp-naming-resource-and-how-to-get-url-2n0

[^31]: https://www.youtube.com/watch?v=9GBXqWKzfIM

[^32]: https://filamentphp.com/plugins/solution-forest-scaffold

[^33]: https://docs.laravel-filament.cn/docs/2.x/admin/resources/relation-managers

[^34]: https://filamentphp.com/docs/3.x/forms/installation/

[^35]: https://hackernoon.com/a-complete-guide-to-laravel-sail

[^36]: https://laracasts.com/discuss/channels/laravel/docker-sail-up-artisan-serve

[^37]: https://www.phparch.com/2024/02/getting-up-and-running-with-laravel-sail/

[^38]: https://laravel.com/docs/12.x/artisan

[^39]: https://kirschbaumdevelopment.com/insights/advanced-guide-to-laravel-sail

[^40]: https://laravel-livewire.com/docs/2.x/artisan-commands

