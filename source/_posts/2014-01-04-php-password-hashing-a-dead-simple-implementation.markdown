---
layout: post
title: "PHP Password Hashing: A Dead Simple Implementation"
date: 2014-01-04 01:55:10 -0600
comments: true
categories: ["PHP", "Security", "Passwords", "Password Validator"]
---

#### [UPDATE: Added a new section at the end of the post]
#### [UPDATE 2: Added a section RE: StorageDecorator]

**tl;dr**: Install [Password Validator][9] and all of your password troubles
will be solved.  All of them.  It'll even upgrade your old hashes
transparently. Sup?

## Hashing Done Wrong

We all know to [encrypt passwords for highest level of security][7].
Unfortunately, too many do it like this:

```php
class SecurityFail
{
    // Encrypt Passwords for Highest Level of Security.
    static public function encrypt($pword)
    {
        return md5($pword);
    }
}
```

While there was never any excuse for getting it *that* wrong, there's now no
excuse for getting it wrong at all. Developers, meet the new(-ish) 
[PHP password hashing][1] functions (and the userland implementation [password-compat][5]).

## Hashing Done Right

First, alter the password column in your user database to `VARCHAR(255)`.
Current `BCRYPT` passwords are 60 characters in length, but when PHP upgrades
the default hash (which will happen at some point), you want to be ready.
Really, just do it.

When it's time to create a new user password, throw the plain text password
into [`password_hash()`][2]:

``` php
$hash = password_hash($plainTextPassword, PASSWORD_DEFAULT);
```

The next time a user logs in, use a little [`password_verify()`][3] action:

``` php
$isValid = password_verify($plainTextPassword, $hashedPassword);
```

If the password is valid, check to see if it needs to be rehashed with
[`password_needs_rehash()`][4]:

``` php
$needsRehash = password_needs_rehash($hashedPassword, PASSWORD_DEFAULT);
```

If the password needs to be rehashed, run it through `password_hash()` again
and persist the result.

Trivial, right? Right!

## Even Trivial-er

Since implementing the code above might take as many as two or three hours out
of your day, I went ahead and implemented it for you. Behold, Password
Validator!

## Password Validator

The [Password Validator][9] library *validates* `password_hash` generated
passwords, *rehashes* passwords as necessary, and can *upgrade* legacy
passwords (if configured to do so).

The really big deal here is the **ease of upgrading** from your current legacy
hashes to the new, more secure, PHP generated hashes. More on that later.

## Usage

### Password Validation

If you're already using [`password_hash`][2] generated passwords in your
application, you need do nothing more than add the validator in your
authentication script. The validator uses [`password_verify`][3] to test 
the validity of the provided password hash.

``` php
use JeremyKendall\Password\PasswordValidator;

$validator = new PasswordValidator();
$result = $validator->isValid($_POST['password'], $hashedPassword);

if ($result->isValid()) {
    // password is valid
}
```

If your application requires options other than the `password_hash` defaults,
you can set both the `salt` and `cost` options with `PasswordValidator::setOptions()`.

``` php
$options = array(
    'salt' => 'SettingYourOwnSaltIsNotTheBestIdea',
    'cost' => 11,
);
$validator->setOptions($options);
```

**IMPORTANT**: If you're using a `cost` other than the default cost of `10`,
your passwords will be rehashed with a cost of `10` *unless* you set the cost
using `PasswordValidator::setOptions()`.

### Rehashing

Each valid password is tested using [`password_needs_rehash`][4]. If a rehash
is necessary, the valid password is rehashed using `password_hash` with the
provided options. The result code `Result::SUCCESS_PASSWORD_REHASHED` will be
returned from `Result::getCode()` and the rehashed password is available via
`Result::getPassword()`.

``` php
if ($result->isValid() && $result->getCode() == Result::SUCCESS_PASSWORD_REHASHED) {
    $rehashedPassword = $result->getPassword();
    // Persist rehashed password
}
```

**IMPORTANT**: If the password has been rehashed, it's critical that you
persist the updated password hash. Otherwise, what's the point, right?

### Upgrading Legacy Passwords

You can use the `PasswordValidator` whether or not you're currently using
`password_hash` generated passwords. The validator will upgrade your current
legacy hashes to the new `password_hash` generated hashes.  All you need to
do is provide a validator callback for your password hashing scheme and then
[decorate][6] the validator with the `UpgradeDecorator`.

``` php
use JeremyKendall\Password\Decorator\UpgradeDecorator;

// Example callback to validate a sha512 hashed password
$callback = function ($password, $passwordHash) {
    if (hash('sha512', $password) === $passwordHash) {
        return true;
    }

    return false;
};

$validator = new UpgradeDecorator(new PasswordValidator(), $callback);
```

The `UpgradeDecorator` will validate a user's current password using the
callback.  If the user's password is valid, it will be hashed with
`password_hash` and returned in the `Result` object, as above.

If the callback determines the password is invalid, the password will be passed
along to the `PasswordValidator` in case it's already been upgraded.

### Persisting Rehashed Passwords

Whenever a validation attempt returns `Result::SUCCESS_PASSWORD_REHASHED`, it's
important to persist the updated password hash.

