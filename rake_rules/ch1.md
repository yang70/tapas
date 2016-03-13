# Personal Website - A Ruby on Rails application

[![Build Status](https://travis-ci.org/yang70/portfolio.svg?branch=master)](https://travis-ci.org/yang70/portfolio)
[![Coverage Status](https://coveralls.io/repos/yang70/portfolio/badge.svg?branch=master&service=github)](https://coveralls.io/github/yang70/portfolio?branch=master)

By [Matthew Yang](http://www.matthewgyang.com).

## Description
This repository is an ongoing project that was created for a couple of reasons:

1. To establish a personal website and presence on the web.
2. As a 'playground' to test out different technologies and methods.

As much as I've tried to keep it updated, there are many areas that need to be cleaned up, refactored and ultimately changed as my experience with Rails (and web development in general) has increased.

Below, I'll try to explain some of the features and techniques that have been incorporated into the application.

## Authentication / Authorization
Basic authentication is implemented with a standard [Devise](https://github.com/plataformatec/devise) integration.

Policies and scope are configured using [Pundit](https://github.com/elabs/pundit), and depend on a user's role (basic user, editor, guest author)

New user's have the option of registering using their Twitter account which was implemented using [Omniauth-Twitter](https://github.com/arunagw/omniauth-twitter).

## Blog
The blog is pretty standard, however I wanted to be able to use Markdown for styling and for inserting code blocks.  I followed [this](http://allfuzzy.tumblr.com/post/27314404412/markdown-and-code-syntax-highlighting-in-ruby-on) blog post which institutes [Redcarpet](https://github.com/vmg/redcarpet) and [CodeRay](https://github.com/rubychan/coderay).

One aspect I have not been able to implement is horizontal scroll for wide code blocks.

## Portfolio
As I mentioned before, the portfolio section uses remote JavaScript for it's CRUD actions and is able to accomplish this without a page reload.

In the show method for projects, I pull in the readme directly from GitHub via the [GitHub API](https://developer.github.com/v3/) and render the markdown like I do in the blog section.

I wrote a helper method to grab the raw markdown:

```ruby
def get_readme(repo_name)

  uri = URI.parse('https://api.github.com/repos/yang70/' + repo_name + '/readme')

  https = Net::HTTP.new(uri.hostname, uri.port)
  https.use_ssl = true

  req = Net::HTTP::Get.new(uri.path, initheader = {'Accept' => 'application/vnd.github.v3.raw'})
  req.basic_auth 'yang70', ENV['GITHUB_ACCESS']

  res = https.request(req)

  res.body
end
```

## Mailers / Sidekiq / Redis

`ActionMailer` is configured to alert the editor of various events with the site, including new user registration and comments.  Email configured in Heroku deployment with the add on [SendGrid](https://sendgrid.com/)

The new user registration welcome email to the user is handled through a background job using [Sidekiq](https://github.com/mperham/sidekiq) and [Redis](http://redis.io/).

I created an `app/workers/sign_up_worker.rb` with the following code:

```ruby
class SignUpWorker
  include Sidekiq::Worker

  def perform(userid)
    user = User.find(userid)
    WelcomeMailer.new_welcome_email(user).deliver_later
  end
end
```

Then in `app/controllers/sessions/registrations_controller.rb` I add a line after deferring to `super` for the rest of the action:

```ruby
def create
  super
  SignUpWorker.perform_async(@user.id) unless @user.invalid?
end
```

## Picture Upload

Picture uploading to articles was enabled with the [CarrierWave](https://github.com/carrierwaveuploader/carrierwave) gem.  It is configured to upload to AWS S3.

Future plans include allowing users to crop an image after upload using something like [Carrierwave Crop](https://github.com/kirtithorat/carrierwave-crop/), or I might switch to [Paperclip](https://github.com/thoughtbot/paperclip) and [Papercrop](https://github.com/rsantamaria/papercrop).

## Remote JavaScript (RJS)
The projects sections has been created to allow quick editing by incorporating AJAX calls and jQuery using Rails [RJS](http://edgeguides.rubyonrails.org/working_with_javascript_in_rails.html).

CRUD action links updated by adding `remote: true` and creating the correct response logic in the controller to respond to html or javascript accordingly.  Then specific javascript 'views' were created, which are just jQuery statements that when execute, preform the desired actions including updating the page structure.

Here's an example of a controller action that can respond to RJS:

```ruby
def create
  @project = Project.new(project_params)
  if @project.save
    respond_to do |format|
      format.html { redirect_to @project, flash[:notice] = "Project has been created." }
      format.js
    end
  else
    respond_to do |format|
      format.html { render :new }
      format.js { render :new}
    end
  end
end
```

## Styling
Styling is my self admitted "weak spot", and I'm working hard to make it a strength!  I started this site by using the [Zurb Foundation](http://foundation.zurb.com/) framework, but have recently swapped it out for [Bootstrap](http://getbootstrap.com/) which I personally find more intuitive.  Additionally there seem to be more resources available for questions/answers.

Ideally, I would like to minimally rely on a framework and institute original styling on my own, which would be future goal.

## Testing
Feature tests are implemented using [Minitest Rails Capybara](https://github.com/blowmage/minitest-rails-capybara) and the corresponding Capybara DSL language.  Example of a Capybara feature test:

```ruby
require "test_helper"

feature "CanAccessWelcome" do
  scenario "is displaying correct content" do
    visit root_path
    page.must_have_content "About"
    page.must_have_content "Recent Blog Posts"
  end
end
```
### Minitest Reporters
Added [Minitest Reporters gem](https://github.com/kern/minitest-reporters) to add a some color and separation to tests.  Here's a [lightning talk](https://www.youtube.com/watch?v=lPu1GwS5QU0&list=PLgUnlWmteg8rV6qFZTs8xjpJVNGv1qt_s&index=41) I gave on the different options.

### Continuous Integration
Continuous integration has been implemented with Travis-CI

### Test Coverage

Test coverage badge on this README from [COVERALLS](https://coveralls.io/)
