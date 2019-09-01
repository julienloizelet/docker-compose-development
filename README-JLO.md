# How to set up a Magento 2 test environment 

1)  once the `bin/dev setup` has been successfully launched, 

2) Install Magento 2 in the workspace folder by running :

     `bin/dev composer create-project --repository=https://repo.magento.com/ magento/project-community-edition julien/magento2`

 Beware that the folder determines the final url as it is said here :

> You will notice that this has a 1-on-1 relation to the hostname provided in your hostfile:
> `workspace/test/project/htdocs` => `https://test.project.localhost/`

3) Files Permission :
     - `cd workspace/julien/magento2`
     - `find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +`
     - `find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +`
     - `sudo chown -R :www-data . `
     - `chmod u+x bin/magento`
4)  Create database :
    - Connect to the phpmyadmin `localhost:8081` with the login root and the password that you will find in the `.env` file.
    - create a database `magento2`

5) Install in command line :

     - From `workspace/julien/magento2` folder , launch `dev php bin/magento setup:install \
                 --base-url=https://julien.magento2.localhost \
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
                 --use-rewrites=1
                 
6) Browse to `https://julien.magento2.localhost`

    - Enjoy OR                
                 
    - If css is visibly broken and the css path contains version number, you should try to disable it in mysql :
`INSERT INTO core_config_data (path, value) VALUES ('dev/static/sign', 0)
 ON DUPLICATE KEY UPDATE value = 0;`


# Use git for a Magento 2 project from scratch

 - Go to root directory of Magento 2
 - run `git init`
 - run `git add .`
 - run `git commit -m "Initial Commit Magento 2 installation"`

                
`
# Working with Varnish and Magento 2

see https://github.com/JeroenBoersma/docker-compose-development/issues/102

     bin/magento varnish:vcl:generate --backend-host=web --backend-port=80 > varnish.vcl
     # added lately and updated today to remove the probe
     dev varnish-vcl-import varnish.vcl
     
     # I've added disable too
     dev varnish-vcl-disable
     # renames file from varnish.vcl -> varnish.vcl.off
     
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
         

# How to manage your own extension



## Working as a merchant developer

Just put your code in `app/code`

## Working as an extension developer

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