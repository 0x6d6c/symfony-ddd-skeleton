# Symfony 4 skeleton with Virtual Kernel for multiple applications

### Name-based Virtual Kernel

The term "Virtual Kernel" refers to the practice of running more than one application (such as `api.example.com` and `admin.example.com`) on a single project repository. Virtual kernels are "name-based", meaning that you have multiple kernel names running on each application. The fact that they are running on the same physical project repository is not apparent to the end user.

In short, each kernel name corresponds to one application.

### Application-based Configuration

First, you need replicate the structure of one application for `config`, `src`, `var` directories and create additionally a `common` config directory for shared bundles and configuration.

Making use of the `Kernel::$name` property you can stand out the application to run with dedicated project files.

### Keeping one entry point for all applications

    ├── public/
    │   └── index.php

Following the same filosofy of Symfony 4, whereas environment variables decides which development environment and debug mode should be used to run your application, you could use `APP_NAME` environment variable to set the application to execute. 
For now, you can play with your applications and PHP's built-in Web server prefixing the new application environment variable:

    $ APP_NAME=app php -S 127.0.0.1:8000 -t public
    $ APP_NAME=api php -S 127.0.0.1:8001 -t public   

### Executing commands per application

    ├── bin/
    │   └── console.php

Use `--kernel` option to run any command different to default one (`app`).

    $ bin/console about -k=api
    
Or if you prefer ignore the previous option and use environment variables:

    $ export APP_NAME=api
    $ bin/console about                         # api application
    $ bin/console debug:router                  # api application
    $
    $ APP_NAME=api bin/console debug:router     # api application

Also you can configure the default `APP_NAME` environment variable in your `.env` file. 

### Running tests per application

    ├── tests/
    │   ├── App/
    │   │   └── AppWebTestCase.php
    │   ├── Api/

The `tests` directory is pretty similar to the `src` directory, just update your `composer.json` to map each directory `tests/<Name>/` with its PSR-4 namespace:

    "autoload-dev": {
        "psr-4": {
            "App\\Tests\\": "tests/App/",
            "Api\\Tests\\": "tests/Api/"
        }
    },

Again, run `composer dump-autoload` to re-generate the autoload config.
    
Here, you might need create a `<Name>WebTestCase` class per application in order to execute all tests together.

### Production and vhosts

Set the environment variable `APP_NAME` for each vhost config in your production server and development machine if prefer:

    <VirtualHost company.com:80>
        # ...
        
        SetEnv APP_NAME app
        
        # ...
    </VirtualHost>

    <VirtualHost api.company.com:80>
        # ...
        
        SetEnv APP_NAME api
        
        # ...
    </VirtualHost>
 
### Adding more applications to the project

With three simple steps you should be able to add new vKernel/applications to the current project:

 1. Add to `config`, `src` and `tests` directories a new folder with the `<name>` of the application and its content.
 2. Add to `config/<name>/` dir at least the `bundles.php` file.
 3. Add to `composer.json` autoload/autoload-dev sections the new PSR-4 namespaces for `src/<Name>/` and `tests/<Name>` directories and update the autoload config file.

Check the new application running `bin/console about -k=<name>`.

License
-------

This software is published under the [MIT License](LICENSE)
