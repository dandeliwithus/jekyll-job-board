# Easy Job Board 

This is a job board created in Jekyll using search via Algolia & Bootstrap 4.

### 1. first you will need to create your jekyll project and give it a name

```bash
$ jekyll new project-name
$ cd project-name
$ bundle exec jekyll serve
```

You have create and started a blank project now you can go over to `http://localhost:4000` to see it in action.

### 2. Creating your enviroment

You will now need to create the following folders and files in your directory.

```bash
├── 404.html
├── Gemfile
├── Gemfile.lock
├── _config.yml
├── _includes
│   ├── footer.html
│   └── navbar.html
├── _layouts
│   ├── default.html
│   └── vacancy-post.html
├── _vacancies
├── css
│   ├── main.css
│   ├── main.css.map
│   └── main.scss
├── index.html
└── readme.md
```

The tree in the directory is pretty simple, we dont need the `_post` folder as personally we will use a project. 

Lets set this up now. You will need to place the following within your `_config.yml` file

```yml
collections:
    vacancies:
        output: true
```

This has now set up the `_vacancies` folder we created. The reason I have used a collection instead of the standard `_post` folder is due to the name of the .md files. We now longer have to use a date in front of the file name. 

We can arrange by date using front matter and jekyll later on.

### 3. Marking up the pages with Bootstrap

