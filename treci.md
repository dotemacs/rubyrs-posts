
А шта ако setup.yaml нема title дефинисан? Или ако нема setup.yaml
ништа у себи?

Отворим нови огранак: 

    $ git checkout -b extra_checks

и ту почнем да пишем тестове у spec/config_spec.rb:

    context "when the title is blank" do
      it "should return nothing" do
        YAML.stub!(:load_file).with("setup.yaml").and_return({"title" => ""}) 
        Config.title.should match("")
      end
    end

Пробам:

    $ bundle exec rspec spec/config_spec.rb
    
Тест прође. Онда додам други:

    context "when the title is missing" do
      it "should return nil" do
        YAML.stub!(:load_file).with("setup.yaml").and_return({}) 
        Config.title.should be_nil
      end
    end
    
И он прође. Онда пробам све тестове заједно:

    $ bundle exec rake
     
    Config
      .title
        should return value
      when the title is blank
        should return nothing
      when the title is missing
        should return nil
     
    blurgh
      should respond to /
      should have a title
     
    Finished in 0.01154 seconds
    5 examples, 0 failures

Сви пролазе. Онда додам напредак у гит и све ставим у главно стабло:

    $ git add spec/config_spec.rb
    $ git commit -m "још тестова за Config.title" 
    $ git checkout master
    $ git merge extra_checks

