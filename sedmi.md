
Било би пожељно да овај систем може да учита чланке и да их
прикаже. Идеја је да чита чланке из одређеног директоријума који је
наведен у конфигурационом фајлу setup.yaml на следећи начин:

    store: posts

За то да направим нови огранак у гиту:

    $ git checkout -b posts

Прво желим да Config чита и .store врати име директоријума. То означим
у тесту овако:

    describe ".store" do
      it "should return value" do
        YAML.should_receive(:load_file).with("setup.yaml").and_return({"store" => "posts"}) 
        Config.store.should match("posts")
      end
    end

покренем тест и он не прође:
     
    1) Config.store should return value
       Failure/Error: Config.store.should match("posts")
       NoMethodError:
         undefined method `match' for nil:NilClass
     
Онда у Config додам нову методу:
     
    def self.store
      options["store"]
    end
    
Поново покренем тест, и овај пут прође. Додам тај рад у гит:
  
    $ git status
    # On branch posts
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #	modified:   blurgh.rb
    #	modified:   spec/config_spec.rb
    #
    no changes added to commit (use "git add" and/or "git commit -a")
    $ git add blurgh.rb spec/config_spec.rb
    $ git commit -m "додана метода Config.store"
    [posts b243446] додана метода Config.store
     2 files changed, 12 insertions(+), 0 deletions(-)

Сада имам тачно одређено место где ће да се држе чланци, али још немам
ништа да их учитам. То желим да ми уради метода get_posts која ће да
врати све фајлове директоријума на који указује store. Тест за ту
методу изгледа овако:


    describe "get_posts" do
      it "should return posts" do
        Dir.should_receive(:entries).with("posts").and_return(["sample_post.md", "another_sample_post.md"])
        get_posts.should == ["sample_post.md", "another_sample_post.md"]
      end
    end

И кад покренем тестове, добијем следеће:
    
     1) blurgh get_posts should return posts
    Failure/Error: get_posts.should == ["sample_post.md", "another_sample_post.md"]
    NameError:
      undefined local variable or method `get_posts' for #<RSpec::Core::ExampleGroup::Nested_2::Nested_2:0x00000001f12da8>

Напишем следећу методу у **blurgh.rb**:

    def get_posts
      Dir.entries(Config.store)
    end

покренем тестирање поново и тест прође.

Но то ми није довољно, јер како ћу да организујем чланке, овако су
враћени у "хрпи". Било би боље када би они могли да буду организовани
по датуму.

И како ћу за наслов ?

То би могло да се реши овако: сваки чланак да буде сачињен делимично у
YAML формату, то јест, први параграф, а остатак да буде сам чланак. На
пример да садржај фајла изгледа овако:

    title: Кападокија
    date: 20110324
    
    Премало се зна о Кападокији пре него што је постала персијска
    сатрапија. Зна се само да је Кападокија била у време Хета веома
    значајна земља и да се управо у њој налазила хетска престоница
    Хатуша. Иако сатрапија у персијско време, Кападокија је сачувала
    известан степен самосталности.
    

То би било у реду за један чланак, али шта са get_posts методом ? У
каквом формату да врати чланке ?

Било би добро да буду враћени у следећем формату:


    [[["20110324"], {"title" => "Кападокија", "body" => "Премало се зна о Кападокији..."}]]

На тај начин могу лако да сортирам чланке. 


И шта још би могао да ту додам? Адресу, тј. URL сваког чланка. Намерно
бих да title буде наслов у самој страници а да име фајла буде
URL. Тако да ако предходни пример чланак назовем: o-kapadokji.md,
формат који би **get_posts** метода враћала би био:

    [["20110324", {"url" => "o-kapadokiji", "title" => "Кападокија", "body" => "Премало се зна о Кападокији..."}]]


Да то направим да буде направићу два фајла која ћу да користим за
тестирање:

    $ mkdir spec/fixtures
    $ emacs spec/fixtures/o-kapadokiji.md 
    
И ту ставим горе наведени текст. Онда отворим други фајл:
    
    $ emacs spec/fixtures/let.md
    
Ставим следећи текст:

    title: Авионски лет
    date: 20110325
     
    25. марта 1923. - Слетањем авиона на линији Париз-Цариград на аеродром
    у Панчеву, Краљевина СХС постала део тада малобројне породице држава
    повезаних ваздушним саобраћајем.

Сада да преправим тест:


    describe "get_posts" do
      it "should return posts" do
        first_article = File.readlines("spec/fixtures/o-kapadokiji.md", "")[1]
        second_article = File.readlines("spec/fixtures/let.md", "")[1]
        YAML.should_receive(:load_file).with("setup.yaml").and_return({"title" => "Naslov", "store" => "spec/fixtures"}) 
       
        get_posts.should == \
        [[20110325,  {"url" => "let", "title" => "Авионски лет", "body" => "#{second_article}"}],\
         [20110324, {"url" => "o-kapadokiji", "title" => "Кападокија", "body" => "#{first_article}"}]]
       
      end
    end
  
