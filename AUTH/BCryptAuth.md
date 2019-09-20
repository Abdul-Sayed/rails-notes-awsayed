## gem install 'BCrypt'

Play around with BCrypt in the console
		`pass = BCrypt::Password.create("abc123")`
pass == "$2a$12$fgLLwqY24TLYCKoYQ3FJk.g7YyVQ5GqaGVZle0SMRAhk8Xmvc6q0."

pass will be an instance of BCrypt (not a string) but it outputs a string when invoked. 
It also recognizes the password its made from; pass == "abc123"   // true 


## rails g resource User name username password_digest

Now create User instances (in seed or forms) with;
		User.create(name: "Abdel", username: "OG", password: "abc123")

## In the User class, write the normal associations, but also write: - gives us setter method for password and user#authenticate
		has_secure_password
		validates :username, unique: true
	

This is the same as creating class methods in the User class;

		def password=(plain_password)
			self.password_digest = BCrypt::Password.create(plain_password)
		end

Someone is authenticated if their entered password is the same as what the encrypted hash converts to (returns true)
		def authenticate(plain_password)
			BCrypt::Password.new(password_digest) == plain_password
		end

## Generate users_controller new/create methods and users/new.html.erb template form

		def new 
			@user = User.new
		end

		def create 
			@user = User.create(user_params)
			if @user.valid?
				session[:user_id] = @user.id
				redirect_to homepage_path
			else 
				redirect_to new_user_path
			end
		end 

		private 
		def user_params
			params.require(:user).permit(:name, :username, :password)  
		end 

## Generate Auth controller (no model)

		rails g controller Auth

In AuthController;

		def new

		end

		def create 
			@user = User.find_by(username: params[:username])
			if @user && @user.authenticate(params[:password])
				flash[:message] = "Logging in #{@user.name}."
				session[:user_id] = @user.id       // session keeps the id of the user thats logged in 
				redirect_to 'homepage here'
			else
				flash[:message] = "Invalid username or password"
				redirect_to new_auth_path
			end
		end

		def destroy 
			session[:user_id] = nil
			redirect_to homepage_path
		end

In views/Auth/new.html.erb, grab their username and password using a form_tag (because Auth has no model)

		<% if flash[:message] %>
			<ul>
				<% flash[:message].each do |message| %>
					<li>
						<%= message %>
					</li>
				<% end %>
			</ul>
		<% end %>

		<h1>Log In</h1>

		<%= form_tag "/auth" do %> 
			<%= text_field_tag :username %>
			<%= password_field_tag :password %>
			<%= submit_tag "Log In %>
		<% end %>


## In config/routes.rb 

		designate a page as the homepage;
		root "page_name#index"

		resources :auth, only: [:new, :create, :destroy]   //or
		get "/login", to: "auth#new"
		post "/auth", to: "auth#create"
		delete "/auth", to: "auth#destroy"


## In homepage, 

		def index 
			@user_id = session[:user_id]
			@logged_in = !!@user_id
			if @logged_in  
				@current_user = User.find(@user_id)
			end
			...
		end


## In Application Controller, write:

		before_action :setup_auth

		def setup_auth
			@user_id = session[:user_id]
			@logged_in = !!@user_id
			if @logged_in
				@current_user = User.find(@user_id)
			end
		end

		def authorized 
			unless @logged_in 
				flash[:message] = "Nice try, you need to log in"
				return redirect_to login_page_path
			end
		end

// This allows all other pages access to @user_id, @logged_in, @current_user

In views/layouts/application.html.erb;

		<% if @logged_in %>
			<p> Logged In As: <%= @current_user.name %> </p>
			<%= button_to "Log Out", "/auth", method: :delete %>    // destroy the logged in instance :user_id from session 
		<% else %>
			<% link_to "Home", 'homepage_path' %>
			<% link_to "Login", '/login' %>
			<% link_to "Register", 'new_user_path' %>
		<% end %>

		<% if flash[:message] %>
			<p> <%= flash[:message] %> </p>
		<% end %>


## IMPORTANT:: In other controllers that display info that only a logged in user should see, write 
		before_action :authorized

No need to do that for pages that don't require a user to be logged in to access

## In other controllers using index method, do 

		before_action :authorized, only: [:index]
		@instances = @current_user.instances  // instead of @instances = Class.all

In other controllers using create methods, write;
		before_action :authorized, only: [:create]

If making a destroy method in a controller, provide for it in
		before_action :authorized, only: [:destroy]

		def destroy
			@instance = @current_user.instances.find(params[:id])
			@instance.destroy
			redirect_to instances_path
		end





<!-- JWT Notes -->

Authentication: Are you who you say you are ?


Authorization: After authenticated, are you allowed to do that action ?


