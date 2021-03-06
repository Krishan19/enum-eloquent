# Konekt Enum Eloquent Bindings


[![Travis Build Status](https://img.shields.io/travis/artkonekt/enum-eloquent.svg?style=flat-square)](https://travis-ci.org/artkonekt/enum-eloquent)
[![Packagist Stable Version](https://img.shields.io/packagist/v/konekt/enum-eloquent.svg?style=flat-square&label=stable)](https://packagist.org/packages/konekt/enum-eloquent)
[![Packagist downloads](https://img.shields.io/packagist/dt/konekt/enum-eloquent.svg?style=flat-square)](https://packagist.org/packages/konekt/enum-eloquent)
[![StyleCI](https://styleci.io/repos/105900484/shield?branch=master)](https://styleci.io/repos/105900484)
[![MIT Software License](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](LICENSE.md)

This package provides support for auto casting [konekt enum](https://github.com/artkonekt/enum) fields in [Eloquent models](https://laravel.com/docs/5.4/eloquent-mutators).

> Supported Konekt Enum versions are 2.0+ and Eloquent 5.0+

[Changelog](Changelog.md)

## Installation

`composer require konekt/enum-eloquent`

## Usage

1. Add the `CastsEnums` trait to your model
2. Define the attributes to be casted via the `protected $enums` property on the model

### Example

**The Enum:**

```php
namespace App;

use Konekt\Enum\Enum;

class OrderStatus extends Enum
{
    const __default = self::PENDING;

    const PENDING   = 'pending';
    const CANCELLED = 'cancelled';
    const COMPLETED = 'completed';

}
```

**The Model:**

```php
namespace App;

use Illuminate\Database\Eloquent\Model;
use Konekt\Enum\Eloquent\CastsEnums;

class Order extends Model
{
    use CastsEnums;

    protected $enums = [
        'status' => OrderStatus::class
    ];
}
```

**Client code:**
```php
$order = Order::create([
    'status' => 'pending'
]);

// The status attribute will be an enum object:
echo get_class($order->status);
// output: App\OrderStatus

echo $order->status->value();
// output: 'pending'

echo $order->status->isPending() ? 'yes' : 'no';
// output: yes

echo $order->status->isCancelled() ? 'yes' : 'no';
// output: no

// You can assign an enum object as attribute value:
$order->status = OrderStatus::COMPLETED();
echo $order->status->value();
// output: 'completed'

// It also works with mass assignment:
$order = Order::create([
    'status' => OrderStatus::COMPLETED()    
]);

echo $order->status->value();
// output 'completed'

// It still accepts scalar values:
$order->status = 'completed';
echo $order->status->isCompleted() ? 'yes' : 'no';
// output: yes

// But it doesn't accept scalar values that aren't in the enum:
$order->status = 'negotiating';
// throws UnexpectedValueException
// Given value (negotiating) is not in enum `App\OrderStatus`
```

### Resolving Enum Class Runtime

It is possible to defer the resolution of an Enum class to runtime.

It happens using the `ClassName@method` notation known from Laravel.

This is useful for libraries, so you can 'late-bind' the actual enum class and let the user to extend it.

#### Example

**The Model:**

```php
namespace App;

use Illuminate\Database\Eloquent\Model;
use Konekt\Enum\Eloquent\CastsEnums;

class Order extends Model
{
    use CastsEnums;

    protected $enums = [
        'status' => 'OrderStatusResolver@enumClass'
    ];
}
```

**The Resolver:**

```php
namespace App;

class OrderStatusResolver
{
    /**
     * Returns the enum class to use as order status enum
     *
     * @return string
     */
    public static function enumClass()
    {
        return config('app.order.status.class', OrderStatus::class);
    }
}
```

This way the enum class becomes configurable without the need to modify the Model code.

---

Enjoy!

For detailed usage of konekt enums refer to the [Konekt Enum Documentation](https://artkonekt.github.io/enum).
