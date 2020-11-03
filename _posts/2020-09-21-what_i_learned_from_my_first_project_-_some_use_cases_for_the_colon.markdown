---
layout: post
title:      "What I learned from my first project - Some use cases for the colon (:)"
date:       2020-09-21 18:00:07 -0400
permalink:  what_i_learned_from_my_first_project_-_some_use_cases_for_the_colon
---


Does the colon go at the beginning or the end the word? During the CLI project, for some reason, I could never remember where to put the colon. Hopefully this post will help beginning programmers.

## It’s just a :symbol

First, when in doubt, prefix the word with the colon. This creates a [symbol](http://rubylearning.com/satishtalim/ruby_symbols.html#:~:text=A%20Symbol%20is%20the%20most,are%20more%20efficient%20than%20strings). So, what’s so special about a symbol? A symbol is just a name and an internal ID, which is to say that it refers to the same object in a Ruby program. For instance,

##### `:symbol.object_id == :symbol.object_id # => results in ‘true’`

The object ids are the same! Recall that this differs from a string, which always gets a new object id when it is generated. So,

##### `“string”.object_id == “string”.object_id # => results in ‘false’`

Why does this matter? Performance. Whenever you create a new object it is stored separately in memory.

That was easy. So what’s the confusion?

## Hashes – The colon can go at the beginning of a :key OR at the end of a key:

Well, in [hashes](https://docs.ruby-lang.org/en/2.0.0/Hash.html#method-c-5B-5D), the colon can be used at the beginning of a key, like this,

##### `{ :name => “Your name” }`

The above syntax is what I like to think of as the “traditional” way, because the key is clearly a symbol. However, hashes can also be written like this,

##### `{ name: “Your name” }`

The above syntax is what I like to think of as the “shortcut”. It’s cleaner and simpler, but as a beginning Rubyist one can sometimes forget that a colon at the end of a word is just syntactic sugar for a “traditional” key in a hash. This will be especially useful when we talk about the colon in keyword arguments.

## Attribute accessors – Put the colon at the beginning of the word

For attribute accessor methods such as [attr_accessor, attr_writer, and attr_reader](https://www.rubyguides.com/2018/11/attr_accessor/) the colon goes at the beginning of the instance variable word(s) you want to reference, like this,

##### `attr_accessor :first_name, :last_name, :address`

Why? Well, the above are symbols, not hash keys.

That said, hash keys can reference symbols used in attribute accessor methods. This is commonly seen in mass assignments. Before we go into mass assignments, let’s first talk about the colon in keyword arguments, since mass assignments are just specific applications of keyword arguments.

## Keyword arguments – Put the colon at the end of the word

To start, let’s get one thing straight: keyword arguments are not hashes passed into methods as an argument. While the word “keyword” might make one think about a hash, keyword arguments are different from hashes.

So, the following method,
 
```
def say_hello(message: "hello")
    puts message
end
```

CANNOT be written as

```
def say_hello({:message => "hello"})
    puts message
end
```

Trying to call the above method will return an error.

## Mass assignments – Where things get fun…

So far, so good. As previously mentioned, keyword assignments are used in mass assignments, like this,

```
class Song
	attr_accessor :name, :artist

  	def initialize(name:, artist:)
       		@name = name
		@artist = artist
    end
end

song_hash = {name: "Hello", artist: "Adele"}
Song.new(song_hash).name # => “Hello”
```

BUT WAIT, song_hash is a hash, not a keyword argument! TRUE! Why does the above code work?!

So, even though a hash cannot be explicitly used as an argument when defining a method, you can call a method that uses keyword arguments by passing in a hash. **This works because Ruby actually converts the hash to a keyword argument.**

Why is this important? Well, <ins>**in future versions of Ruby (Ruby 3 and above) Ruby will no longer do this**</ins>. That is, you will need to pass in a keyword argument, not a hash, when calling the above method. You can easily do this by using the [double splat operator](https://blog.simplificator.com/2015/03/20/ruby-and-the-double-splat-operator/), which converts a hash to keywords, like this:

##### `Song.new(**song_hash).name # => “Hello”`

You can also rewrite the method to take in a hash like this,

```
class Song
	attr_accessor :name, :artist

  	def initialize(hash)
         song_hash.each{ |key, value| self.send("#{key}=", value)}
    end
end
```

Again, don’t mistake keyword arguments for hashes! While Ruby might convert hashes into keywords for now, this feature will be deprecated. [See more about this change here.](https://www.ruby-lang.org/en/news/2019/12/12/separation-of-positional-and-keyword-arguments-in-ruby-3-0/)

## Can you ever pass a symbol as an argument?

You can when passing in a function as the argument. This is more advanced than the scope of this blog post, so I won’t go any further. Suffice it to say, it can be done. [Here’s how, if you’re curious.](https://dev.to/halented/passing-functions-as-arguments-in-ruby-5b5i)

## Conclusion
 
This post isn’t meant to be exhaustive, but it should cover many scenarios of the colon that beginning Rubyists encounter. To summarize:

* For **symbols** and **attribute accessor methods** – Place the colon **at the beginning** of the word
* For **keyword arguments** – Place the colon **at the end** of the word
* For **hash keys** – **It depends** on the hash syntax you decide to use

