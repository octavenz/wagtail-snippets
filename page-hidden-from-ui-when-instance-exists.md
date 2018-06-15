### Page dissapears from UI when an instance exists

``` python
  # app.models.py
  class StandardPage(Page):
      @classproperty
      def parent_page_types(self):
          return [] if self.objects.count() > 0 else None
```
