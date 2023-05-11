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
Edit `config/deploy.rb` or put the following into it:
```ruby
# config valid for current version and patch releases of Capistrano
lock "~> 3.17.2"
# ================================
# Set the following according to your configuration
# FQDN for your application
server 'ror.erfp.ru', port: 22, roles: [:web, :app, :db], primary: true
# Set folder name on remote server
set :application, 'erfp'
# Set repo url
set :repo_url, 'git@github.com:erfp/erfp-rails.git'
# Set username on remote server
set :user, 'meole'
# ================================
set :puma_threads,    [4, 16]
set :puma_workers,    0

set :pty,             true
set :use_sudo,        false
set :stage,           :production
set :deploy_via,      :remote_cache
set :deploy_to,       "/home/#{fetch(:user)}/apps/#{fetch(:application)}"
set :puma_bind,       "unix://#{shared_path}/tmp/sockets/#{fetch(:application)}-puma.sock"
set :puma_state,      "#{shared_path}/tmp/pids/puma.state"
set :puma_pid,        "#{shared_path}/tmp/pids/puma.pid"
set :puma_access_log, "#{release_path}/log/puma.access.log"
set :puma_error_log,  "#{release_path}/log/puma.error.log"
set :ssh_options,     { forward_agent: true, user: fetch(:user), keys: %w(~/.ssh/id_rsa.pub) }
set :puma_preload_app, true
set :puma_worker_timeout, nil
set :puma_init_active_record, true  # Change to false when not using ActiveRecord

append :linked_files, "config/master.key"
append :linked_dirs, "log", "tmp/pids", "tmp/cache", "public/uploads"

# Default branch is :master
# ask :branch, `git rev-parse --abbrev-ref HEAD`.chomp

# Default deploy_to directory is /var/www/my_app_name
# set :deploy_to, "/var/www/my_app_name"

# Default value for :format is :airbrussh.
# set :format, :airbrussh

# You can configure the Airbrussh format using :format_options.
# These are the defaults.
# set :format_options, command_output: true, log_file: "log/capistrano.log", color: :auto, truncate: :auto

# Default value for :pty is false
# set :pty, true

# Default value for :linked_files is []
# append :linked_files, "config/database.yml", 'config/master.key'

# Default value for linked_dirs is []
# append :linked_dirs, "log", "tmp/pids", "tmp/cache", "tmp/sockets", "tmp/webpacker", "public/system", "vendor", "storage"

# Default value for default_env is {}
# set :default_env, { path: "/opt/ruby/bin:$PATH" }

# Default value for local_user is ENV['USER']
# set :local_user, -> { `git config user.name`.chomp }

# Default value for keep_releases is 5
# set :keep_releases, 5

# Uncomment the following to require manually verifying the host key before first deploy.
# set :ssh_options, verify_host_key: :secure

```


