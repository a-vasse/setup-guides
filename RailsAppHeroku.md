## Creating a Rails app for Heroku deployment

### Table of Contents
* [Introduction](#introduction)
* [Prerequisites](#prerequisites)
  * [Ruby](#rubyinstall)
  * [Gems](#geminstall)
  * [Node.js](#nodeinstall)
  * [Yarn](#yarninstall)
  * [PostgreSQL](#postgresqlinstall)
  * [Heroku](#herokuinstall)
* [Setting up the project](#setup)
* [Setting up Devise](#devise)
  * [Initial setup](#devisesetup)
  * [Adapting for Rails 7](#deviseadapt)
  * [Alerts](#devisealerts)
  * [Controller](#devisecontroller)
  * [Editing the User](#deviseuser)
* [Setting up Cloudinary and Active Storage](#clas)
  * [Install and integrate dotenv](#clasdot)
  * [Install Cloudinary](#clasinstall)
  * [Install Active Storage](#activeinstall)
  * [Install Active Storage](#clasconfig)
  * [Usage in your project](#clasuse)
* [Local Testing](#local)
* [Deployment on Heroku](#herokudeploy)
  * [Initial Deployment](#herokuinitial)
  * [Database Management](#herokudb)
  * [Cloudinary Configuration](#herokucloud)
  * [Troubleshooting](#herokutrouble)
* [Conclusion](#conclusion)

## Introduction:<a id="introduction"></a>

This guide will allow the user to:

- Create and configure a CRUD app with Ruby on Rails.
- Install and integrate Devise.
- Install and integrate Cloudinary and Active Storage.
- Configure and deploy the app on Heroku.

## Prerequisites:<a id="prerequisites"></a>
The following are required for operation. If these are already present in your configuration, you can skip these steps.

Follow the necessary guides for installation:

### Ruby:<a id="rubyinstall"></a>
- Ensure `rbenv` is installed by executing `brew install rbenv`.
- Install Ruby via `rbenv install 3.1.2`. This will take approx. **5 minutes** .
- Configure the system to use 3.1.2 by default by executing `rbenv global 3.1.2`.
- Restart your terminal.
- Confirm Ruby version with `ruby -v`.

### Gems:<a id="geminstall"></a>
- Execute the following to install key gems needed for this project:
  ```
  gem install colored faker http pry-byebug rake rails rest-client rspec rubocop-performance sqlite3
  ```
- Confirm Rails version with `rails -v`

### Node.js<a id="nodeinstall"></a>
- Install the Node.js version manager `nvm` by executing the following:
  ```
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh
  ```

- Restart your terminal.
- Confirm version with `nvm -v`.
- Install Node.js by executing `nvm install 16.15.1`.
- Confirm version with `node -v`.
- Execute `nvm cache clear` to clear current cache.

### Yarn<a id="yarninstall"></a>
- Install the Yarn package manager by executing `npm install --global yarn`.
- Confirm version with `yarn -v`.

### PostgreSQL<a id="postgresqlinstall"></a>
 - Install by executing the following commands:
   ```
   sudo apt install -y postgresql postgresql-contrib libpq-dev build-essential
   ```

   ```
   sudo /etc/init.d/postgresql start
   ```

   ```
   sudo -u postgres psql --command "CREATE ROLE \"`whoami`\" LOGIN createdb superuser;"
   ```
- Configure PostgreSQL to autostart on opening a terminal:
  ```
  sudo echo "`whoami` ALL=NOPASSWD:/etc/init.d/postgresql start" | sudo tee /etc/sudoers.d/postgresql
  ```
  ```
  sudo chmod 440 /etc/sudoers.d/postgresql
  ```
  ```
  echo "sudo /etc/init.d/postgresql start" >> ~/.zshrc
  ```

- Restart your terminal
- Confirm version with `psql --version`
- The message `Starting PostgreSQL 14 database server` will confirm successful installation.

### Heroku<a id="herokuinstall"></a>
- Install by executing `curl https://cli-assets.heroku.com/install.sh | sh`
- Confirm version with `heroku --version`

## Setting up the project:<a id="setup"></a>
The following will instruct you on how to create the initial setup for your Rails app:

- Create a new rails app by executing `rails new [APP-NAME] -d postgresql -j webpack`
- Navigate to the folder via `cd [APP-NAME]`
- Open the project in your preferred code editor (ex. VS Code via `code .`)
- Run initial `gem` install by executing `bundle install`

## Setting up Devise<a id="devise"></a>

This step will install Devise, needed to authenticate users. If your project requires a user login/logout logic please follow this installation guide. Otherwise this step can be skipped.

### Initial setup<a id="devisesetup"></a>
- Install the `gem` by executing `gem "devise"`,
- Rerun your `gem` installation. (`bundle install`)
- Install devise by executing `rails generate devise:install`
- Refer to `config/initializers/devise.rb`, ensuring the following is present:
```
config.mailer_sender = 'please-change-me-at-config-initializers-devise@example.com'
```
- Generate the `User` model by executing `rails generate devise User`
- This will generate the following:
    - `app/models/user.rb`
    - `db/migrate/TIMESTAMP_devise_create_users.rb`
- Generate custom devise views by executing `rails generate devise:views`

### Adapting for Rails 7<a id="deviseadapt"></a>

As Devise is currently fully compatible with Rails 7, `Turbo` used in forms will require disabling. This can be implemented as follows:
- In `app/views/devise/registrations/new.html.erb` it should read:
  ```
  <%= form_for(resource, as: resource_name, url: registration_path(resource_name),
  data: {turbo: false}) do |f| %>
  ```

- In `app/views/devise/sessions/new.html.erb` it should read:
  ```
  <%= form_for(resource, as: resource_name, url: session_path(resource_name),
  data: {turbo: false}) do |f| %>
  ```

- In `app/views/shared/_navbar.html.erb` it should read:
  ```
  <%= link_to "Log out", destroy_user_session_path, data: {turbo_method: :delete},
  class: "dropdown-item" %>
  ```

### Alerts<a id="devisealerts"></a>

Alerts will need to be created. This can be done as follows:
- Create the corresponding view:
  ```
  touch app/views/shared/_flashes.html.erb
  ```
- Insert the following code:
  ```
  <% if notice %>
    <div class="alert alert-info alert-dismissible fade show m-1" role="alert">
      <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
      <%= notice %>
    </div>
  <% end %>
  <% if alert %>
    <div class="alert alert-warning alert-dismissible fade show m-1" role="alert">
      <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
      <%= alert %>
    </div>
  <% end %>
  ```

### Controller<a id="devisecontroller"></a>

The following configuration needs to take place for Devise to function:
- Initially configure all routes as requiring login to access.
  - For example, in `app/controllers/application_controller.rb`:
    ```
    class ApplicationController < ActionController::Base
      before_action :authenticate_user!
    end
    ```

- When whitelisting pages to skip authentication do as follows:
  - For example, in `app/controllers/pages_controller.rb`
  ```
  class PagesController < ApplicationController
    skip_before_action :authenticate_user!, only: :home

    def home
    end
  end
  ```
### Editing the User<a id="deviseuser"></a>

In order to create new columns to our User table, there are several adjustments to the User that need to be made.

In this example, we will be adding `name` and `age` to the user.

- First, the columns need to be added to the migration file:
  ```
  db/migrate/TIMESTAMP_devise_create_users.rb
  ```
  Add the following:
  ```
  t.string   :name, null: false
  t.integer  :age, null: false
  ```

- Then, new fields need to be added in the sign up/edit/log in pages:
  ```
  app/views/devise/registrations/new.html.erb
  ```
  ```
  app/views/devise/registrations/edit.html.erb
  ```
  ```
  app/views/devise/sessions/new.html.erb
  ```
  Adding the following:
  ```
    <%= f.input :name, required: true %>
    <%= f.input :age, required: true %>
  ```

- Finally `app/controllers/application_controller.rb` needs to be updated:
  ```
  class ApplicationController < ActionController::Base

    before_action :configure_permitted_parameters, if: :devise_controller?

    def configure_permitted_parameters
      # For additional fields in app/views/devise/registrations/new.html.erb
      devise_parameter_sanitizer.permit(:sign_up, keys: [:name, :age])

      # For additional in app/views/devise/registrations/edit.html.erb
      devise_parameter_sanitizer.permit(:account_update, keys: [:name, :age])
    end
  end
  ```

## Setting up Cloudinary/Active Storage<a id="clas"></a>
In order to upload images to a project, such as avatars, we will integrate the Cloudinary online service, combined with the Active Storage functionality. If you do not intend to upload images into your project, this step can be skipped.

A free account can be registered at [Cloudinary](https://cloudinary.com/users/register/free).

### Install and integrate dotenv<a id="clasdot"></a>

`dotenv` will need to be installed to allow us to safety store our keys in our code.
- Install the `gem` by executing `gem "dotenv-rails", groups: [:development, :test]`
- Run `bundle install`
- Execute the following code to create the neccessary file:
  ```
  touch .env
  echo '.env*' >> .gitignore
  ```
### Install Cloudinary<a id="clasinstall"></a>
- Install the `gem` by executing `gem "cloudinary"`
- Run `bundle install`
- Set the following code in `.env`, created above:
  ```
  CLOUDINARY_URL=cloudinary://298522699261255:Qa1ZfO4syfbOC-################*8
  ```
  _* Note: Your key can be obtained from your Cloudinary account._

### Install Active Storage<a id="activeinstall"></a>
- Install by executing `rails active_storage:install`
- Run `rails db:migrate`
- Configure your `config/storage.yml` file as follows:
  ```
  cloudinary:
  service: Cloudinary
  folder: <%= Rails.env %>
  ```
- Configure your `config/environments/development.rb` as follows:
  ```
  config.active_storage.service = :cloudinary
  ```
- Configure your `config/environments/production.rb` as follows:
  ```
  config.active_storage.service = :cloudinary
  ```
### Configuring files<a id="clasconfig"></a>

In order to set up your project to use Cloudinary and Active storage, the following edits need to be made to your `models`, `controllers` and `views`:

- For your `models` that will use uploaded images, the following needs to be inserted:
  - For single images:
    ```
    has_one_attached :photo
    ```
  - For multiple images:
    ```
    has_many_attached :photos
    ```

- For your `controllers` that involve methods with uploading involved, the following is required:
  - For single images:
    ```
    def article_params
    params.require(:article).permit(:title, :body, :photo)
    end
    ```
  - For multiple images:
    ```
    def article_params
    params.require(:article).permit(:title, :body, photos: [])
    end
    ```
### Usage in your project<a id="clasuse"></a>

The following is a quick explanation of how Cloudinary and Active Storage can be implemented into a project:
- When displaying images, the following syntax is applied:
  - For single images:
    ```
    <%= cl_image_tag @article.photo.key, height: 300, width: 400, crop: :fill %>
    ```
  - For multiple images:
    ```
    <% @article.photos.each do |photo| %>
      <%= cl_image_tag photo.key, height: 300, width: 400, crop: :fill %>
    <% end %>
    ```
- For forms that allow the user to upload, the following field needs to be inserted:
  - For single images:
    ```
    <%= f.input :photo, as: :file %>
    ```
  - For multiple images:
    ```
    <%= f.input :photos, as: :file, input_html: { multiple: true } %>
    ```
## Local testing<a id="local"></a>
In order to run the project locally, execute the following:
  - Local server: `rails s` to run on `localhost:3000`
  - Yarn build: `yarn build --watch`

To setup your database for testing, execute the following:
- `rails db:create`
- `rails db:migrate`
- `rails db:seed`

_* Note: if you have an existing DB, you will need to run `rails db:reset`_

## Deployment on Heroku<a id="herokudeploy"></a>

### Initial deployment<a id="herokuinitial"></a>
- Log in to Heroku by executing `heroku login` and following the login instructions.
- Create the name of your app by executing `heroku create [APP_NAME] --region eu`

  _Note: `--region eu` or `region us` can be selected based on your location, to reduce latency._
- Run `git push heroku master` in order to push your latest build to Heroku

Your app should now be live. You can check the URL (ex. `APP_NAME.herokuapp.com`) to confirm.

### Database management<a id="herokudb"></a>
- In order to generate your database on Heroku, the following commands need to be run:
  - `heroku run rails db:migrate`
  - `heroku run rails db:seed`

*Warning: Do not run `heroku run rails db:create`, it is not necessary in this instance.*

### Cloudinary Configuration<a id="herokucloud"></a>

- Run the following `heroku config:set CLOUDINARY_URL=cloudinary://166....` replacing the end with your own personal key.
- Confirm the status by executing `heroku config`

### Troubleshooting<a id="herokutrouble"></a>

- In order to pinpoint issues with Heroku deployment, run `heroku logs --tail`
- This will output the logs of the deployment, indicating any issues experienced.

- If you encounter issues with your current database, it can be reset as follows:
  - `heroku restart`
  - `heroku pg:reset DATABASE`

- Followed by:
  - `heroku run rails db:migrate`
  - `heroku run rails db:seed`


## Conclusion<a id="conclusion"></a>

With the above steps followed, you are now ready to code your own Ruby on Rails app, and deploy it to Heroku!
