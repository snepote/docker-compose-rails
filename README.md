# docker-compose-rails
A base Ruby on Rails application with Docker Compose

## Build

### Generate the Rails skeleton app inside ./web folder
```bash
docker-compose run web rails new ./web --force --no-deps --database=postgresql
```

## Source
[Quickstart: Compose and Rails](https://docs.docker.com/compose/rails)
