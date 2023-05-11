# Deploy rails application to VPS/VDS Ubuntu server with Capistrano
Howto: Deploy rails app
We'll use `erfp` as an application name for new rails project

## Ubuntu version
---
Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-71-generic x86_64)

# Create new rails App
```bash
rails new erfp --database=postgresql --css=bootstrap
```
1. Prepare your local development and testing environment with postgres
`config/database.yml` add folowing credentials into `development` and `test` environments
Example:
```ruby
test:
  <<: *default
  database: erfp_test
```
add following:
```ruby
  username: postgres
  password: YourLocalPostgresPassword
```
2. Create `development` and `test` databases
```bash
rake db:setup
```
3. Migrate database
```bash
rake db:migrate
```
4. Start local server (Puma)
```bash
bin/rails s
```
Check `http://127.0.0.1:3000` if app now available.

# Prepare Postgres database on ubuntu server
---
Login to your new VPS/VDS server via SSH:
```bash
ssh ruby
```
Switch to `postgres` user and jump into its working directory
```bash
su postgres && cd
```
Create new database `erfp_production`
```bash
createdb erfp_production
```
Jump into PSQL Shell:
```bash
psql
```
Create user (ROLE) for new database
```bash
create user erfp with password 'myStrongProductionPassword';
```
Grant all priveleges on newly created role:
```bash
grant all privileges on database erfp_production to erfp;
```
Exit PSQL Shell
```bash
exit
```
Back to root user
```bash
exit
```
Put password for newly created postgres role into production encrypted environment variables
```bash
EDITOR="nano --wait" bin/rails credentials:edit --environment production
```
add your password by adding following key and its value:
```
ERFP_DATABASE_PASSWORD: myStrongProductionPassword
```

# Install Capistrano
---
Add the following gems to your `Gemfile` to `group :development`:
```ruby
# CAPISTRANO DEPLOY TOOL
gem 'capistrano',         require: false
gem 'capistrano3-puma',   require: false
gem 'capistrano-bundler', require: false
gem 'capistrano-rails',   require: false
gem 'capistrano-rvm',     require: false
```
Install gems via bundler:
```bash
bundle install
```
Install Capistrano (init setup)
```bash
cap install
```
Replace `Capfile` content with the following:
```ruby
# Load DSL and set up stages
require 'capistrano/setup'

# Include default deployment tasks
require 'capistrano/deploy'
require 'capistrano/rails'
require 'capistrano/bundler'
require 'capistrano/rvm'
require 'capistrano/scm/git'
install_plugin Capistrano::SCM::Git

require 'capistrano/puma'
install_plugin Capistrano::Puma
install_plugin Capistrano::Puma::Systemd
Dir.glob('lib/capistrano/tasks/*.rake').each { |r| import r }
```