Да објасним овде да **first_article** и **second_article** су само
фајлови учитани, да не бих писао цео њихов садржај. И **store** сада
чита фајлове из **spec/fixtures**.

Покренм тестирање:

    $ bundle exec rspec spec/blurgh_spec.rb

али тест не прође:

    1) blurgh get_posts should return posts
         Failure/Error: get_posts.should == \
         Errno::ENOENT:
           No such file or directory - posts
	   
То је зато што на самом почетку теста имам:

    before :each do
      YAML.should_receive(:load_file).with("setup.yaml").and_return({"title" => "Naslov", "store" => "posts"}) 
    end

који попуњава **store** са **posts**. Да би то спречио одвојићу прва
неколико теста у једну целину на следећи начин:

     context "routes and content" do
       before :each do
         YAML.should_receive(:load_file).with("setup.yaml").and_return({"title" => "Naslov", "store" => "posts"}) 
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
       end
     end

док је други тест сам за себе.

Покренем тест и он опет не пролази:

    1) blurgh get_posts should return posts
       Failure/Error: [["20110325"], {"url" => "let", "title" => "Авионски лет", "body" => "#{second_article}"}]]
         expected: [[["20110324"], {"url"=>"o-kapadokiji", "title"=>"Кападокија",
         ...
	   
што је и пожељан резултат. Јер ако се анлизира грешка, драгачија је од
предходне: не жали се о томе како не може да прочита неки фајл, већ да
резултат методе није идентичан очекиваном.

Сада да прилагодим методу, да би прошао тест:

    def get_posts
     
      all_posts = Hash.new {
        |h,k| h[k] = Hash.new(&h.default_proc) 
      }
     
      post_dir = File.join(Config.store + "/" + "*.md")
     
      Dir.glob(post_dir).each do |post|
        header, body = File.readlines(post, "")
        data = YAML.load(header)
        all_posts[data['date']]['url']   = post.gsub("\.md", "").gsub(Config.store + "/", "")
        all_posts[data['date']]['title'] = data['title']
        all_posts[data['date']]['body']  = body
      end
      
      all_posts.sort.reverse
     
    end
    
**all_posts** je "хаш хашева" (следи објашњење!). **Dir.glob** чита
фајлове из **Config.store** и попуњава **all_posts**.

Покренем тест и ... он падне:

    1) blurgh get_posts should return posts
       Failure/Error: YAML.should_receive(:load_file).with("setup.yaml").and_return({"title" => "Naslov", "store" => "spec/fixtures"})
         (Syck).load_file("setup.yaml")
             expected: 1 time
             received: 3 times

Ово ми говори да сам добио жељени резултат, но само је метода позивала
**YAML.load_file** више него што је означено у тесту. Овде ми тест не
помаже да програм буде коректан, већ и ефикаснији.

Преправим методу у:

    def get_posts
     
      all_posts = Hash.new {
        |h,k| h[k] = Hash.new(&h.default_proc) 
      }
     
      store = Config.store
      post_dir = File.join(store + "/" + "*.md")
     
      Dir.glob(post_dir).each do |post|
        header, body = File.readlines(post, "")
        data = YAML.load(header)
        all_posts[data['date']]['url']   = post.gsub("\.md", "").gsub(store + "/", "")
        all_posts[data['date']]['title'] = data['title']
        all_posts[data['date']]['body']  = body
      end
      
      all_posts.sort.reverse
     
    end

покренем тест и сада прође. Што значи да треба да се убаци у гит:

    $ git status
    # On branch posts
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #	modified:   blurgh.rb
    #	modified:   spec/blurgh_spec.rb
    #
    # Untracked files:
    #   (use "git add <file>..." to include in what will be committed)
    #
    #	spec/fixtures/
    no changes added to commit (use "git add" and/or "git commit -a")
    $ git add blurgh.rb spec/blurgh_spec.rb spec/fixtures/
    $ git status
    # On branch posts
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #	modified:   blurgh.rb
    #	modified:   spec/blurgh_spec.rb
    #	new file:   spec/fixtures/let.md
    #	new file:   spec/fixtures/o-kapadokiji.md
    #
    $ git commit -m "урађена метода get_posts"
    [posts fba900e] урађена метода get_posts
     4 files changed, 81 insertions(+), 31 deletions(-)
     rewrite spec/blurgh_spec.rb (73%)
     create mode 100644 spec/fixtures/let.md
     create mode 100644 spec/fixtures/o-kapadokiji.md



Наставља се... (у следећој епизоди RSpec пије пиво и осваја свет)


Да видим како би то сада могао да организујем да се листа чланака
прикаже на почетној страни.

Додам следећи тест:

    context "on the index page" do
      it "the posts URLs should be listed" do
        get '/'
        last_response.body.should match("let")
        last_response.body.should match("o-kapadokiji")
      end
    end
    
под **describe "get_posts" **, покренем тестирање и оно пада:

    1) blurgh get_posts on the index page the posts URLs should be listed
       Failure/Error: last_response.body.should match("let")
         expected "<html>\n  <body>\n    <h1>Пази сад</h1>\n\n  </body>\n</html>\n" to match "let"
       # ./spec/blurgh_spec.rb:47:in `block (4 levels) in <top (required)>'
       
што ми указује да мој тест чита прави setup.yaml, што треба исправити
тако што наместим да се YAML објекат учита пре самог теста. То урадим
на следећи начин, тако што омогућим у **before** блоку:

    describe "get_posts" do
     
      before do
        YAML.should_receive(:load_file).with("setup.yaml")\
          .and_return({"title" => "Naslov", "store" => "spec/fixtures"})
      end
     
      it "should return posts" do
        first_article = File.readlines("spec/fixtures/o-kapadokiji.md", "")[1]
        second_article = File.readlines("spec/fixtures/let.md", "")[1]
         
        get_posts.should == \
        [[20110325,  {"url" => "let", "title" => "Авионски лет", "body" => "#{second_article}"}],\
         [20110324, {"url" => "o-kapadokiji", "title" => "Кападокија", "body" => "#{first_article}"}]]
      end
     
      context "on the index page" do
        it "the posts URLs should be listed" do
          get '/'
          last_response.body.should match("let")
          last_response.body.should match("o-kapadokiji")
        end
      end
     
    end
    
Тест опет пада, али са садржајем који сам ја дефинисао у тесту:

    1) blurgh get_posts on the index page the posts URLs should be listed
       Failure/Error: last_response.body.should match("let")
         expected "<html>\n  <body>\n    <h1>Naslov</h1>\n\n  </body>\n</html>\n" to match "let"
       # ./spec/blurgh_spec.rb:53:in `block (4 levels) in <top (required)>'    
       
