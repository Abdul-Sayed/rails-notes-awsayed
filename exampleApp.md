# Example Rails App called bread-app

## Generate rails app

  `rails new bread-app`  

	`bundle install`


  `rails g resource Bread name:string flavor:string`  
  `rails g migration add_price_to_breads price:integer`  

	OR 

	`rails g model bread name:string flavor:string price:integer`  
	`rails g controller breads index show new edit`

## Seed with data

In seeds.rb   
  `Bread.create(name: "Pumpernickel", flavor: "Pumpernickelly, price: "10")`  
  `Bread.create(name: "Whole Wheat", flavor: "Cardboardy, price: "2")`  
  `Bread.create(name: "Peasant Bread", flavor: "Delicious, price: "1")`  
  `Bread.create(name: "Rye", flavor: "of the Tiger, price: "7")`  

Migrate tables 

	`rails db:migrate`
	`rails db:seed`


## Serve App  
  `rails s`   

## Create CRUD routes 
  `resources :breads` 

## Specify controller action & create its view
In app/controllers/breads_controller, create CRUD methods

Make breads#index Read controller action for GET '/breads'

The index page links to the new page
The index page also shows a list of model instances that can be clicked, linking to their show pages  

    def index
      @breads = Bread.all  
    end  

In app/views/breads/index.html.erb, create the view for the index method;  
Link to the new page, search for a bread, show a list of Bread instances each linking to the show page of that bread
Use `link_to bread.name, bread_path(bread.id)` or implicitly connect to show page; `link_to bread.name, bread`

		<p>
			<%= link_to "Bake a Bread", new_bread_path%>
		</p>

		<%= form_tag "/breads", method: :get do %>

			<%= text_field_tag :search_term %>
			<%= submit_tag "Search for a Bread" %>
			
		<% end %>

    <h1>List of breads in our Bread Box</h1>
    <h1>Click on a bread to be shown its details</h1>

    <ul>
      <% @breads.each do |bread|%>
        <li>

          <p>Taking user to show page using `link_to linkDescription, linkPath`</p>
          <%= link_to bread.name, bread_path(bread.id)%>

        </li>
      <% end %>
    </ul>

#

Make breads#new Read controller action for GET '/breads/new' 

Displays form to save a new instance
The new page redirects to the show page for that instance

    def new
      @bread = Bread.new
    end

In app/views/breads/new.html.erb, create the view for the new method;  
This will be a form that captures user input and creates a new instance to post to the database
Clicking on the button fires off the create method

    <p>Welcome to the Bakery</p>

    <%= form_for @bread, method: :post do |f| %>
      <%= f.label :name, "Name your bread:" %> <%= f.text_field :name %>
      <br>
      <%= f.label :flavor, "Flavor of bread:" %> <%= f.text_field :flavor %>
      <br>
      <%= f.label :price, "Bread price:" %> <%= f.number_field :price %>
      <br>
      <%= f.submit "Put it in the oven"%>
    <% end %>

# 

Make breads#create Post controller action for POST '/breads'
Create a new instance (post to the db), redirect user to show page after creating a new bread instance
Use `redirect_to bread_path(@bread.id)` or implicitly connect to show page; `redirect_to @bread`

    def create
      @bread = Bread.create(bread_params)
      redirect_to bread_path(@bread.id)
    end


    private

    # private helper method to capture form input method arguments from new page. Write below all other methods
    def bread_params
      params.require(:bread).permit(:name, :flavor, :price)
    end

There is no seperate view for the create action other than the form within the new view. 


#

Make breads#show Read controller action for GET '/breads/:id'  

The show page displays the details of a model instance
It links to the edit page and back to the index page
And provides a button to destroy the shown instance, and redirect to the index page

    def show
      find_bread
    end

private helper method to find form entered instance by id from db, as needed by multiple other methods
		private
		def find_bread
			@bread = Bread.find(params[:id])
		end

