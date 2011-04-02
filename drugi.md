
Блог у синатри, епизода друга

Да би додали нешто ново, пожељно је да се направи нови огранак (branch) у гиту:

    $ git checkout -b config_file    
    $ git branch
    \* config_file
      master

Са задњом сам направио нови огранак и наместио њега као тренутно место
у гиту где ће ићи сав нов рад.
    

Овај систем ће читати своју конфигурацију из setup.yaml фајла, чији формат ће изгледати овако:

    title: Име мог блога
    store: posts


Title ће бити сам наслов, suffix је завршни део назива фајлова, у овом
случају у Markdown формату и store означава где ће се фајлови стављати.

Тако да када будем отишао на почетну страну ја требам да видим наслов
или назив. То треба да се подеси у тесту:

    it "should have a title" do
      get '/'
      last_response.body.should match("Naslov")
    end

И ако покренем тестирање:

    $ bundle exec rake
    
следећа грешка се појављује:


    Failures:
     
      1) blurgh should have a title
         Failure/Error: last_response.body.should match("Naslov")
           expected "" to match "Naslov"
         # ./spec/blurgh_spec.rb:16:in `block (2 levels) in <top (required)>'


Да би то решио, '/' адреса треба да испише наслов који ће да добије из фајла у YAML формату. 

Пошто ништа не чита тај фајл, направићу модул који га чита. Али прво да кренем са тестом:

    emacs spec/config_spec.rb
    
И ту опишем како би да се понаша тај модул:

    require 'spec_helper'
     
    describe "Config" do
     
      describe ".title" do
        it "should return value" do
          Config.title.should match("Naslov")
        end
      end

    end  

Ту означим да метода title ће вратити вредност дефинисану у фајлу, у
овом случају то ће бити "Naslov".

Онда напишем Config модул у **blurgh.rb**:


    module Config
      def self.title
        options["title"]
      end
         
      private
      def self.options
        YAML.load_file("setup.yaml")
      end
    end

где приватна метода options чита YAML конфигурацију. Модул се 

Да би могао да читам YAML формат, додам:

    require 'yaml'


Вратим се тесту, покренем га, он не ради:

     Failure/Error: Config.title.should match("Naslov")
     Errno::ENOENT:
       No such file or directory - setup.yaml

То је зато што тај фајл и не постоји. Зато требa да се створи "лажни" фајл:

    YAML.stub!(:load_file).with("setup.yaml").and_return({"title" => "Naslov"})

**stub** је део RSpec-а и он намести да метода врати пожељне
резултате. У овом случају намести да YAML врати учитан фајл setup.yaml
са садржајем "title: Naslov".

Ако погледате пажљиво, видећете да он у ствари не вараћа 

    "title: Naslov" 

већ Hash са садржајем: 

    "title" => "Naslov"
    
То је зато што у овом случају ја желим да YAML објекат врати што
"реалнији" резултат. Шта то значи? Када YAML учита неки фајл, он учита
вредности из њега и стави их у Hash. А пошто ја овде хоћу да наместим
YAML да ми врати пожељне вредности, ја их сам попуним.

Сада да пробам само задњи тест:

    $ bundle exec rspec spec/config_spec.rb
    
Он прође.

Сада се вратим на првобитни тест у spec/blurgh_spec.rb и додам у **it
"should have a title" do**:

    YAML.stub!(:load_file).with("setup.yaml").and_return({"title" => "Naslov"}) 

покренем 

    $ bundle exec rake
    
и опет тест за наслов пада. То је зато што ништа се не враћа из главне
синатрине методе. Да би то исправио додам следеће:

    Config.title

покренем 

    $ bundle exec rake

И уместо да сви тестови прођу, први пада. То је првобитни тест, зато
што он не може да нађе setup.yaml фајл. Да би то исправио, убацим
следећи блок:


    before :all do
      YAML.stub!(:load_file).with("setup.yaml").and_return({"title" => "Naslov"})     
    end

Али тада само први тест прође, други пада. Ако изменим блок у **before
:each**, онда сваки тест у blurgh_spec добије "лажни" фајл, то јест
stub setup.yaml фајла:

    require 'spec_helper'
     
    describe "blurgh" do
     
      before :each do
        YAML.stub!(:load_file).with("setup.yaml").and_return({"title" => "Naslov"})     
      end
     
      def app
        @app ||= Sinatra::Application
      end
     
      it "should respond to /" do
        get '/'
        last_response.should be_ok
      end
     
      it "should have a title" do
        get '/'
        last_response.body.should match("Naslov")
      end
     
    end


    $ bundle exec rake
     
    Config
      .title
        should return value
     
    blurgh
      should respond to /
      should have a title
     
    Finished in 0.01046 seconds
    3 examples, 0 failures

После овог напредка, додам то у гит. Прво да видим статус:

     
    $ git status
    # On branch config_file
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
    #	spec/config_spec.rb
    no changes added to commit (use "git add" and/or "git commit -a")

Што значи да blurgh.rb и spec/blurgh_spec.rb су измењене а
spec/config_spec.rb је нови фајл.

Све промене додам гит (у два корака):

    $ git add blurgh.rb spec/blurgh_spec.rb spec/config_spec.rb
    $ git status
    # On branch config_file
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #	modified:   blurgh.rb
    #	modified:   spec/blurgh_spec.rb
    #	new file:   spec/config_spec.rb
    $ git commit -m "додан нови модул Config"

Погледао сам статус гита, чисто да видим да ли је све тачно унешено.

Тренутно сам и даље на огранку config_file, да потврдим:

    $ git branch
    \* config_file
      master

Да би унео нови део напредка у главно стабло гита, требам да се
пребацим у њега и да упишем нове рад. То радим овако:

    $ git checkout master
    Switched to branch 'master'

    $ git merge config_file
    Updating b740f6e..b95240c
    Fast-forward
     blurgh.rb           |   13 +++++++++++++
     spec/blurgh_spec.rb |   12 +++++++++++-
     spec/config_spec.rb |   12 ++++++++++++
     3 files changed, 36 insertions(+), 1 deletions(-)
     create mode 100644 spec/config_spec.rb