Сада да наместим да index.html добије линкове чланака, преправим
оригинални **get '/'** у:

    get '/' do
      @title = Config.title
      @posts = get_posts
      erb :index 
    end

и у **index.erb** додам:

    <% @posts.each do |post| %>
       <%= post[1]['url'] %>
    <% end %>    

покренем тест, али овај пут само одређени, задњи тест на којем радим:


    bundle exec rspec -e "blurgh get_posts on the index page" spec/blurgh_spec.rb         

резултат је:

    Failures:
     
        1) blurgh get_posts on the index page the posts URLs should be listed
           Failure/Error: YAML.should_receive(:load_file).with("setup.yaml")\
             (Syck).load_file("setup.yaml")
                 expected: 1 time
                 received: 2 times
           # ./spec/blurgh_spec.rb:36:in `block (3 levels) in <top (required)>'

што ми говори да очекивани резултат теста је тачан, али да се
конфигурациони фајл чита два пута. Једном се чита:

    @title = Config.title

а други пут:    

    @posts = get_posts
    
Могао бих да преправим Config модул да чита резултате само једном и да
их све врати. Нешто овако:

    self.all
      options["title"], options["store"]
    end

Али што се тиче овог чланка, овако је једноставније за сада, ма да
можда није изванредно ефикасно. Но да не заборавим, ставићу белешку да
се овоме вратим и погледам поново:

    module Config
      # TODO: moze biti efikasnije

Сада да подесим тест:

    YAML.should_receive(:load_file).exactly(2).times.with("setup.yaml")\
      .and_return({"title" => "Naslov", "store" => "spec/fixtures"})

Додао сам: **.exactly(2).times** што ће омогућити да тест прође:

    $ bundle exec rspec -e "blurgh get_posts on the index page" spec/blurgh_spec.rb
    Run filtered using {:full_description=>/(?-mix:blurgh get_posts on the index page)/}
     
    blurgh
      routes and content
        the view
      get_posts
        on the index page
          the posts URLs should be listed
     
    Finished in 0.02366 seconds
    1 example, 0 failures
    
Покренем тест за цео програм и добијам следећу грешку за четири случаја:

    blurgh
      routes and content
        should respond to / (FAILED - 1)
        should have a title (FAILED - 2)
        the view
          should have a body html elements (FAILED - 3)
      get_posts
        should return posts (FAILED - 4)

     ...	

     Failure/Error: YAML.should_receive(:load_file).with("setup.yaml").and_return({"title" => "Naslov", "store" => "posts"})
       (Syck).load_file("setup.yaml")
           expected: 1 time
           received: 2 times    

