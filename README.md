# Tabby

![Tabby](tabby.jpg)

Titular application benchmarking blueprints, y'all

A Wiki for tracking projects, Slab information and institutional history, or whatever.

# Contributing
There are two ways to contribute to the wiki: Edit pages from the website directly; edit the markdown files in the Git repository

## Website
- In order to edit through the website interface, you'll need to use the default admin account or use that account to create your own, personal account.
- Once you are logged in, browse to the page you want to edit, and click the "EDIT" tab in the top right corner of the site.
- To create a new page, click the "CREATE" tab.
- Changes take about 5 minutes to show up in the GitHub Repo.

## Git repo
- Using and existing GitHub account, simply clone the git repository to your computer.
  - `git clone https://github.com/scholarslab/tabby.git`
- Now you can edit any of the Markdown files in the newly created 'tabby' folder.
- You can create new Markdown pages as well.
- To save the files, 'add' them, and then 'commit' them.
  - `git add <changed/added filename>`
  - `git commit -m "Short message about changes."`


# Setup Instructions
These are the instructions for setting up WikiJS from scratch (on production or your computer for development).

## Clone this repo
- `git clone https://github.com/scholarslab/tabby.git`

## Modify three files.

### config.yml
- Create a wikijs `config.yml` file:
  - These are just the lines that need to be changed.

  ```
  title: "Scholars' Lab Development Tracker"

  host: http://tabby.scholarslab.org

  port: 3000

  # ---------------------------------------------------------------------
  # Site Authentication
  # ---------------------------------------------------------------------

  public: true

  # ---------------------------------------------------------------------
  # Secret key to use when encrypting sessions
  # ---------------------------------------------------------------------
  # Use a long and unique random string (256-bit keys are perfect!)

  sessionSecret: 1uq9joj4df9l2roe2sk2la7u6ijq38djfq5

  # ---------------------------------------------------------------------
  # Database Connection String
  # ---------------------------------------------------------------------

  db: mongodb://tabby_wikidb:27017/wiki

  # ---------------------------------------------------------------------
  # Git Connection Info
  # ---------------------------------------------------------------------

  git:
    url: $(GIT_REPO_URL)
    branch: master
    auth:

      # Type: basic or ssh
      type: basic

      # Only for Basic authentication:
      username: $(GIT_USER_NAME)
      password: $(GIT_USER_PASS)

      serverEmail: $(WIKI_ADMIN_EMAIL)
  ```

  - The 'host' option is the domain name that we type into the browser (including port if we use that)
    - For local development, change the host to 'http://localhost'
  - The 'port' is the internal port that NodeJS will use for the app. This will also be the internal port of the Docker container.
  - The 'public' = true allows visitors to see pages. If set to false, you would need to log in before seeing pages on the site.
  - The database connection string will use the name of the database container, in this case 'tabby_wikidb'.
  - The git 'url' line uses Docker's environment variable capabilities. We set this value in the '.env' file.
  - With git 'type' set to 'basic', this will utilize https requests, rather than the recommended ssh connection. We don't want to deal with SSH keys from the production server connecting to GitHub. (we could, but I don't want to set that up. :) )
  - 'username' and 'password' also use environment variables.

