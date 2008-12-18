---
layout: post
title: ManyToMany in templates
---
Given these models..

{% highlight python %}
from django.db import models


class Tag(models.Model):

	def __str__(self):
		return self.title

	class Admin:
		fields = (
			(None, {'fields':('title','slug','description')}),
		)						
		list_display = ('title','description')

	slug = models.SlugField('Slug',prepopulate_from=("title",),help_text='Automagically generate from title.',primary_key=True)
	title = models.CharField('Title', maxlength=30, core=True)
	description = models.TextField('Description', help_text='Short summary of this tag')


class Post(models.Model):

	def __str__(self): 
		return self.title

	def save(self):
		import textile
		self.body_html = textile.textile(self.body)
		super(Post, self).save()	

	def get_absolute_url(self):
		return "/%s/%s/" % (self.pub_date.strftime("%Y/%b/%d").lower(), self.slug)			

	class Admin:
		fields = (
			(None, {'fields':('title','slug','pub_date','body','tags','enable_comments','lead_image')}),
		)						
		list_display = ('title','pub_date','enable_comments')
		list_filter = ['pub_date','enable_comments','tags']
		search_fields = ['title','body']
		date_hierarchy = 'pub_date'

	title = models.CharField(maxlength=200)
	slug = models.SlugField(prepopulate_from=('title',))
	pub_date = models.DateTimeField('Date Published',auto_now_add=True)
	body = models.TextField(help_text='You can use Textile')
	body_html = models.TextField('Entry as html',blank=True, null=True)
	tags = models.ManyToManyField(Tag)
	lead_image = models.ImageField(upload_to='img/blog-posts/lead', blank=True)
	enable_comments = models.BooleanField(default=True)

{% endhighlight %}


I wanted to output a list of tags relating to a Post in a template. After some trial and error, hit upon..

{% highlight python %}
    for article in latest 
	 get_free_comment_count for news.post article.id as comment_count 
	<h2><a href=" article.get_absolute_url"> article.title </a></h2>
			<p class="date small">Posted on  article.pub_date|date:"F j, Y"  <a href=" article.get_absolute_url }}#comments"> comment_count comment comment_count|pluralize </a></p>
			article.body_html 
			<p>tagged:  for tag in article.tags.all tag.title endfor </p>
    endfor 

{% endhighlight %}
