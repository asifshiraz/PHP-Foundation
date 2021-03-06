# PHP-Foundation

Writing modern PHP applications efficiently

## Requirements

 * Apache HTTP Server 2.2.0+
   * `mod_rewrite`
   * `mod_headers`
 * PHP 5.6.0+
   * Multibyte String extension (`mbstring`)
   * PDO (PHP Data Objects) extension (`pdo`)
   * OpenSSL extension (`openssl`)
   * GMP (GNU Multiple Precision) extension (`gmp`)
 * MySQL 5.5.3+ **or** MariaDB 5.5.23+ (for authentication only)

## Installation

 1. Copy all files from this repository to your development path or web server.

    **Note:** This framework does currently work in the web root (at `/` under your domain) only.

 1. Copy the configuration template `config/.env.example` to `config/.env`. This new file is where your private configuration will be stored. Fill in the correct settings for your environment. The most important setting is `APP_PUBLIC_URL` which is required for your application to work in the first place.
 1. If you want to use the built-in authentication component, create the database tables for [MySQL](https://github.com/delight-im/PHP-Auth/blob/master/Database/MySQL.sql) or [MariaDB](https://github.com/delight-im/PHP-Auth/blob/master/Database/MySQL.sql) in the database that you specified in `config/.env`.
 1. Get [Composer](https://getcomposer.org/) and run `composer install` in this root directory to set up the framework and its dependencies.
 1. Make sure that the content of the `storage/` directory is writable by the web server, e.g. using

    ```
    $ sudo chown -R www-data:www-data /var/www/html/storage/*
    ```

    on Linux if this directory (containing the `README.md`) is in `/var/www/html/` on the server.

## Usage

### Application structure

 * `app/`: This is the most important directory: It's where you'll write all your PHP code. It's entirely up to you how you structure your application in this directory. Create as many files and subdirectories here as you wish. The `index.php` file is the main entry point to your application. The complete folder is exclusively for you.
 * `config/`: Store your local configuration, which should include all confidential information, keys and secrets, here. Put everything in `config/.env` while keeping an up-to-date copy of that file in `config/.env.example` which should have exactly the same keys but all the confidential values removed. The first file will be your private configuration, the second file can be checked in to version control for others to see what configuration keys they need. Always remember to securely back up the private configuration file (`config/.env`) somewhere outside of your VCS. Do *not* store it in version control.
 * `public/`: This is where you can store your static assets, such as CSS files, JavaScript files, images, your `robots.txt` and so on. The files will be available at the root URL of your application, i.e. `public/favicon.ico` can simply be accessed at `favicon.ico`. The complete folder is exclusively for you.
 * `storage/app/`: Storage that you can use in your app to store files temporarily or permanently. This space is private to your application. Feel free to add any number of subfolders and files here.
 * `storage/framework/`: Storage used by the framework, e.g. for caching. Do *not* add, modify or delete anything here.
 * `vendor/`: This directory is where Composer will install all dependencies. Do *not* add, modify or delete anything here.
 * `views/`: If you use templates for HTML, XML, CSV or LaTeX (or anything else) that should be rendered by the built-in template engine, this is where you should store them.
 * `.htaccess`: Rules and optimizations for Apache HTTP Server. You may add your own rules here, but only in the `CUSTOM` section at the bottom.
 * `composer.json`: The dependencies of your application. This framework must *always* be included in the `require` section as `delight-im/framework`, so please don't remove that entry. But otherwise, feel free to add or modify any of your own entries.
 * `index.php`: This is the main controller of this framework. It sets everything up correctly and passes on control to your application code. Do *not* change or delete this.

### Routing

Your application will be available at the URL where this root folder is accessible on your web server.

**Note:** You should have added this URL to the `config/.env` already. This is required for your application to work.

The single pages or endpoints of your application will be served at the routes that you define in the `app/` directory. A few sample routes have already been added.

For more detailed information on routes, please refer to the [documentation of the router](https://github.com/delight-im/PHP-Router).

### Charsets and encodings

Everything is UTF-8 all the way through! Just remember to save your own source files as UTF-8, too.

### Storage

You may store files private to your application in `storage/app/`. Feel free to store the files either in the root directory or in any number of subfolders that you create.

The storage path can be retrieved in your application code via `$app->getStoragePath('/path/to/subfolder/file.txt')`, for example.

Files inside `storage/app/` are private to your application. You may use PHP's file handling utilities to work with the files or offer them to the user as a download (see "File downloads" below).

### Database access

Did you set your database credentials in `config/.env`? If the configuration is valid, getting a database connection is as simple as this:

```php
$app->db();
```

For information on how to read and write data using this instance, please refer to the [documentation of the database library](https://github.com/delight-im/PHP-DB).

**Note:** You don't have to construct the database instance or establish the connection yourself. This will be done for you automatically. Just call the methods to read and write data on the instance provided.

### String handling

Convenient string handling in an object-oriented way with full Unicode support is available on any string after wrapping it in the `s(...)` helper function.

For information on all the methods available with strings wrapped in `s(...)`, please refer to the [documentation of the string library](https://github.com/delight-im/PHP-Str).

**Note:** There is no need for you to set up the `s(...)` shorthand anymore. This has been done already.

### Input validation

This framework comes with easy and safe input validation for `GET`, `POST` and cookie data as well as for any existing variable.

Both validation and filtering are performed automatically and *correctly typecast* values are returned for you.

If a value does not exist or if it's invalid, you're guaranteed to receive `null` as the result. This way, you don't have to check for empty strings, `null`, `false` or invalid formats separately.

```php
$app->input()->get('username'); // equivalent to TYPE_STRING
$app->input()->get('profileId', TYPE_INT);
$app->input()->get('disabled', TYPE_BOOL);
$app->input()->get('weight', TYPE_FLOAT);
$app->input()->get('message', TYPE_TEXT);
$app->input()->get('recipientEmail', TYPE_EMAIL);
$app->input()->get('linkTo', TYPE_URL);
$app->input()->get('whoisIp', TYPE_IP);
$app->input()->get('transmission', TYPE_RAW);

$app->input()->post('country'); // equivalent to TYPE_STRING
$app->input()->post('zipCode', TYPE_INT);
$app->input()->post('subscribe', TYPE_BOOL);
$app->input()->post('price', TYPE_FLOAT);
$app->input()->post('csv', TYPE_TEXT);
$app->input()->post('newsletterEmail', TYPE_EMAIL);
$app->input()->post('referringSite', TYPE_URL);
$app->input()->post('signUpIp', TYPE_IP);
$app->input()->post('print', TYPE_RAW);

$app->input()->cookie('realName'); // equivalent to TYPE_STRING
$app->input()->cookie('lastLoginTimestamp', TYPE_INT);
$app->input()->cookie('hideAds', TYPE_BOOL);
$app->input()->cookie('cartTotalAmount', TYPE_FLOAT);
$app->input()->cookie('draft', TYPE_TEXT);
$app->input()->cookie('friend', TYPE_EMAIL);
$app->input()->cookie('social', TYPE_URL);
$app->input()->cookie('antiFraud', TYPE_IP);
$app->input()->cookie('display', TYPE_RAW);

$app->input()->value($username); // equivalent to TYPE_STRING
$app->input()->value($profileId, TYPE_INT);
$app->input()->value($disabled, TYPE_BOOL);
$app->input()->value($weight, TYPE_FLOAT);
$app->input()->value($message, TYPE_TEXT);
$app->input()->value($recipientEmail, TYPE_EMAIL);
$app->input()->value($linkTo, TYPE_URL);
$app->input()->value($whoisIp, TYPE_IP);
$app->input()->value($transmission, TYPE_RAW);
```

### HTML escaping

In order to escape any string for safe use in HTML, just wrap the string in the `e(...)` helper function:

```php
echo e('Bob <b>says</b> "Hello world"');
// => Bob &lt;b&gt;says&lt;/b&gt; &quot;Hello world&quot;

// or

echo '<img src="'.e($profilePictureUrl).'" alt="Bob" width="96" height="96">';
```

```html
<!-- or -->

<img src="<?= e($profilePictureUrl) ?>" alt="Bob" width="96" height="96">
```

If you use templates stored in the `views/` directory (see below), you don't need this, as templates come with automatic escaping by default.

### Templates

Place your HTML, XML, CSV or LaTeX (or anything else) templates in the `views/` folder. Make them powerful and re-usable with the [Twig language](http://twig.sensiolabs.org/doc/templates.html). A few examples for such templates have already been added.

In order to render a template, just load it in your PHP code:

```php
echo $app->view('welcome.html');
```

Usually, you'll want to pass data to the templates as well:

```php
echo $app->view('welcome.html', [
    'userId' => $id
    'name' => $name
    'messages' => $messages
]);
```

The `APP_DEBUG` setting from your `config/.env` file determines whether templates are always reloaded (in debug mode) or cached (in production).

The data that you passed in the second parameter of the `$app->view(...)` method will be available in your template:

```html
<h1>Hello, {{ name }}!</h1>
```

All output is escaped (and thus safe) by default! If you need to disable escaping for a variable, you can do so using the `raw` filter:

```html
<span>Goodbye, {{ name | raw }}!</span>
```

Conditional expressions and loops are available as control structures:

```html
{% if messages|length > 0 %}
    <p>You have <strong>{{ messages|length }}</strong> new messages!</p>
{% endif %}
```

```html
<div>
    {% for picture in pictures %}
        <img src="{{ picture.url }}" alt="{{ picture.description }}" width="100" height="100">
    {% endfor %}
</div>
```

For less code and more markup re-use, templates can be embedded inside each other:

```html
{{ include('includes_header.html') }}
```

Embedding can even be combined with loops:

```html
<ul>
    {% for user in users %}
        <li>{{ user.username }}</li>
        {% for picture in user.pictures %}
            {{ include('picture_box.html') }}
            <!-- `picture_box` has access to `picture` -->
        {% endfor %}
    {% endfor %}
</ul>
```

You can add custom filters in your PHP code which can then be used to modify data in the templates:

```php
$app->getTemplateManager()->addFilter('repeatAndReverse', function ($str) {
    return strrev($str.' '.$str);
});
```

```html
<p>We will repeat {{ myVariable | repeatAndReverse }} and reverse this!</p>
```

Moreover, you can add variables as globals in your PHP code which can then be accessed in the templates:

```php
$app->getTemplateManager()->addGlobal('googleAnalytics', $ga);
```

```html
{{ googleAnalytics.trackingCode | raw }}
</body>
```

The main application object is available as a global variable named `app` by default. This means that you can access all methods from the application object in your templates:

```html
<link rel="icon" type="image/png" href="{{ app.url('/favicon.ico') }}">
```

Please refer to the [documentation of the template engine](http://twig.sensiolabs.org/doc/templates.html) for more information on how to write templates.

### Authentication

Convenient and secure authentication is always available via the `$app->auth()` helper method.

For information on how to register or log in users with a few lines of code, please refer to the [documentation of the authentication library](https://github.com/delight-im/PHP-Auth).

### Sessions and cookies

Convenient management of session data and cookies is available via the two classes `\Delight\Cookie\Cookie` and `\Delight\Cookie\Session`.

For information on how to use these two classes, please refer to [the documentation of the session and cookie library](https://github.com/delight-im/PHP-Cookie).

**Note:** You don't have to include these two classes yourself anymore. They are included and loaded automatically.

### Flash messages

If you want to store short messages to be displayed right upon the next request, you can use the built-in flash message helpers:

```php
$app->flash()->success('Congratulations!');
// or
$app->flash()->info('Please note!');
// or
$app->flash()->warning('Watch out!');
// or
$app->flash()->danger('Oh snap!');
```

In order to *display* the messages, which you'll probably want to do in your templates, use the methods available on the `$app->flash()` instance for retrieving the data. Alternatively, use one of the built-in partial templates for popular front-end solutions:

 * [Bootstrap 3](http://getbootstrap.com/)

   ```
   {% include 'includes/flash/bootstrap-v3.html' %}
   ```

### Mail

The pre-configured mailing component is always available via the `$app->mail()` helper:

```php
$message = $app->mail()->createMessage();

$message->setSubject('This is the subject');
$message->setFrom([ 'john.doe@example.com' => 'John Doe' ]);
$message->setTo([ 'jane@example.org' => 'Jane Doe' ]);
$message->setBody('Here is the message');

if ($app->mail()->send($message)) {
    // Success
}
else {
    // Failure
}
```

Please refer to the [documentation of the mailing component](http://swiftmailer.org/docs/messages.html) for more information on how to build message instances.

### Obfuscation of IDs

Do you want to prevent your internal IDs from leaking to competitors or attackers when using them in URLs or forms?

Just use the following two methods to encode or decode your IDs, respectively:

```php
$app->ids()->encode(6); // => e.g. "43Vht7"
// and
$app->ids()->decode('43Vht7'); // => e.g. 6
```

If you want to configure the underlying transformation, you simply have to change the following configuration values in your `config/.env` file:

 * `SECURITY_IDS_ALPHABET`
 * `SECURITY_IDS_PRIME`
 * `SECURITY_IDS_INVERSE`
 * `SECURITY_IDS_RANDOM`

Please refer to the [documentation of the ID obfuscation library](https://github.com/delight-im/PHP-IDs) for information on what constitutes proper values and how to generate them.

### Uploading files

Whenever you want to let users upload files to your application, there's a built-in component that does this in a convenient and safe way:

```php
$upload = $app->upload('/uploads/users/' . $userId . '/avatars');
$upload->from('my-input-name');
$upload->save();
```

For more information on how to use the upload handler, including more advanced controls, please refer to the [documentation of the file upload library](https://github.com/delight-im/PHP-FileUpload#usage).

### Serving files

You can serve (static) files to the client from your PHP code, e.g. after performing access control:

```php
$app->serveFile($app->getStoragePath('/photos/314.png'), 'image/png');
```

Most commonly, this will be used to serve images. If certain files aren't public or cannot be accessed directly for other reasons, thus requiring more dynamic behavior, the helper above will be useful.

If you want to offer files for download, however, e.g. documents or videos, there is another helper that is more suitable (see "File downloads").

### File downloads

In order to send strings or files to the client and prompt a download, use one of the following methods:

```php
$app->downloadContent('<html>...</html>', 'Bookmarks-Export.html', 'text/html');
// or
$app->downloadFile($app->getStoragePath('/photos/42.jpg'), 'My Photo.jpg', 'image/jpeg');
```

### Helpers and utilities

The following helpers and utilities are available throughout your application code and in all templates:

 * `$app->url($toPath)` (`app.url($toPath)` in templates)
 * `$app->redirect($toPath)`
 * `$app->setStatus($code)`
 * `$app->setContentType($type)`
 * `$app->flash()` (`app.flash()` in templates)
 * `$app->currentRoute()` (`app.currentRoute()` in templates)
 * `$app->currentUrl()` (`app.currentUrl()` in templates)
 * `$app->getHost()` (`app.getHost()` in templates)
 * `$app->getClientIp()` (`app.getClientIp()` in templates)
 * `$app->isHttp()`
 * `$app->isHttps()`
 * `$app->getProtocol()`
 * `$app->getPort()`
 * `$app->getQueryString()`
 * `$app->getRequestMethod()`

### Security

 * **SQL injections** are prevented if you write to the database using prepared statements only. That is, just follow the examples from the "Database access" section above.
 * **Cross-site scripting (XSS)** protection is available via the convenient escaping and templating features (see sections "HTML escaping" and "Templates" above).
 * **Clickjacking** prevention is built-in and applied automatically.
 * **Content sniffing (MIME sniffing)** protection is built-in and applied automatically.
 * **Cross-site request forgery (CSRF)** mitigation is built-in as well using [same-site cookies](https://github.com/delight-im/PHP-Cookie). For this automatic protection to be sufficient, however, you must prevent XSS attacks and you must not use the HTTP method `GET` for "dangerous" operations.
 * Safe and convenient input validation is built-in as a solid basis for handling untrusted input.

## Contributing

All contributions are welcome! If you wish to contribute, please create an issue first so that your feature, problem or question can be discussed.

## License

This project is licensed under the terms of the [MIT License](https://opensource.org/licenses/MIT).
