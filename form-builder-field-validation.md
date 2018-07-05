### Adding custom validation to Form Builder form fields

Occasionally you need to be able to perform additional validation on the Form Builder built in forms. This can be achieved
by overriding the get_form method on the AbstractForm class and subsequently overriding the clean method on the retrieved form.

``` python

from wagtail.contrib.forms.models import AbstractForm

def clean_form(self):
    """
    Override the form builder form clean method to add validation to check for existing
    submissions
    :param self:
    :return:
    """
    email = self.cleaned_data.get('email')
    if email:
        existing_submissions = self.submission_class.objects.filter(
            page=self.submission_page,
            form_data__icontains=email,
        )
        # Add a non field error to this submission if we have any existing entries.
        if existing_submissions.count():
            self.add_error(None, 'You\'ve already submitted!')

    return self.cleaned_data


class MyCustomForm(AbstractForm):


    def get_form(self, *args, **kwargs):

        # Call AbstractForm get_form method to retrieve form normally
        form = super(MyCustomForm, self).get_form(*args, **kwargs)

        # Dynamically override the clean method of the form builder form with a descriptor method
        form.clean = clean_form.__get__(form)

        # Attach anything else you need to the form in order to do your validation
        # This examples attaches the Form Submission class and the Page as properties of the form to aid validation.
        form.submission_class = self.get_submission_class()
        form.submission_page = self

        return form

```
