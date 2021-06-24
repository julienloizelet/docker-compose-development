> Here is a list of personal notes. 

# Before starting :

Suppose that you have clone the `docker-compose-development` in `/some/path/to/the/clone`.

1. Add this to your aliases : 

      alias dev='/some/path/to/the/clone/bin/dev'
2. Create a `docker-custom.yml` file based on the `docker-custom.yml.dist` in order to add a 
`phpmyadmin` container that we will use to create an empty database.

# How to set up a Magento 2 test environment 

1)  Launch the `dev setup` process. (say "yes" for elastic search as it is mandatory for 2.4)

2) Install Magento 2 in the workspace folder by running :

     `dev composer create-project --repository=https://repo.magento.com/ magento/project-community-edition some-name/project-name`

 Beware that the folder determines the final url as it is said here :

> You will notice that this has a 1-on-1 relation to the hostname provided in your hostfile:
> `workspace/test/project/htdocs` => `https://test.project.localhost/`


Example : `dev composer create-project --repository=https://repo.magento.com/ magento/project-community-edition m24/magento2 2.4.0`
wil install Magento 2.4.0 and url will be `http://m24.magento2.localhost/`
(php 7.3 is mandatory for a 2.4 installation, so create a file `.php73` on the workspace directory.)

If you get an error like `PHP Fatal error:  Allowed memory size ...`, use `dev php -d memory_limit=-1 
/usr/local/bin/composer ...` instead of `dev composer ...`

@see https://github.com/JeroenBoersma/docker-compose-development/issues/124#issue-601933753

3) Files Permission :
     - `cd workspace/some-name/project-name`
     - `find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +`
     - `find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +`
     - `sudo chown -R :www-data . `
     - `chmod u+x bin/magento`
4)  Create database :
    - Connect to the phpmyadmin `localhost:8081` with the login root and the password that you will find in the `.env` file.
    - create a database `magento2`

5) Install in command line :

     - From `workspace/some-name/project-name` folder , launch `dev php bin/magento setup:install \
                 --base-url=https://some-name.project-name.localhost \
                 --db-host=db \
                 --db-name=magento2 \
                 --db-user=root \
                 --db-password=THE-ROOT-PASSWORD-IN-ENV-FILE \
                 --backend-frontname=admin \
                 --admin-firstname=admin \
                 --admin-lastname=admin \
                 --admin-email=admin@admin.com \
                 --admin-user=admin \
                 --admin-password=admin123 \
                 --language=en_US \
                 --currency=USD \
                 --timezone=America/Chicago \
                 --use-rewrites=1 \
                 --elasticsearch-host=elasticsearch
                 
6) Browse to `https://some-name.project.localhost`

    - Enjoy OR                
                 
    - known issue 1 : If css is visibly broken and the css path contains version number, you should try to disable it in mysql :
`INSERT INTO core_config_data (path, value) VALUES ('dev/static/sign', 0)
 ON DUPLICATE KEY UPDATE value = 0;`

# Use git for a Magento 2 project from scratch

 - Go to root directory of Magento 2
 - run `git init`
 - run `git add .`
 - run `git commit -m "Initial Commit Magento 2 installation"`
         
# Working with Varnish and Magento 2

see https://github.com/JeroenBoersma/docker-compose-development/issues/102

     bin/magento varnish:vcl:generate --backend-host=web --backend-port=80 > varnish.vcl
     # added lately and updated today to remove the probe
     dev varnish-vcl-import varnish.vcl
     
     # I've added disable too
     dev varnish-vcl-disable
     # renames file from varnish.vcl -> varnish.vcl.off
     
     au cas où:
     https://github.com/JeroenBoersma/docker-compose-development/issues/65
     
     
To flush the Varnish cache (https://github.com/JeroenBoersma/docker-compose-development/issues/113):

     dev varnishadm ban "req.url ~ /"
     
# Working with Mage2TV clean-cache

see https://github.com/JeroenBoersma/docker-compose-development/issues/92


     # Install globally
     dev composer global require mage2tv/magento-cache-clean
     ## Navigate to your Magento2 base-dir
     # In your project create the var/cache-clean-config.json
     dev php /data/.composer/vendor/mage2tv/magento-cache-clean/bin/generate-cache-clean-config.php
     # Run to start watching
     dev node  /data/.composer/vendor/mage2tv/magento-cache-clean/bin/cache-clean.js --watch
     
You should add some alias in your `.bash_aliases` for example :

    cache-clean.js () {
       dev php /data/.composer/vendor/mage2tv/magento-cache-clean/bin/generate-cache-clean-config.php
       dev node /data/.composer/vendor/mage2tv/magento-cache-clean/bin/cache-clean.js "$@"
    }
With this alias, you just have to run `cache-clean.js --watch` in your Magento 2 folder.
         

# How to manage your own extension separately

## Option 1 (lazy one)

Clone your module in `app/code/VendorName/ModuleName`


## Option 2 (composer way)


Suppose that you want to work on a module called `my-module-name` and that your local sources for this module
are in `some/local/path/to/my-module`. We suppose too that your Magento 2 project is in a subfolder of `workspace` called `some-name/project`.


We will use composer :
- First, shut down your dev environment : `dev down`
- Create a `my-own-modules/my-module-name` folder in your Magento 2 project and add `/my-own-modules` to the `.gitignore` file.
- add a volume in `docker-custom.yml` to link  `some-name/project/my-own-modules/my-module-name` folder and `some/local/path/to/my-module`.
You can use the `docker-custom.yml.dist` file as a model.
- Up your dev environment :  `dev up`
- Add the path to your module as a repository path :

      dev composer config repositories.some-name-for-this-module-path-repo path ./my-own-modules/my-module-name/ 
      
- Then, supposing that the package name of your module (defined in its `composer.json`) is `vendorName/moduleName` you have to run 

      dev composer require vendorName/moduleName
          
and composer will install it by symlinking from `some-name/project/my-own-modules/my-module-name`     

So now you can modify your code in `some/local/path/to/my-module` ànd your modifications will be instantanly 
visible in your docker project.

## Dockerized Magento 2 and Phpstorm


To generate xsd schema mapping, go to the path of Magento 2 installation directory 

    cd /some/path/to/the/clone/workspace/m23/magento2

And run the following commmand :

    dev php bin/magento dev:urn-catalog:generate .idea/misc.xml 

Maybe you should have to delete first the `.idea/misc.xml`.    


## Marketplace validation : 

### copy/paste validation

To run copy paste detector on the full project :

    dev php -d memory_limit=-1 /data/m23/magento2/./vendor/bin/phpcpd --log-pmd '/data/m23/magento2/dev/tests/static/report/phpcpd_report.xml' --names-exclude "*Test.php"  --min-lines 13  --exclude 'generated/code' --exclude 'dev' /data/m23/magento2 
`



## Sample data :

- go to the path of Magento 2 installation directory
- `dev php -d memory_limit=-1 bin/magento sampledata:deploy`
- `dev php bin/magento setup:upgrade`
- wait for ages


## Elastic search

- Since 2.4, if elastic search is version 6, you have to modify catalog search engine (version AND host)

then (if you want to see products on front ...)

- magento->admin->stores->configuration->catalog->catalog
  search->Search Engine->... and selected "Elasticsearch 6.x (Deprecated)".
- `dev php bin/magento cache:flush`
- `dev php bin/magento indexer:reindex`



## Magento 2.4 2FA (2 factor authorization)
To disable this module : 

```
dev php bin/magento module:disable Magento_TwoFactorAuth
dev php bin/magento cache:flush
```

## Mailhog
http://mail.localhost/
