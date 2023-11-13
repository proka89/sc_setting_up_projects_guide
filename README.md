## StuntCoders guide for setting up new projects in local environment

This guide aims to provide a step-by-step process for setting up WordPress and Magento projects in a local environment, utilizing Docker or LocalWP.

### Prerequisites

- **Docker Desktop**: Ensure Docker Desktop is installed on your machine. [Download Docker Desktop](https://www.docker.com/products/docker-desktop).
- **LocalWP**: For WordPress-specific projects, LocalWP is a more user-friendly option. [Download LocalWP](https://localwp.com/).
- **Basic Command Line Knowledge**: Familiarity with terminal or command prompt commands is required.
- **Code Editor**: Use any code editor like Visual Studio Code, Sublime Text, etc.

### Table of Contents

- [Setting up projects via Docker](https://github.com/proka89/sc_setting_up_projects_guide#setting-up-projects-via-docker)
  - [Setting up WordPress projects via Docker](https://github.com/proka89/sc_setting_up_projects_guide#setting-up-wordpress-projects-via-docker)
  - [Useful docker commands](https://github.com/proka89/sc_setting_up_projects_guide#useful-docker-commands)
    - [SSH into web container](https://github.com/proka89/sc_setting_up_projects_guide#ssh-into-web-container)
    - [Update a plugin or theme](https://github.com/proka89/sc_setting_up_projects_guide#update-a-plugin-or-theme)
    - [Activate or deactivate a plugin or a theme](https://github.com/proka89/sc_setting_up_projects_guide#activate-or-deactivate-a-plugin-or-a-theme)
    - [Import or Dump a database](https://github.com/proka89/sc_setting_up_projects_guide#import-or-dump-a-database)
  - [Troubleshooting Docker WordPress setup](https://github.com/proka89/sc_setting_up_projects_guide#troubleshooting-docker-wordpress-setup)
    - [Importing a Database](https://github.com/proka89/sc_setting_up_projects_guide#importing-a-database)
    - [Exporting a Database](https://github.com/proka89/sc_setting_up_projects_guide#exporting-a-database)
    - [Checking WordPress Logs](https://github.com/proka89/sc_setting_up_projects_guide#checking-wordpress-logs)
    - [Restarting Containers](https://github.com/proka89/sc_setting_up_projects_guide#restarting-containers)
    - [Accessing MySQL Shell](https://github.com/proka89/sc_setting_up_projects_guide#accessing-mysql-shell)
    - [Flushing WordPress Cache](https://github.com/proka89/sc_setting_up_projects_guide#flushing-wordpress-cache)
    - [Troubleshooting WordPress Permissions](https://github.com/proka89/sc_setting_up_projects_guide#troubleshooting-wordpress-permissions)
  - [Setting up Magento 1 projects via Docker](https://github.com/proka89/sc_setting_up_projects_guide#setting-up-magento-1-projects-via-docker)
  - [Troubleshooting Docker Magento 1 setup](https://github.com/proka89/sc_setting_up_projects_guide#troubleshooting-docker-magento-1-setup)
    - [Checking Container Logs](https://github.com/proka89/sc_setting_up_projects_guide#checking-container-logs)
    - [Accessing the Container Shell](https://github.com/proka89/sc_setting_up_projects_guide#accessing-the-container-shell)
    - [Inspecting Containers](https://github.com/proka89/sc_setting_up_projects_guide#inspecting-containers)
    - [Managing Magento Cache](https://github.com/proka89/sc_setting_up_projects_guide#managing-magento-cache)
    - [Database Connectivity](https://github.com/proka89/sc_setting_up_projects_guide#database-connectivity)
    - [Restarting Containers](https://github.com/proka89/sc_setting_up_projects_guide#restarting-containers-1)
    - [Checking File Permissions](https://github.com/proka89/sc_setting_up_projects_guide#checking-file-permissions)

## Setting up projects via Docker

If you are not already familiar with Docker, it's recommended to start with the [Docker Getting Started documentation](https://docs.docker.com/get-started/). Key areas to pay special attention to include:

- **Overview of Docker concepts**: Understanding the fundamental concepts such as images, containers, and Dockerfiles.
- **Setting up your Docker environment**: Learn how to install and configure Docker on your specific operating system.
- **Building and running your first container**: This section provides practical experience in building and running Docker containers, which is crucial for setting up your development environment.
- **Docker Compose**: Understanding Docker Compose is essential for managing multi-container applications, like those you'll be creating for WordPress and Magento.

In the instructions below, we will be using variable such as `project_name` or `database_name`, these should be replaced by the actual project or database names. `database_name` variable will be the name of the demo database dump, which can be found in the root of the project. For example this could look something like `pickles.sql.zip`, in such case you would be using `pickles` instead of `database_name` in the commands below.

**Have in mind that these setups are based on alrady existing preconfigured Docker files. Always make sure to check project readme file for more specific instructions.**

### Setting up WordPress projects via Docker

- Clone project from the GitHub repo
- Unzip demo database dump:

  ```bash
  unzip -o database_name.sql.zip > database_name.sql
  rm -r __MACOSX
  ```

- All database credentials and wp-config.php data are stored in the `.env` file. Run:

  ```bash
  cp .env.example .env
  ```

- Go into project root directory and run the containers with:

  ```bash
  docker-compose build && docker-compose up -d
  ```

- If you haven't already, you will have to add an entry to hosts file like this:

  ```bash
  127.0.0.1 project_name.local
  ::1       project_name.local
  ```

An additional container is included that lets you use the `wp-cli` app without having to install it on your local machine.

You can now visit your site(http://project_name.local), or go to phpMyAdmin dashboard(http://project_name.local:8000)

### Useful docker commands

To use these commands we will first create aliases for them. Creating or adding an alias on a Mac involves modifying your shell's configuration file. The process can slightly vary depending on the shell you are using (e.g., Bash or Zsh, with Zsh being the default on newer macOS versions). If you are unsure which shell you are using, type `echo $SHELL` in your Terminal.

Next step vould be to open the shell from the terminal with editors such as `vim` or `nano`. In this case we will use `vim`:

- Type `vim ~/.zshrc` in the Terminal, to open the `.zshrc` file. Then press `i` to enter the `INSERT` mode
- We will be adding the aliases and two Docker related functions from the snipet below

  ```bash
  alias docker-compose="docker compose"
  alias dcb="docker-compose build"
  alias dcu="docker-compose up"
  alias dcd="docker-compose down"
  alias dcl="docker-compose exec --user $(id -u):$(id -g) web /bin/bash"
  alias dcdebug="docker-compose down && docker-compose run --service-ports web"
  alias dclc="dockerlogin"
  alias dwp="docker_wpcli"

  docker_wpcli() {
    declare ARGS

    # if CWD is not WordPress project, exit
    if ! [[ -d ./wp-includes && -d ./wp-admin && -d ./wp-content ]]; then
      echo 'Not in WordPress project.'
      return 1
    fi

    # If WordPress and DB are not running, exit
    CWD=${PWD##*/}
    CONTAINERS_ACTIVE=$(docker ps | grep $CWD |  wc -l)

    if [ $CONTAINERS_ACTIVE -lt 2 ]; then
      echo 'Start WordPress and Database containers first.'
      return 1
    fi

    # if container already exists, recreate it
    COUNT=$(docker ps -a | grep wp_cli | wc -l)

    if [ $COUNT -gt 0 ]; then
      echo Recreating wp_cli container..
      docker rm wp_cli
    fi

    # If no arguments are passed, display wp info
    if [ $# -eq 0 ]; then
      ARGS="--info"
    else
      ARGS=("$@")
    fi

    docker-compose run --name wp_cli --rm wp_cli "${ARGS[@]}"
  }

  dockerlogin() {
    if [ -n "$1" ]
    then
      docker-compose exec --user $(id -u):$(id -g) $1 /bin/bash
    fi
  }
  ```

- Press `esc` on the keyboard to exit the `INSERT` mode, and then type `:wq` to save the changes and exit from the file.

After adding the alias and reloading the shell configuration, you can test it by typing the alias in the Terminal. For example, if you added `alias dps='docker ps'`, just type `dps` and it should execute `docker ps`.

#### SSH into web container

```bash
dcl # Calls dockerlogin function
```

Or if you don't have an alias set:

```bash
docker-compose exec <service> /bin/bash # services: web, db, phpmyadmin, etc..
```

#### Update a plugin or theme

```bash
dwp plugin list
dwp plugin update <plugin> # paste plugin name from the list

```

Or if you don't have an alias set:

```bash
docker-compose run wp_cli --rm --name wp_cli plugin/theme update
```

#### Activate or deactivate a plugin or a theme

```bash
dwp theme activate <theme>
dwp theme deactivate <theme>
```

#### Import or Dump a database

```bash
dwp db export dump.sql
dwp db import dump.sql
```

For more information about `wp-cli`, read [WP-CLI](https://developer.wordpress.org/cli/commands/) documentation.

### Troubleshooting Docker WordPress setup

Docker commands and scenarios that could be useful for someone managing WordPress environments with Docker. These examples cover various common tasks and troubleshooting scenarios.

#### Importing a Database

If you need to import a database into a WordPress container, you can use the following commands:

```bash
# List running containers to find the database container ID
docker ps

# Import a database using the container ID
docker exec -i CONTAINER_ID mysql -uUSERNAME -pPASSWORD DATABASE_NAME < your_database.sql

# Or, using docker-compose
docker-compose exec -T DB_SERVICE_NAME mysql -uUSERNAME -pPASSWORD DATABASE_NAME < your_database.sql
```

#### Exporting a Database

To export a database from your Docker container, use these commands:

```bash
# Run Docker container in detached mode
docker-compose up -d

# List containers to find the database container ID
docker ps

# Export the database
docker exec CONTAINER_ID /usr/bin/mysqldump -u root --password=PASSWORD DATABASE_NAME > exported_db.sql

# Or, using docker-compose
docker-compose exec -T DB_SERVICE_NAME /usr/bin/mysqldump -u root --password=PASSWORD DATABASE_NAME > exported_db.sql
```

#### Checking WordPress Logs

To check the logs of your WordPress container:

```bash
# Using Docker
docker logs CONTAINER_ID

# Using docker-compose
docker-compose logs SERVICE_NAME
```

#### Restarting Containers

If you need to restart your containers:

```bash
# Restart all containers using Docker Compose
docker-compose restart

# Restart a specific service
docker-compose restart SERVICE_NAME
```

#### Accessing MySQL Shell

To access the MySQL shell within your Docker container:

```bash
# Using Docker
docker exec -it CONTAINER_ID mysql -uUSERNAME -pPASSWORD

# Using docker-compose
docker-compose exec DB_SERVICE_NAME mysql -uUSERNAME -pPASSWORD
```

#### Flushing WordPress Cache

To flush the cache in WordPress:

```bash
# Flush cache using WP CLI
docker exec CONTAINER_ID wp cache flush
```

#### Troubleshooting WordPress Permissions

To fix file permissions in a WordPress container:

```bash
# Adjust permissions for the WordPress directory
docker exec CONTAINER_ID chown -R www-data:www-data /var/www/html
```

Replace `CONTAINER_ID`, `DB_SERVICE_NAME`, `USERNAME`, `PASSWORD`, `DATABASE_NAME`, and `your_database.sql` with your actual container ID, service name, database username, password, database name, and SQL file name. These commands assume that you have a running Docker environment with Docker Compose and that your `docker-compose.yml` file is set up correctly for your WordPress and database services.

### Setting up Magento 1 projects via Docker

- Clone the project from the GitHub repo
- If you haven't already, you will have to add an entry to hosts file. To open the file run:

  ```bash
  sudo vim /etc/hosts
  ```

  At the end of the file add:

  ```bash
  127.0.1.1 project_name.local
  ```

- Unzip demo database dump:

  ```bash
  unzip -o database_name.sql.zip > database_name.sql
  ```

- Go into project root directory and run the containers with:

  ```bash
  docker-compose up -d
  ```

- Update local.xml(`app/etc/local.xml`) file with these credentials:

  ```xml
  <username><![CDATA[magento]]></username>
  <password><![CDATA[secret]]></password>
  <dbname><![CDATA[magento]]></dbname>
  ```

- If you have any issues or errors when loading `project_name.local`, delete the cache located in the `/var/cache` folder in project root. Build always takes time so be patient! Magento cache in the admin is probably disabled, which will slow down projects in the local environment.

### Troubleshooting Docker Magento 1 setup

Docker commands and scenarios that could be useful for someone managing Magento 1 environments with Docker. These examples cover various common tasks and troubleshooting scenarios.

#### Checking Container Logs

Logs are often the first place to look when something goes wrong.

```bash
# View logs of a specific container
docker logs CONTAINER_ID

# Follow the logs in real-time
docker logs -f CONTAINER_ID
```

#### Accessing the Container Shell

Accessing the shell of your container can be invaluable for direct inspection and troubleshooting.

```bash
# Access shell of a specific container
docker exec -it CONTAINER_ID /bin/bash
```

#### Inspecting Containers

Docker's inspect command provides detailed information about a container's configuration and state.

```bash
# Inspect a specific container
docker inspect CONTAINER_ID
```

#### Managing Magento Cache

Clearing the Magento cache can resolve many issues.

```bash
# Clear Magento cache from the container
docker exec CONTAINER_ID rm -rf /var/www/html/var/cache/*
```

#### Database Connectivity

Ensure that your application can connect to the database.

```bash
# Access MySQL shell
docker exec -it DB_CONTAINER_ID mysql -uUSERNAME -pPASSWORD

# Check if you can connect to the database from your Magento container
docker exec CONTAINER_ID mysql -hDB_HOST -uUSERNAME -pPASSWORD -e "SHOW DATABASES;"
```

#### Restarting Containers

Sometimes, a simple restart can fix a problem.

```bash
# Restart a specific container
docker restart CONTAINER_ID

# Restart all containers in docker-compose
docker-compose restart
```

#### Checking File Permissions

Incorrect file permissions can cause errors, especially in a Magento environment.

```bash
# Check file permissions
docker exec CONTAINER_ID ls -l /var/www/html

# Correct file permissions
docker exec CONTAINER_ID chmod -R 755 /var/www/html
docker exec CONTAINER_ID chown -R www-data:www-data /var/www/html
```

Replace `CONTAINER_ID`, `DB_SERVICE_NAME`, `USERNAME`, `PASSWORD`, `DATABASE_NAME`, and `your_database.sql` with your actual container ID, service name, database username, password, database name, and SQL file name. These commands assume that you have a running Docker environment with Docker Compose and that your `docker-compose.yml` file is set up correctly for your WordPress and database services.

### Setting up Magento 2 projects via Docker

- Clone the project from the GitHub repo
- If you haven't already, you will have to add an entry to hosts file. To open the file run:

  ```bash
  sudo vim /etc/hosts
  ```

  At the end of the file add:

  ```bash
  127.0.0.1 project_name.local
  ::1 project_name.local
  ```

- Rename `package.json.sample` to `package.json`. You can do this with:

  ```bash
  cp package.json.sample package.json
  ```

- Rename `app/etc/env_local.php` to `app/etc/env.php`. You can do this with:

  ```bash
  cp app/etc/env_local.php app/etc/env.php
  ```

- If there is a WordPress in combination with Magento
    - Rename `wp/wp-config_local.php` to `wp/wp-config.php`
    - Change DB_HOST (wp/wp-config) & host (app/etc/env.php) IP address if different then `172.19.0.2` is assigned to your container. To check IP addresses of containers run the following command:

      ```bash
      docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)
      ```

- Go into project root directory and run the containers with:

  ```bash
  docker-compose up -d
  ```

- Unzip project demo database dumb (`database_name.sql.zip`) and import the database once container is set:

  ```bash
  unzip -o database_name.sql.zip > database_name.sql
  ```

  ```bash
  docker exec -i DB_CONTAINER mysql -uUSER -pPASSWORD magento < database_name.sql
  ```

You can run `docker ps` to see containers info such as `DB_CONTAINER` name.

- Once done, you can visit the site at `project_name.local`

### Troubleshooting Docker Magento 2 setup

#### No products is showing on frontend

Run reindexer from inside the container `bin/magento indexer:reindex`

#### Cached page, no new changes are showing

Clear the cache `bin/magento cache:clean`

Sometimes, you might also need to clear the generated content: `rm -rf var/cache/* var/page_cache/* generated/code/*`.

#### I've changed something inside `app/code/` directory but it's not showing

Run setup upgrade script `bin/magento setup:upgrade`

Also, consider running `bin/magento setup:di:compile` if you're working with dependency injection or plugins.

#### Container run out of memory when performing CLI commands

Add following before command then run `php -dmemory_limit=2G`

#### Apache throws error 500

SSH into web container and run `a2enmod rewrite` then restart apache2:

```bash
# Display a list of all active containers. Look for the container that is running your web server and note its CONTAINER ID or NAME.
docker ps

# SSH into the Docker Container
docker exec -it [CONTAINER_ID_OR_NAME] /bin/bash

# Restart Apache
a2enmod rewrite
```

#### Executor failed running [/bin/sh -c php -d memory_limit=-1 bin/magento setup:upgrade] on `dcu`

1. Comment out these lines in Dockerfile

    ```bash
    php -d memory_limit=-1 bin/magento setup:upgrade
    php -d memory_limit=-1 bin/magento cache:clean
    ```

2. Start the container

    ```bash
    docker-compose up
    ```

3. SSH into web container and run

    ```bash
    composer install --ignore-platform-reqs
    ```

### Setting up WordPress projects via LocalWP

To set WordPress projects locally in a more user friendly way you can use [LocalWP](https://localwp.com/).

- Download the Local WP to your computer
- Add new local project with default settings and `project_name`, and set site domain to `project_name.local`
- Clone this project to your `Sites` folder, and from there run this command:

    ```bash
    rsync -avzhp --progress --exclude 'wp-config.php' . ~/Local\ Sites/project_name/app/public/
    ```

    Alternatively you can delete the WordPress files added by Local WP, and clone this project in that place(~/Local Sites/project_name/app/public), and then update the wp-config.php file with these:

    ```bash
    'DB_NAME': 'local'
    'DB_USER': 'root'
    'DB_PASSWORD': 'root'
    ```

- Import demo database from the project. But to do so first unzip the database:

    ```bash
    unzip -o database_name.sql.zip > database_name.sql
    rm -r __MACOSX
    ```

    Import the database included in the project using the Adminer tool provided in Local. (This is a simple phpMyAdmin alternative.). Or if there are any issues with this you can open the site SSH shell and type:

    ```bash
    mysql -uroot -proot local < database_name.sql
    ```

    In case if there are issues with loading the database, you can try to delete and recreate the local wp database from the site shell. First login to mysql with:

    ```bash
    mysql -uroot -proot
    ```

    Then delete and recreate WordPress local database, with:

    ```bash
    drop database local;
    create database local;
    exit
    ```

- Sort out the SSL certificate trust issues by following this [guide](https://localwp.com/help-docs/getting-started/managing-local-sites-ssl-certificate-in-macos/)
- Test the site by opening WP Admin from Local.
- Happy coding!

#### LocalWP CLI

To use the WP CLI in your project set with Local WP, you will need to open the SSH shell, and from there you can use WP CLI commands. Some useful commands:

- Export the local database

  ```bash
  wp db export backup.sql
  ```

- Import a local database

  ```bash
  wp db import backup.sql
  ```

- Search and Replace in project database

  ```bash
  # Useful if you have loaded the database from staging or production
  wp search-replace '.com' '.local'
  wp search-replace '.websitetotal.com' '.local'
  ```

- Ggenerate `.pot` file from theme or plugin folder

  ```bash
  # Make sure this one is run from the correct folder
  wp i18n make-pot . languages/pot_file_name.pot
  ```

- List all the plugins

  ```bash
  wp plugin list
  ```

- Update the plugin

  ```bash
  wp plugin update plugin_name
  ```

- Activate the plugin

  ```bash
  wp plugin activate plugin_name
  ```

- Deactivate the plugin

  ```bash
  wp plugin deactivate plugin_name
  ```

- Show only a list of all active plugins which have updates available

  ```bash
  wp plugin list --field=name --status=active --update=available
  ```

- List all users

  ```bash
  wp user list
  ```

- List admin users

  ```bash
  wp user list --role=administrator
  ```

- Create a new administrator user

  ```bash
  wp user create admin name@stuntcoders.com --role=administrator --user_pass=sc123123
  ```

- Update existing administrator user password

  ```bash
  wp user update USER_ID  --user_pass=sc123123
  ```

- Delete the user and reassign that user’s posts to any other user

  ```bash
  wp user delete USER_ID --reassign=OTHER_USER_ID
  ```

- Regenerate all media attachments. Useful when changing image sizes

  ```bash
  wp media regenerate
  ```

- Flushe the WordPress site cache

  ```bash
  wp cache flush
  ```

- List all meta data associated with the specified post

  ```bash
  wp post meta list POST_ID --format=table
  ```

- Get a Specific Meta Field for a Post

  ```bash
  wp post meta get POST_ID META_KEY
  ```

- List All Meta Data for a User

  ```bash
  wp user meta list USER_ID --format=table
  ```

- Get a Specific Meta Field for a User

  ```bash
  wp user meta get USER_ID META_KEY
  ```

- List Rewrite Rules

  ```bash
  wp rewrite list --format=table
  ```

- Export content to an XML file

  ```bash
  wp export --dir=/path/to/export --user=author --start_date=2021-01-01 --end_date=2021-12-31
  ```
