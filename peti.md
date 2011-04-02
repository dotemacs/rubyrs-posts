
До сада, радио сам само са тестовима, нисам заиста пробао
апликацију. Најлакши начин да се покрене синатра апликација је
користећи shotgun. Дакле:

    $ git checkout -b proba
    $ emacs Gemfile

И у њу додам:

    gem 'shotgun'
    
Па
    $ bundle install
    
Да би покренуо апликацију:

    $ shotgun blurgh.rb
    
И отворим http://localhost:9393/, али ту је грешка, јер нема setup.yaml фајла.

Направим setup.yaml са следећим садржајем:    

    title: Пази сад
    
Поново отворим страницу, и видим "Пази сад" у **h1** таговима као што
очекујем.

Да убацим садржај у гит:

    $ git add Gemfile setup.yaml
    $ git commit -m "апликација приказује странице"
    $ git checkout master
    $ git merge proba
