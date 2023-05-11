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
Check `http://http://127.0.0.1:3000` if app now available.
