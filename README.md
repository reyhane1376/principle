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

# YAGNI principle (You Aren't Gonna Need It)

The principle helps developers avoid wasted effort on features that are assumed to be needed at some point. The idea is that this assumption often ends up being incorrect. Even if a feature ends up being desired, it still may turn out that the implementation is not necessary. The argument is for developers to not waste time on creating extraneous elements that may not be necessary and can hinder or slow the development process.


# SRP (single responsibility principle)

## A class should have one, and only one, reason to change.

### Violations of the single responsibility principle:

There are too many instance variables in the class.
There are too many public methods in the class.
Each method of the class uses different instance variables.
Specific tasks are delegated to private methods. 

### before

```php
<?php

namespace Src\Solid\Srp;

class ConfirmationMailMailer
{
    private $templating;
    private $translator;
    private $mailer;

    public function __construct(
        TemplatingEngineInterface $templating,
        TranslatorInterface $translator,
        MilerInterface $mailer
    ) {
        $this->templating = $templating;
        $this->translator = $translator;
        $this->mailer     = $mailer;
    }

    public function sendTo(User $user)
    {
        $message = $this->createMessageFor($user);
        $this->sendMessage($message);
    }

    private function createMessageFor(User $user): Message {
        $subject = $this->translator->translate(text: 'please confirm your email address');
        $body = $this->templating->render('email.confirm', [
            'confirm_code' => $user->getConfirmCode(),
        ]);

        return new Message($subject, $body, $user->getEmailAddress());
    }

    private function sendMessage(Message $message)
    {
        $this->mailer->send($message);
    }
}

```

```php

<?php

namespace Src\Solid\Srp;

class Message
{
    private $subject;
    private $body;
    private $emailAddress;

    public function __construct($subject, $body, $emailAddress)
    {
        $this->subject      = $subject;
        $this->body         = $body;
        $this->emailAddress = $emailAddress;
    }

    public function get_subject()
    {
        return $this->subject;
    }

    public function get_body()
    {
        return $this->body;
    }

    public function get_email_address()
    {
        return $this->emailAddress;
    }
}
```

```php
<?php

namespace Src\Solid\Srp;

interface TranslatorInterface {
    public function translate(string $text): string;
}

```

```php
namespace Src\Solid\Srp;

interface TemplatingEngineInterface {
    public function render(string $template, array $params): string;
}
```

```php
namespace Src\Solid\Srp;

interface MailerInterface
{
    public function send(Message $message);
}
```

### after

```php
<?php

use TemplatingEngineInterface;
use TranslatorInterface;
use Message;
use User;

class ConfirmationMailFactory
{
    private TemplatingEngineInterface $templating;
    private TranslatorInterface $translator;

    public function __construct(TemplatingEngineInterface $templating, TranslatorInterface $translator)
    {
        $this->templating = $templating;
        $this->translator = $translator;
    }

    public function createMessageFor(User $user): Message
    {
        $subject = $this->translator->translate(text: 'please confirm your email address');
        $body = $this->templating->render(template: 'email.confirm', [
            'confirm_code' => $user->getConfirmCode()
        ]);

        return new Message($subject, $body, $user->getEmailAddress());
    }
}
```

```php
<?php

namespace Src\Solid\Srp;

class ConfirmationMailMailer
{
    private $confirmationMailFactory;
    private $mailer;

    public function __construct(
        ConfirmationMailFactory $confirmationMailFactory,
        MilerInterface $mailer
    ) {
        $this->confirmationMailFactory = $confirmationMailFactory;
        $this->mailer     = $mailer;
    }

    public function sendTo(User $user)
    {
        $message = $this->createMessageFor($user);
        $this->sendMessage($message);
    }

    private function createMessageFor(User $user): Message {
        return $this->confirmationMailFactory->createMessageFor($user);
    }

    private function sendMessage(Message $message)
    {
        $this->mailer->send($message);
    }
}

```