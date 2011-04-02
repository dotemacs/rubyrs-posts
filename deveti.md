
Резултат тестирања изгледа овако:

    $ bundle exec rake

    Config
      .title
        should return value
      when the title is blank
        should return nothing
      when the title is missing
        should return nil
      .store
        should return value
      .all
        should return all the values
     
    blurgh
      routes and content
        should respond to /
        should have a title
        the view
          should have a body html elements
          the posts URLs should be listed
      get_posts
        should return posts
        on the index page
          the posts URLs should be listed
          the posts titles should be shown
      get_post
        should return a post in two parts
     
    Finished in 0.02477 seconds
    13 examples, 0 failures
    
То је зато што сам одредио да формат резултата теста буде: **--format
doc** у .rspec фајлу.

Но проблем је што сам ставио:

    on the index page
      the posts URLs should be listed
      the posts titles should be shown

да је унутар **get_posts**. То би приуредим и да наместим да буде читљивије.

Тренутно **blurgh_spec.rb** изгледа овако:

    # -*- coding: utf-8 -*-
    require 'spec_helper'
     
    describe "blurgh" do
     
      def app
        @app ||= Sinatra::Application
      end
     
      context "routes and content" do
        before :each do
          YAML.should_receive(:load_file)\
           .with("setup.yaml").and_return({"title" => "Naslov", "store" => "posts"})
        end
     
        it "should respond to /" do
          get '/'
          last_response.should be_ok
        end
     
        it "should have a title" do
          get '/'
          last_response.body.should match("Naslov")
        end
     
        context "the view" do
          it "should have a body html elements" do
            get '/'
            last_response.body.should match("<body>")
          end
     
          it "the posts URLs should be listed" do
            get '/'
            # last_response.body.should match("let")
            last_response.body.should match('<a href="let">Авионски лет</a>')
          end
        end
      end
     
      describe "get_posts" do
     
        before do
          YAML.should_receive(:load_file).with("setup.yaml")\
            .and_return({"title" => "Naslov", "store" => "spec/fixtures"})
        end
     
        it "should return posts" do
          first_article = File.readlines("spec/fixtures/o-kapadokiji.md", "")[1]
          second_article = File.readlines("spec/fixtures/let.md", "")[1]
          store = Config.all['store']
     
          get_posts(store).should == \
          [[20110325,  {"url" => "let", "title" => "Авионски лет", "body" => "#{second_article}"}],\
           [20110324, {"url" => "o-kapadokiji", "title" => "Кападокија", "body" => "#{first_article}"}]]
        end
     
        context "on the index page" do
          it "the posts URLs should be listed" do
            get '/'
            last_response.body.should match("let")
            last_response.body.should match("o-kapadokiji")
          end
     
          it "the posts titles should be shown" do
            get '/'
            last_response.body.should match('<a href="let">Авионски лет</a>')
            last_response.body.should match('<a href="o-kapadokiji">Кападокија</a>')
          end
     
        end
     
      end
     
      describe "get_post" do
     
        before do
          YAML.should_receive(:load_file).with("setup.yaml")\
            .and_return({"title" => "Naslov", "store" => "spec/fixtures"})
        end
     
        it "should return a post in two parts" do
          get_post("o-kapadokiji").should have(2).parts
        end
     
      end
     
    end

Да би то променио, променим прво:

    context "routes and content" do
      before :each do
        YAML.should_receive(:load_file)\
         .with("setup.yaml").and_return({"title" => "Naslov", "store" => "posts"})
      end

у 

    context "routes and content" do
      before :each do
        YAML.should_receive(:load_file)\
         .and_return({"title" => "Naslov", "store" => "spec/fixtures"})
      end      

Покренем тестирање и оно прође.

Али сада се то понавља три пута, што да не ставим то пре почетка свих
тестова:

    before :each do
      YAML.should_receive(:load_file)\
        .and_return({"title" => "Naslov", "store" => "spec/fixtures"})
    end

а остале избришем. Покренем тест и пролази без жалби.

Вратим се предходној жељи где би да групишем сродне тестове, тако да
преуредим фајл да изгледа овако:


    # -*- coding: utf-8 -*-
    require 'spec_helper'
     
    describe "blurgh" do
     
      before :each do
        YAML.should_receive(:load_file)\
          .and_return({"title" => "Naslov", "store" => "spec/fixtures"})
      end
     
      def app
        @app ||= Sinatra::Application
      end
     
      context "routes and content" do
     
        it "should respond to /" do
          get '/'
          last_response.should be_ok
        end
     
        context "the view" do
     
          it "should have a title" do
            get '/'
            last_response.body.should match("Naslov")
          end
     
          it "should have a body html elements" do
            get '/'
            last_response.body.should match("<body>")
          end
     
          it "the posts titles should be shown" do
            get '/'
            last_response.body.should match('<a href="let">Авионски лет</a>')
            last_response.body.should match('<a href="o-kapadokiji">Кападокија</a>')
          end
     
        end
      end
     
      describe "get_posts" do
     
        it "should return posts" do
          first_article = File.readlines("spec/fixtures/o-kapadokiji.md", "")[1]
          second_article = File.readlines("spec/fixtures/let.md", "")[1]
          store = Config.all['store']
     
          get_posts(store).should == \
          [[20110325,  {"url" => "let", "title" => "Авионски лет", "body" => "#{second_article}"}],\
           [20110324, {"url" => "o-kapadokiji", "title" => "Кападокија", "body" => "#{first_article}"}]]
        end
     
      end
     
      describe "get_post" do
     
        it "should return a post in two parts" do
          get_post("o-kapadokiji").should have(2).parts
        end
     
      end
     
    end

Што је у мом мишљењу, читљивије и логичније. Покренем тест и он
прође. Да додам у гит:

    $ git status
    # On branch posts
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #	modified:   spec/blurgh_spec.rb
    #
     
    $ git add spec/blurgh_spec.rb 
    $ git commit -m "смршао blurgh_spec.rb"
    [posts 0e3722c] смршао blurgh_spec.rb
     1 files changed, 18 insertions(+), 34 deletions(-)

     