In app/views/breads/show.html.erb, create the view for the show method;  
Display the details of the bread
Link back to index page, to edit page, and provide a button to delete the bread


    <p>Name: <%= @bread.name%></p>
    <p>Flavor: <%= @bread.flavor%></p>
    <p>Price: $<%= @bread.price%></p>

    <p>To link back to index page, path shown in the prefix column in rails routes is breads so use breads_path</p>
    <%= link_to "Return to the bread box", breads_path%>

    <p>To link to edit page, rails routes shows edit_bread as path for '/breads/:id/edit' so use edit_bread_path</p>
    <%= link_to "Edit this bread", edit_bread_path(@bread)%>

		or 

    <%= link_to "Edit this bread", edit_bread_path(@bread.id)%>

    <br>

    <%= button_to "Destroy Bread", bread_path(@bread), method: :delete %>

# 

Make breads#edit Read controller action for GET '/breads/:id/edit'

Displays a form to update an instance

    def edit
      find_bread
    end

private helper method to find form entered instance by id from db, as needed by multiple other methods
		private
		def find_bread
			@bread = Bread.find(params[:id])
		end

In app/views/breads/edit.html.erb, create the view for the edit method;  

    <h1>Edit your bread</h1>

    <%= form_for @bread, method: :patch do |f| %>
      <%= f.label :name, "Name your bread:" %> <%= f.text_field :name %>
      <br>
      <%= f.label :flavor, "Flavor of bread:" %> <%= f.text_field :flavor %>
      <br>
      <%= f.label :price, "Bread price:" %> <%= f.number_field :price %>
      <br>
      <%= f.submit "Change your order"%>
    <% end %>

# 

Make breads#update Patch/Put controller action for PATCH/PUT '/breads/:id'
Update the instance in the db and redirect user to show page
Use `redirect_to bread_path(@bread.id)` or implicitly connect to show page; `redirect_to @bread`

    def update
      find_bread
      @bread.update(bread_params)
      redirect_to bread_path(@bread.id)
    end

private helper method to find form entered instance by id from db, as needed by multiple other methods
private helper method to capture form input method arguments from edit page. Write below all other methods
    private
		def find_bread
			@bread = Bread.find(params[:id])
		end
    def bread_params
      params.require(:bread).permit(:name, :flavor, :price)
    end

There is no seperate view for the update action other than the form within the edit view. 

# 

Make breads#destroy Delete controller action for DELETE '/breads/:id'
Destroy the shown instance, and redirect to the index page

    def destroy
      find_bread
      @bread.destroy
      redirect_to breads_path
    end

private helper method to find form entered instance by id from db, as needed by multiple other methods
		private
		def find_bread
			@bread = Bread.find(params[:id])
		end


# Do the same full crud for Bakeries

***
Join Model
***
---
Create join model with a column, and specify which other model(s) it belongs to. This will set up the join with foreign keys of the other models in its migration file, and establish that it belongs to the other models in its class model file. 
		`rails g resource Review content:string bread:belongs_to bakery:belongs_to`

Establish model associations in the classes of the other models. Be sure to include dependent: :destroy_all
ie for Bread:
		`has_many :reviews, dependent: :destroy`
		`has_many :bakeries, through: :reviews`
For Bakery 
		`has_many :reviews, dependent: :destroy`
		`has_many :breads, through: :reviews`

Provide seed data for join in db/seeds.rb
		Review.create(bread: Rye, bakery: Dough, content: "Rye is dry")
		Review.create(bread: Pumpernickel, bakery: Levain_Bakery, content: "Awesome")
		Review.create(bread: Peasant_Bread, bakery: Dough, content: "Yummy")

Run migration. scheme.rb should be updated with the join table
		`rails db:migrate`

Re-seed the database. First reset, to clear out any previous seeded instances 
		rails db:reset 
		rails db:seed  

Test if things were set up right
		rails c 
		> Bread.all
		> Review.all    

Example deliverables: 

1) In a bakery's show page, show a list of its reviews and a link to the reviewed bread's show page

In views/bakeries/show.html.rb ;

		<h3>Bakery Reviews:</h3>

		<ul>
			<% @bakery.reviews.each do |review| %>
					<li>
						<%= review.content  %> - <%= review.bread.name %>  <%= link_to review.bread.name, bread_path(review.bread.id) %>
					</li>
			<% end %>
		</ul>

