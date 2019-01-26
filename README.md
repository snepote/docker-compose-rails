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
**Note:** allow a few seconds to start. Since is running on the background, you have no feedback to know when the application is ready to accept requests.

## Stop the application
From the root directory of your project, run the following command:
```bash
docker-compose down
```
You can use the same terminal window in which you started the database, or another one where you have access to a command prompt. This is a clean way to stop the application.

## Rebuild the application
If you make changes to the Gemfile or the Compose file to try out some different configurations, you need to rebuild. Some changes require only
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

## Sources
[Quickstart: Compose and Rails](https://docs.docker.com/compose/rails)
[Creating a new Rails application project with Docker](https://github.com/IcaliaLabs/guides/wiki/Creating-a-new-Rails-application-project-with-Docker)
[Testing Your Rails Application with Docker](https://blog.codeship.com/testing-rails-application-docker/)

## Running Rspec
### Rspec setup (one time only)
Edit `Gemfile` and add Rspec gem to development and test group
```bash
group :development, :test do
  gem 'rspec', '~> 3.8'
  # ...
end
```
and rebuild the application with
```bash
docker-compose run web bundle install
docker-compose up --build
```

### Creat Rspec config files
```bash
docker-compose run web bundle exec rake rspec --init
```

### Run Rspec
```bash
docker-compose up -d
docker-compose run -e "RAILS_ENV=test" web rake db:create db:migrate
docker-compose run -e "RAILS_ENV=test" web bundle exec rspec
```
