# Django-starter

## Create venv and activate
- `.\.venv\Scripts\activate `

## Create requirements.txt and install packages
`touch requirements.txt`

- Add required packages
Django
pillow
django-cleanup
django-allauth
django-htmx

- then install `pip install -r requirements.txt`
`pip install --upgrade pip`
`pip3 freeze --local > requirements.txt`

## start project `a_core` and setup DB
- `django-admin startproject a_core .`
- `python manage.py migrate`
- `python manage.py runserver`

## Create home page app `a_home`
- `python manage.py startapp a_home`
- Add it to INSTALLED_APPS in setting.py
- create view in a_home/views.py:
    def home_view(request):
    return render(request,'home.html')
- add path to views in root urls.py as:
    `from a_home.views import *`
    path('', home_view, name="home"),

## Add html files in templates dir
- `mkdir templates`
- register 'templates' dir in TEMPLATES var of settings.py 
- `'DIRS': [BASE_DIR / 'templates'],`
- File 1: base.html with code: https://github.com/andyjud/django-starter-assets
- create subfolder `templates/includes` and add these files:
- File 2: move `<messages>` into new includes/messages.html add `{% include 'includes/messages.html' %}` where it was
- File 3: move `<header>` into new includes/header.html add `{% include 'includes/header.html' %}` where it was
- create subfolder `templates/layout` and add these files:
- File 4: move `<content>` into new layout/blank.html add `{% block content %}` and `{% endblock %}`   where it was
- in blank.html: add `{% extends 'base.html' %}`, `<content>` between `{% block content %}` and `{% endblock %}`   
- File 5: create `home.html` in templates dir
- in home.html add: add `{% extends 'layouts/blank.html' %}`, `{% block content %}` and `{% endblock %}`   


## Create static folder in root dir
- `mkdir static`
- add static folder settings.py 
- add below STATIC_URL as `STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static'), ]`

## Create user profile pages
- create app `a_users` with: `python manage.py startapp a_users`
- add app to INSTALLED_APPS in settings.py
- create a model in models.py `Profille`
- Run `makemigrations` and `migrate`
- Register add in the admin.py wit view in admnin interface
- `admin.site.register(Profile)`

## Create signals.py  to create new profiles when users register.
- Create file `a_users/signals.py`
- Add code for creating user
- register `signals.py` in the `apps.py`
- by adding : `def ready(self):import a_users.signals` unser class `AUsersConfig`

## Create superuser to access admin panel
- `python manage.py createsuperuser`
- pass username, email and passwords
- start the server `python manage.py runserver`

## Add avatar and username variable in `templates/includes/header.html`
- add {{ user.profile.avatar }} to img src
- add {{ user.profile.name }} replacing 'username' text

## Create a profile page in views.py
- add code:
def profile_view(request):
    profile = request.user.profile
    return render(request, 'a_users/profile.html', {'profile':profile})
- folder in `a_users/templates/a_users`
- add `profile.html` with code: https://github.com/andyjud/django-starter-assets/blob/main/profile.html
- create path in root urls `path('profile/', include('a_users.urls')),`
- NB remember to import `include` next to `path`
- Create `urls.py` in app folder `a_users/urls.py`
- add code:
from django.urls import path
from a_users.views import *

urlpatterns =[
    path('',profile_view, name='profile'),
]
- add new url in `header.html`in href of 'My Profile' as `{% url 'profile' %}`
- `runserver`and check updates
- NB: ensure `{% block}` has the same block name, template its extending from e.g. 'layout' 

## Create edit profile page
- create a `forms.py` in `a_users` app folder
- add code:
from django.forms import ModelForm
from django import forms
from .models import Profile


class ProfileForm(ModelForm):
    class Meta:
        model = Profile
        exclude = ['user']
        widgets = {
            'image': forms.FileInput(),
            'displayname': forms.TextInput(attrs={'placeholder': 'Add display name'}),
            'info': forms.Textarea(attrs={'rows': 3, 'placeholder': 'Add information'})
        }
- create a view for edit profile in views.py
- add code:
from django.shortcuts import render
from django.contrib.auth.decorators import login_required
from .forms import *

def profile_view(request):
    profile = request.user.profile
    return render(request, 'a_users/profile.html', {'profile':profile})

@login_required
def profile_edit_view(request):
    form = ProfileForm(isinstance=request.user.profile)
    return render(request, 'a_users/profile_edit.html', {'form':form})

- create `profile_edit.html` file in app templates
- add code: https://github.com/andyjud/django-starter-assets/blob/main/profile_edit.html
- add path in call `urls.py` as `path('edit/', profile_edit_view, name='profile-edit'),`
- updated href in the `header.html` in 'Edit Profile'