И то исто могу да исправим тако што учиним исту измену и за те
тестове, али опет добијам следећу грешку:

    1) blurgh get_posts should return posts
       Failure/Error: YAML.should_receive(:load_file).exactly(2).times.with("setup.yaml")\
         (Syck).load_file("setup.yaml")
             expected: 2 times
             received: 1 time
       # ./spec/blurgh_spec.rb:37:in `block (3 levels) in <top (required)>'

Што ми указује да ћу више труда потрошити на измени тестова, него да
применим нову методу **all** у **Config** модулу. Зато вратим ове две
задње измене и додам нови тест:

    describe ".all" do
      it "should return all the values" do
        YAML.should_receive(:load_file).with("setup.yaml")\
          .and_return({"title" => "Naslov", "store" => "posts"}) 
        Config.all.should == {"title" => "Naslov", "store" => "posts"}
      end
    end

са методом:

    def self.all
      options
    end
Покренм тестове за **Config**:

    $ bundle exec rspec spec/config_spec.rb     
    
сви прођу. 

Сада да преправим тестове за blurgh:

    it "should return posts" do
      first_article = File.readlines("spec/fixtures/o-kapadokiji.md", "")[1]
      second_article = File.readlines("spec/fixtures/let.md", "")[1]
      store = Config.all['store']

      get_posts(store).should == \
      [[20110325,  {"url" => "let", "title" => "Авионски лет", "body" => "#{second_article}"}],\
       [20110324, {"url" => "o-kapadokiji", "title" => "Кападокија", "body" => "#{first_article}"}]]
    end

тест у овом случају неће да прође јер **get_posts** метода очекује
аргумент. Њу преправим у:

    def get_posts(store)
     
      all_posts = Hash.new {
        |h,k| h[k] = Hash.new(&h.default_proc) 
      }
     
      post_dir = File.join(store + "/" + "*.md")
     
      Dir.glob(post_dir).each do |post|
        header, body = File.readlines(post, "")
        data = YAML.load(header)
        all_posts[data['date']]['url']   = post.gsub("\.md", "").gsub(store + "/", "")
        all_posts[data['date']]['title'] = data['title']
        all_posts[data['date']]['body']  = body
      end
      
      all_posts.sort.reverse
     
    end

Покренем тест, али он пада:

    1) blurgh routes and content should respond to /
       Failure/Error: get '/'
       ArgumentError:
         wrong number of arguments (0 for 1)
       ...

Порука је да у **get '/'** користим нешто са погрешним бројем
аргумената. Изменим блок да сада користи аргумент у позиву у
**get_posts** методи:

    get '/' do
      blurgh_conf = Config.all
      @title = blurgh_conf['title']
      @posts = get_posts(blurgh_conf['store'])
      erb :index 
    end

Покренем тестирање и све прође:

    $ bundle exec rake

И пре него што убацим то у гит, било би добро да погледам како то
заправо изгледа. 

Отворим setup.yaml и додам:

    store: spec/fixtures
    
покренем апликацију:

    $ shotgun blurgh.rb

И погледам [http://localhost:9393/](http://localhost:9393/) и видим
имена фајлова. 

Вратим измену у setup.yaml и додам рад у гит:

    $ git status
    # On branch posts
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #	modified:   blurgh.rb
    #	modified:   spec/blurgh_spec.rb
    #	modified:   spec/config_spec.rb
    #	modified:   views/index.erb
    #
    # Untracked files:
    #   (use "git add <file>..." to include in what will be committed)
    #
    #	.bundle/
    no changes added to commit (use "git add" and/or "git commit -a")
    $ git add blurgh.rb spec/ views/
    $ git status
    # On branch posts
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #	modified:   blurgh.rb
    #	modified:   spec/blurgh_spec.rb
    #	modified:   spec/config_spec.rb
    #	modified:   views/index.erb
    #
    # Untracked files:
    #   (use "git add <file>..." to include in what will be committed)
    #
    #	.bundle/
    $ git commit -m "приказује листу чланака на index.html"
    [posts 21d9938] приказује листу чланака на index.html
     4 files changed, 45 insertions(+), 13 deletions(-)

Једна напомена овде, кад сам урадио **git status** приказана су четири
фајла. Али ја сам уписао само три аргумента кад сам радио **git add**:

    $ git add blurgh.rb spec/ views/
    
То је зато што када се гиту дају аргументи са завршном **/**, он учита
фајлове из тог директоријума.


У следећој епизоди: претварање фајлова у линкове и можда још нешто...
