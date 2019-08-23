# How to set up a Magento 2 test environment 

1)  once the `bin/dev setup` has been successfully launched, 

2) run `bin/dev composer create-project --repository=https://repo.magento.com/ magento/project-community-edition julien/magento2`

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
    - create a database `magento`

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

                 
`
  

