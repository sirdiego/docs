# Getting Started

First, let's [install Deployer](installation.md). Run the following commands in a terminal:

```sh
curl -LO https://deployer.org/deployer.phar
mv deployer.phar /usr/local/bin/dep
chmod +x /usr/local/bin/dep
```

Now you can use Deployer via the `dep` command. 
Open up a terminal in your project directory and run the following command:

```sh
dep init
```

This command will create `deploy.php` file in the current directory. It is called *recipe* and contains the configuration and tasks for your deployment.
By default all recipes extend the [common](https://github.com/deployphp/deployer/blob/master/recipe/common.php) recipe. 


Define your task really simple:
 
```php
task('test', function () {
    writeln('Hello world');
});
```

To run that task, run the following command:

```sh
dep test
```

And you should get this output:

```text
➤ Executing task test
Hello world
✔ Ok
```

Now lets create a task which will run commands on the remote server. For that we must configure a server. 
Your created `deploy.php` file should contain a `server` declaration line like this:
 
```php
server('production', 'domain.com')
    ->user('username')
    ->identityFile()
    ->set('deploy_path', '/var/www/domain.com');
```

You can find more about all server configurations [here](servers.md). Now let's define a task which will output the `pwd` command from a remote server:
 
```php
task('pwd', function () {
    $result = run('pwd');
    writeln("Current dir: $result");
});
```

Run it like `dep pwd`, and you will get this:

```text
➤ Executing task pwd
Current dir: /var/www/domain.com
✔ Ok
```

Now lets prepare for our first deployment. You need to configure such parameters as `repository`, `shared_files` and others:
   
```php
set('repository', 'git@domain.com:username/repository.git');
set('shared_files', [...]);
```

You can get a parameters value in tasks using the `get` function. 
Also you can override each config for each server:

```php
server('production', 'domain.com')
    ...
    ->set('shared_files', [...]);
```

More about configuration can be found [here](configuration.md).


Now let's deploy our application:
 
```sh
dep deploy
```

To see what exactly is happening you can increase verbosity of the output with the `--verbose` option: 

* `-v`  for normal output,
* `-vv`  for more verbose output,
* `-vvv`  for debug.
 
On first Deployer will create the following directories on the server:

* `releases`  contains releases dirs,
* `shared` contains shared files and dirs,
* `current` symlink to current release.

Configure your server to serve your public directory from the `current` symlink.

> Note that deployer use [ACL](https://en.wikipedia.org/wiki/Access_control_list) by default for setting up permissions.
> You can change this behavior with the `writable_mode` config.    

By default deployer keeps the last 5 releases, but you can increase this number by modifying the parameter `keep_releases`:
 
```php
set('keep_releases', 10);
```

If something went wrong during the deployment process, or something is wrong with your new release, 
simply run the following command to rollback to the previous working release:

```sh
dep rollback
```

You may want to run some task before/after another tasks. Configuring this is really simple!

Let's reload php-fpm after the `deploy` task finished:

```php
task('reload:php-fpm', function () {
    run('sudo /usr/sbin/service php5-fpm reload');
});

after('deploy', 'reload:php-fpm');
```

Read more about [configuring](configuration.md) deploy. 
