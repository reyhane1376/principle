# Tell Don't Ask (TDA)

In Object-oriented programming, TDA (Tell Don’t Ask) means telling an object to perform an action rather than asking for information and taking action on behalf of the object.

### before
```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;

class UsersController extends Controller
{
    public function updateWallet(Request $request)
    {
        $user = User::find(1);
        $user_wallet = $user->wallet;
        $user_wallet += 100000;
        $user->wallet = $user_wallet;
        $user->save();

        // $user->update(['wallet' => $user_wallet]);
    }
}
```


### after
```php
<?php

class User
{
    protected $hidden = [...];

    protected $casts = [...];

    public function increaseWallet(int $amount): void
    {
        $currentWallet = $this->attributes['wallet'];
        $this->attributes['wallet'] = $currentWallet + $amount;
        $this->increment('wallet', $amount);
        $this->save();
    }
}
```

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;

class UsersController extends Controller
{
    public function updateWallet(Request $request)
    {
        $user = User::find(1);
        $user->increaseWallet(100000);
    }
}
```

