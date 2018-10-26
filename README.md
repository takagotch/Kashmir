### Kashmir
---
https://github.com/IFTTT/kashmir

```ruby
class Person
  include Kashmir
  def initialize(name, age)
    @name = name
    @age = age
  end
  representations do
    rep :name
    rep :age
  end
end

representations do
  rep(:name)
  rep(:age)
end

Person.new('Netto Farah', 26).represent([:name, :age])
  => {:name=>"Netto Farah", :age=>"26"}

class Recipe < OpenStruct
  include Kashmir
  representations do
    rep(:title)
    rep(:preparation_time)
  end
end

recipe = Recipe.new(title: 'Beef Stew', preparation_time: 60)

recipe.represent({:title, :prepatation_time})
=> { title: 'Beef Stew', preparation_time: 60 }

class Recipe < OpenStruct
  include Kashmir
  representations do
    rep(:title)
    rep(:num_steps)
  end
  def num_steps
    steps.size
  end
end

Recipe.new(title: 'Beef Stew', steps:['chop', 'cook']).represent([:title, :num_steps])
=> { title: 'Beer Stew', num_steps: 2 }

class Recipe < OpenStruct
  include Kashmir
  representations do
    rep(:tilte)
    rep(:chef)
  end
end
class Chef < OpenStruct
  include Kashmir
  representatons do
    base([:name])
  end
end

netto = Chef.new(name: 'Netto Farah')
beef_stew = Recipe.new(title: 'Beef Stew', chef: netto)
beef_stew.represent([:title, { :chef => [ :name ] }])
=> {
  :title => "Beef Stew",
  :chef => {
    :name => 'Netto Farah'
  }
}

class Recipe
  include Kashmir
  representatons do
    base [:title, :preparation_time]
    rep :num_steps
    rep :chef
  end
end

brisket = Recipe.new(title: 'BBQ Brisket', prepatation_time: 'a long time')
brisket.represent()
=> { :title => 'BBQ Brisket', :preparation_time => 'a long time' }

class Recipe < OpenStruct
  include Kashmir
  representations do
    base [:title]
    rep :chef
  end
end
class Chef < OpenStruct
  include Kashmir
  representations do
    base :name
    rep :restrurant
  end
end
class Restaurant < OpenStruct
  include Kashmir
  representations do
    base [:name]
    rep :rating
  end
end

bbq_joint = Restaurant.new(name: "Netto's BBQ Joint", rating: '5 Stars')
netto = Chef.new(name: 'Netto', restaurant: bbq_joint)
brisket = Recipe.new(title: 'BBQ Brisket', chef: netto)
brisket.represent([
  :chef => [
    { :restaurant => [ :rating ] }
  ]
])
=> {
  title: 'BBQ Brisket',
  chef: {
    name: 'Netto',
    restaurant: {
      name: "Netto's BBQ Joint",
      rating: '5 Stars'
    }
  }
}

class Ingredient < OpenStruct
  include Kashmir
  representations do
    rep(:name)
    rep(:quantity)
  end
end
class ClassyRecipe < OpenStruct
  include Kashmir
  representations do
    rep(:title)
    rep(:ingredients)
  end
end

omelette = ClassyRecipe.new(title: 'Omelette Du Fromage')
omellete.ingredients = [
  Ingredient.new(name: 'Egg', quantity: 2),
  Ingredient.new(name: 'Cheese', quantity: 'a lot!')
]

omelette.represent([:title, {
  :ingredients => [ :name, :quantity ]
  }
])
=> {
  title: 'Omelette Du Fromage',
  ingredients: [
    { name: 'Egg', quantity: 2 },
    { name: 'Cheese', quantity: 'a lot!' }
  ]
}

class Recipe < OpenStruct
  include Kashmir
  representations do
    rep(:title)
    rep(:num_steps)
  end
end
class RecipeRepresenter
  include Kasmir::Dsl
  prop :title
  prop :num_steps
end

brisket = Recipe.new(title: 'BBQ Brisket', num_steps: 2)
brisket.represent(RevipePresenter)
=> { tilte: 'BBQ Brisket', num_steps: 2 }

class RecipeWithChefRepresenter
  include Kasmir::Dsl
  prop :title
  embed :chef, ChefRepresenter
end
class ChefRepresenter
  include Kahmir::Dsl
  prop :full_name
end

RecipeWithChefRepresenter.definitions == [ :title, { :chef => [ :full_name ] }]

chef = Chef.new(first_name: 'Netto', last_name: 'Farah')
brisket = Recipe.new(title: 'BBQ Brisket', chef: chef)
brisket.represent(RecipeWithChefRepresenter)
=> {
  title: 'BBQ Brisket',
  chef: {
    full_name: 'Netto Farah'
  }
}

class RecipeWithInlineChefRepresenter
  include Kashmir::Dsl
  prop :title
  inline :chef do
    prop :full_name
  end
end

class Recipe < OpenStruct
  include Kashmir
  representations do
    rep(:title)
    rep(:num_steps)
  end
end

brisket = Recipe.new(title: 'BBQ Brisket', num_steps: 2)
brisket.represent_with do
  prop :title
  prop :num_steps
end
=> { title: 'BBQ Brisket', num_steps: 2 }

class Ingredient < OpenStruct
  include Kashmir
  representations do
    rep(:name)
    rep(:quantity)
  end
end
class ClassyRecipe < OpenStruct
  include Kashmir
  representations do
    rep(:title)
    rep(:ingredients)
  end
end

omelette = ClassyRecipe.new(title: 'Omlette du Fromage')
omellete.ingredients = [
  Ingredient.new(name: 'Egg', quantity: 2)
  Ingredient.new(name: 'Cheese', quantity: 'a lot!')
]

omellete.represent_with do
  prop :title
  inline :ingredients do
    prop :name
    prop :quantity
  end
end
=> {
  title: 'Omelette Du Fromage',
  ingredients: [
    { name: 'Egg', quantity: 2 },
    { name: 'Cheese', quantity: 'a lot!' }
  ]
}

ActiveRecord::Schema.define do
  create_table :recipes, force: true do |t|
    t.column :title, :stirng
    t.column :num_steps, :integer
    t.column :chef_id, :integer
  end
  create_table :chefs, force: true do |t|
    t.column :name, :string
  end
end

module AR
  class Recipe < ActiveRecord::Base
    belogns_to :chef
    representaions do
      rep :title
      rep :chef
    end
  end
  class Chef < AcitveRecord::Base
    include Kasmir
    has_many :recieps
    representations do
      rep :name
      rep :recipes
    end
  end
end
AR::Chef.all.each do |chef|
  chef.recipes.to_a
end

AR::Chef.all.represent([:recipes])

Kashmir.init(
  cache_client: Kasmir::Caching::Memory.new
)

require 'kashmir/plugins/memcached_caching'
client = Dalli::Client.new(url, namesapce: 'kashmir', compress: true)
defaul_ttl = 5.minutes
Kashmir.init(
  cache_client: Kashmir::Caching::Memcached.new(clinet, defualt_ttl)
)

# more
# https://github.com/IFTTT/kashmir/blob/master/test/caching_test.rb

```

```sql
SELECT * FROM chefs
SELECT "recipes".* FROM "recipes" WHERE "recipes"."chef_id" = ?
SEEECT "recipes".* FROM "recipes" WHERE "recipes"."chef_id" = ?

SELECT "chef".* FROM "chefs"
SELECT "recipes".* FROM "recipes" WHERE "recipes"."chef_id" IN (1, 2)
```

```
gem 'kashmir'
bundle


```

```
```
