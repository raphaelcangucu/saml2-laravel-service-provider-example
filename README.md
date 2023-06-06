
## About this Repo

This repo is a demo repository to show how to use a SAML2 Service provider using the lib `24slides/laravel-saml2` in a Laravel Application.

We need to change just a small thing at the library to make it work on the SAIL Laravel environment.

## Steps to reproduce this demo

### Step 1. Install dependencies

In composer.json file add the repository

    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/raphaelcangucu/laravel-saml2.git"
        }
    ]

Add the package using composer

    composer require 24slides/laravel-saml2:dev-fixSailPort


If you are using Laravel 5.5 and higher, the service provider will be automatically registered.

For older versions, you have to add the service provider and alias to your `config/app.php`:

```php
'providers' => [
    ...
    Slides\Saml2\ServiceProvider::class,
]

'alias' => [
    ...
    'Saml2' => Slides\Saml2\Facades\Auth::class,
]
```


### Step 2. Publish the configuration file.

```
php artisan vendor:publish --provider="Slides\Saml2\ServiceProvider"
```

### Step 3. Run migrations

```
php artisan migrate
```

### Step 4. Configuring a new tenant -  Identity Providers (IdPs)


To distinguish between identity providers there is an entity called Tenant that represent each IdP.

When request comes to an application, the middleware parses UUID and resolves the Tenant.

You can easily manage tenants using the following console commands:

- `artisan saml2:create-tenant`
- `artisan saml2:update-tenant`
- `artisan saml2:delete-tenant`
- `artisan saml2:restore-tenant`
- `artisan saml2:list-tenants`
- `artisan saml2:tenant-credentials`

> To learn their options, run a command with `-h` parameter.

Each Tenant has the following attributes:

- **UUID** — a unique identifier that allows to resolve a tenannt and configure SP correspondingly
- **Key** — a custom key to use for application needs
- **Entity ID** — [Identity Provider Entity ID](https://spaces.at.internet2.edu/display/InCFederation/Entity+IDs)
- **Login URL** — Identity Provider Single Sign On URL
- **Logout URL** — Identity Provider Logout URL
- **x509 certificate** — The certificate provided by Identity Provider in **base64** format
- **Metadata** — Custom parameters for your application needs

##### Step 4.1 Example for a local IDP

```bash
    sail artisan saml2:create-tenant --entityId=http://localhost/saml/metadata --loginUrl=http://localhost/auth/login --logoutUrl=http://localhost/auth/logout -x509cert=ASKFORTHEIDPCERTIFICATE

```


##### Step 4.2  See the IDP / Tenant credentials

```bash
    sail artisan saml2:tenant-credentials 1
```

It will return something like this

    Identifier (Entity ID): http://localhost:8888/saml2/5d32bb4a-88fb-4d0b-890d-a785f3c64162/metadata
    Reply URL (Assertion Consumer Service URL): http://localhost:8888/saml2/5d32bb4a-88fb-4d0b-890d-a785f3c64162/acs
    Sign on URL: http://localhost:8888/saml2/5d32bb4a-88fb-4d0b-890d-a785f3c64162/login
    Logout URL: http://localhost:8888/saml2/5d32bb4a-88fb-4d0b-890d-a785f3c64162/logout
    Relay State:  (optional)

#### Step 5. Testing the SAML2 Login

    Open this a URL like this: `http://localhost:8888/saml2/5d32bb4a-88fb-4d0b-890d-a785f3c64162/login?returnTo=https://communitysite.dev.com`

    This will redirect to the IDP and after the user Login it will return to the returnTo query param.