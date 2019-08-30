---
title: Mailer
description: Mailer Guide for the OriginPHP Framework
extends: _layouts.documentation
section: content
---
# Mailer
The Mailer object enables to you easily create reusable and easy to test email classes.


## Creating a Mailer
To create a Mailer and its test file run the following command

```linux
$ bin/console generate mailer SendWelcomeEmail
```

This will create 4 files

```
[ OK ] /var/www/app/Mailer/SendWelcomeEmailMailer.php
[ OK ] /var/www/tests/TestCase/Mailer/SendWelcomeEmailMailerTest.php
[ OK ] /var/www/app/View/Mailer/SendWelcomeEmail/html.ctp
[ OK ] /var/www/app/View/Mailer/SendWelcomeEmail/text.ctp
```

Now open `app/Mailer/SendWelcomeEmail.php` and adjust the email subject

```php
namespace App\Mailer;
use App\Mailer\AppMailer;

class SendWelcomeEmailMailer extends AppMailer
{
    public function execute(Entity $user)
    {
        $this->user = $user;

        $this->mail([
            'to' => $user->email,
            'subject' => 'email subject goes here',
        ]);
    }
}
```

Now open the HTML template `/app/View/Mailer/SendWelcomeEmail/html.ctp` and adjust the message

```php
<p>Hi <?= $user->name ?></p>
<p>Welcome to FunkyApp</p>
```

If a text version of the HTML mailer is not found and you are sending the email in both HTML and text, OriginPHP will automatically convert the HTML version to a text version using the [Html Utility](/docs/utilitity/html).

> It is considered a best practice to send both the HTML and Text version, and it will improve deliverability.

### Sending Defaults

The code generation command will create Mailers which extend the `AppMailer` class, there you can set your default settings for your Mailers.

> Mailer is new in version 1.29, so if you have upgraded then you will need to create this file manually.

```php
namespace App\Mailer;
use Origin\Mailer\Mailer;

class AppMailer extends Mailer
{
    public $defaults = [
        'from' => ['noreply@somewhere.com','Funky App'],
        'replyTo' => 'noreply@somewhere.com',
    ];

    # app/View/Layout/mailer.ctp
    public $layout = 'mailer';

    # Email account
    public $account = 'default';
}
```

### Creating Layouts

if you want to wrap your HTML emails in a template you can use layouts for this

Create `app/View/Layout/mailer.ctp`

```php
<!DOCTYPE html>
<html>
  <head>
    <meta content='text/html; charset=UTF-8' http-equiv='Content-Type' />
  </head>
  <body>
    <?= $this->content() ?>
  </body>
</html>
```

## Sending a Mailer

Use the dispatch method to generate and send the email, this will also execute callbacks such as `startup` and `shutdown`.

Here is simple example

```php
use App\Mailer\SendWelcomeEmailMailer;

class UserController extends Controller
{
  public function add()
    {
        $user = $this->User->new();

        if ($this->request->is(['post'])) {
            $user = $this->User->new($this->request->data());
            if ($this->User->save($user)) {

                $this->Flash->success('Your user has been created.');

                (new SendWelcomeEmailMailer())->dispatch($user);

                return $this->redirect(['action' => 'view', $this->User->id]);
            }
            $this->Flash->error('Your user could not be saved');
        }

        $this->set('user', $user);
    }
}
```

### Queuing Emails

If you have have configured [Queues](/docs/queue) you can queue emails to be sent later, when you queue a Mailer, it is added to the `mailer` queue using the `default` queue connection.

To a queue a Mailer to be sent

```php
(new SendWelcomeEmailMailer())->dispatchLater($user);
```

## Previewing the Email

To preview a message, this will do everything except send the email, and will return a message object.

```php
use App\Mailer\SendWelcomeEmail;
$message = (new SendWelcomeEmailMailer())->preview($user);
$headers = $message->header();
$body = $message->body();
```

## Email Configuration

You will want to setup both a `default` and `test` account, when you run tests the mailer will be switched to the test account. Setup your email accounts in your `config/email.php` we have created a template for you, just rename the file and fill your details.

```php
use Origin\Mailer\Email;

Email::config('default',[
        'host' => 'smtp.example.com',
        'port' => 25,
        'username' => 'demo@example.com',
        'password' => 'secret',
        'timeout' => 5,
        'ssl' => true,
        'tls' => false
    ]);

Email::config('test',[
        'engine' => 'Test'
    ]);
```

For more information on configuration see the [Email Utility](/docs/utility/email').