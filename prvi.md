Блог систем у синатри, епизода прва

Идеја овог чланка је писање апликације у синатри, са тестирањем и
гитом. Предходно знање ових технологија/апликација није потребно.

Блог систем треба да има YAML фајл у коме се убаци наслов, и место где
ће да се држе чланци.

Од самог почетка користићу тестирање:

    $ mkdir blurgh
    $ cd blurgh 
    $ emacs Gemfile

и додам:

    source 'http://rubygems.org'
     
    gem 'sinatra'
    gem 'rack-test'
    gem 'rspec'
    gem 'rake'
    
онда инсталирам:
   
    $ gem install -r bundle 
    $ bundle install    

Да би омогућио да RSpec функционше, требам да направим следеће фајлове:

    $ mkdir spec
    $ emacs spec/spec_helper.rb

И додам следеће:

    require File.join(File.dirname(__FILE__), '..', 'blurgh')
     
    require 'sinatra'
    require 'rack/test'
    require 'rspec'
     
     
    # set test environment
    set :environment, :test
    set :run, false
    set :raise_errors, true
    set :logging, false
     
     
    RSpec.configure do |c|
      c.include Rack::Test::Methods
    end    
    
Кључни део овде је додавање: Rack::Test::Methods. То омогућава
тестирање уз помоћ [**rack**](http://rack.rubyforge.org/).

Онда направим први тест:

    $ emacs spec/blurgh_spec.rb
    
и напишем први тест:    

    require 'spec_helper'
     
    describe "blurgh" do
     
      def app
        @app ||= Sinatra::Application
      end
     
      it "should respond to /" do
        get '/'
        last_response.should be_ok
      end
    end

да покренем тај тест:

    $ bundle exec rspec spec/blurgh_spec.rb

он наравно неће да прође јер још увек нема главне апликације, зато

    $ emacs blurgh.rb

и додам следеће:

    #!/usr/bin/env ruby
     
    require 'sinatra'
     
    get '/' do
    end

поново покренем тестирање:

    $ bundle exec rspec spec/blurgh_spec.rb
    
и избаци следеће резултате:

    $ bundle exec rspec spec/blurgh_spec.rb 
    .
     
    Finished in 0.01479 seconds
    1 example, 0 failures

да би то било мало разговетније, покренем тестирање са "документационом" опцијом и бојом, овако:

    $ bundle exec rspec -f doc --colour spec/blurgh_spec.rb 
     
    blurgh
      should respond to /
     
    Finished in 0.01517 seconds
    1 example, 0 failures


То је добар почетак, и то ћу да убацим у гит:

    $ git init
    $ emacs .gitignore

и ту ставим следеће:

    Gemfile.lock

    ## Mac OS
    .DS_Store
     
    ## Textmate
    *.tmproj
    tmtags
     
    ## Emacs
    *~
    \#*
    .\#*
     
    ## Vim
    *.swp

додам фајлове у гит:

    $ git add .
    $ git status
   
који врати следеће:

    # On branch master
    #
    # Initial commit
    #
    # Changes to be committed:
    #   (use "git rm --cached <file>..." to unstage)
    #
    #	new file:   .gitignore
    #	new file:   Gemfile
    #	new file:   blurgh.rb
    #	new file:   spec/blurgh_spec.rb
    #	new file:   spec/spec_helper.rb
    #

онда упишем те промене фајлове:

    $ git commit -m "сам почетак"

што испише следеће резултате

    [master (root-commit) ea5dd64] сам почетак
     5 files changed, 58 insertions(+), 0 deletions(-)
     create mode 100644 .gitignore
     create mode 100644 Gemfile
     create mode 100755 blurgh.rb
     create mode 100644 spec/blurgh_spec.rb
     create mode 100644 spec/spec_helper.rb

Али пошто је то досадно да се стално куца, користићу rake да то аутоматизујем:

    $ emacs Rakefile
    
у коју додам ово:

    require 'rspec/core/rake_task'
     
    RSpec::Core::RakeTask.new do |t|
      t.rspec_opts = ["-r ./spec/spec_helper.rb"]
      t.pattern = 'spec/**/*_spec.rb'
    end

то омогућује RSpec да покреће тестове када се укуца: 

    $ bundle exec rake spec 
    
Али пошто ће то бити честа радња, да омогућим да куцам што мање,
додајем и ово:
     
    task :default => "spec"

Тако да могу да укуцам само:

    $ bundle exec rake 

Да би имао боју и опис тестова, отворим нови фајл **.rspec** и у њега убацим:

    --colour --format doc

Сада резултат изгледа боље:

    
    $ bundle exec rake 
     
    blurgh
      should respond to /
     
    Finished in 0.0092 seconds
    1 example, 0 failures

То сада може да се дода у гит:

    $ git status
    
ми покаже који су нови фајлови које могу да додам:

    $ git add Rakefile .rspec
    $ git commit -m "додајем Rakefile и .rspec"

