# Pagination

--------------------------------------------------------

* [Usage](#usage)
	- [Basics](#usage_basics)
	- [With the query builder](#usage_with_the_query_builder)
* [Pagination rendering](#pagination_rendering)
	- [Example view](#pagination_rendering_example_view)

--------------------------------------------------------

The pagination class makes easy to generate pagination links.

--------------------------------------------------------

<a id="usage"></a>

### Usage

<a id="usage_basics"></a>

#### Basics

First you'll have to create a pagination object. The first parameter is the total count of the items you want to paginate. The second parameter is optional and lets you set the number of items to be displayed on each page. If left empty then it'll use the default value specified in the pagination config file. There's also an optional third parameter that lets you override the other settings set in the configuration file.

```
$pagination = $this->pagination->create(Articles::count());
```

Once the pagination object is created we can fetch the articles from the database. Use the `limit` and `offset` methods to set the range of your query.

```
$articles = Articles::limit($pagination->limit())->offset($pagination->offset())->all();
```

<a id="usage_with_the_query_builder"></a>

#### With the query builder

You can also use the `pagination` method of the [query builder](:base_url:/docs/:version:/databases-sql:query-builder). It will automatically perform the count and apply the appropriate limit and offset.

The first parameter is optional and lets you set the number of items to be displayed on each page. If left empty then it'll use the default value specified in the pagination config file. There's also an optional second parameter that lets you override the other settings set in the configuration file.

```
$articles = Articles::paginate();
```

--------------------------------------------------------

<a id="pagination_rendering"></a>

### Pagination rendering

You can render the pagination partial in the article list view using the `render` method.

```
$pagination->render('partials.pagination');
```

If you used the `paginate` method of the query builder then you can access the pagination object using the `getPagination` method on the result set.

```
$articles->getPagination()->render('partials.pagination');
```

<a id="pagination_rendering_example_view"></a>

#### Example view

Here's an example of what a pagination partial can look like if you're using Twitter Bootstrap.

```
<ul class="pagination">
	{% if(isset($previous)) %}
		<li><a href="{{preserve:$first}}">First</a></li>
		<li><a href="{{preserve:$previous}}">Prev</a></li>
	{% endif %}
	{% foreach($pages as $page) %}
		{% if($page['is_current']) %}
			<li class="active"><span>{{$page['number']}}</span></li>
		{% else %}
			<li><a href="{{preserve:$page['url']}}">{{$page['number']}}</a></li>
		{% endif %}
	{% endforeach %}
	{% if(isset($next)) %}
		<li><a href="{{preserve:$next}}">Next</a></li>
		<li><a href="{{preserve:$last}}">Last</a></li>
	{% endif %}
</ul>
```
