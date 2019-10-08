# Creating a Many to Many relationship with metadata attached

If you're creating a many to many relationship between two objects and you want to edit that relationship using an 
Inline Panel in the admin interface, you will probably end up using something like the following:

```python
from django.db import models
from modelcluster.fields import ParentalKey
from wagtail.admin.edit_handlers import PageChooserPanel


class RelatedPages(Orderable):
    child = models.ForeignKey(
        'app.ContentPage',
        on_delete=models.CASCADE,
        related_name='related_page'
    )
    index = ParentalKey('app.IndexPage', related_name='related_pages_list')

    panels = [
        PageChooserPanel('child', 'app.ContentPage'),
    ]
    

class IndexPage(Page):

    # ...

    content_panels = [
        MultiFieldPanel([
            InlinePanel('related_pages_list'),
        ], heading=_("Related Pages"))
    ]
```

This would then allow you to query related pages like so:

```python
IndexPage().related_pages_list.all()
```

However, the objects returned will be of the `RelatedPage` type. In some cases this might be what you want, but often you
will want to return the underlying model, eg. `ContentPage`. To do this you can add a `ManyToManyField` to the `IndexPage`:

```python
class IndexPage(Page):
    # ...
    related_pages = models.ManyToManyField(
        ChildPage,
        through=RelatedPages,
        through_fields=('index', 'child')
    )
```

Now, you can query related pages like so:

```python
IndexPage().related_pages.all()
```

The objects returned now will be of the `ChildPage` type. What's more, you can query based on the intermediary table using
the related name, for example:

```python
IndexPage().related_pages.order_by('related_page__sort_order')
```

This will give you the child page objects, but in the order defined in the admin interface using inline panels.
