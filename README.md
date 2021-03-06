# Travis CI and WordPress Integration for Composer Packages

[Travis CI](https://travis-ci.org/) WordPress configuration and setup for PHP 5.6 and 5.5 for composer and PHP Unit tests. Read more about [Continuous Integration from Martin Fowler](http://martinfowler.com/articles/continuousIntegration.html). 

- With this setup, WordPress is fully loaded before tests are run.
- Disable `wp_mail` to prevent tests from failing
- Any WordPress hooks will not be implemented - like `add_action` and `add_filter`. You can configure it to be otherwise but for my needs, I have not (no instructions for this).
- This configuration is only setup for [composer](https://getcomposer.org/) package projects ([using composer](https://getcomposer.org/doc/01-basic-usage.md)). Edit the `composer.json` to your specification and needs.

Composer's default is [PHP Unit 4.8](https://phpunit.de/manual/4.8/en/writing-tests-for-phpunit.html) because I need compatibility with PHP 5.5.9. If you do not have these requirements update the `composer.json` file accordingly.

## Travis CI Tests

Use this project as your base configuration and you are ready to go. Otherwise, copy the files and settings you need. 

When you push to GitHub with Travis CI integrated your Tests in the `tests` folder will be run as expected.

Here is the Travis CI documentation you will care most about:
- [Configuring PHP](https://docs.travis-ci.com/user/languages/php/)
- [Including MySQL](https://docs.travis-ci.com/user/database-setup/#MySQL)
- [Default Environment Variables](https://docs.travis-ci.com/user/environment-variables/#Default-Environment-Variables)
- [Speed Up Tests](https://docs.travis-ci.com/user/speeding-up-the-build/#PHP-optimisations)
- [Complex Builds](https://docs.travis-ci.com/user/customizing-the-build#Implementing-Complex-Build-Steps) (don't set `script: ./scripts/run-tests.sh` like in the docs. That will bypass the `phpunit.xml` file and test nothing - only do this if you are sure it is what you want. You will want to use .sh files for provisioning)

## Local Tests
When doing local tests I use [Laravel Homestead](https://github.com/laravel/homestead) `vagrant box add laravel/homestead`. Then from the project directory like `~/Code/project` I run a few command on the VM to download and install WordPress. Finally I use [PHPStorm](https://www.jetbrains.com/phpstorm/) to [run tests with coverage](https://www.jetbrains.com/help/phpstorm/2016.1/running-with-coverage.html).

![PHP Storm Example](https://s3-us-west-2.amazonaws.com/kevindees-github-readme/example-tests.png)

### Database
Create the `wordpress` database on the guest machine Homestead:

```bash
mysql -e "SET NAMES utf8; create database IF NOT EXISTS wordpress;" -uroot
```

### Download WordPress
Download and install WordPress with `wp-cli.phar`:

```bash
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
./wp-cli.phar core download --allow-root --path=wordpress
./wp-cli.phar core config --allow-root --dbname=wordpress --dbuser=homestead --dbpass=secret --dbhost=127.0.0.1 --path=wordpress
./wp-cli.phar core install --allow-root --admin_name=admin --admin_password=admin --admin_email=admin@example.com --url=http://tests.wp --title=WordPress --path=wordpress
```

### Configure Homestead
In the `Homestead.yaml` file I add the wordpress site:

```yaml
sites:
  - map: tests.wp
    to: /home/vagrant/Code/project/wordpress
```

### Update hosts file
In the host computers `hosts` file I point `tests.wp` to the Homestead IP. In my `Homestead.yaml` file it is:

```yaml
ip: "192.168.10.10"
```

So, in my `hosts` file on my Mac using [Hostbuddy](https://clickontyler.com/hostbuddy/) I add a new host and flush the DNS cache on my machine:

```hosts
192.168.10.10 tests.wp
```

### Update WP Config
If you are running tests form the host computer then be sure to edit the `wp-config.php` file to that configuration. Here is what I do because the MySQL port is forwarded on the host computer:

```php
/** MySQL hostname */
if( php_sapi_name() !== 'cli' || gethostname() == 'homestead' ) {
    define('DB_HOST', 'localhost');
} else {
    define('DB_HOST', '127.0.0.1:33060');
}
```

If you are running tests on the guest machine, Homestead, you do not need to do this.

# Circle CI
If you need help with Circle CI this is a great resource http://blog.wppusher.com/continuous-integration-with-wordpress-and-circleci/
