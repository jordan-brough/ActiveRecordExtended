# frozen_string_literal: true

require "bundler/gem_tasks"
require "rspec/core/rake_task"

RSpec::Core::RakeTask.new(:spec)

task default: :spec

task :setup do
  if File.exist?(".env")
    puts "This will overwrite your existing .env file"
  end
  db_name     = fetch_input("Enter your database name: [active_record_extended_test] ")
  db_user     = fetch_input("Enter your database user: [] ")
  db_password = fetch_input("Enter your database password: [] ")
  db_server   = fetch_input("Enter your database server: [localhost] ")

  db_name     = "active_record_extended_test" if db_name.empty?
  db_password = ":#{db_password}"             unless db_password.empty?
  db_server   = "localhost"                   if db_server.empty?
  db_server   = "@#{db_server}"               unless db_user.empty?
  env_path    = File.expand_path("./.env")

  File.open(env_path, "w") do |file|
    file.puts "DATABASE_NAME=#{db_name}"
    file.puts "DATABASE_URL=\"postgres://#{db_user}#{db_password}#{db_server}/#{db_name}\""
  end

  puts ".env file saved"
end

# rubocop:disable Metrics/BlockLength

namespace :db do
  task :load_db_settings do
    require "active_record"
    unless ENV["DATABASE_URL"]
      require "dotenv"
      Dotenv.load
    end
  end

  task drop: :load_db_settings do
    `dropdb #{ENV["DATABASE_NAME"]}`
  end

  task create: :load_db_settings do
    `createdb #{ENV["DATABASE_NAME"]}`
  end

  task migrate: :load_db_settings do
    ActiveRecord::Base.establish_connection(ENV["DATABASE_URL"])

    ActiveRecord::Schema.define do
      enable_extension "hstore"

      create_table :people, force: true do |t|
        t.integer  "tag_ids",      array: true
        t.string   "tags",         array: true
        t.integer  "personal_id"
        t.hstore   "data"
        t.jsonb    "jsonb_data"
        t.datetime "created_at"
        t.datetime "updated_at"
      end

      create_table :tags, force: true do |t|
        t.belongs_to :person
      end

      create_table :profile_ls, force: true do |t|
        t.belongs_to :person
        t.integer :likes
      end

      create_table :profile_rs, force: true do |t|
        t.belongs_to :person
        t.integer :dislikes
      end
    end

    puts "Database migrated"
  end

  task setup: :load_db_settings do
    unless ENV["DATABASE_URL"]
      Rake::Task["setup"].invoke
      Dotenv.load
    end

    Rake::Task["db:create"].invoke
    Rake::Task["db:migrate"].invoke
  end
end

# rubocop:enable Metrics/BlockLength

def fetch_input(message)
  print message
  STDIN.gets.chomp
end