## Create a new layout `box.html` in `templates/layouts`
- create file: `box.html`
- add code: https://github.com/andyjud/django-starter-assets/blob/main/box.html
- change `{% extends %}` from `blank.hmtl` to `box.html`

## Add functionality to save form to database
- in `profile_edit_view` in app views.py add code:
if request.method == 'POST':
    form = ProfileForm(request.POST, request.FILES, instance=request.user.profile)
    if form.is_valid():
        form.save()
        return redirect('profile')
- import `redirect` next to `render`
- add media folder to store uploaded user files
- create folder `media` in root dir 
- add 'media' to settings.py
- add below STATICFILES_DIRS
MEDIA_URL = 'media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
- Add path to the root `urls.py`
imports:
from django.conf.urls.static import static
from django.conf import settings
add code below `urlpatterns =[]`:
# only used in development
urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
- refresh page and test 
- Add django cleanup to delete duplicated images
- in the INSTALLED_APPS, add code above custom apps
`'django_cleanup.apps.CleanupConfig',`

## Setup user logout - allauth
- refer: https://docs.allauth.org/en/latest/installation/quickstart.html
- in `settings.py`:
add `AUTHENTICATION_BACKENDS` below `MIDDLEWARE`:
AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
    'allauth.account.auth_backends.AuthenticationBackend'
]

add allauth as in `INSTALLED_APPS` above custom apps:
'allauth',
'allauth.account',
'allauth.socialaccount',

add allauth middleware in `MIDDLEWARE`:
'allauth.account.middleware.AccountMiddleware',

add accounts path to the root `urls.py`:
path('accounts/',  include('allauth.urls')),

- Run migration to impelement changes
`python manage.py migrate`

- add logout href 'Log Out' to `header.html` as `{% url 'account_logout' %}`
- `runserver` and test logout

## Style logout page
create dir `templates/allauth/layouts`
add `base.html` file and add code:
{% extends 'layouts/box.html' %}

{%block class %}allauth{% endblock %}

{% block content %}
{% endblock %}

## Add login link in `header.html`
- add login href 'Login' to `header.html` as `{% url 'account_login' %}`
- add configrs to `settings.py` at the bottom:
LOGIN_REDIRECT_URL = '/'

EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
ACCOUNT_AUTHENTICATION_METHOD = 'email'
ACCOUNT_EMAIL_REQUIRED= True

## Override notification pop-ups for login/logout
- create folder in `templates/account/messages`
- add file `logged_in.txt` and `logged_out.txt`

## Add sign-up link in `header.html`
- add sign-up href 'Sign up' to `header.html` as `{% url 'account_signup' %}?next={% url 'profile-onboarding' %}`
- `?next=` directs user to the `profile-onboarding` page
- add `profile-onboarding` page to app `urls.py`:
`path('onboarding/', profile_edit_view, name='profile-onboarding'),`

- in `profile_edit_view` in `a_users/views.py` add:
if request.path == reverse('profile-onboarding'):
    onboarding = True
else:
    onboarding = False

- add `'onboarding':onboarding` to rendered context object
- import reverse `from django.urls import reverse`

- in `profile_edit.html` revise `<h1>` element:
{% if onboarding %}
<h1 class="mb-4">Complete your Profile</h1>
{% else %}
<h1 class="mb-4">Edit your Profile</h1>
{% endif %}

- in `profile_edit.html` revise `cancel` button in `<form>` element:
{% if onboarding %}
<a class="button button-gray ml-1" href="{% url 'home' %}">Skip</a>
{% else %}
<a class="button button-gray ml-1" href="{{ request.META.HTTP_REFERER }}">Cancel</a>
{% endif %}

## Ensure only lower case username in DB
- in `a_users/signals.py` add:
@receiver(pre_save,sender=User)
def user_presave(sender, instance, **kwargs):
    if instance.username:
        instance.username = instance.username.lower()

- import `pre_save` next to `post_save`

## Create path to access profiles by `@kakaort`
- in root `urls.py` add path as:
`path('@<str:username>/', profile_view, name="profile"),`
- import `profile_view`: `from a_users.views import profile_view`
- in `profile_view` of `a_users/profile_view` in views.py: 
- import `get_object_or_404` next to `redirect`; import `from django.contrib.auth.models import User`
- Add `username=None` as input arg into `profile_view()`
full revised code:
def profile_view(request, username=None):
    if username:
        profile = get_object_or_404(User, username=username).profile
    else:
        try:
            profile = request.user.profile
        except:
            redirect('account_login')
            
    return render(request, 'a_users/profile.html', {'profile':profile})

