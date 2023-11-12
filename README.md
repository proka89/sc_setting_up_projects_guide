## StuntCoders guide for setting up new projects in local environment

This guide aims to provide a step-by-step process for setting up WordPress and Magento projects in a local environment, utilizing Docker or LocalWP.

### Prerequisites

- **Docker Desktop**: Ensure Docker Desktop is installed on your machine. [Download Docker Desktop](https://www.docker.com/products/docker-desktop).
- **LocalWP**: For WordPress-specific projects, LocalWP is a more user-friendly option. [Download LocalWP](https://localwp.com/).
- **Basic Command Line Knowledge**: Familiarity with terminal or command prompt commands is required.
- **Code Editor**: Use any code editor like Visual Studio Code, Sublime Text, etc.

## Setting up projects via Docker

If you are not already familiar with Docker, it's recommended to start with the [Docker Getting Started documentation](https://docs.docker.com/get-started/). Key areas to pay special attention to include:

- **Overview of Docker concepts**: Understanding the fundamental concepts such as images, containers, and Dockerfiles.
- **Setting up your Docker environment**: Learn how to install and configure Docker on your specific operating system.
- **Building and running your first container**: This section provides practical experience in building and running Docker containers, which is crucial for setting up your development environment.
- **Docker Compose**: Understanding Docker Compose is essential for managing multi-container applications, like those you'll be creating for WordPress and Magento.

### Setting up WordPress projects via Docker

In the instructions below, we will be using variable such as `project_name` or `database_name`, these should be replaced by the actual project or database names. `database_name` variable will be the name of the demo database dump, which can be found in the root of the project. For example this could look something like `pickles.sql.zip`, in such case you would be using `pickles` instead of `database_name` in the commands below.

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

- Go into project root directory and just run the containers with:

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

- Type `vim ~/.zshrc` in the Terminal, to opet the `.zshrc` file. Then press `i` to enter the `INSERT` mode
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

Or if you don't have an [alias](https://github.com/stuntcoders/stunt_dev_setup/blob/master/install.sh#L72) set:

```bash
docker-compose exec <service> /bin/bash # services: web, db, phpmyadmin, etc..
```

#### Update a plugin or theme

```bash
dwp plugin list
dwp plugin update <plugin> # paste plugin name from the list

```

Or if you don't have an [alias](https://github.com/stuntcoders/stunt_dev_setup/blob/master/install.sh#L75-L111) set:

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