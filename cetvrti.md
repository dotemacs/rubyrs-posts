
Пошто је ово систем за блог, било би пожељно да има начин како ће моћи
да прикаже садржај.

Правим нови огранак:

    $ git checkout -b views
    
Напишем следећи тест у **blurgh_spec.rb**:

    context "the view" do
      it "should have a body html elements" do
        get '/'
        last_response.body.should match("<body>")
      end
    end

следеће:
  
    $ mkdir views
    $ emacs views/layout.erb

у који додам:
      
    <html>
      <body>
        <%= yield %>
      </body>
    </html>

у фајл views/index.erb ставим:    

    <h1><%= @title %></h1>

У главном програму додам следеће:

    get '/' do
      @title = Config.title
      erb :index 
    end

 покренем тест и он прође.
 
 Онда рад додам у гит:
 
    $ git status
    # On branch master
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
    #	views/
    no changes added to commit (use "git add" and/or "git commit -a")

То ми говори да су измењена два фајла: blurgh.rb и spec/blurgh_spec.rb и додан views. 


    $ git add blurgh.rb spec/blurgh_spec.rb
    asimic@byzantium:~/dev/blurgh2$ git add views/
    asimic@byzantium:~/dev/blurgh2$ git status
    # On branch master
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #	modified:   blurgh.rb
    #	modified:   spec/blurgh_spec.rb
    #	new file:   views/index.erb
    #	new file:   views/layout.erb

Друго додавање у гиту: **git add views/**, намерно сам користио **/**
на самом крају јер то онда дода гиту све фајлове у том директоријому.

И за крај: 

    $ git commit -m "додани views"
    $ git checkout master
    $ git merge views
