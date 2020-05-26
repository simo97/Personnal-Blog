## Handling SEO with Wagtail CMS

One of the most important things to do when building a website is to handle SEO (Search Engine Optimization), it allows the web pages to be well referenced by the various search engines over the internet. It follows a set of specifics rules and those rules has to be used by content editor, writer or who ever is in charge of producing content for the website. But the point is Wagtail as a content management system comes with the barely minimum for to address this feature, fortunately some clever developers has written down some package to handle this.

# What is available ?

A quick tour  [here](https://github.com/springload/awesome-wagtail#seo-and-smo)  shows us a list of them. We have :
-  [wagtail-metadata](https://github.com/takeflight/wagtail-metadata) 
-  [wagtail-metadata-mixin](https://github.com/bashu/wagtail-metadata-mixin) 
-  [wagtail-schema.org](https://github.com/takeflight/wagtail-schema.org) 
-  [wagtail-opengraph-image-generator ](https://github.com/candylabshq/wagtail-opengraph-image-generator) 
-  [wagtail-redirect-importer](https://github.com/Frojd/wagtail-redirect-importer)  

# How to install ?

In this post we will focus on wagtail-metadata which is quite easy to install (like every others wagtail plugin);
```sh
pip install wagtail-metadata
```
Add the application to `INSTALLED_APPS` in django settings.
```python
INSTALLED_APPS = [
    ...
    'wagtailmetadata',
]
```
and it's done. No migrations to apply (at this level at least). 

# How does it work ?
To properly add SEO support on a web page you should follow a set of rules and add some meta tags to the HTML document, these are named Opengraph meta tag, in the `<head>` tag. These tags are basically: 
- Meta content type
```html
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
```
- Meta title
```html
<meta name="title" content="page title here">
```
- Meta description
```html
<meta name="description" content="some description goes here">
```

- Meta author
```html
<meta name="author" content="name of person who wrote the content">
```

It's more explained here on  [https://moz.com/](https://moz.com/blog/the-ultimate-guide-to-seo-meta-tags).

Our goal is to easily add and update those tags and their values on the site's pages and it's the goal of that lib. To do that as in wagtail every page is based on a type defined in models we will need to extends `MetadataPageMixin` from `from wagtailmetadata.models import MetadataPageMixin` and we will have a code looking like this : 

```python
from wagtail.core.models import Page
from wagtailmetadata.models import MetadataPageMixin

class HomePage(MetadataPageMixin, Page):
    pass

class AboutPage(MetadataPageMixin, Page):
    pass
```
Because of our inheritance works in python and this lib implementation `MetadataPageMixin` should be put before `Page` class. 
At this point you can makemigrations on you apps and go to the site admin and try to create or update a page to see some changes. 
```python
python manage.py makemigrations
python manage.py migrate
python manage.py runserver 
```

![Capture d’écran 2020-05-24 à 21.41.57.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1590352915239/ihaNzg5_A.png)

Now we have more fields on the page admin where we can edit SEO related field under the `promotion` panel, now let write some tag into the template to display those informations at the right place. 
As it need to appear on all the site's pages we will have to put it into the base template of the site, by default is named `base.html` and is found in you main project folder, if your project is named `mysite` you will find it out inside `mysite/templates/base.html`. The point is now to write useful Django tag to return this into the `head` element like in this snippet:

```html
{% load wagtailmetadata_tags %}
<html>

<head>
{% meta_tags %}
</head>
<body>
</body>
</html>
```

This is enough for a basic SEO support, it will handle the meta field generation process and will output meta tag for social media (facebook meta, Twitter meta, etc). At this point you can checkout the rendered HTML and see the result. 

![Capture d’écran 2020-05-24 à 21.44.47.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1590353104556/Xey1tuGtT.png)

As you see, by default this provide meta tags for twitter and OpenGraph (which is more common), it also provide simple meta description content for HTML itself.

To test the render you can open a OpenGraph checker like [twitter card validator](https://cards-dev.twitter.com/validator) to see how links to you pages will be interpreted and displayed on twitter. Here is the result of validation on one website i've build using Wagtail CMS. There is a lot of other tools out there to do this kind of tests.
![Capture d’écran 2020-05-24 à 21.50.12.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1590353445557/8hpy1-5H5.png)

There is a lot more stuff achievable with this lib, the purpose of this post was to introduce it and the concept. You can do more by checking it documentation which is well written with a lot of examples.

Thank you for reading, you can hit me up on  [twitter](https://twitter.com/adonis__simo)  if you want to know more.