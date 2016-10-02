![composer logo](https://getcomposer.org/img/logo-composer-transparent2.png)

# Create your private composer repository


##### Prerequisite

 * php >= 5.3.4
 * git


##### Introduction

In this tutorial we're going to see how to create a private composer repository package.\
This will allow us to re-use code easily with composer dependency management.



##### 1.Install composer

first we need to install composer and add composer.phar archive file location to $PATH variable\
more instructions can be found on [offical website](https://getcomposer.org/)

download composer installer
```bash
wget https://getcomposer.org/installer
```

Run installer
```bash
php installer
```

install composer globally (optionnaly rename file with --filename=[your_file_name])

```bash
php composer-setup.php --filename=composer
sudo mv composer /usr/local/bin/composer
```

test if it work by calling the cli tool without argument

```bash
composer
```

you shoud see something like

![composer cli help](https://community.1and1.com/wp-content/uploads/2015/04/Composer.png)

If you got any errors, refer to [composer documentation](https://getcomposer.org/doc/00-intro.md)


##### 2.Create your package code

In my case, i will create a dead-simple config class that will allow me to create php config file containing multidimensionnal array with config value and easily retrieve and edit them inside my application.

first we're going to init the projet with composer 


```bash
composer init
```

Now, let's add some code.
We're going to create the Config class.


```bash
touch Config.php
```


```php

<?php

class Config {


	private  $vars;
	private static $instance;

	private function __construct(){
		$this->vars = require CONFIG_FILE;
	}

	public static function instance(){

		if(!isset(self::$instance))
			self::$instance = new Config;
		return self::$instance;
	}

	private  function get_item_rec($arr, $path){

		if(!isset($arr) || empty($arr))
			return NULL;	

		if(strpos($path, '.') === FALSE)
		{
			if(isset($arr[$path]))
				return ($arr[$path]);
			return NULL;
		}

		$curr_key = substr($path, 0, strpos($path, '.'));
		$next_path = substr($path, strpos($path, '.') + 1);

		return $this->get_item_rec($arr[$curr_key], $next_path);

	}
 	
	private function set_item_rec(&$arr, $path, $value){

		if(!isset($arr) || empty($arr))	return FALSE;

		if(!isset($value)) return FALSE;

		if(strpos($path, '.') === FALSE)
		{
			if(isset($arr[$path]))
				{
					$arr[$path] = $value;
					return TRUE;
				}
			return FALSE;
		}

		$curr_key = substr($path, 0, strpos($path, '.'));
		$next_path = substr($path, strpos($path, '.') + 1);

		return $this->set_item_rec($arr[$curr_key], $next_path, $value);

	}

	public function get($name){

		$item = $this->get_item_rec($this->vars, $name);
		return ($item);
	}

	public  function set($name, $value){
		return $this->set_item_rec($this->vars, $name);
	}


	public  function dump(){
		print_r($this);
	}
	// Config
}


```



Then you will need to tell composer that we want to use PSR-4 autoloading.\
Open composer.json and add the following at the root of the json object :

```json
"autoload": {
        "psr-4": {"YourVendorName\\": ""}
 },
```

my full composer.json file:


```json
{
    "name": "erwan/easy-config",
    "description": "easy configuration PHP class coupled with multi-dimensional array config php file",
    "type": "PHP config class helper",
    "authors": [
        {
            "name": "Erwan Touba",
            "email": "erwan.touba@epitech.eu"
        }
    ],
    "autoload": {
        "psr-4": {"Erwan\\": ""}
    },
    "require": {}
}
```


install composer autoloaded & vendor directory


```bash
composer install
```


##### 2.Create your repository folder with [satis](https://github.com/composer/satis)

First we will install satis, more detailed info [here](https://getcomposer.org/doc/articles/handling-private-packages-with-satis.md#satis)

```bash
cd ~
composer create-project composer/satis --stability=dev --keep-vcs
cd satis
```

Now we're gonna edit satis.json configuration file to setup repository infos


```json
{
    "name": "Config class composer repository",
    "homepage": "https://google.fr",
    "repositories": [
        { "type": "vcs", "url": "https://bitbucket.org/erwan_touba/apitech.git" }
    ],
    "require-all": true
}
```


Then we generate the composer repository : ```php bin/satis build <configuration file> <build-dir> ```\
example:
```bash
php bin/satis build satis.json www
```