For the markup and styling we will be using [Bootstrap 4](https://getbootstrap.com/docs/4.3/getting-started/introduction/) and some simple SCSS. Go ahead and grab the starter template.

We're going to make some simple adjustments to the template so we're able to always have a sticky footer. For more info on that check the example [here](https://getbootstrap.com/docs/4.3/examples/sticky-footer/)

We will need to place the same template in both `default.html` and `vacancies.html` we will be modifiing `vacancies.html` later to layout a job posting.

Lets come back to the styles and HTML later.

### 4. Adding Algolia

`jekyll-algolia` is a Jekyll plugin that lets you push all your content in an Algolia index.

##### Requirments

- Jekyll >= 3.6.0
- Ruby >= 2.3.0
- Bundler

##### Installation

You need to add `jekyll-algolia` to your `Gemfile`, as part of the `:jekyll-plugins` group. If you do not yet have a Gemfile, here is the minimal content you’ll need:

```ruby
source 'https://rubygems.org'

gem 'jekyll', '~> 3.6'

group :jekyll_plugins do
    gem 'jekyll-algolia'
end
```

Then, run `bundle install` to update your dependencies.

If everything went well, you should be able to run `jekyll help` and see the `algolia` subcommand listed.

##### Configuration 

You need to provide your Algolia credentials for this plugin to index your site.

_If you don’t yet have an Algolia account, you can open a free [Community plan here](https://www.algolia.com/users/sign_up/hacker). Once signed in, you can get your credentials from your [dashboard](https://www.algolia.com/api-keys)._

Once you have your credentials, you should define your `application_id` and `index_name` inside your `_config.yml` file like this:

```yml
# _config.yml

algolia:
    application_id: your_application_id
    index_name: vacancies
    search_only_api_key: your_search_only_api_key

    # custom settings ---
    extensions_to_index:
        - md
    settings:
    searchableAttributes:
        - jobtitle
        - tags
```

We will go into the custom settings later on.

##### Usage 

Once your credentials are setup, you can run the indexing by running the following command:

`ALGOLIA_API_KEY='your_admin_api_key' bundle exec jekyll algolia`

Note that `ALGOLIA_API_KEY` should be set to your admin API key. This key has write access to your index so will be able to push new data. This is also why you have to set it on the command line and not in the `_config.yml` file: you want to keep this key secret and not commit it to your versioning system.

For more info on Algolia for Jekyll go [here](https://community.algolia.com/jekyll-algolia/getting-started.html)

#### 5. Setting up the vancany posts

We now have "backend" and the search functionality built into our pages. We now need to create some job descriptions in our `_vacancies` folder. 

If you notice from the return statement above there is the following elements.

- `${hit.jobtitle}`
- `${hit.dateposted}`
- `${hit.salary}`
- `${hit.location}`

These are custom bits of Front Matter on every `.md` file for example:

```yml
---
layout: vacancy-post
jobtitle:  "Awesome Jekyll Developer"
dateposted:   2019-02-10

salary: £1,000,000 per year
location: Anywhere

tags:
    - developer
    - front-end
    - jekyll

reference: 12345678
---
```

The `${content}` block will pull out the first paragraph of the job description and use this as a snippet. So just remember not to use any headings at the beginning of the job description.

### 6. Adding InstantSearch.js

Because the `minima` is pre-packaged as a dependency, if you want to edit it, you need to overwrite some of its files locally. For this tutorial, we’ll need to update [one file](https://raw.githubusercontent.com/jekyll/minima/master/_layouts/home.html) from the original theme.

```html
<div class="container">

    <div id="search-searchbar"></div>
    
    <div class="post-list" id="search-hits">
        {% assign sorted = (site.vacancies | sort: 'dateposted') | reverse %}
        {% for job in sorted %}
        <div class="post-item"></div>
        {% endfor %}
    </div>

    {% include algolia.html %}

</div>
```

I am just going to give you the code I have created, if you want to start from scratch using the official InstantSearch.js tutorial you can from [here](https://community.algolia.com/jekyll-algolia/blog.html)

To use what I have created, in your `_includes` folder create a file called `algoia.html` and paste the following:

```html
<link rel="stylesheet" type="text/css"
    href="https://cdn.jsdelivr.net/npm/instantsearch.js@2.10.4/dist/instantsearch.min.css">
<script src="https://cdn.jsdelivr.net/npm/instantsearch.js@2.10.4"></script>
<link rel="stylesheet" type="text/css"
    href="https://cdn.jsdelivr.net/npm/instantsearch.js@2.10.4/dist/instantsearch-theme-algolia.min.css">

<script>
    // Instanciating InstantSearch.js with Algolia credentials
    const search = instantsearch({
        appId: '{{ site.algolia.application_id }}',
        indexName: '{{ site.algolia.index_name }}',
        apiKey: '{{ site.algolia.search_only_api_key }}'
    });

    const hitTemplate = function (hit) {

        let date = '';
        if (hit.date) {
            date = moment.unix(hit.date).format('MMM D, YYYY');
        }

        let url = `{{ site.url }}${hit.url}#${hit.anchor}`;

        const title = hit._highlightResult.title.value;

        let breadcrumbs = '';
        if (hit._highlightResult.headings) {
            breadcrumbs = hit._highlightResult.headings.map(match => {
                return `<span class="post-breadcrumb">${match.value}</span>`
            }).join(' > ')
        }

        const content = hit._highlightResult.html.value;

        return `
            <div class="post-item">
                <h2><a class="post-link" href="${hit.url}">${hit.jobtitle}</a></h2>
                <span class="text-muted text-lower date-posted">Posted ${hit.dateposted}</span>
                <div class="listing-meta my-3">
                    <span class="listing-tags text-lower">${hit.salary}</span>
                    <span class="listing-tags text-lower">${hit.location}</span>
                </div>
                
                <div class="post-snippet">${content}</div>
                <hr>
            </div>
        `;
    }

    // Adding searchbar and results widgets
    search.addWidget(
        instantsearch.widgets.searchBox({
            container: '#search-searchbar',
            placeholder: 'Search for vacancies',
            poweredBy: true // This is required if you're on the free Community plan
        })
    );
    search.addWidget(
            instantsearch.widgets.hits({
                container: '#search-hits',
                empty: 'No results',
                templates: {
                    item: hitTemplate 
                }
            })
        );

    // Starting the search
    search.start();
</script>

```

After all that you should have a page that looks like the following:

--- insert phot ---

