
Да објасним мало о овом пројекту, да ставим кратак опис о њему:

    emacs README.md
    
и додам:

    Једноставни блог у Синатри за [http://ruby.rs](http://ruby.rs)
     
    Simple blog in Sinatra for [http://ruby.rs](http://ruby.rs)
    
па то ставим у гит:

    $ git add README.md 
    $ git commit -m "додајем README.md"
    [master 67807a9] додајем README.md
     1 files changed, 3 insertions(+), 0 deletions(-)
     create mode 100644 README.md
    

Било би добро да када посетим неку страницу која не постоји, да
прикажем грешку, попут: **Nothing to see here, move along**.

Отворим нови огранак у гиту:

    $ git checkout -b 404
    Switched to a new branch '404'

У blurgh_spec.rb, додам следећу спецификацију:
    
    describe "error page" do
      it "should display a pre defined 404 message" do
        get '/non-existent-page'
        last_response.body.should match("Nothing to see here")
      end
    end    

покренем тест који наравно не пролази:

    Failures:
     
      1) blurgh routes and content error page should display a pre defined 404 message
         Failure/Error: get '/non-existent-page'
         Errno::ENOENT:
           No such file or directory - spec/fixtures/non-existent-page.md
         # ./blurgh.rb:50:in `readlines'
         # ./blurgh.rb:50:in `get_post'
         # ./blurgh.rb:61:in `block in <top (required)>'
         # ./spec/blurgh_spec.rb:67:in `block (4 levels) in <top (required)>'

Да бих то исправио, морам да променим **get_post** методу из овога:

    def get_post(post)
      header, body = File.readlines(Config.options['store'] + "/" + post + ".md", "")
    end

у ово:

    def get_post(post)
      begin
        header, body = File.readlines(Config.options['store'] + "/" + post + ".md", "")
      rescue Errno::ENOENT
        not_found
      end
    end

    not_found do
      "Nothing to see here"
    end

Да објасним шта се овде изменило: **begin** ... **rescue** омогућава
да се изврши **File.readlines** метода, али ако се види грешка коју се
приказала предходно: **Errno::ENOENT** онда да се покрене
**not_found** метода. **not_found** метода је специфична, јер је већ
унапред дефинисана у самом **sinatra** пројекту за употребу у оваквим
случајевима, када се појављује грешка. И уместо да се види грешка,
путем ње, може да се прикаже пожељно упозорење.

Покренем тестирање и све пролази.

Па да испробам и саму апликацију, додам:

    store: spec/fixtures
    
у setup.yaml. И покренем:

    $ shotgun blurgh.rb     
    
Видим два чланка који се налазе под **spec/fixtures**, као што се
видело и пре. И да би испробао нову методу укуцам
[http://localhost:9393/let2](http://localhost:9393/let2) на којој се
види порука: Nothing to see here.

Да то сада додам у гит:

    $ git status
    # On branch 404
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #	modified:   blurgh.rb
    #	modified:   setup.yaml
    #	modified:   spec/blurgh_spec.rb
    #

    $ git add blurgh.rb setup.yaml spec/blurgh_spec.rb
    
    $ git commit -m "приказивање грешке за непостојећу страну"
    [404 fb7e5c8] приказивање грешке за непостојећу страну
     3 files changed, 19 insertions(+), 2 deletions(-)

И да додам то у главно стабло гита:

    $ git checkout master
    Switched to branch 'master'

    $ git merge 404
    Updating 67807a9..fb7e5c8
    Fast forward
     blurgh.rb           |   11 ++++++++++-
     setup.yaml          |    3 ++-
     spec/blurgh_spec.rb |    7 +++++++
     3 files changed, 19 insertions(+), 2 deletions(-)

Сада има сисем за блоговање, написан у 75 линија рубија. Који је
тестиран. Може још шта да се дода, систем није комплетан. Али има
добру основу.

Сав овај рад, окачен је [овде](http://github.com/dotemacs/blurgh).
     



