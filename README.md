# Install django social login

    pip install django django-ninja django-allauth
    
    django-admin startproject django_socail_login


## `django_social_login/settings.py`

    INSTALLED_APPS = [
        ...
        'django.contrib.sites',
        'allauth',
        'allauth.account',
        'allauth.socialaccount',
        'allauth.socialaccount.providers.openid_connect'
    ]

    MIDDLEWARE = [
        ...
        'allauth.account.middleware.AccountMiddleware',
    ]

    TEMPLATES = [
        {
            ...
            "DIRS": [str(BASE_DIR.joinpath("templates"))],
            ...
        },
    ]


    AUTHENTICATION_BACKENDS = (
        "allauth.account.auth_backends.AuthenticationBackend",
    )

    SITE_ID = 1
    ACCOUNT_EMAIL_VERIFICATION = 'none'
    LOGIN_REDIRECT_URL = 'home'
    ACCOUNT_LOGOUT_ON_GET = True


    SOCIALACCOUNT_PROVIDERS = {
        "openid_connect": {
            "APPS": [
                {
                    "provider_id": "openid_connect",
                    "name": "openid_connect",
                    "client_id": "web-app",
                    "secret": "<COPY IT FROM THE KEYCLOAK CLIENT>",
                    "settings": {
                        "server_url": "http://localhost:8080/auth/realms/django_social_login_keycloak/.well-known/openid-configuration",
                    },
                }
            ]
        }
    }

## `templates/base.html`

    <html lang="en">
      <head>
        <meta charset="UTF-8" />
        <link
          href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/css/bootstrap.min.css"
          rel="stylesheet"
        />
        <link
          rel="stylesheet"
          href="https://stackpath.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css"
        />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Django Social Login</title>
      </head>
      <body>
        {% block content %} {% endblock content %}
      </body>
    </html>


## `templates/home.html`

    {% extends '_base.html' %} {% load socialaccount %}

    {% block content %}

    <div class="container" style="text-align: center; padding-top: 10%;">
      <h1>Django Social Login</h1>

      <br /><br />

      {% if user.is_authenticated %}
        <h3>Welcome {{ user.username }} !!!</h3>
        <br /><br />
        <a href="{% url 'account_logout' %}" class="btn btn-danger">Logout</a>
      {% else %}
        <!-- GitHub button starts here -->
        <a href="{% provider_login_url 'openid_connect' %}" class="btn btn-secondary">
          <i class="fa fa-github fa-fw"></i>
          <span>Login with GitHub</span>
        </a>
        <!-- GitHub button ends here -->
      {% endif %}
    </div>

    {% endblock content %}



## `django_social_login/views.py`

    from django.views.generic import TemplateView


    class Home(TemplateView):
        template_name = "home.html"


## `django_social_login/urls.py`

    from django.contrib import admin
    from django.urls import path, include
    from .views import Home

    urlpatterns = [
        path('admin/', admin.site.urls),

        path('accounts/', include('allauth.urls')),
        path("", Home.as_view(), name="home")
    ]


## migrate

    ./manage.py makemigrations
    ./manage.py migrate

## create `superuser`

    ./manage.py createsuperuser

## setup KeyCloak

### start the docker-compose

    docker-compose -f docker/docker-compose.yml up -d

### create a realm

1. navigate to [localhost:8080](http://localhost:8080) and login with `admin` `admin`  
2. click on the realm dropdown and create a new realm (realmname: `django_social_login_keycloak`)
3. save

### crete a client

1. navigate to `Clients → Create client`
2. set Client ID to `web-app` and click on next
3. allow `Client authentication` and `Authorization`, `Standard flow` and `Direct access grants` have to be checked and click on save
4. set the Root URL an Home URL to `http://localhost:8000`
5. set the Valid redirect URIs to `*` (only for testing in production use the valid one)
6. set the Valid post logout redirect URIs to `+` and click on save
7. navigate to `Credentials` and copy the `Client secret` and paste it in your `settings.py` file under `SOCIALACCOUNT_PROVIDERS`

### create a user

1. navigate to `Users → Add user`
2. set the username to `user` and click on save
3. navigate to credentials and set the password to `user`

## start the application

    ./manage.py runserver

## test the login

1. navigate to [localhost:8080](http://localhost:8080)
2. select 'Login with Keycloak'
3. login with `user` `user`
