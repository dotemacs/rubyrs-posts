Сада да претворим фајлове у HTML линкове.

Прво направим тест под **describe "get_posts"**:

    it "the posts titles should be shown" do
      get '/'
      last_response.body.should match('<a href="let">Авионски лет</a>')
      last_response.body.should match('<a href="o-kapadokiji">Кападокија</a>')
    end

Онда да подесим **index.erb**:

    <h1><%= @title %></h1>
     
    <% @posts.each do |post| %>
       <a href="<%= post[1]['url'] %>"><%= post[1]['title'] %></a>
    <% end %>

покренем тест, и он успешно прође. Да додам напредак у гит:

    $ git status
    # On branch posts
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #	modified:   spec/blurgh_spec.rb
    #	modified:   views/index.erb
    #
     
    $ git add spec/blurgh_spec.rb views/index.erb
     
    $ git commit -m "омогућено претварање наслова чланака у URL на почетној страници"
    [posts b1ad84c] омогућено претварање наслова чланака у URL на почетној страници
     2 files changed, 8 insertions(+), 1 deletions(-)
     
Било би добро да сада могу да видим и саме чланке. Пошто су они сами
фајлови, треба ми нека метода која ће да их чита. Да кренем са тестом:


    describe "get_post" do
      pending
    end
  
покренем тестирање:

    $ bundle exec rspec spec/blurgh_spec.rb 

и оно пада са:

    Pending:
      blurgh get_post 
        # Not Yet Implemented
        # ./spec/blurgh_spec.rb:69
  
Када размислим мало боље о овоме, како ће се фајлови читати, то ће
бити у суштини овако:

    /име-фајла

Но како они треба да изгледају кад су прочитани. Слично као и када их
**get_posts** чита, сачињени од два дела. Први део је YAML
конфигурација а други, сам садржај.

Тако да пишем следећи тест:

    describe "get_post" do
      it "should return a post in two parts" do
        get_post("o-kapadokiji").should have(2).parts
      end
    end

Покренем тест, који пада, зато пишем следећу методу:

    def get_post(post)
      header, body = File.readlines(Config.options['store'] + "/" + post + ".md", "")
    end

покренем тест и он пада:

    1) blurgh get_post should return a post in two parts
       Failure/Error: get_post("o-kapadokiji").should have(2).parts
       NoMethodError:
         undefined method `+' for nil:NilClass
       # ./blurgh.rb:46:in `get_post'
       # ./spec/blurgh_spec.rb:76:in `block (3 levels) in <top (required)>'

46 линија је средина предходне методе и она очекује конфигурацију за
**Config**. Зато преправим тест да изгледа овако:

    describe "get_post" do
     
      before do
        YAML.should_receive(:load_file).with("setup.yaml")\
          .and_return({"title" => "Naslov", "store" => "spec/fixtures"})
      end
     
      it "should return a post in two parts" do
        get_post("o-kapadokiji").should have(2).parts
      end
     
    end

Покренем тест, који успешно прође.

Па додам то у гит:

    $ git status
    # On branch posts
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #	modified:   blurgh.rb
    #	modified:   spec/blurgh_spec.rb
    no changes added to commit (use "git add" and/or "git commit -a")
     
    $ git add blurgh.rb spec/blurgh_spec.rb
     
    $ git commit -m "учитавање чланка"
    [posts b1290dd] учитавање чланка
     2 files changed, 17 insertions(+), 0 deletions(-)
     
Овде би било добро да обратим пажњу на:

    get_post("o-kapadokiji").should have(2).parts     
    
специфично на **have(2).parts** део. **parts** је само реч која је ту
додана да би тест био читљивији. Она може бити замењена са неком
другом и тест ће и даље да прође. На пример:

    get_post("o-kapadokiji").should have(2).сарма
    
пролази као важећи тест.

Ако заменим **сарма** са **class**, добијам следећу грешку:

    1) blurgh get_post should return a post in two parts
         Failure/Error: get_post("o-kapadokiji").should have(2).class
         NoMethodError:
           undefined method `matches?' for RSpec::Matchers::Have:Class
         # ./spec/blurgh_spec.rb:76:in `block (3 levels) in <top (required)>'    
	 
што ми указује на то да је **have(2)** сачињена од **Have** класе. Ако погледам [https://github.com/rspec/rspec-expectations/blob/master/lib/rspec/matchers/have.rb](https://github.com/rspec/rspec-expectations/blob/master/lib/rspec/matchers/have.rb) видим следеће под примерима:

    #   should have(number).named_collection__or__sugar
    
... 

    # If the receiver IS the collection, you can use any name
    # you like for <tt>named_collection</tt>. We'd recommend using
    # either "elements", "members", or "items" as these are all
    # standard ways of describing the things IN a collection.

па примери:
    
    # Passes if [1,2,3].length == 3
    # [1,2,3].should have(3).items #"items" is pure sugar
    
    
Наставак следи...
