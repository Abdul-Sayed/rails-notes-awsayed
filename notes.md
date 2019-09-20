
## Creating an empty rails app; `rails new app_name`
  	`rails new bread-app`  
		`bundle install`

-------------------------------------------------------------------
## Rails Generators - rails generate resource is preferred

### `rails g resource model_name` Generates migration + model + controller + view folder
==> generates migration file in db/migrate, empty model in app/models, breads_controller file in app/controllers, and an empty breads folder in app/views/  
  `rails g resource bread`  
  Populate migration table with columns; `rails g resource bread name:string flavor:string`  
	
### `rails g migration create_[model_name_in_plural] colm1:type colm2:type`  Generates migration
==> generates migration file in db/migrate  
  For an empty table; `rails g migration create_breads`  
  To initialize table with columns; `rails g migration create_breads name:string flavor:string`   

### `rails g model model_name` Generates model + migration
==> generates an empty model in app/models, generates migration file in db/migrate
  `rails g model bread`  
  For migration table to have added columns; `rails g model bread name:string flavor:string`  

### `rails g controller [model_name_in_plural] cnt_act1 cnt_act2` Generates controller + view folder with empty view files it
==> generates breads_controller file in app/controllers and creates an empty breads folder in app/views/      
  `rails g controller breads index show new edit`


-------------------------------------------------------------------
### Adding columns to migration table; 
`rails g migration add_columnName_to_tableName columnName:type`

	rails g migration add_price_to_breads price:integer

------------------------------------------------------------------
### Adding Model relationships

	rails g resource className model1:belongs_to model2:belongs_to
	rails g resource className model1:references model2:references

-------------------------------------------------------------------
### Seeding Data in db/seeds.rb 
Use Model_name.create(arg1, arg2, ...) to create multiple instances of the model as seed data

		Bread.create(name: "Pumpernickel", flavor: "Pumpernickely", price: 10)
		Bread.create(name: "Whole Wheat", flavor: "Wheaty", price: 2)
		Bread.create(name: "Rye", flavor: "Dry", price: 5)
		Bread.create(name: "Sourdough", flavor: "sour", price: 6)

-------------------------------------------------------------------
### Migrate tables 

	`rails db:migrate`
	`rails db:seed`  or  `rails db:reset`

-----------------------------------------------------------------------
### Create Routes

In config/routes.rb, explicit way to specify rest routes: `get "url", to: "controller#action", as: alt_URL`

    get "/breads", to: "breads#index", as: "breads"  
    post "/breads", to: "breads#create", as: "breads"  
    get "/breads/new", to: "breads#new", as: "new_bread"  
    get "/breads/:id/edit", to "breads#edit", as: edit_bread  
    get "/breads/:id", to: "breads#show", as: "bread"  
    patch "/breads/:id", to "breads#update", as: "bread"  
    put "/breads/:id", to "breads#update", as: "bread"  
    delete "/breads/:id", to "breads#destroy", as: "bread"  

When only using some of the rest routes  
		resources :breads, only: [:index, :create, :new, :edit, :show]

Alternative way to create full CRUD routes  
		`resources :breads` 

### rails routes Full CRUD route prefixes

To see route paths,
Enter rails routes in terminal or go to http://localhost:3000/rails/info/routes

        Prefix Verb   URI Pattern                    Controller#Action
        _______________________________________________________________
        breads GET    /breads(.:format)              breads#index
        ----------------------------------------------------------
        breads POST   /breads(.:format)              breads#create
        ----------------------------------------------------------
    new_bread GET    /breads/new(.:format)          breads#new
        ----------------------------------------------------------
    edit_bread GET    /breads/:id/edit(.:format)     breads#edit
        ----------------------------------------------------------
        bread GET    /breads/:id(.:format)          breads#show
        ----------------------------------------------------------
        bread PATCH  /breads/:id(.:format)          breads#update
        ----------------------------------------------------------
        bread PUT    /breads/:id(.:format)          breads#update
        ----------------------------------------------------------
        bread DELETE /breads/:id(.:format)          breads#destroy

The index page links to the new page where a new instance can be created (posted to the db)
The new page redirects to the show page for that instance
The index page also shows a list of model instances that can be clicked, linking to their show pages

The show page displays the details of a model instance 
It links back to the index page 
It also links to the edit page, where an instance can be updated, redirecting back to the show page upon updating
It also provides a button to destroy the shown instance, and redirecting to the index page 

-------------------------------------------------------------------