- if user trying to access 'profile' url while not logged in, gets redirected to login

## Setup the `settings` page:
- create a view `profile_settings_view` in a_users/views.py:
@login_required
def profile_settings_view(request):
    return render(request, 'a_users/profile_settings.html')

- create `profile_settings.html` in `a_users/templates`:
- add code: https://github.com/andyjud/django-starter-assets/blob/main/profile_settings.html
- create path in `a_users/urls.py`:
path('settings/', profile_settings_view, name='profile-settings'),
- Add urls to `header.html`

## Add the `edit` link in `profile_settings.html` using htmx:
- ensure `django-htmx` package is installed else `pip install django-htmx`
- add `django-htmx` to INSTALLED_APPS in settings.py, above custom apps
- add `django_htmx.middleware.HtmxMiddleware` to INSTALLED_APPS in settings.py, bottom of list
- add an edit email form in `forms.py` and add:
class EmailForm(ModelForm):
    email = forms.EmailField(required=True)

    class Meta:
        model= User
        fields = ['email']
- NB `from django.contrib.auth.models import User`
- create a view for the form
@login_required
def profile_emailchange(request):

    if request.htmx:
        form = EmailForm(instance=request.user)
        return render(request, 'partials/email_form.html', {'form':form})
    
    return redirect('home')
- if htmx request, pass the snippet to front end 

- Create `partials/email_form.html` in root template dir
add code: `https://github.com/andyjud/django-starter-assets/blob/main/email_form.html`

- Create path in app `urls.py`:
path('emailchange/', profile_emailchange, name='profile-emailchange'),

- Add htmx code in `profile_settings.html`:
- replace `href=""` in the Edit '<a>' element and add htmx attributes:
hx-get="{% url 'profile-emailchange' %}"
hx-target="#email-address"
hx-swap="innerHTML"
- include a `cursor-pointer` class

- Add code to saved data to database:
-import messages: `from django.contrib import messages`
add code and check if email already exists else `save()`.
    if request.method == "POST":
        form = EmailForm(request.POST, instance=request.user)

        if form.is_valid():

            # Check if the email already exists 
            email= form.cleaned_data['email']
            if User.objects.filter(email=email).exclude(id=request.user.id).exists():
                messages.warning(request, f"{email} is already in use.")
                return redirect('profile-settings')
            
            form.save()

- in `signals.py` with `user_postsave()` 
- import `from allauth.account.models import EmailAddress`
add below `if created:`:
    else:
        #update allauth emailaddresse if exist else create one
        try:
            email_address = EmailAddress.objects.get_primary(user)

            if email_address.email !=user.email:
                email_address.email= user.email
                email_address.verified= False
                email_address.save()
            
        except:
            EmailAddress.objects.create(
                user = user,
                email =user.email,
                primary = True,
                verified = False
            )

- back to `singals.py`:
- import `from allauth.account.utils import send_email_confirmation`
- add below `form.save` in `profile_emailchange()` of app `views.py`
    # Then send confirmation email
    send_email_confirmation(request, request.user)

    return redirect('profile-settings')

else:
    messages.warning(request, "Form not valid")
    return redirect('profile-settings')


## Enable the `verify` link in `profile_settings.html`
- in views.py add code:
@login_required
def profile_emailverify(request):
    send_email_confirmation(request, request.user)
    return redirect('profile-settings')

- add path is app `urls.py`:
`path('emailverify/', profile_emailverify, name='profile-emailverify'),`

- add url in `profile_settings.html`:
- add `{% url 'profile-emailverify' %}` in 'Verify' "<a>" href attribute

## Setup profile delete
- in the app `views.py`:
@login_required
def profile_delete_view(request):
    return render(request, 'a_users/profile_delete.html')

- create `a_users/profile_delete.html` file
add code: `https://github.com/andyjud/django-starter-assets/blob/main/profile_delete.html`

- add path to app `urls.py`:
`path('delete/', profile_delete_view, name='profile-delete'),`

- Add url to `profile_settings.html`:

- add functionality to `profile_delete_view` in `views.py`:
- import: `from django.contrib.auth import logout`
add code:
    user =request.user
    if request.method == "POST":
        logout(request)
        user.delete()
        messages.success(request, 'Account deleted, what a pity')
        return redirect('home')

## Add 404 page
- add `404.html` to root templates folder
- add code: `https://github.com/andyjud/django-starter-assets/blob/main/404.html`
- test by changing `DEBUG` to False in settings.py
- add `ALLOWED_HOSTS`: `'127.0.0.1',` for dev only






