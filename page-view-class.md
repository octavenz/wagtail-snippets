### Hand off view logic from Page models to view classes

#### Note:

If you like this pattern you can enforce it by installing the flake8 linting plugin: https://github.com/octavenz/flake8-wagtail-no-serve

#### Description

Wagtail's Page model does a lot of things besides being a django model. Some would say too many things. Included in this list is handling view logic, which can be (in our experience) a common cause of excessively large and complicated Page models. A solution to this is to instead defer to a view class for all view logic. To do this we can create (or update an existing) BasePage class with the following:

``` python
  # app.models.pages.base.py
  from wagtailcore.models import Page
  
  class BasePage(Page):
      """
      Wagtail Page model that defers view logic to a separate view class,
      thus (in part) restoring the Django MVT architecture.
      """

    view_class = PageView

    def serve(self, request, *args, **kwargs):
        view = self.get_view_class(request, *args, **kwargs).as_view(page=self)
        return view(request, *args, **kwargs)

    def get_view_class(self, request, *args, **kwargs):
        return self.view_class
        
    class Meta:
        abstract = True
```

Where PageView can be defined as:

``` python
    from django.template.response import TemplateResponse
    from django.views import View

    class PageView(View):
        """
        Basic view class that emulates the serve method of a Wagtail Page
        model.

        This class allows for separation of view logic from the Page model,
        thus (in part) restoring Django's MVT architecture.
        """

        page = None

        def get(self, request, *args, **kwargs):
            """Emulates Page.serve method."""
            request.is_preview = getattr(request, 'is_preview', False)

            return TemplateResponse(
                request,
                template=self.get_template(request, *args, **kwargs),
                context=self.get_context_data(request, *args, **kwargs),
            )

        def get_template(self, request, *args, **kwargs):
            """
            Allows overriding the template for the view while defaulting to
            the Wagtail page's template.
            """
            return self.page.get_template(request, *args, **kwargs)

        def get_context_data(self, request, *args, **kwargs):
            """
            Generates context for the view.

            General pattern here is that anything that is request specific
            should be defined in this method, while anything that is page
            instance specific should live in the get_context method of that
            page.
            """
            return self.page.get_context(request, *args, **kwargs)

```

In the above case we're inherting from the basic django View class, but you can do anything you want here really. The main point is to get the view logic out of the Page model itself.

Because the Page model retains one of it's other functions as a url router we can add logic to it to conditionally return different views, i.e we can return a differnt view on a route depending if it is via an ajax request (i.e. a request for additional paginated content) or not:

```python
    class NewsPage(BasePage):

        def get_view_class(self, request, *args, **kwargs):
            if request.headers.get('x-requested-with') == 'XMLHttpRequest':
                return NewsListingView
            return NewsLandingPageView

```

The final piece of the puzzle is for when you are using the `@route` decorator, use the following mixin instead:

```python
    from wagtail.contrib.routable_page.models import RoutablePageMixin, route

    class RoutablePageViewMixin(RoutablePageMixin):
        """
        Custom routable mixin that works with pages extending from BasePage.
        """

        @route(r'^$')
        def index_route(self, request, *args, **kwargs):
            """Switches out the default index route for one that uses the view class."""
            view = self.get_view_class(request, *args, **kwargs).as_view(page=self)
            return view(request, *args, **kwargs)
```