2) In Bakeries show page, provide a button to delete each review, and refresh show page

In views/bakeries/show.html.rb, just add a delete button 
		<h3>Bakery Reviews:</h3>

		<ul>
			<% @bakery.reviews.each do |review| %>
					<li>
						<%= review.content  %> - <%= link_to review.bread.name, bread_path(review.bread.id) %>  <%= button_to "X", review_path(review.id), method: :delete %>
					</li>
			<% end %>
		</ul>

Create the destroy method in reviews_controller  -- be sure to include dependent: :destroy in the class models for bakery and bread

		def destroy
			find_review
			@bakery = @review.bakery
			@review.destroy
			redirect_to bakery_path(@bakery.id)
		end


3) Create a new Review, but redirect to the review's bakery show page. 
Review must specify the bakery and bread its associated with. Use a collection select drop down menu in the form to choose a bakery/bread

Collection select syntax; -- only used in template of join
<%= f.collection_select arg1 arg2 arg3 arg4 %>
arg1 = what column of the form instance the drop down is for; bread_id or bakery_id
arg2 = what is being iterated over, Bread.all
arg3 = sort parameter; :id   
arg4 = what the user sees; :name   

		<%= f.label :bakery_id %>
		<%= f.collection_select :bakery_id, Bakery.all, :id, :name %>

In app/controllers/reviews_controller.rb

		def new 
			@review = Review.new
		end

		def create
			@review = Review.create(review_params)
			redirect_to bakery_path(@review.bakery.id)
		end

		private

		# private helper method to find form entered instance by id from db, as needed by multiple other methods
		def find_review
			@review = Review.find(params[:id])
		end

		# private helper method to capture form input method arguments from edit page. Write below all other methods
		def review_params
			params.require(:review).permit(:content, :bakery_id, :bread_id)
		end

In app/views/reviews/new.html.erb 

<%# Display form to create a new model instance to post to database %>

<p>Create New Review</p>

<%= form_for @review, method: :post do |f| %>

  <%= f.label :content %>
  <%= f.text_field :content %>

  <%= f.label :bakery_id %>
  <%= f.collection_select :bakery_id, Bakery.all, :id, :name_address %>

  <%= f.label :bread_id %>
  <%= f.collection_select :bread_id, Bread.all, :id, :name %>

  <%= f.submit "Construct Review"%>
<% end %>

Where the bakery class has a method;

		def name_address
			"#{self.name} - #{self.address}"
		end


****
## Form Partials
***

When using the same form multiple times, 
1) create a _form.html.erb file in the views directoy and paste a form There
2) wherever your using a form, paste <%= render "form" %>


*** 
## Validations
***

 errors only get generated upon attempting to save to the database; via create and update methods. A way to check if an object is savable to the db is with object.valid?

 review1 = Review.create(content: "Da Good Content")
 review1.valid? ==> false
 review1.errors
 review1.errors.full_messages

To require that certain columns be filled before saving to the db, in the class model, Write
		validates :column_name, presence:true
		validates :column_name, presence:true, uniqueness: true   --> if it should be unique

Example for bread, in bread.rb

		validates :name, presence: true, uniqueness: true
		validates :flavor, presence: true
		validates :price, presence: true, numericality: { only_integer: true, :greater_than => 0 }
		validate :name_cannot_be_ciabatta

		def name_cannot_be_ciabatta
			if name.downcase == "ciabatta"
				errors.add(:name, "can't be called ciabatta")
			end
		end

Prevent redirects to show page if saving to db failed;

		def create
			@bread = Bread.create(bread_params)
			if @bread.valid?
				redirect_to bread_path(@bread.id)
				# redirect_to @bread
			else
				flash[:errors] = @bread.errors.full_messages
				redirect_to new_bread_path
			end
		end

In new page,

		<% if flash[:errors] %>
			<ul>
				<% flash[:errors].each do |error_message| %>
					<li>
						<%= error_message %>
					</li>
				<% end %>
			</ul>
		<% end %>
	...

Do same as above for update method and edit view

