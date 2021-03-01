# Dockerised Rails Development Environment

This repository acts as a base for developing Rails applications using Docker. It uses docker-compose to set up and run a Postgres database in a separate container, running alongside the Rails application container. The production environment will require further setup before it can start functioning.

## Contents

The current versions of various dependencies are:

* Ruby 3.0 - this is the base Docker image 
* postgresql-client latest via apt-get
* Nodejs 14.x
* NPM latest
* yarn latest
* Rails 6.1.3
* Ruby gems as defined in the Gemfile.lock (which is included in the repo!)

A Rails app has already been generated as there are several setup steps that would otherwise be missing.

## Usage

This repo is meant to contain a generic Rails setup, *not* the application that gets built on it. To use this, either fork https://github.com/danwhitston/rails-docker to your new application repo on GitHub, git clone it, and start work, or git clone it, start work and set the remote to a new destination. Either way, you should rewrite this README to give instructions for working with your own application based on this.

To bring up a development instance of the docker setup:

* First ensure that Docker is installed on your system, along with docker-compose
* Run `docker-compose build` to set up the Rails and Postgres docker instances
* Run `docker-compose up` to start a development instance
* Your shell should show the usual Rails and Postgres command line outputs and should not give you a command line
* Create the test and dev databases from another shell window with `docker-compose run web rake db:create`
* Navigate to localhost:3000 and you should see a `Yay! You're on Rails` welcome screen

By uncommenting some lines in the docker-compose.yml, you can also enable an Adminer instance to examine your databases. 

TODO: This doesn't currently have IDE or editor config files, and usage with editors requires documentation.

## How it was created

Useful for recreating or updating:

* To start, use the following files from this repo
    ```shell
    .dockerignore
    .gitignore
    docker-compose.yml
    Dockerfile
    entrypoint.sh
    ```
* Create a `./Gemfile` with just the following lines
    ```ruby
    source 'https://rubygems.org'
    gem 'rails', '~>6.1.0'
    ```
* Use `touch Gemfile.lock` to create an empty lockfile, needed for the docker build to work correctly
* Create the rails app with `docker-compose run --no-deps web rails new . --force --database=postgresql --skip-test`
* Use `sudo chown -R $USER:$USER .` to fix user ownership of the created files
* Set up database connection in `./config/database.yml`
    ```yaml
      default: &default
        adapter: postgresql
        encoding: unicode
        host: db
        username: postgres
        password: password
        pool: 5
    ```
* Add the following code to `./Gemfile` to install RSpec-Rails
    ```ruby
    # Run against the latest stable release
    group :development, :test do
      gem 'rspec-rails', '~> 4.0.2'
    end
    ```
* Then `docker-compose build` was run to install RSpec-Rails, followed by `docker-compose run --no-deps web rails generate rspec:install` to set up RSpec test generation
* Use `sudo chown -R $USER:$USER .` *again* to fix ownership of the remaining created files
* Finally, run `docker-compose up` to run the two Docker instances, and in another shell run `docker-compose run web rake db:create` to set up the test and development databases in the PostgreSQL instance

This was based on https://docs.docker.com/compose/rails/ but required several changes to get working on the current Ruby and Rails versions, as NodeJS / NPM / Yarn did not install correctly.
