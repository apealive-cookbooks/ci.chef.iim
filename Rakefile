require 'rspec/core/rake_task'
require 'rubocop/rake_task'
require 'foodcritic'
require 'kitchen'
require 'emeril/rake_tasks'

desc 'Cleans the workspace and the bundle dir !!!!, bundle install required afterwards.'
task :clean do
  rm_rf %w(.bundle build berks-cookbooks Gemfile.lock Berksfile.lock .kitchen .vagrant .rakeTasks)
end

desc 'Run all style checks'
task :style => ['style:chef', 'style:ruby']

desc 'Run all tests'
task :test => ['test:unit', 'test:kitchen']

desc 'Regenerate README.md'
task :doc do
  `bundle exec knife cookbook doc .`
end

desc "Run all regular tasks"
task :default => ['style', 'doc']

namespace :style do
  desc 'Run Ruby style checks'
  RuboCop::RakeTask.new(:ruby) do |task|
    task.patterns = ['**/*.rb']
    # only show the files with failures
    task.formatters = ['simple']
    # don't abort rake on failure
    task.fail_on_error = true
  end

  desc 'Run Chef style checks'
  FoodCritic::Rake::LintTask.new(:chef) do |t|
    t.options = {
            :tags => %w(FC0034),
            :fail_tags => ['any']
    }
  end
end

namespace :test do
  desc 'Run Test Kitchen with Vagrant'
  task :kitchen do
    Kitchen.logger = Kitchen.default_file_logger
    Kitchen::Config.new.instances.each do |instance|
      instance.test(:always)
    end
  end

  desc 'Runs specs with chefspec.'
  RSpec::Core::RakeTask.new(:unit, [:recipe, :output_file]) do |t, args|
    args.with_defaults(:recipe => '*', :output_file => nil)
    t.verbose = false
    t.fail_on_error = true
    t.rspec_opts = '--color --format progress --format RspecJunitFormatter --out build/chefspec-reports.xml'
    t.ruby_opts = '-W0' #it supports ruby options too
    t.pattern = "spec/#{args.recipe}_spec.rb"
  end
end

desc "Releases Cookbook"
Emeril::RakeTasks.new do |t|
  # turn on debug logging
  t.config[:logger].level = :debug

  # set a category for this cookbook
  t.config[:category] = "Library"

  t.config[:publish_to_supermarket] = false
end