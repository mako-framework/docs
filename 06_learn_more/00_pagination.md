# Pagination

--------------------------------------------------------

* [Usage](#usage)
* [Example view](#example_view)

--------------------------------------------------------

The pagination class makes easy to generate pagination links.

--------------------------------------------------------

<a id="usage"></a>

### Usage

First you'll have to create a pagination object. The first parameter is the pagination partial view and the second is the total count of the items you want to paginate.

There's also a third optional parameter that lets you set the number of items to be displayed on each page. If this is left empty then it'll use the default value specified in the pagination config file.

	$pagination = $this->pagination->create(Articles::count());

Once the pagination object is created we can fetch the articles from the database. Use the ```limit``` and ```offset``` methods to set the range of your query.

	$articles = Articles::limit($pagination->limit())->offset($pagination->offset())->all();

The query builder and ORM will also allow you to use the pagination object directly.

	$articles = Articles::paginate($pagination)->all();

You can render the pagination partial view in the article list view using the ```render``` method.

	$paginationLinks = $pagination->render('partials.pagination');

--------------------------------------------------------

<a id="example_view"></a>

### Example view

Here's an example of what the pagination partial view can look like (using the template syntax) if you're using Twitter Bootstrap.

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