### docker-compose.yml
- Create a `docker-compose.yml` file
  - Using version 2 of Docker Compose file because our server's docker-compose version can only take up to version 2

  ```
  version: '2'
  services:
    wikidb:
      image: 'mongo:3.7-jessie'
      expose:
        - '27017'
      command: '--smallfiles --bind_ip wikidb'
      environment:
        - 'MONGO_LOG_DIR=/dev/null'
      volumes:
        - ./data/mongo:/data/db
    wikijs:
      image: 'requarks/wiki:master'
      links:
        - wikidb
      depends_on:
        - wikidb
      ports:
        - '8088:3000'
      environment:
        WIKI_ADMIN_EMAIL: ${WIKI_ADMIN_EMAIL}
        GIT_USER_NAME:    ${GIT_USER_NAME}
        GIT_USER_PASS:    ${GIT_USER_PASS}
        GIT_REPO_URL:     ${GIT_REPO_URL}
      volumes:
        - ./config.yml:/var/wiki/config.yml
        - /var/log/www/tabby:/logs
  ```

  - **MongoDB section**
    - Everything is mostly the default settings. 
    - We'll use a set version of the mongoDB so that future versions don't break this version.
    - Create a volume connection to the host so that the DB data persists
      across server/docker reboots. The only purpose for the mongoDB is to hold
      the user account data when using the local account options (like we are).
      Even if this data doesn't persist, it's not a big deal. The default admin
      account uses the email from the '.env' file, and the default admin
      password for WikiJS (which you can find on their website).
      - ***NOTE: If the database data is reset (ie. the 'data' directory is
        deleted) then make sure to change the password as soon as the site is
        live again!***
  - **wikijs section**
    - Use the 'master' version of wikijs for stability and such
    - 'links' makes an internal Docker connection with the mongoDB container.
    - 'depends_on' makes sure the mongoDB is set up before the wikijs gets spun up.
    - 'ports' specifies the 'outside' port that Docker is listening on in the host, and the 'inside' port that the 'outside' is sent to. So Docker listens on port 8088, and forwards that data on the inside to 3000 (the NodeJS app).
    - 'environment' pulls in the environment variables from the '.env' file (the ${VARIABLE} part) and makes them available in the Docker's internal environment as system variables.
    - 'volumes' basically mounts the local/host file 'config.yml' into the path in the Docker container.
      - For local development, make sure you have the folder '/var/log/www/tabby' created already, or comment out that line.

### .env
- Create an `.env` file for environment variables

  ```
  GIT_USER_NAME=slabrd
  GIT_USER_PASS=github_user_pass
  GIT_REPO_URL=https://github.com/scholarslab/tabby.git
  WIKI_ADMIN_EMAIL=slabrd@virginia.edu
  ```

  - The first three variables are the access to GitHub, and which repo to use.
  - 'WIKI_ADMIN_EMAIL' is the default admin email address created by WikiJS when booted up.
  - Using Docker's environment variables works because Docker creates the VM
    using the '.env' file which then puts those in the system's environment.
    This is done before WikiJS starts, so the variables are already there and
    waiting.

# Web server setup
- The production server uses Apache as reverse proxy. 
  - For local development, skip this.
- Add an http config for this virtual host.
- Apache http config:

  ```
  <VirtualHost *:80>
      ServerName tabby.scholarslab.org

      ProxyPass / http://localhost:8088/
      ProxyPassReverse / http://localhost/
  </VirtualHost>

  ```

  - 'VirtualHost' looks for traffic on port 80
  - 'ServerName' looks for the domain name makerwiki.scholarslab.org
  - 'ProxyPass' everything from the root of the domain is passed to the localhost port 8088 (which is what Docker is listening on).
  - 'ProxyPassReverse' - fixes the request sent back to the browser when HTTP redirect responses are sent from the app. 

- Restart Apache as needed.
  - `sudo service httpd graceful`

# Run Docker

- While in the project directory, run 
  - `docker-compose up -d`

# Access the website
- For production, the website is accessible at http://tabby.scholarslab.org
- For local development, the website is accessible at http://localhost
  - Note: any changes made to settings, the admin password, etc will not be carried over to the production site.
  - Note: any changes to pages, and adding and deleting pages WILL affect the
    production site, because this local instance is connected to the production
    GitHub account.


# Things to note
- You can add pages to the site by using the webpage and logging in as the
  admin, OR by directly adding them to the GitHub repo. The web app checks for
  updates every five minutes, so it takes that long for direct additions to
  GitHub to show up on the site.
- When creating a WikiJS from scratch, the GitHub repository must already be created, and not be empty.
