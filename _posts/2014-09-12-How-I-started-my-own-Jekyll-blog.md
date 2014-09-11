---
layout: post
title: How I started my own Jekyll blog
categories:
- Jekyll
- blog
- Setup
comments: true
---


The idea to start a personal blog was in my mind for quite some time but
I could not motivated enough myself to do it. Even after I read various
posts suggesting that every developer should have one, as a self note
keeper, as a way to improve his communication skills and  his 'brand'.

I was feeling that it was not enough fun. Just a
[WordPress](http://wordpress.org/) install, a nice looking theme and
that's all it would take to build it. Boring.

Then I read about [Jekyll](http://jekyllrb.com/),
[Octopress](http://octopress.org/) and all the other static site
generators. I was intrigued.

<!--more-->

First I tried Octopress because it was based on Jekyll, the most popular generator,
and because Google search gave more recent results for Octopress than Jekyll etc. Soon I
discovered that with all the extra work already implemented (extra features, plugins etc)
it added a lot of complexity compared to plain Jekyll. So I went for Jekyll.

### Jekyll Setup

To install Jekyll I just ran :
{% highlight bash %}
sudo apt-get install ruby nodejs
sudo gem install jekyll
{% endhighlight %}


I used the excellent [Hyde](https://github.com/poole/hyde) theme as a
base and modified it to my liking. It's main characteristic is that it
has a two column layout containing a sidebar on the right or left side.

On the downside, it uses plain css (I am fan of [Sass](http://sass-lang.com/))
 and almost none of the new html5 elements. I replaced some divs on the
 layout with some more semantic html5 elements like article, footer, header
 etc and on the next days I will  hopefully migrate on Sass.


### Configuration

Here is the <code>_config.yml</code> of my blog:

{% highlight yaml %}

markdown:         redcarpet
highlighter:      pygments

# Permalinks
permalink:        /:year/:title/


# Setup
title:            'Anastasios Bourazanis'
tagline:          'Thoughts on software development'
description:      ''
url:              http://abourazanis.com
baseurl:          /

author:
  name:           'Anastasios Bourazanis'
  url:            https://twitter.com/ABourazanis
  email:          a.bourazanis@gmail.com

paginate:         3


# Google Analytics
google_analytics_tracking_id: UA-36113001-3

#Disqus comments
disqus_short_name: abourazanis
disqus_show_comment_count: true


github:
  repo:           https://github.com/tas0s


# Social links
facebook_user: anastasios.bourazanis
googleplus_user: AnastasiosBourazanis
twitter_user: ABourazanis
github_user: tas0s
coderwall_user:
stackoverflow_user:
linkedin_user: abourazanis
pinterest_user:
delicious_user:
pinboard_user:
douban_user:
quora_user:
instagram_user:
behance_user:
subscribe_rss: atom.xml

# Plugins
gems:
  - jemoji
  - jekyll-sitemap

# Exclude from production _site
exclude:
    - Gemfile
    - Gemfile.lock
    - LICENSE
    - README.md

{% endhighlight %}

<i>Note: I removed the line about post excerpt due to a [Jekyll bug](https://github.com/jekyll/jekyll/issues/1401)
that was preventing the blog from building.</i>

### Blog directory structure
My blog folder looks like this:
{% highlight bash %}
.
├── _assets
|   └──css
|       ├── hyde.css
|       ├── monokai.css
|       └── poole.css
├── _includes
|   ├── categories.html
|   ├── comments.html
|   ├── comments_count.html
|   ├── ga.html
|   ├── head.html
|   ├── sidebar.html
|   └── tags.html
├── _layouts
|   ├── archive.html
|   └── category_index.html
|   └── default.html
|   └── page.html
|   └── post.html
|   └── tags.html
├── _plugins
|   ├── archive_generator.rb
|   └── categories_generator.rb
|   └── jekyll_asset_pipeline.rb
|   └── tags_generator.rb
|   └── youtube.rb
├── _posts
├── public
|   ├── images
|   |   ├──social
|   |   └──me.jpg
|   └── javascripts
|       └──md5.js
├── _site
├── 404.html
├── CNAME
├── LICENCE.md
├── Rakefile
├── _config.yml
├── about.md
├── archive.html
├── atom.xml
├── index.xml
└── robots.txt
{% endhighlight %}


### Archive Page

After the base configuration of my blog and the layout modifications, I wanted to have
an archive page based on month. In order to achieve this, I used the following
[script](http://joseoncode.com/2011/11/27/generating-monthly-archives-with-jekyll/)
from [Jose Romaniello](http://joseoncode.com/) (<code>archive_generator.rb</code>):
{% highlight ruby linenos=table %}
module Jekyll
  class ArchiveIndex < Page
    def initialize(site, base, dir, period, posts)
      @site = site
      @base = base
      @dir = dir
      @name = 'index.html'
      self.process(@name)
      self.read_yaml(File.join(base, '_layouts'), 'archive.html')
      self.data['period'] = period
      self.data['period_posts'] = posts
      archive_title_prefix = site.config['archive_title_prefix'] || 'Archive: "'
      archive_title_suffix = site.config['archive_title_suffix'] || '"'
      if period["month"]
        self.data['title'] = "#{archive_title_prefix}#{period["month"]}-#{period["year"]}#{archive_title_suffix}"
      else
        self.data['title'] = "#{archive_title_prefix}#{period["year"]}#{archive_title_suffix}"
      end
    end
  end
  class ArchiveGenerator < Generator
    safe true
    def generate(site)
      if site.layouts.key? 'archive'
        site.posts.group_by{ |c| {"month" => c.date.month, "year" => c.date.year} }.each do |period, posts|
          archive_dir = File.join(period["year"].to_s(), "%02d" % period["month"].to_s())
          write_archive_index(site, archive_dir, period, posts)
        end

        site.posts.group_by{ |c| {"year" => c.date.year} }.each do |period, posts|
          archive_dir = File.join(period["year"].to_s())
          write_archive_index(site, archive_dir, period, posts)
        end
      end
    end
    def write_archive_index(site, dir, period, posts)
      index = ArchiveIndex.new(site, site.source, dir, period, posts)
      index.render(site.layouts, site.site_payload)
      index.write(site.dest)
      site.pages << index
    end
  end
end

{% endhighlight %}

For the archive page layout I created the following template (<code>archive.html</code>):
{% highlight html linenos=table %}
---
layout: default
---
<h1 class="page-title">Archive</h1>
<div id="archive-nav">
{% raw %}
{% for post in site.posts %}
  {% capture year %}{{ post.date | date: '%Y' }}{% endcapture %}
  {% capture nyear %}{{ post.next.date | date: '%Y' }}{% endcapture %}
  {% if year != nyear %}
    <a {% if page.url contains year %}class="active"{% endif %} href="/{{year}}/">{{year}}</a>
  {% endif %}
{% endfor %}
</div>
<h2> Posts : {% if page.period["month"] != nil %} {{ page.period["month"] }} - {% endif %} {{page.period["year"]}} </h2><br>
<ul style="list-style-type: none;">
  {% for post in page.period_posts reversed %}
    {% unless post.next %}
        <h3>{{ post.date | date: '%Y %b' }}</h3>
      {% else %}
        {% capture currPostDate %}{{ post.date | date: '%Y %b' }}{% endcapture %}
        {% capture nextPostDate %}{{ post.next.date | date: '%Y %b' }}{% endcapture %}
        {% if currPostDate != nextPostDate %}
          <h3>{{ post.date | date: '%Y %b' }}</h3>
        {% endif %}
      {% endunless %}

    <li>{{ post.date | date: '%b %d' }} - <a href="{{post.url}}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
{% endraw %}
{% endhighlight %}

With the above code each month will have the corresponding archive page.
I wanted also to have a global archive page so the archives can be accessed from the sidebar menu.
For this, I created the following page (<code>archive.html</code>) on the same level of the <code>about.md</code> and <code>index.html</code> page:
{% highlight html linenos=table %}
---
layout: page
title: Archive
permalink: /archive/
---
{% raw %}
<div id="archive-nav">
{% for post in site.posts %}
  {% capture year %}{{ post.date | date: '%Y' }}{% endcapture %}
  {% capture nyear %}{{ post.next.date | date: '%Y' }}{% endcapture %}
  {% if year != nyear %}
    <a {% if page.url contains year %}class="active"{% endif %} href="/{{year}}/">{{year}}</a>
  {% endif %}
{% endfor %}
</div>

<h2> Posts :</h2>
<ul style="list-style-type: none;">
  {% for post in site.posts %}
    {% unless post.next %}
        <h3>{{ post.date | date: '%Y %b' }}</h3>
      {% else %}
        {% capture currPostDate %}{{ post.date | date: '%Y %b' }}{% endcapture %}
        {% capture nextPostDate %}{{ post.next.date | date: '%Y %b' }}{% endcapture %}
        {% if currPostDate != nextPostDate %}
          <h3>{{ post.date | date: '%Y %b' }}</h3>
        {% endif %}
      {% endunless %}

    <li>{{ post.date | date: '%b %d' }} - <a href="{{post.url}}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
{% endraw %}
{% endhighlight %}

### Categories Page
I wanted to add a page like the previous archives page but for post categories.
I used [Dave Perret's](http://www.daveperrett.com/) category generator from his
jekyll plugins [repo](https://github.com/recurser/jekyll-plugins) (<code>categories_generator.rb</code>).

{% highlight ruby %}
module Jekyll

  # The CategoryIndex class creates a single category page for the specified category.
  class CategoryPage < Page

    # Initializes a new CategoryIndex.
    #
    #  +template_path+ is the path to the layout template to use.
    #  +site+          is the Jekyll Site instance.
    #  +base+          is the String path to the <source>.
    #  +category_dir+  is the String path between <source> and the category folder.
    #  +category+      is the category currently being processed.
    def initialize(template_path, name, site, base, category_dir, category)
      @site  = site
      @base  = base
      @dir   = category_dir
      @name  = name

      self.process(name)

      if File.exist?(template_path)
        @perform_render = true
        template_dir    = File.dirname(template_path)
        template        = File.basename(template_path)
        # Read the YAML data from the layout page.
        self.read_yaml(template_dir, template)
        self.data['category']    = category
        # Set the title for this page.
        title_prefix             = site.config['category_title_prefix'] || 'Category: '
        self.data['title']       = "#{title_prefix}#{category}"
        # Set the meta-description for this page.
        meta_description_prefix  = site.config['category_meta_description_prefix'] || 'Category: '
        self.data['description'] = "#{meta_description_prefix}#{category}"
      else
        @perform_render = false
      end
    end

    def render?
      @perform_render
    end

  end

  # The CategoryIndex class creates a single category page for the specified category.
  class CategoryIndex < CategoryPage

    # Initializes a new CategoryIndex.
    #
    #  +site+         is the Jekyll Site instance.
    #  +base+         is the String path to the <source>.
    #  +category_dir+ is the String path between <source> and the category folder.
    #  +category+     is the category currently being processed.
    def initialize(site, base, category_dir, category)
      template_path = File.join(base, '_layouts', 'category_index.html')
      super(template_path, 'index.html', site, base, category_dir, category)
    end

  end

  # The CategoryFeed class creates an Atom feed for the specified category.
  class CategoryFeed < CategoryPage

    # Initializes a new CategoryFeed.
    #
    #  +site+         is the Jekyll Site instance.
    #  +base+         is the String path to the <source>.
    #  +category_dir+ is the String path between <source> and the category folder.
    #  +category+     is the category currently being processed.
    def initialize(site, base, category_dir, category)
      template_path = File.join(base, '_includes', 'custom', 'category_feed.xml')
      super(template_path, 'atom.xml', site, base, category_dir, category)

      # Set the correct feed URL.
      self.data['feed_url'] = "#{category_dir}/#{name}" if render?
    end

  end

  # The Site class is a built-in Jekyll class with access to global site config information.
  class Site

    # Creates an instance of CategoryIndex for each category page, renders it, and
    # writes the output to a file.
    #
    #  +category+ is the category currently being processed.
    def write_category_index(category)
      target_dir = GenerateCategories.category_dir(self.config['category_dir'], category)
      index      = CategoryIndex.new(self, self.source, target_dir, category)
      if index.render?
        index.render(self.layouts, site_payload)
        index.write(self.dest)
        # Record the fact that this pages has been added, otherwise Site::cleanup will remove it.
        self.pages << index
      end

      # Create an Atom-feed for each index.
      feed = CategoryFeed.new(self, self.source, target_dir, category)
      if feed.render?
        feed.render(self.layouts, site_payload)
        feed.write(self.dest)
        # Record the fact that this pages has been added, otherwise Site::cleanup will remove it.
        self.pages << feed
      end
    end

    # Loops through the list of category pages and processes each one.
    def write_category_indexes
      if self.layouts.key? 'category_index'
        self.categories.keys.each do |category|
          self.write_category_index(category)
        end

      # Throw an exception if the layout couldn't be found.
      else
        throw "No 'category_index' layout found."
      end
    end

  end


  # Jekyll hook - the generate method is called by jekyll, and generates all of the category pages.
  class GenerateCategories < Generator
    safe true
    priority :low

    CATEGORY_DIR = 'categories'

    def generate(site)
      site.write_category_indexes
    end

    # Processes the given dir and removes leading and trailing slashes. Falls
    # back on the default if no dir is provided.
    def self.category_dir(base_dir, category)
      base_dir = (base_dir || CATEGORY_DIR).gsub(/^\/*(.*)\/*$/, '\1')
      category = category.gsub(/_|\P{Word}/, '-').gsub(/-{2,}/, '-').downcase
      File.join(base_dir, category)
    end

  end


  # Adds some extra filters used during the category creation process.
  module Filters

    # Outputs a list of categories as comma-separated <a> links. This is used
    # to output the category list for each post on a category page.
    #
    #  +categories+ is the list of categories to format.
    #
    # Returns string
    def category_links(categories)
      base_dir = @context.registers[:site].config['category_dir']
      categories = categories.sort!.map do |category|
        category_dir = GenerateCategories.category_dir(base_dir, category)
        # Make sure the category directory begins with a slash.
        category_dir = "/#{category_dir}" unless category_dir =~ /^\//
        "<a class='category' href='#{category_dir}/'>#{category}</a>"
      end

      case categories.length
      when 0
        ""
      when 1
        categories[0].to_s
      else
        categories.join(', ')
      end
    end

    # Outputs the post.date as formatted html, with hooks for CSS styling.
    #
    #  +date+ is the date object to format as HTML.
    #
    # Returns string
    def date_to_html_string(date)
      result = '<span class="month">' + date.strftime('%b').upcase + '</span> '
      result += date.strftime('<span class="day">%d</span> ')
      result += date.strftime('<span class="year">%Y</span> ')
      result
    end

  end

end

{% endhighlight %}

I altered the layout in order to match the rest of my blog (<code>category_index.html</code>):
{% highlight yaml %}
{% raw %}
---
layout: default
---


<h1> {{ page.title }} </h1><br>
<ul style="list-style-type: none;">
  {% for post in site.categories[page.category] %}
    {% unless post.next %}
        <h3>{{ post.date | date: '%Y %b' }}</h3>
      {% else %}
        {% capture currPostDate %}{{ post.date | date: '%Y %b' }}{% endcapture %}
        {% capture nextPostDate %}{{ post.next.date | date: '%Y %b' }}{% endcapture %}
        {% if currPostDate != nextPostDate %}
          <h3>{{ post.date | date: '%Y %b' }}</h3>
        {% endif %}
      {% endunless %}

    <li>{{ post.date | date: '%b %d' }} - <a href="{{post.url}}">{{ post.title }}</a>
    <div class="meta">{% include categories.html %}</div>
</li>
  {% endfor %}
</ul>
{% endraw %}
{% endhighlight %}

### Assets minify
In order to combine and compress my css and js files I used the
[Jekyll Asset Pipeline Reborn](https://github.com/kitsched/japr).
I just ran:
{% highlight bash %}
sudo gem install japr
{% endhighlight %}

I moved all my css and js files on a new folder, named <code>_assets</code>
and I added on the <code>head.html</code> the following liquid block in order to include the generated
files
{% highlight yaml %}
{% raw %}
{% css_asset_tag global %}
    - /_assets/css/poole.css
    - /_assets/css/monokai.css
    - /_assets/css/hyde.css
{% endcss_asset_tag %}
{% endraw %}
{% endhighlight %}

Then I created the file <code>categories_generator.rb</code> on the <code>_plugins</code> folder with the following:
{% highlight ruby linenos=table %}
require 'japr'

module JAPR
  class CssCompressor < JAPR::Compressor
    require 'yui/compressor'

    def self.filetype
      '.css'
    end

    def compress
      return YUI::CssCompressor.new.compress(@content)
    end
  end

  class JavaScriptCompressor < JAPR::Compressor
    require 'yui/compressor'

    def self.filetype
      '.js'
    end

    def compress
      return YUI::JavaScriptCompressor.new(munge: true).compress(@content)
    end
  end
end

{% endhighlight %}

The two last classes are responsible for the compression of css and js files.

That was my blog's main functionality/features.<br>
For any other feature change or addition I will write a separate post.