``` php
if ($result->getCode() === Result::SUCCESS_PASSWORD_REHASHED) {
    $rehashedPassword = $result->getPassword();
    // Persist rehashed password
}
```

While you can always perform the test and then update your user database
manually, if you choose to use the **Storage Decorator** all rehashed passwords
will be automatically persisted.

The Storage Decorator takes two constructor arguments: An instance of
`PasswordValidatorInterface` and an instance of the
`JeremyKendall\Password\Storage\StorageInterface`.

#### StorageInterface

The `StorageInterface` includes a single method, `updatePassword()`. A class
honoring the interface might look like this:

``` php
<?php

namespace Example;

use JeremyKendall\Password\Storage\StorageInterface;

class UserDao implements StorageInterface
{
    public function __construct(\PDO $db)
    {
        $this->db = $db;
    }

    public function updatePassword($identity, $password)
    {
        $sql = 'UPDATE users SET password = :password WHERE username = :identity';
        $stmt = $this->db->prepare($sql);
        $stmt->execute(array('password' => $password, 'username' => $identity));
    }
}
```

#### Storage Decorator

With your `UserDao` in hand, you're ready to decorate a
`PasswordValidatorInterface`.

``` php
use Example\UserDao;
use JeremyKendall\Password\Decorator\StorageDecorator;

$storage = new UserDao($db);
$validator = new StorageDecorator(new PasswordValidator(), $storage);

// If validation results in a rehash, the new password hash will be persisted
$result = $validator->isValid('password', 'passwordHash', 'username');
```

**IMPORTANT**: You must pass the optional third argument (`$identity`) to
`isValid()` when calling `StorageDecorator::isValid()`.  If you do not do so,
the `StorageDecorator` will throw an `IdentityMissingException`.

### Validation Results

Each validation attempt returns a `Result` object. The object provides some
introspection into the status of the validation process.

* `Result::isValid()` will return `true` if the attempt was successful
* `Result::getCode()` will return one of three possible `int` codes:
    * `Result::SUCCESS` if the validation attempt was successful
    * `Result::SUCCESS_PASSWORD_REHASHED` if the attempt was successful and the password was rehashed
    * `Result::FAILURE_PASSWORD_INVALID` if the attempt was unsuccessful
* `Result::getPassword()` will return the rehashed password, but only if the password was rehashed

### Database Schema Changes

As mentioned above, because this library uses the `PASSWORD_DEFAULT` algorithm,
it's important your password field be `VARCHAR(255)` to account for future
updates to the default password hashing algorithm.

## Helper Scripts

There are two helper scripts available, both related to the password hash
functions (these functions are only available after running `composer
install`).

### version-check

If you're not already running PHP 5.5+, you should run `version-check` to
ensure your version of PHP is capable of using password-compat, the userland
implementation of the PHP password hash functions.  Run `./bin/version-check`
from the root of your project. The result of the script is pass/fail.

### cost-check

The default `cost` used by `password_hash` is 10.  This may or may not be
appropriate for your production hardware, and it's entirely likely you can use
a higher cost than the default. `cost-check` is based on the [finding a good
cost][8] example in the PHP documentation. Simply run `./bin/cost-check` from the command line and an appropriate cost will be returned.

**NOTE**: The default time target is 0.2 seconds.  You may choose a higher or lower 
target by passing a float argument to `cost-check`, like so:

``` bash
$ ./bin/cost-check 0.4
Appropriate 'PASSWORD_DEFAULT' Cost Found:  13
```

## Wrapping Up

The addition of native [password hashing functions][1] is the most important
security update to PHP since, well, I don't know when. There's no excuse for
not implementing them in your applications, and the [Password Validator][9]
library makes it trivial. That's especially true when it comes to updating your
legacy password hashes, which many of us need to do.  Even if you only use the
Password Validator as a roadmap for your own implementation, I strongly
recommend upgrading ASAP.

## Kudos

I was remiss to not add this bit of kudos when I originally published this
post.  Better late than never.

Credit for the new password hashing functions goes to [Anthony Ferrara][10].
He submitted the [original RFC][11] and created the [password-compat][5]
library. The PHP community owes Anthony a debt of gratitude for making password
hash security so ridiculously simple.  Seriously, if *I* can grok it, you
know it's idiot proof :-)

Without Anthony's hard work (and [PHP core's unanimous 'Yes' votes][13], and
the [password-compat contributors][12]), my small contribution wouldn't have
been possible.  Kudos to you all.

[1]: http://www.php.net/manual/en/ref.password.php
[2]: http://www.php.net/manual/en/function.password-hash.php
[3]: http://www.php.net/manual/en/function.password-verify.php
[4]: http://www.php.net/manual/en/function.password-needs-rehash.php
[5]: https://github.com/ircmaxell/password_compat
[6]: http://en.wikipedia.org/wiki/Decorator_pattern
[7]: http://csiphp.com/blog/2012/02/16/encrypt-passwords-for-highest-level-of-security/
[8]: http://php.net/password_hash#example-875
[9]: https://packagist.org/packages/jeremykendall/password-validator
[10]: https://twitter.com/ircmaxell
[11]: https://wiki.php.net/rfc/password_hash
[12]: https://github.com/ircmaxell/password_compat/graphs/contributors
[13]: https://wiki.php.net/rfc/password_hash#vote
