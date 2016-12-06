# Coding standard

## Bundler

* Put gems in alphabetical order in the Gemfile.

* Do not use any unnecessary gems in your project.

* Use only established gems in your projects. If you're contemplating on
  including some little-known gem you should do a careful review of its source
  code first.

* If your project needs different gems for different environments, then group it for that specific environment.
  ```ruby
  group :development, :test do
    gem 'mysql2'   #mysql for development
    gem 'pry-rails', "~> 0.3.4"
  end

  group :production do
    gem 'pg'    #postgresql for production
  end
  ```
  
* Do not remove the `Gemfile.lock` from version control. This is not some
  randomly generated file - it makes sure that all of your team members get the
  same gem versions when they do a `bundle install`.
  
## Migrations

* Keep the `schema.rb` under version control and arrange it well, so that it clearly understandable.

* Use `rake db:schema:load` instead of `rake db:migrate` to initialize an empty
  database.

* Add database level validation as well model level validation

* Enforce foreign-key constraints.

* When writing constructive migrations (adding tables or columns),
  use the `change` method instead of `up` and `down` methods.
  ```Ruby
  # the old way
  class AddNameToPeople < ActiveRecord::Migration
    def up
      add_column :people, :name, :string
    end

    def down
      remove_column :people, :name
    end
  end

  # the new preferred way
  class AddNameToPeople < ActiveRecord::Migration
    def change
      add_column :people, :name, :string
    end
  end
  ```
  
## Routing

* When you need to add more actions use `member` and `collection` routes.

  ```Ruby
  # bad
  get 'subscriptions/:id/unsubscribe'
  resources :subscriptions

  # good
  resources :subscriptions do
    get 'unsubscribe', on: :member
  end

  # bad
  get 'photos/search'
  resources :photos

  # good
  resources :photos do
    get 'search', on: :collection
  end
  ```

* If you need to define multiple `member/collection` routes use the
  alternative block syntax.

  ```Ruby
  resources :items do
    member do
      get 'allocate'
      put 'reallocate'
      # more routes
    end
  end

  resources :photos do
    collection do
      get 'search'
      # more routes
    end
  end
  ```

* Use nested routes to express better the relationship between ActiveRecord
  models.

  ```Ruby
  class Post < ActiveRecord::Base
    has_many :comments
  end

  class Comments < ActiveRecord::Base
    belongs_to :post
  end

  # routes.rb
  resources :posts do
    resources :comments
  end
  ```
* Skip unnecessary routes if those routes are not using `except/only`
  
  ```ruby
  resources :users, except: [:show]
  resources :employee, except: [:show, :edit, :update]
  ```
  
## Controllers

* Keep the controllers skinny - they should only retrieve data for the view
  layer and shouldn't contain any business logic (all the business logic
  should naturally reside in the model).

* If more than one action in a controller contains same lines of code, then write
  it into seperate a method and use callback funcation to invoke it on those action.
  
  ```ruby
  # bad
  # controller.rb
  def edit
    @category = Category.find(params[:id])  
  end

  def show
    @category = Category.find(params[:id])
    @category_items = @category.items.paginate(page: params[:page], per_page: 5)
  end
  
  #good
  # controller.rb
  before_action :get_category, only: [:edit, :show]
  def show
    @category_items = @category.items.paginate(page: params[:page], per_page: 5)
  end
  
  def get_category
    @category = Category.find(params[:id])
  end
  ```
  
