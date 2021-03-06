# Laraman - Laravel Data Manager

Laraman is a Laravel based administration panel.

Laraman provides a quick user interface for reviewing and managing data stored in your database.

Laraman is really good at the index route, searching, filtering and pagination.  It leaves the create, update and delete to the application.

## Installation

### Composer

Require this package with composer:

```
composer require christhompsontldr/laraman
```

### Service Provider

After updating composer, add the ServiceProvider to the providers array in config/app.php

#### Laravel 5.x:

```
Christhompsontldr\Laraman\ServiceProvider::class,
```

### Config

Copy the `config/laraman.php` file from the packge to your applications config directory.

## Routes

Laraman utilizes the `resource` method in routes to build all the required routes.

In `routes/web.php` add"
```
Laraman::resource('users');
```

Laraman will now look for a `app/Http/Controllers/Manage/UserController.php`.

The namespace of the Laraman controllers can be changed in the `config/laraman.php` file.  `Manage` is the default namespace.

## Models

Include the Laraman trait on your model

```
use Christhompsontldr\Laraman\Traits\LaramanModel;
```

and then use it

```
use LaramanModel;
```

Laraman utilizes something we call `formatters`.  We have included a few default formatters, but you are welcome to write your own.  Review the `Christhompsontldr\Laraman\Traits\LaramanModel` class for examples.

Think of these as post-accessors.  This allows Laraman to manipulate model data after the application's accessors have been applied.

Example of using the date formatter

```
public function __configure()
{
    $this->columns = [
        [
            'field' => 'created_at',
            'display' => 'Created',
            'formatter' => 'datetime',
            'options'   => [
                'format' => 'F j, Y g:ia',
            ]
        ],
```

## Controllers

Include the Laraman trait on your controller

```
use Christhompsontldr\Laraman\Traits\LaramanController;
```

and then use it

```
use LaramanController;
```

Laraman expects your controller to have a `__configure()` method where a few things are configured.

```
public function __configure()
{
    $this->columns = [
        [
            'field' => 'id',
        ],
        [
            'field' => 'name',
        ],
        [
            'field' => 'email',
        ],
        [
            'field'   => 'organization.name',
            'display' => 'Organization',
        ],
    ];

    $this->buttons = [
        config('laraman.view.hintpath') . '::buttons.view',
    ];
}
```

This example will build an index route with a table with 4 columns and 1 button.

## Options

### Model

If the model name you want to use doesn't make the naming convention you used for your controller, it can be set with the model attribute

```
    public function __configure()
    {
        $this->model = \App\Mail::class;
```

### Views

Need to load views from another path, use the `viewPath` attribute

```
    public function __configure()
    {
        $this->viewPath = config('laraman.view.hintpath') . '::mail';
```

### Route

The route where laraman lives for this controller can be changed

```
    public function __configure()
    {
        $this->routePath = config('laraman.route.prefix') . '.mail';
```

### Search

You can enable model level searches with the `searchEnabled` attribute

```
public function __configure()
{
    $this->searchEnabled = true;
```

Your model will need to have implemented a `search()` method.  This is commonly found in the Laravel Scout library or the Algolia Search for Laravel library.

### Columns

The only required array key for a column is the `field`.  This will be the database column name you want to display.

#### display

`display` will change the name displayed to the user in the top of the table.

#### related model data

The dot notation can be used to reach related model data.

```
public function __configure()
{
    $this->columns = [
        [
            'field' => 'id',
        ],
        [
            'field' => 'name',
        ],
        [
            'field' => 'email',
        ],
        [
            'field'   => 'organization.name',
            'display' => 'Organization',
        ],
    ];
```

`organization.name` will load the name from the related organization.

#### blade

If you need to use a custom blade for a field, define it like this

```
public function __configure()
{
    $this->columns = [
        [
            'field' => 'braintree_customer_id',
            'display' => 'Braintree Customer',
            'options' => [
                'blade' => config('laraman.view.hintpath') . '::fields.memberships.customer'
            ]
        ],
```

### Filters

Laraman can utilize filters defined on the model

```
public function __configure()
{
    $this->filters = [
        [
            'field' => 'event',
            'display' => 'Event',
            'type' => 'select',
            'values' => [
                'send'  => 'send',
                'hard_bounce' => 'hard bounce',
                'open'   => 'open',
                'soft_bounce' => 'soft bounce',
                'deferral' => 'clickdeferral',
                'delivered' => 'delivered',
                'reject' => 'reject',
                'spam' => 'spam',
            ]
        ],
    ];
```

If the model has a `filterEvent` defined, it will be utilized

```

    public function filterEvent($builder, $val)
    {
        return $builder->{$val}();
    }
```

Could be used to apply model scopes like `scopeSend()` and `scopeOpen()`.

### Buttons

Action buttons can be added with the `buttons` attribute

```
public function __configure()
{
    $this->buttons = [
        'laraman::buttons.braintree-transaction',
        'laraman::buttons.receipt',
    ];
```

### Scopes

If you need to scope the model being used, define a `scope` method in your controller

```
class TrialController extends Controller
{
    use LaramanController;

    public function scope($builder)
    {
        //  only show trials
        return $builder->trial();
    }
```

### Extras

Have extra data to pass from the controller to the view, use `extras`

```
class TrialController extends Controller
{
    use LaramanController;

    public function __configure()
    {
        $this->columns = [
            [
                'field' => 'created_at',
                'display' => 'Created',
                'formatter' => 'datetime',
                'options'   => [
                    'format' => 'F j, Y g:ia',
                ]
            ]
        ];

        //  active trials
        $this->extras['active'] = Membership::trial()->active()->count();

        //  by day
        $this->extras['byday'] = [];
        foreach (range(0, 30) as $day) {
            $date = Carbon::now()->subDays($day)->format('Y-m-d');

            $this->extras['byday'][$date] = Membership::trial()->active()->whereDate('created_at', $date)->count();
        }
    }
```