
<!-- Environment Setup -->
	rails new appNameBackEnd --api --database=postgresql -T

In Gemfile, ensure these gems and bundle install/update
	gem 'jwt'
	gem 'bcrypt'
	gem 'rack-cors'
	gem 'active_model_serializers'
	gem 'fast_jsonapi'
	gem 'rest-client'
	gem 'json',
	gem 'faker'

In config/initializers/cors.rb and config/application.rb uncomment and enable CORS;

	config.middleware.insert_before 0, Rack::Cors do
		allow do
			origins '*'    # ideally should be "localhost:3000, or hosting domain address"
			resource '*', headers: :any, methods: [:get, :post, :put, :patch, :delete, :options, :head]
		end
	end


Delete existing config/credentials.yml.enc file. Then in a terminal in same directory, enter:
	EDITOR='code --wait' rails credentials:edit
where code is for VSCODE, and use atom for atom, etc.

In new credentials.yml window that opens, type, save, and close:
	jwt_secret: 'someAppSecret'

Then in config/credentials.yml.enc, your jwt_secret will be encrypted. 
To check is its there, open rails console, and enter
	Rails.application.credentials.jwt_secret

Your .gitignore file should have this;
	# Ignore master key for decrypting credentials and more.
	/config/master.key


<!-- File Generation -->
Generate models/migrations/controllers.

This example is for a two model; user =< note
For User, include password_digest
	rails g resource user username:string password_digest
	
	rails g resource note title:string body:string user:references

Generate controller only for Auth (doesn't need models or migrations)
	rails g controller auth

Generate Active Model Serializers for the models
(You shouldn't have to if you used --api flag during rails new)
	rails g serializer user
	rails g serializer note

<!-- Models -->
In the User class, include this in addition to its normal associations,

	class User < ApplicationRecord
			has_secure_password
			has_many :notes
			validates :username, unique: true
	end

Complete the other class models;

	class Note < ApplicationRecord
		belongs_to :user
	end

<!-- Migrations -->

Should look like this;

	class CreateUsers < ActiveRecord::Migration[5.2]
		def change
			create_table :users do |t|
				t.string :username
				t.string :password_digest

				t.timestamps
			end
		end
	end

	class CreateNotes < ActiveRecord::Migration[5.0]
		def change
			create_table :notes do |t|
				t.integer :user_id
				t.string :title
				t.string :body
				t.timestamps
			end
		end
	end

Schema

	create_table "users", force: :cascade do |t|
		t.string "username"
		t.string "password_digest"
		t.datetime "created_at", null: false
		t.datetime "updated_at", null: false
	end

	create_table "notes", force: :cascade do |t|
    t.integer  "user_id"
    t.string   "title"
    t.string   "body"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
	end	

<!-- Routing -->
Define Routes 

	Rails.application.routes.draw do

		# Custom Routes //=> get '/users', to: 'users#action'
		# OR //=> get '/users' => 'users#action'
		post '/login', to: 'auth#login'	
		post '/signup', to: 'users#create'
		get '/profile', to: 'users#profile'
		
		resources :users, only: [:show, :create, :update, :delete]
		resources :notes
	end

<!-- Serializers -->
Create serializers, and their attributes 
Attributes can include any other custom instance methods defined in the class
Also include the relationships

	class UserSerializer < ActiveModel::Serializer 
		include FastJsonapi::ObjectSerializer
		attributes :id, :username, :password
		has_many :notes

	end

	class NoteSerializer < ActiveModel::Serializer 
		include FastJsonapi::ObjectSerializer
		attributes :id, :title, :body, :updated_at
		belongs_to :user
	end


<!-- Controllers -->
Define controller actions

class ApplicationController < ActionController::API
  def token
    request.headers["Authorization"].split(" ")[1] if request.headers["Authorization"]
		# request.headers["Authorization"].split(" ")[1].split("\"")[1]
  end

  def secret
    # ENV['jwt_secret']
    Rails.application.credentials.jwt_secret
  end

  def decoded_token
    JWT.decode(token, secret, true, { algorithm: 'HS256' })
  end

  def current_user
    User.find(decoded_token[0]["user_id"]) if decoded_token
  end

  def create_token(user_id)
    payload = { user_id: user_id }
    JWT.encode(payload, secret, 'HS256')
  end
end

<!--  -->

class AuthController < ApplicationController
  def login
    # find the user
    user = User.find_by(username: params[:username])
    # if user exists and they are really the user via password, send the returning user a token
    if user && user.authenticate(params[:password])
      render json: {token: create_token(user.id)}
    else
      render json: {errors: ["Invalid username or password"]}, status: 422
    end

  end
end

<!--  -->

class UsersController < ApplicationController
  def profile
    render json: current_user
  end

  def create
    user = User.new(user_params)
    if user.valid?
      (render json: { token: create_token(user.id) }, user, include: [:username], status: 201) if user.save
    else
      render json: { errors: user.errors.full_messages }, status: 422
    end
  end

  private

  def user_params
    params.permit(:username, :password)
  end
end

<!--  -->

class NotesController < ApplicationController
  before_action :find_note, only: [:show, :update, :destroy]

  def index
    notes = Note.all
    render json: notes, status: 200
		# or render json: notes, include: [:title, :body]
		# same as //=> render json: notes, only: [:id, :title, :body]
		# //=> Another option: render json: notes, except: [:created_at, :updated_at]    
		# To show nested models, use render json: notes, include '**'  
  end

  def show
			if note
				render json: note, status: 200
				# or render json: note, only: [:id, :title, :body]
			else
				render json: { message: 'Note not found' }
			end
  end

  def create
    # note = Note.create(note_params)
    # render json: note, status: 201

		note = Note.new(note_params)
		if note.valid?
			(render json: note, include: [:title, :body], status: 201) if note.save
		else
			render json: { message: 'Note cannot be saved' }
		end
  end

  def update
    note.update(note_params)
    # render json: note, status: 200
		render json: note, include: [:title, :body], status: 200
  end

  def destroy
    noteId = note.id
    note.destroy
    render json: { message: "Zap! note deleted", noteId: noteId }
  end

  private

  def find_note
    note = Note.find(params[:id])
  end

  def note_params
    params.permit(:title, :body, :user_id)
  end
end


<!-- Seeding -->
Fill seeds.rb with instances

User.create(username: "Abdel", password: "abc123")
Note.create(title: "Take shower", body: "Use bar of soap")

<!-- 10.times do
  Note.create(user: User.create(username: Faker::Name.name), title: Faker::Lorem.sentence(rand(4) + 1, true), body: Faker::Lorem.paragraphs(3, true).join('\n'))
end -->
	
<!-- Serve App -->
task manager -> kill all postgres instances
	sudo service postgresql start
  rake db:create
	rails db:migrate
	rails db:seed
	rails s -p 3001
Enter rails routes in terminal or go to http://localhost:3001/rails/info/routes



