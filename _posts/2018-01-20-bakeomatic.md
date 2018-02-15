---
layout: post
title: bakeomatic
date: 2018-01-20 12:00 -0600
categories: projects
image: "/assets/images/bakeomatic-overall.png"
summary: | 
  Java CLI app written as TDD project in IntelliJ
---

**Motivation**

A project to learn Java and gain experience with TDD 

**What is bakeomatic**

A Simple cake baking app that reads an inventory and cake mixes from a JSON file. When a cake is baked, the ingredients are reduced in inventory. The inventory can be restocked in the app.
**Technologies**

main: Java 
testing: JUnit, Hamcrest
IDE: IntelliJ

**Implementation**

A user is presented with an initial inventory, menu, and list of options, they can then select a particular cake to bake

![markdown-img]({{ "/assets/images/bakeomatic-initial.png" | absolute_url}}) 

Inventory will run down if too many cakes are baked without re-stocking

![markdown-img]({{ "/assets/images/bakeomatic-rundown.png" | absolute_url}}) 

If inventory is run down, it can be restocked using option 'r'

![markdown-img]({{ "/assets/images/bakeomatic-restock.png" | absolute_url}}) 

Ingredients are an individual class which are loaded into the inventory class. The cakes are also a class which query the inventory to see if they are available. 

{% highlight java %}
{% raw %}

public class Cake {
    public String name;
    public HashMap<String, Integer> ingredients;
    private BigDecimal itemCost;

    public Cake(String name){
        this.name = name;
        this.ingredients = new HashMap<String, Integer>();
    }

    public boolean isAvailable(Inventory inventory){
        for(String name: this.ingredients.keySet()){
            int itemQty = inventory.getItem(name).getQty();
            if (itemQty == 0) {
                return false;
            } else if (itemQty < this.ingredients.get(name)){
                return false;
            }
        }
        return true;
...

{% endraw %}
{% endhighlight %}

The logic loop is in the main class and the main methods and inventory used in the main loop are in the bakerlogic class

{% highlight java %}
{% raw %}

public class BakerLogic {
    private HashMap<String, Cake> allCakes;
    private Inventory inventory;
    private ArrayList<Cake> availableCakes;

    public BakerLogic(Inventory inventory){
        this.inventory = inventory;
        this.allCakes = new HashMap<>();
        this.availableCakes = new ArrayList<>();
    }

{% endraw %}
{% endhighlight %}

One of the bigger challenges was writing the code for reading the JSON

{% highlight java %}

public void loadInitialCakes(){
        JsonObject rootObject = this.getRoot();
        JsonObject cakes = rootObject.getAsJsonObject("cakes");
        for(String cake: cakes.keySet()){
            JsonObject ingredients = cakes.getAsJsonObject(cake);
            Cake newCake = new Cake(cake);
            for(String item: ingredients.keySet()){
                newCake.ingredients.put(item, ingredients.get(item).getAsInt());
            }

            this.allCakes.put(cake, newCake);
        }
    }

    ...

    private JsonObject getRoot(){
        InputStream inputStream = getClass().getClassLoader().getResourceAsStream("initial.json");
        Reader reader = new InputStreamReader(inputStream);
        JsonParser parser = new JsonParser();
        JsonElement rootElement = parser.parse(reader);
        JsonObject rootObject = rootElement.getAsJsonObject();
        return rootObject;
    }

{% endhighlight %}

Another challenge was implementing testing. I brought in a second framework to handle some assertions that I used to in Javascript.

{% highlight java %}

import static org.hamcrest.CoreMatchers.containsString;
import static org.hamcrest.MatcherAssert.assertThat;

...

@Test
    public void printsAdditionalOptions(){
        this.baker.loadCakeOptions();
        System.setOut(new PrintStream(outContent));
        this.baker.printAdditionalOptions();
        assertThat(outContent.toString(), containsString("order the cake with the corresponding number in the menu"));
    }

{% endhighlight %}

<a href="https://github.com/ansachs/bakermatic" target="_blank">bakermatic</a>
