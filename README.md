# jekyll-related-posts

Jekyll can show up to ten related posts using the variable ```site.related_posts```, but it needs to run with the ```--lsi``` option. If you're hosting the site on Github Pages you're out of luck, because such option is not supported. In that case it's only possible to show the ten most recent posts, which might not be related to the current post at all.

### why

Several solutions exists, but they are in form of plugins (which also won't work on Github Pages) or for various reasons did't serve my purpose. The best one I could find is [this one](https://blog.webjeda.com/jekyll-related-posts/), which however requires to define a minimum number of tags. This presents a limitation: if you set the number too high you might not see enough posts, if you set it too low you might see posts related only by a few tags, thus missing better matching posts.

So I came up with my own solution: it allows to display a defined amount of **related posts**, and sort them by the number of tags they share with the **current post** (for the sake of clarity, from now on we'll call it **current page**). Let's see

### the code
```
{% assign showRelatedPosts = 3 %}

{% capture posts %} 
    {% for post in site.posts %}
        {% if post.url != page.url %}
            {% assign matchingTags = "" | split: ","  %}
    
            {% for tag in post.tags %}
                {% assign currentPostTag = tag | downcase %}
                {% assign tagsPage = page.tags | downcase %}
                {% if tagsPage contains currentPostTag %}
                    {% assign matchingTags = matchingTags | push: currentPostTag %}    
                {% endif %}
            {% endfor %}
            
            {% assign tagsTotal = matchingTags.size %}
    
            {% if tagsTotal < 10 %}
                {% assign tagsTotal = tagsTotal | prepend: "0" %}
            {% else %}
                {% assign tagsTotal = tagsTotal  %}
            {% endif %}
            
            |
            <article>
                <strong>{{ tagsTotal }} tags</strong>
                <header>    
                    <h5>{{ post.title }}</h5>    
                </header>
                <section>
                <a href="{{ site.baseurl }}{{ post.url }}">
                    <img width="320" height="200" src="{{ post.image }}" 
                    class="img-fluid" alt="{{ post.image }}" />
                </a>
                </section>
                <small>{{ matchingTags | join: ", " }}</small>
            </article>
                
        {% endif %}
    {% endfor %}
{% endcapture %}

{% assign relatedPosts = posts | split: '|' | sort | reverse %}

<header>
    <h4>You might also like:</h4>
    <hr>
</header>

{% for i in (0..showRelatedPosts) %}
    {{ relatedPosts[i] }}
{% endfor %}
```
A visual example: 
![jekyll-related-posts-example](https://raw.githubusercontent.com/wu-mng/jekyll-related-posts/master/related_posts_example.png)

### how it works

We start with an easy one: here you set the total number of related posts you want to show.  
```
{% assign showRelatedPosts = 3 %}
```
All the following defintions in quotes are from Liquid [manual](https://shopify.github.io/liquid/) (Liquid is the templating language used by Jekyll).
> Using ```capture```, you can create complex strings using other variables created with ```assign```.
```
{% capture posts %}
  {% for post in site.posts %}
    {% if post.url != page.url %}
```
We use ```{% capture %}``` to group all the posts of the site, then, for every post, excluding the current one:
```
{% assign matchingTags = "" | split: ","  %}
```
we create an empty variable, ```matchingTags```. Every tag matching the tags of the current page, will be added here, separated by commas. 
> ```split``` Divides an input string into an array using the argument as a separator. 

Splitting tags will allow us to properly count the elements of the ```matchingTags``` array.
```
{% for tag in post.tags %}
  {% assign currentPostTag = tag | downcase %}
``` 
We loop through every tag of the post, then we "downcase" the tag, and assign it to the ```currentPostTag``` variable. 
This way we'll be able to consider as matching even the tags that were written improperly.  
```
{% assign tagsPage = page.tags | downcase %}
``` 
we also downcase the tags of the current page.
```
{% if tagsPage contains currentPostTag %}
``` 
If we have a match
```
{% assign matchingTags = matchingTags | push: currentPostTag %}
``` 
we "push" the matching tag to the ```matchingTags``` array.

```
{% assign tagsTotal = matchingTags.size %}
```    
```.size``` 
> Returns the number of characters in a string or the number of items in an array.

In this case we count the total number of items in the "matchingTags" array. <br>
Now we know how many tags the post we are looping through has in common with the current page.
```
{% if tagsTotal < 10 %}
    {% assign tagsTotal = tagsTotal | prepend: "0" %}
{% else %}
    {% assign tagsTotal = tagsTotal  %}
{% endif %}
```    
We need to sort the posts by the total number of tags, but apparently is not possible to do numeric sort in Liquid, [see here](https://github.com/Shopify/liquid/issues/980). We use strings: so to get a proper sorting we prepend a "0" to numbers lower than ten. 

Below is what will actually "print" to screen. 
```
|
<article>
    <header>    
        <strong>{{ tagsTotal }} tags</strong>
        <h5 class="title">{{ post.title }}</h5>    
    </header>
    <section>
    <a href="{{ site.baseurl }}{{ post.url }}">
        <img width="320" height="200" src="{{ post.image }}" 
            alt="{{ post.image }}" />
    </a>
    </section>
    <small>{{ matchingTags | join: ", " }}</small>
</article>
```    

be sure to not omit the intial ```|``` (but you can put something else instead): we need it to properly separate and output all the entries. 

We **must** include ```{{ tagsTotal }}```, and it also needs to be the first Liquid object. This is not ideal, but it's the only way I managed to get Liquid to sort the posts correctly. However, if you don't want to show ```{{ tagsTotal }}```, you can comment it out the HTML way: ```<!-- {{ tagsTotal }} -->```.

Then mark up the HTML code however you like. Here I'm showing the post title and ```{ post.image }``` (which is the equivalent of Wordpress "featured image"), and ```{{ matchingTags | join: ", " }}```, the list of matching tags. 

```
{% assign relatedPosts = posts | split: '|' | sort | reverse %}
```    
Here we create another variable from the ```posts``` we "captured" at the beginning, we split them by ```|``` and we sort them by the number of matching tags. We reverse the array, so the posts with the biggest number of matches is shown first.
```
<header>
    <h4>You might also like:</h4>
    <hr>
</header>
{% for i in (0..showPosts) %}
    {{ relatedPosts[i] }}
{% endfor %}
```
Then we end with a simple for loop, showing the related posts.
That's all!
