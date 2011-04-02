
Да би документација тестова била читљивија, преправим

    context "the view" do
    
у

    context "the index view" do    
    
и додам нови тест:

    context "the post view" do
      it "should show post content" do
        get '/let'
        last_response.body.should match("25. марта 1923. - Слетањем авиона на линији Париз-Цариград на")
      end
    end
    
Тај тест наравно не пролази, зато што не постоји ништа што би
приказало садржај фајла. То могу да омогућим тако што створим методу
која ће приказивати индивидалне чланке:

    get '/:post' do
      @config, @content = get_post(:post)
      erb :post
    end

и додам и **post.erb** у **views/**:

    <%= @content %>

само да би било што једноставније и да би омогућио да тест прође. 

Покренем тест, и он пада са следећом поруком:

    1) blurgh routes and content the post view should show post content
         Failure/Error: get '/let'
         TypeError:
           can't convert Symbol into String
         # ./blurgh.rb:46:in `+'
         # ./blurgh.rb:46:in `get_post'
         # ./blurgh.rb:58:in `block in <top (required)>'
         # ./spec/blurgh_spec.rb:44:in `block (4 levels) in <top (required)>'

што значи да се **get_post** метода не добија одговарајући
аргумент. Зато преправим:

    @config, @content = get_post(:post)
  
у

    @config, @content = get_post(params[:post])  
    
покренем тестирање поново и све прође. 

Но нисам баш задовољан предходним тестом, он само проверава прву линију чланка

    last_response.body.should match("25. марта 1923. - Слетањем авиона на линији Париз-Цариград на")
    
Било би боље када би учитавао сав фајл. Зато изменим тест у:

    it "should show post content" do
      get '/let'
      article = File.readlines("spec/fixtures/let.md", "")[1]
      last_response.body.should match(article)
    end

јер ми изгледа читљивије. Па покренем тест... који прође.
    
Додам напредак у гит:

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
    #	views/post.erb
    no changes added to commit (use "git add" and/or "git commit -a")

Овде се види да је **views/post.erb** нови фајл који још није унет у
гит. Зато требам да га додам са осталим фајловима где је направљена
промена:

    $ git add blurgh.rb spec/blurgh_spec.rb views/post.erb 

    $ git commit -m "приказивање чланака"
    [posts 5540485] приказивање чланака
     3 files changed, 20 insertions(+), 5 deletions(-)
     create mode 100644 views/post.erb

Било би добро да додам да може да се види наслов када се прикаже
чланак. 

    it "should show post title" do
      get '/let'
      config = File.readlines("spec/fixtures/let.md", "")[0]
      post_options = YAML.load(config)
      last_response.body.should match(post_options['title'])
    end

Покренем тест, и он не пролази, зато што немам начин да прикажем
наслов на страници која приказује чланке.

Зато преуредим **get '/:post'** у:

    get '/:post' do
      @config, @content = get_post(params[:post])
      @title = YAML.load(@config)['title']
      erb :post
    end

подесим **post.erb**:

    <h2>
      <%= @title %>
    <h2>
     
    <%= @content %>
    
покренем тест и он прође. Зато да учитам у гит:

    $ git status
    # On branch posts
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #	modified:   blurgh.rb
    #	modified:   setup.yaml
    #	modified:   spec/blurgh_spec.rb
    #	modified:   views/post.erb
    
    $ git add blurgh.rb spec/blurgh_spec.rb views/post.erb 
    $ git commit -m "приказивање наслова чланака"

Сада да додам датум за чланке. Прво да додам тест:

    it "should show post date" do
      get '/let'
      config = File.readlines("spec/fixtures/let.md", "")[0]
      post_options = YAML.load(config)
      last_response.body.should match("20110325")
    end

Па да се уверим да тест пада:

    $ bundle exec rake
    
И онда да преправим **get '/:post'**:

    get '/:post' do
      @config, @content = get_post(params[:post])
      post_options = YAML.load(@config)
      @title = post_options['title']
      @date =  post_options['date']
      erb :post
    end

и да променим **post.erb**:

    <h2>
      <%= @title %>
    </h2>
     
    <div>
      <%= @date %>
    </div>
     
    <%= @content %>
    
Покренем тест и он прође. Па да додам напредак у гит:

    $ git status
    # On branch posts
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #	modified:   blurgh.rb
    #	modified:   spec/blurgh_spec.rb
    #	modified:   views/post.erb
    #

    $ git add blurgh.rb spec/blurgh_spec.rb views/post.erb

    $ git commit -m "приказивање датума чланака"
    [posts d005862] приказивање датума чланака
     3 files changed, 14 insertions(+), 1 deletions(-)
     
Па да додам овај огранак у главни, тако што се прво пребацим у
**master** огранак:

    $ git checkout master
    Switched to branch 'master'
     
и сада да додам **posts** огранак у **master**:
     
    $ git merge posts
    Auto-merging blurgh.rb
    CONFLICT (add/add): Merge conflict in blurgh.rb
    Auto-merging spec/blurgh_spec.rb
    CONFLICT (add/add): Merge conflict in spec/blurgh_spec.rb
    Auto-merging spec/config_spec.rb
    CONFLICT (add/add): Merge conflict in spec/config_spec.rb
    Auto-merging views/index.erb
    CONFLICT (add/add): Merge conflict in views/index.erb
    Automatic merge failed; fix conflicts and then commit the result.     

Ово је ново. Порука ми говори да измене које сам направио се не
подударају апсолутно са садржајем **master** огранка. 

Да би то решио, морам да направим измене ручно. Отворим прво
**blurgh.rb** и видим следеће:

    module Config
    <<<<<<< HEAD:blurgh.rb
      def self.title
        options["title"]
      end
      
    =======
    
Да би то поправио, морам сам да коригујем у едитору. Бацим се на
посао, фајл по фајл... То су тестови да ми помогну да ништа није
пропуштено.

Али ако у међу времену покушам да додам огранак **posts** поново,
добићу следећу грешку:

    $ git merge posts
    fatal: You have not concluded your merge. (MERGE_HEAD exists)

то је зато што гит није могао аутоматски да дода нови огранак, па је
направио нове фајлове, спајањем постојећих фајлова у **master**
огранку са фајловима из **posts** огранка.

Да би то решио, морам прво да средим све фајлове и да сви тестови
пролазе. Затим да видим статус гита, и тек онда да додам фајлове:

    $ git status
    blurgh.rb: needs merge
    spec/blurgh_spec.rb: needs merge
    spec/config_spec.rb: needs merge
    views/index.erb: needs merge
    # On branch master
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #	new file:   spec/fixtures/let.md
    #	new file:   spec/fixtures/o-kapadokiji.md
    #	new file:   views/post.erb
    #
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #	unmerged:   blurgh.rb
    #	unmerged:   spec/blurgh_spec.rb
    #	unmerged:   spec/config_spec.rb
    #	unmerged:   views/index.erb
    #

    $ git add blurgh.rb spec/config_spec.rb spec/blurgh_spec.rb views/index.erb
    
али требам да додам и нове фајлове:

    $ git add spec/fixtures/o-kapadokiji.md spec/fixtures/let.md views/post.erb    

На крају учитам све промене у гит:

    $ git commit -m "додајем рад на чланцима"    
    
Ако погледам лог гита, видим да су то и поруке из **posts** огранка:

    $ git log --oneline | head -5
    88f2449 додајем рад на чланцима
    d005862 приказивање датума чланака
    4a40483 приказивање наслова чланака
    5540485 приказивање чланака
    a79f8c8 смршао blurgh_spec.rb    
    

Наставља се ...    


