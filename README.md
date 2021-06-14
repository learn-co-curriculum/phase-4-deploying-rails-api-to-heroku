# Deploying a Rails API to Heroku

## Learning Goals

- Set up your local environment for deploying with Heroku
- Deploy a basic Rails application to Heroku

## Introduction

In this lesson, we'll be deploying a basic, standalone Rails API application to
Heroku. We'll give instructions to generate the application from scratch and
talk through the steps to get the code running on a Heroku server.

In coming lessons, we'll learn how to add more complexity to the application
with a React frontend. Since the setup for a Rails-React application is a bit
trickier, it'll be beneficial to see the setup for Rails alone first. Let's get
started!

## Environment Setup

To make sure you're able to deploy your application, you'll need to do the
following:

### Sign Up for a [Heroku Account][heroku signup]

You can sign up at for a free account at
[https://signup.heroku.com/devcenter][heroku signup].

### Download the [Heroku CLI][heroku cli] Application

Download the Heroku CLI at
[https://devcenter.heroku.com/articles/heroku-cli#download-and-install][heroku cli].

After downloading, you can login via the CLI in the terminal:

```sh
heroku login
```

This will open a browser window to log you into your Heroku account. After
logging in, close the browser window and return to the terminal. You can run
`heroku whoami` in the terminal to verify that you have logged in successfully.

### Install the Latest Ruby Version

Verify which version of Ruby you're running by entering this in the terminal:

```sh
ruby -v
```

Make sure that the Ruby version you're running is listed in the [supported
runtimes][] by Heroku. At the time of writing, supported versions are 2.6.7,
2.7.3, or 3.0.1. Our recommendation is 2.7.3, but make sure to check the site
for the latest supported versions.

If it's not, you can use `rvm` to install a newer version of Ruby:

```sh
rvm install 2.7.3
rvm --default use 2.7.3
```

You should also install the latest version of `bundler` and `rails`:

```sh
gem install bundler
gem install rails
```

### Install Postgresql

Heroku requires that you use Postgresql for your database instead of SQLite.
Postgresql (or just Postgres for short) is an advanced database management
system with more features than SQLite. If you don't already have it installed,
you'll need to set it up.

To install Postgres for WSL, follow this guide:
[https://docs.microsoft.com/en-us/windows/wsl/tutorials/wsl-database#install-postgresql][postgresql wsl]

To install Postgres for OSX, you can use Homebrew:

```sh
brew install postgresql
```

Phew! With that out of the way, let's get started on building our Rails
application and deploying it to Heroku.

## Creating a Rails App to Deploy

We'll be following the steps in the
[Heroku Rails Deploying Guide][heroku rails deploying guide], so if you get
stuck and are looking for more assistance, check that guide first.

The first thing we'll need to do is create our new Rails application. Make
sure you're in a non-lab directory, then run:

```sh
rails new bird-app --api --minimal --database=postgresql
```

This will set up our app to run in API mode, with the minimum dependencies
needed, and with Postgresql as the database.

`cd` into the app, and run this command:

```sh
bundle lock --add-platform x86_64-linux --add-platform ruby
```

This will add additional platforms to your `Gemfile.lock` file that will allow
the necessary dependencies to be installed after you deploy your app.

## Building the Demo App

Next, let's set up up a migration, model, route, and controller so that
we have some data to display in our application:

```sh
rails g resource Bird name species
```

Add this data to the `db/seeds.rb` file:

```rb
Bird.create!(name: 'Black-Capped Chickadee', species: 'Poecile Atricapillus')
Bird.create!(name: 'Grackle', species: 'Quiscalus Quiscula')
Bird.create!(name: 'Common Starling', species: 'Sturnus Vulgaris')
Bird.create!(name: 'Mourning Dove', species: 'Zenaida Macroura')
```

Then run this command to generate the database and run the migrations and seed
file:

```sh
rails db:create db:migrate db:seed
```

> `rails db:create` creates a new Postgresql database to be associated with your
> application based on the configuration in the `config/database.yml` file.
> Unlike with SQLite, the actual database file isn't created in the `db` folder;
> it lives elsewhere in your file system, depending on your Postgresql
> configuration.

Finally, edit the `app/birds_controller.rb` file and add an `index` action:

```rb
  # GET /birds
  def index
    birds = Bird.all
    render json: birds
  end
```

To make sure the app works locally before deploying, run `rails s` and visit
[http://localhost:3000/birds](http://localhost:3000/birds).

## Deploying

Now that we've got some working code, it's time to get that code to run on a
Heroku server! The process of uploading our code to Heroku is managed by Git.
This makes it easy to deploy new versions using a tool most developers,
including yourself, are already familiar with.

Make a commit to save your changes:

```sh
git add .
git commit -m 'Initial commit'
```

Next, you'll need to create an application on Heroku:

```sh
heroku create
```

This command will generate a new application in your Heroku account, and
configure a new remote repository where you can push up your code. You can
confirm the remote repository was created successfully by running:

```sh
git config --list --local | grep heroku
```

You should see output like this:

```txt
remote.heroku.url=https://git.heroku.com/aqueous-sierra-44713.git
remote.heroku.fetch=+refs/heads/*:refs/remotes/heroku/*
```

Now, deploying your code is as simple as using `git push` to upload the changes
from your repository to Heroku:

```sh
git push heroku main
```

> Note: depending on your Git configuration, your default branch might be named
> `master` or `main`. You can verify which by running
> `git branch --show-current`. If it's `master`, you'll need to run
> `git push heroku master` instead.

You've successfully pushed up your code!

## Building and Migrating

After pushing up your code, you'll notice a flurry of activity in the terminal.
This indicates that Heroku is in the process of building your application, by:

- Setting up a Ruby environment to run your code in
- Installing the gems for your project with `bundle install`
- Running `rails server` to start up your server

If the build process succeeds, your app is live and online!

Before visiting the site, let's also set up the database. Remember, the database
is a separate application from your Rails application. Thankfully, we can get
it set up easily by running a few Rake commands on the server.

To migrate and seed the database on the server, run:

```sh
heroku run rails db:migrate db:seed
```

When you prefix any command with `heroku run`, it will run that command on the
server where your application was deployed. This command is very useful for
troubleshooting: you can even run `heroku run rails c` to open a Rails console
on the server!

Note: if you run into trouble connecting to the server, you may see an error
similar to the following when you try running the command above:

```text
psql: could not connect to server: No such file or directory
        Is the server running locally and accepting
        connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
```

One option for solving this problem for Mac users is to install the Postgres
app. To do this, first uninstall `postgresql`:

```sh
brew remove postgresql
```

Then download the app from the [Postgres downloads page][] and install it.
Launch the app and click "Initialize" to create a new server. You should now be
able to run the migration and seed the database using the command above.

[Postgres downloads page]: https://postgresapp.com/downloads.html

You can now visit the site in the browser by running `heroku open`. Note that,
because there is no root path (`'/'`) defined in our routes, you will see a
Page Not Found error when the app opens. Navigate to the `/birds` endpoint and
verify that you are able to see an array of JSON data for all the birds in the
database. If you aren't able to, check out the troubleshooting section below, or
the [troubleshooting guide on Heroku][troubleshooting guide on heroku].

## Adding New Features

Since Heroku integrates the deploying process with Git, it's straightforward
to add new features to your code and deploy them. Let's start by adding a new
controller action in the `BirdsController`:

```rb
def show
  bird = Bird.find(params[:id])
  render json: bird
rescue ActiveRecord::RecordNotFound
  render json: "Bird not found", status: :not_found
end
```

Test your code locally by running `rails s` and visiting
[http://localhost:3000/birds/1](http://localhost:3000/birds/1).

After adding this code, make a commit:

```sh
git add app/controllers/birds_controller.rb
git commit -m 'Added show action'
```

Then, to deploy the changes, push the new code up to Heroku:

```sh
git push heroku main
```

After pushing new code, Heroku will run through the build process again and
deploy your changes. You don't have to run your migrations again since the
database already exists on the server. You would have to run the migrations if
you created a new migration file.

## Troubleshooting

If you ran into any errors along the way, here are some things you can try to
troubleshoot:

- If your app failed to deploy at the build stage, make sure your local
  environment is set up correctly by following the steps at the beginning of
  this lesson. Check that you have the latest versions of Ruby and Bundler, and
  ensure that Postgresql was installed successfully.
- If you deployed successfully, but you ran into issues when you visited the
  site, make sure you migrated and seeded the database. Also, make sure that
  your application works locally and try to debug any issues on your local
  machine before re-deploying. You can also check the logs on the server by
  running `heroku logs`.

For additional support, check out these guides on Heroku:

- [Deploying a Rails 6 App to Heroku][heroku rails deploying guide]
- [Rails Troubleshooting on Heroku][troubleshooting guide on heroku]

## Conclusion

Congrats on deploying your first Rails app to the world wide web! Understanding
the deployment process and what it takes to run your application on another
computer is an important step toward becoming a full-stack developer. Like
anything new, this process can be daunting the first time you try it, but with
practice and exposure, you'll build confidence over time.

In the next lesson, we'll work on deploying a more complex application with a
Rails API backend and a React frontend, and talk through some of the challenges
of running these two applications together.

## Resources

- [Deploying a Rails 6 App to Heroku][heroku rails deploying guide]
- [Rails Troubleshooting on Heroku][troubleshooting guide on heroku]

[heroku signup]: https://signup.heroku.com/devcenter
[heroku cli]: https://devcenter.heroku.com/articles/heroku-cli#download-and-install
[supported runtimes]: https://devcenter.heroku.com/articles/ruby-support#supported-runtimes
[postgresql wsl]: https://docs.microsoft.com/en-us/windows/wsl/tutorials/wsl-database#install-postgresql
[heroku rails deploying guide]: https://devcenter.heroku.com/articles/getting-started-with-rails6
[troubleshooting guide on heroku]: https://devcenter.heroku.com/articles/getting-started-with-rails6#troubleshooting
