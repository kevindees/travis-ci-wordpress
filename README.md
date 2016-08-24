# Travis CI and WordPress Integration

Travis CI WordPress Configuration and Setup for PHP 5.6 and 5.5. With this setup WordPress is fully loaded and run before tests are run. Any WordPress hooks will not be implemeneted - like `add_action` and `add_filter`. You can configure it to be otherwise but for my needs I have not.

Also, note that this configuration is only setup for [composer](https://getcomposer.org/) projects ([using composer](https://getcomposer.org/doc/01-basic-usage.md)). Edit the `composer.json` to your specification and needs.

## Local Tests
When doing local tests I use [Laravel Homestead](https://github.com/laravel/homestead) `vagrant box add laravel/homestead`.

Then from the project direcroty like `~/Code/project` I run a few command on the VM to download and install WordPress:

### Database
Create the `wordpress` Database:

```bash
mysql -e "SET NAMES utf8; create database IF NOT EXISTS wordpress;" -uroot
```

### Download WordPress
Download and install WordPress with `wp-cli.phar`:

```bash
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
./wp-cli.phar core download --allow-root --path=wordpress
./wp-cli.phar core config --allow-root --dbname=wordpress --dbuser=travis --dbhost=127.0.0.1 --path=wordpress
./wp-cli.phar core install --allow-root --admin_name=admin --admin_password=admin --admin_email=admin@example.com --url=http://127.0.0.1 --title=WordPress --path=wordpress
```

### Configure Homestead
In the `Homestead.yaml` file I add the wordpress site:

```yaml
sites:
  - map: tests.wp
    to: /home/vagrant/Code/project/wordpress
```

### Update hosts file
In the host computers `hosts` file I point `tests.wp` to the Homestead IP. Im my `Homestead.yaml` file it is:

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
if( isset($_SERVER['SERVER_NAME']) ) {
    define('DB_HOST', '127.0.0.1');
} else {
    define('DB_HOST', '127.0.0.1:33060'); // Port forwarded by Homestead by default
}
```

If you are running tests from the guest machine, Homestead, you do not need to do this.

# Circle CI
If you need help with Circle CI this is a greate resource http://blog.wppusher.com/continuous-integration-with-wordpress-and-circleci/
