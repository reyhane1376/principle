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
# OCP (open close principle)

## You should be able to extend a class's behavior without modifying it.

## Two different perspective:

1. Reducing the amount of change (Robert C Martin)
2. Backward compatibility (Bertrand Meyer)

### Violations of the open/close principle:

It contains conditions to determine a strategy.

Conditions using the same variables or constants are recurring inside the class or related classes. 

The class contains hard-coded references to other classes or class names.

Inside the class, objects are being created using the new operator. 

The class has protected properties or methods to allow changing its behavior by overriding state or behavior. 

### before

```php 
namespace Src\Solid\OCP;

class GenericEncoder {
    public function encode($data, string $format): string {
        if ($format === 'json') {
            $encoder = new JsonEncoder();
        } elseif ($format === 'xml') {
            $encoder = new XMLEncoder();
        } else {
            trow new InvalidArgumentException('invalid format')
        }
        return $encoder->encode($data);
    }
}
```

```php
namespace Src\Solid\OCP;

class JsonEncoder {
    public function __construct() {
    }

    public function encode($data): string {
        return '{data:""}';
    }
}
```

### after

first fix it with srp:

```php
namespace Src\Solid\OCP;

interface EncoderInterface
{
    public function encode($data): string;
}
```
```php
class JsonEncoder implements EncoderInterface
{
    // ... implementation of encode() method ...
}
```

```php
class XMLEncoder implements EncoderInterface
{
    // ... implementation of encode() method ...
}
```
then ocp:

```php
namespace Src\Solid\OCP;

class EncoderFactory
{
    public function createEncoder(string $format): EncoderInterface
    {
        if ($format === 'json') {
            $encoder = new JsonEncoder();
        } elseif ($format === 'xml') {
            $encoder = new XMLEncoder();
        } else {
            throw new InvalidArgumentException(message: 'invalid format');
        }

        return $encoder;
    }
}
```

```php
class GenericEncoder
{
    private $encoderFactory;

    /**
     * @param EncoderFactory
     */
    public function __construct(EncoderFactory $encoderFactory)
    {
        $this->encoderFactory = $encoderFactory;
    }

    public function encode($data, string $format): string
    {
        $encoder = $this->encoderFactory->createEncoder($format);

        return $encoder->encode($data);
    }
}
```

then make it better:

```php
namespace Src\Solid\OCP;

class EncoderFactory
{
    private $factories = [];

    public function addEncoderFactory(string $format, callable $factory)
    {
        $this->factories[$format] = $factory;
    }

    public function createEncoder(string $format): EncoderInterface
    {
        if (!isset($this->factories[$format])) {
            throw new InvalidArgumentException(message: 'invalid format');
        }

        $factory = $this->factories[$format];

        return $factory();
    }
}
```

then make it better:(immutable)

```php
namespace Src\Solid\OCP;

interface EncoderFactoryInterface
{
    public function createEncoder(string $format): EncoderInterface;
}
```
```php
namespace Src\Solid\OCP;

interface EncoderFactoryConfigInterface
{
    public function addEncoderFactory(string $format, callable $factory): void;
}
```
```php
class EncoderFactory implements EncoderFactoryConfigInterface, EncoderFactoryInterface
```
```php
class GenericEncoder
{
    private $encoderFactory;

    /**
     * @param EncoderFactoryInterface $encoderFactory
     */
    public function __construct(EncoderFactoryInterface $encoderFactory)
    {
        $this->encoderFactory = $encoderFactory;
    }

    public function encode($data, string $format): string
    {
        $encoder = $this->encoderFactory->createEncoder($format);

        return $encoder->encode($data);
    }
}
```
# lsp (open close principle)

## Derived classes must be substitutable for their base classes.

### Violations of the Liskov substitution principle:

A derived class does not have an implementation for all methods.
Different substitutes return things of different types.
A derived class is less permissive with regard to method arguments.
Secretly programming against a more specific type.

### before

```php
interface FileInterface
{
    public function rename();
    public function move();
    public function copy();
    public function download();
}
```
```php
namespace Src\Solid\LSP;

class GoogleDriveFile implements FileInterface
{
    public function rename()
    {
        // TODO: Implement rename() method.
    }

    public function move()
    {
        // TODO: Implement move() method.
    }

    public function copy()
    {
        // TODO: Implement copy() method.
    }

    public function download()
    {
        // TODO: Implement download() method.
    }
}
```

```php
namespace Src\Solid\LSP;

class DropBoxFile implements FileInterface
{
    public function rename()
    {
        // TODO: Implement rename() method.
    }

    public function move()
    {
        // TODO: Implement move() method.
    }

    public function copy()
    {
        // TODO: Implement copy() method.
    }

    public function download()
    {
        // TODO: Implement download() method.
    }
}
```
```php
namespace Src\Solid\LSP;

class LocalFile implements FileInterface
{
    public function rename()
    {
        // TODO: Implement rename() method.
    }

    public function move()
    {
        // TODO: Implement move() method.
    }

    public function copy()
    {
        // TODO: Implement copy() method.
    }

    public function download()
    {
        // TODO: Implement download() method.
    }
}
```

