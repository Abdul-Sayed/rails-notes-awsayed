 Because Rails is only used as a backend API, it no longer controls views and doesn't have access to sessions. 
Another way of securing sessions must be implemented.  

	task manager -> kill all postgres instances
	rails new auth_api --api --database=postgresql -T

In Gemfile, uncomment 'bcrypt', 'rack-cors' and add `gem 'jwt'` and bundle install

In config/initializers/cors.rb and config/application.rb uncomment and enable CORS (uncommented);


	rails g model User username password_digest

	class User < ApplicationRecord
		has_secure_password
	end

	rails g controller Auth 
	rails g controller Users

	sudo service postgresql start
	rails db create
	rails db:migrate
	rails db:seed
	rails s -p 3001



### SETUP for 3 models: bird --< sighting >-- location

	rails new bird-watcher --api --database=postgresql
	gem 'rack-cors'
	gem 'active_model_serializers', '~> 0.10.0'
	gem 'fast_jsonapi'
	gem 'rest-client', '~> 2.0.2'
	gem 'json', '~> 2.1.0'
	gem 'faker', '~> 1.6', '>= 1.6.6'
	bundle install

In config/application.rb, ensure this is enabled (uncommented);

	config.middleware.insert_before 0, Rack::Cors do
		allow do
			origins '*'
			resource '*', headers: :any, methods: [:get, :options]
		end
	end

Also, in config/initializers/cors.rb, uncomment out:

	Rails.application.config.middleware.insert_before 0, Rack::Cors do
		allow do
			origins "*"  # ideally should be "localhost:3000, or hosting domain address"

			resource "*",
				headers: :any,
				methods: [:get, :post, :put, :patch, :delete, :options, :head]
		end
	end

Generate models/migrations/controllers
	rails g resource bird name species
	rails g resource location latitude longitude 
	rails g resource sighting bird:references location:references

Generate Active Model Serializers for the models
You wont have to if you used --api flag during rails new
	rails g serializer bird
	rails g serializer location
	rails g serializer sighting

Update the end models with has_many;

	class Bird < ApplicationRecord
		has_many :sightings
		has_many :locations, through: :sightings

		validates :name, uniqueness: true 
	end

	class Location < ApplicationRecord
		has_many :sightings
		has_many :birds, through: :sightings
	end

Fill seeds.rb with instances

Define Routes 

	Rails.application.routes.draw do
		resources :birds, only: [:index, :show, :create, :update]
		# OR //=> get '/birds', to: 'birds#index'
		# OR //=> get '/birds' => 'birds#index'
	end

Start postgresql & run migrations
	task manager -> kill all postgres instances
	sudo service postgresql start
	rails db:create
	rails db:migrate
	rails db:seed
	rails s

Create serializers, and their attributes 
Attributes can include any other custom instance methods defined in the class
Also include the relationships

	class BirdSerializer < ActiveModel::Serializer 
		include FastJsonapi::ObjectSerializer
		attributes :id, :name, :species
		has_many :sightings
		has_many :locations
	end

	class SightingSerializer < ActiveModel::Serializer 
		include FastJsonapi::ObjectSerializer
		attributes :id, bird_id, location_id
	end

	class LocationSerializer < ActiveModel::Serializer 
		include FastJsonapi::ObjectSerializer
		attributes :id, :latitude, :longitude
		has_many :sightings
		has_many :birds
	end



Define controller actions
	class BirdsController < ApplicationController

		def index
			birds = Bird.all    //=> or Bird.order('id ASC')   
			render json: birds, include: [:name, :species]
			# same as //=> render json: birds, only: [:id, :name, :species]
			# //=> Another option: render json: birds, except: [:created_at, :updated_at]    
			### To show nested models, use render json: birds, include '**'  
			### or  render json: {info: BirdSerializer.new(@bird)}, include '**'
		end

		def show
			find_bird
			if bird
				render json: bird, only: [:id, :name, :species]
			else
				render json: { message: 'Bird not found' }
			end
		end

		def create 
			bird = Bird.new(bird_params)
			if bird.save 
				render json: bird, include: [:name, :species]
			else
				render json: { message: 'Bird cannot be saved' }
			end
		end

		def update  
			find_bird
			bird.update(bird_params)
			render json: bird, include: [:name, :species]
		end


		private 

		def find_bird
			bird = Bird.find(params[:id])
		end

		def bird_params
			params.permit(:name, :species)
		end

	end


For Join 

	def index
		sightings = Sighting.all
		render json: sightings
	end

	def show
		sighting = Sighting.find_by(id: params[:id])
		options = {
			include: [:bird, :location]
		}
		render json: include: [:sighting, :options]
	end

