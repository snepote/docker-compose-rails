# docker-compose-rails
A base skeleton for a Ruby on Rails application with Postgres using Docker Compose.

## Build: initial setup

### Generate the Rails skeleton app inside ./web folder
```bash
docker-compose run web rails new ./web --force --no-deps --database=postgresql
```
**Note:** if you are running Docker on Linux, the files rails new created are owned by root. This happens because the container runs as the root user. If this is the case, change the ownership of the new files.
```bash
sudo chown -R $USER:$USER .
```
Now that you’ve got a new Gemfile, you need to build the image again. (This, and changes to the Gemfile or the Dockerfile, should be the only times you’ll need to rebuild.)
```bash
docker-compose build
```
### Setup the database connection
The app is now bootable, but you’re not quite there yet. By default, Rails expects a database to be running on `localhost` - so you need to point it at the `db` container instead. You also need to change the database and username to align with the defaults set by the `postgres` image.

Replace the contents of `web/config/database.yml` as it follows (take into account that the prefix `web_` is the name of your app):
```yml
default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  username: postgres
  password:
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

development:
  <<: *default
  database: web_development

test:
  <<: *default
  database: web_test

production:
  <<: *default
  database: web_production
  url: <%= ENV['DATABASE_URL'] %>
```

### Boot the application
#### Set basic ENV VARs
Before starting the application you will need to set a value for RACK_ENV. On the root folder of the project, create an `.env` file. **IMPORTANT:** this file *must* be on `.gitgnore` and `.dockerignore`
```bash
touch .env
```
Then add the following (and any other necessary) content:
```bash
RACK_ENV=development
```

#### Let's create the database (one time only)
```bash
docker-compose up
```
Finally, you need to create the database. In another terminal, run:
```bash
docker-compose run web rake db:create
```

## Restart the application
To restart the application run `docker-compose up` in the project directory. Alternatively, you can use the following command to run it on the background:
```bash
docker-compose up -d
```

## Stop the application
From the root directory of your project, run the following command:
```bash
docker-compose down
```
You can use the same terminal window in which you started the database, or another one where you have access to a command prompt. This is a clean way to stop the application.

## Rebuild the application
f you make changes to the Gemfile or the Compose file to try out some different configurations, you need to rebuild. Some changes require only
```bash
docker-compose up --build
```
but a full rebuild requires a re-run of
```bash
docker-compose run web bundle install
```
to sync changes in the `Gemfile.lock` to the host, followed by
```bash
docker-compose up --build
```

## Source
[Quickstart: Compose and Rails](https://docs.docker.com/compose/rails)
