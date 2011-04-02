
Пошто имам сада основну структуру, мало да "затегнем" тестирање.

Створим нови огранак у коме ћу да радим:

    $ git checkout -b mocks

Кад сам намештао YAML објекат да врати садржај фајла:

    YAML.stub!(:load_file).with("setup.yaml").and_return({"title" => "Naslov"})

ја сам користио **stub!**. А описивао сам **stub**. Иако обе методе
врше исту сврху, **stub!** (са узвичником) потиче из RSpec верзије 1,
док је **stub** из RSpec 2. Обе раде у верзији 2.

То може да се преправи, али би било боље да се пређе на нешто још и
боље: **mocks**.

Пре него што то урадим да објасним шта је шта.

**Stub** је замена за објекат или делове, тј. методе, за објекат.

**Mock** је "лажни" објекат.

Која је разлика ?

Разлика између **stub** и **mocks** је у томе што је **mock**
"интерактиван". Шта то тачно значи? У овом случају то значи да
**mock** јасно дефинише да треба да се назове/користи. Ако није
назван/коришћен током теста, тест је неуспешан. Док **stub** неће да
се жали ако се не користи у тесту.

Ево и примера:

    describe ".title" do
      it "should return value" do
        YAML.stub!(:load_file).with("setup.yaml").and_return({"title" => "Naslov"}) 
        Config.title.should match("Naslov")
      end
    end

ако додам у њега додам YAML.stub(:load):

    describe ".title" do
      it "should return value" do
        YAML.stub!(:load_file).with("setup.yaml").and_return({"title" => "Naslov"}) 
        YAML.stub(:load) # <-- додатак
        Config.title.should match("Naslov")
      end
    end
    
Тест ће да успе. Али ако то урадим са **mock**-ом:

    describe ".title" do
      it "should return value" do
        YAML.stub!(:load_file).with("setup.yaml").and_return({"title" => "Naslov"}) 
        YAML.should_receive(:load)  # <-- додатак
        Config.title.should match("Naslov")
      end
    end

тест не пролази:


    Failures:
     
      1) Config.title should return value
         Failure/Error: YAML.should_receive(:load)
           (Syck).load(any args)
               expected: 1 time
               received: 0 times


Да би направили тест да буде што реалнији, боље би било да се пише овако:

    describe ".title" do
      it "should return value" do
        YAML.should_receive(:load_file).with("setup.yaml").and_return({"title" => "Naslov"}) 
        Config.title.should match("Naslov")
      end
    end

Онда заменим све **stub!** са **should_receive** и покренем тестове:

    $ bundle exec rake 
    
сви пролазе. Да видим шта је измењено:

    $ git status
    # On branch mocks
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #	modified:   spec/blurgh_spec.rb
    #	modified:   spec/config_spec.rb

Онда да их додам у гит:

    $ git add spec/blurgh_spec.rb spec/config_spec.rb
    $ git commit -m "мала измена у тестовима, са stub на mock"

И онда додам нове измене у главно стабло кода:

    $ git checkout master
    Switched to branch 'master'
    asimic@byzantium:~/dev/blurgh2$ git merge mocks
    Updating 4f37cd4..c13635d
    Fast-forward
     spec/blurgh_spec.rb |    2 +-
     spec/config_spec.rb |    6 +++---
     2 files changed, 4 insertions(+), 4 deletions(-)

    $ git merge mocks
    Updating 4f37cd4..c13635d
    Fast-forward
     spec/blurgh_spec.rb |    2 +-
     spec/config_spec.rb |    6 +++---
     2 files changed, 4 insertions(+), 4 deletions(-)
