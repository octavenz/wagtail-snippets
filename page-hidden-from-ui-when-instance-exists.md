### Page dissapears from UI when an instance exists

Wagtail's Page model has a [is_creatable](http://docs.wagtail.io/en/v2.1/reference/pages/model_reference.html#wagtail.core.models.Page.is_creatable) class attribute that can be used to hide the page from Wagtail's admin. We can use this to enforce rules around Page creation using a descriptor.

``` python
  # app.models.py


  class OnlyOneDescriptor(object):
      """A descriptor that only allows a single instance of a Page type to
      be created."""

      def __get__(self, obj, objtype):
          if objtype.objects.count() == 0:
              return True
          return False

  class StandardPage(Page):
      is_creatable = OnlyOneDescriptor()
```