## download method not defined for localFile class

### after

change download method to new interface

```php
interface DownloadableFileInterface extends FileInterface
{
    public function download();
}
```
```php
class GoogleDriveFile implements DownloadableFileInterface
{
}
```
```php
namespace Src\Solid\LSP;

class LocalFile implements FileInterface
{
    public function rename()
    {
        // TODO: Implement rename() method.
    }

    public function move()
    {
        // TODO: Implement move() method.
    }

    public function copy()
    {
        // TODO: Implement copy() method.
    }
}
```

----------------------
Is it mandatory to specify a return type for methods within an interface 
```php
interface DownloadableFileInterface extends FileInterface
{
    public function download(): bool;
}
```


-----------
### before
```php
interface FileServiceInterface
{
    public function encode(FileInterface $file);
}
```

```php
namespace Src\Solid\LSP;

class FileService implements FileServiceInterface
{
    public function encode(FileInterface $file)
    {
        if (!($file instanceof LocalFile)) {
            throw new InvalidArgumentException(message: 'only local file can be encoded.');
        }
    }
}
```
### after

```php
interface EncodeableFileInterface extends FileInterface
{
}
```
```php
namespace Src\Solid\LSP;

class FileService implements FileServiceInterface
{
    public function encode(EncodeableFileInterface $file)
    {
    }
}
```
```php
interface FileServiceInterface
{
    public function encode(EncodeableFileInterface $file);
}
```
# isp (Interface Segregation Principle)

## Make fine-grained interfaces that are client specific.

### Violations of the interface segregation principle:

Multiple use cases.
No interface, just a class.

### before

```php
interface Notifier
{
    public function sendSMS();
    public function sendEmail();
    public function sendWebSocket();
}
```
```php
namespace Src\Solid\ISP;

class KaveNegarSMSProvider implements Notifier
{
    public function sendSMS()
    {
        // TODO: Implement sendSMS() method.
    }

    public function sendEmail()
    {
        // TODO: Implement sendEmail() method.
    }

    public function sendWebSocket()
    {
        // TODO: Implement sendWebSocket() method.
    }
}
```
```php
namespace Src\Solid\ISP;

class MailChimpEmailProvider implements Notifier
{
    public function sendSMS()
    {
        // TODO: Implement sendSMS() method.
    }

    public function sendEmail()
    {
        // TODO: Implement sendEmail() method.
    }

    public function sendWebSocket()
    {
        // TODO: Implement sendWebSocket() method.
    }
}
```
### after

```php
interface SMSProvider
{
    public function sendSMS();
}

```
```php
interface EmailProvider
{
}
```
```php
interface WebSocketProvider
{
    public function sendWebSocket();
}
```
```php
namespace Src\Solid\ISP;

class KaveNegarSMSProvider implements SMSProvider
{
    public function sendSMS()
    {
        // TODO: Implement sendSMS() method.
    }
}
```
```php
namespace Src\Solid\ISP;

class MailChimpEmailProvider implements EmailProvider
{
    public function sendEmail()
    {
        // TODO: Implement sendEmail() method.
    }
}
```

# DIP (Dependancy Inversio Principle)

## Depend on abstractions, not on concretions.

### Violations of the dependency inversion principle:

A high-level class depends on a low-level class.
Vendor lock-in.

### before

```php
namespace Src\Solid\DIP;

class Authentication
{
    private $connection;

    /**
     * @param Connection $connection
     */
    public function __construct(Connection $connection)
    {
        $this->connection = $connection;
    }

    public function check(string $username, string $password): bool
    {
        $user = $this->connection->query("SELECT * FROM users WHERE username = ?", [$username]);

        if (!$user) {
            throw new RuntimeException(message: 'invalid username or password');
        }

        // ...
    }
}
```

```php
namespace Src\Solid\DIP;

class Connection
{
    public function query(string $query, array $params): bool
    {
        return true;
    }
}
```
```php
interface UserProviderInterface
{
    public function findUser(string $username): bool;
}
```
```php
namespace Src\Solid\DIP;

class UserProvider implements UserProviderInterface
{
    private $connection;

    /**
     * @param Connection $connection
     */
    public function __construct(Connection $connection)
    {
        $this->connection = $connection;
    }

    public function findUser(string $username)
    {
        $user = $this->connection->query("SELECT * FROM users WHERE username=?", ['username' => $username]);

        if (!$user) {
            throw new RuntimeException(message: 'invalid username or password');
        }
    }
}
```
```php
namespace Src\Solid\DIP;

class Authentication
{
    private $user_provider;

    /**
     * @param UserProviderInterface $user_provider
     */
    public function __construct(UserProviderInterface $user_provider)
    {
        $this->user_provider = $user_provider;
    }

    public function check(string $username, string $password): bool
    {
        $user = $this->user_provider->findUser($username);

        if (!$user) {
            throw new RuntimeException(message: 'invalid username or password');
        }
    }
}
```
```php
namespace Src\Solid\DIP;

class MongoUserProvider implements UserProviderInterface
{
    public function findUser(string $username): bool
    {
        // TODO: Implement findUser() method.
        return true;
    }
}
```
