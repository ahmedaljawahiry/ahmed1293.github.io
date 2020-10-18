---
title: "CRA Served from Django"
---

![](/static/images/django-react.png)

## Objectives:
- React running inside [Django](https://www.djangoproject.com/), with [create-react-app](https://create-react-app.dev/) (CRA).
    - Frontend and backend served from the same domain.
    - Run frontend on port 3000 with hot-reloading and all CRA features working nicely.
    - Run full app on port 8000 after running `yarn build`.
- [Django routing](https://docs.djangoproject.com/en/3.1/topics/http/urls/) and [React Router](https://reactrouter.com/web/) 
playing nicely together.
    - Redirects work.
    - Subroutes work.
- Login page created using Django.
- APIs using [SessionAuthentication](https://www.django-rest-framework.org/api-guide/authentication/#sessionauthentication).
- Project is [Docker](https://www.docker.com/) -ised.

## Result:
![](/static/images/django-react-demo.gif)

## Why:
- Get up-and-running quickly.
- Don't worry about user authentication - let Django handle that.
- Don't worry about handling API authentication from your frontend.
- Write your custom, dynamic, pages with React while using Django's' built-in 
views (see [CCBV](https://ccbv.co.uk/)) for more common pages.
- Single deployment.
- If your frontend needs to make API calls to various other systems then you'll want a backend, even if it's just to 
proxy the API calls.

## Why not:
- Fully decoupling your frontend and backend is important to you.
- Django + React routing isn't ideal.
- You have dedicated frontend and backend teams.

## How:

Full repo can be found [here](https://github.com/ahmedaljawahiry/django_react_routing). Note that TypeScript is used.

1. Start a Django project.
2. Create a "frontend" Django app: `django-admin startapp frontend`.
3. Create a React app (called "react", or whatever) inside your frontend/ directory: `yarn create react-app react`.
4. Add the below to `settings.py` so that Django knows where CRA's templates and static files are:
    <pre><code>
    STATICFILES_DIRS = [
        "frontend/react/build/static/",
    ]
    
    TEMPLATES = [
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',  # unchanged
            'DIRS': ['frontend/react/build'],  # add this
            'APP_DIRS': True,  # unchanged
            'OPTIONS': {...},  # unchanged
        },
    ]
    </code></pre>
5. While you're in `settings.py`, add:
    <pre><code>
    LOGIN_URL = '/user/login'
    LOGIN_REDIRECT_URL = '/ui/'
    LOGOUT_REDIRECT_URL = '/user/login'
    </code></pre>
6. In `frontend/views.py`, create some views:
    <pre><code>
    @login_required()
    def index(request, *args):
        return render(request, 'index.html')
    
    
    class Login(LoginView):
        template_name = 'frontend/login.html'
        redirect_authenticated_user = True
    
    
    class Logout(LogoutView):
        pass
    </code></pre>
7. `index.html` is created after you run `yarn build` inside your React directory.
8. `frontend/login.html` is a standard Django template you'll have to create for yourself. Something like 
<a href="https://github.com/ahmedaljawahiry/django_react_routing/blob/master/frontend/templates/frontend/login.html" target="_blank" rel="noopener">this<a/>
will do.
9. In `frontend/urls.py`, add the following:
    <pre><code>
    urlpatterns = [
        path('user/login/', Login.as_view(), name='login'),
        path('user/logout/', Logout.as_view(), name='logout'),
        re_path('^ui.*', index, name='spa'),  # react router handles anything after /ui/
    ]
    </code></pre>
10. Create another Django app, e.g. "backend_app", and setup DRF with some basic APIs:
    <pre><code>
    ### backend_app/api.py:
    
    class APIWithAuth(APIView):
    
        authentication_classes = [SessionAuthentication]
        permission_classes = [IsAuthenticated]
    
        def get(self, *args, **kwargs):
            return Response({
                'auth': True,
                'description': 'this api uses session auth'
            })
    
    
    class APIWithoutAuth(APIView):
    
        def get(self, *args, **kwargs):
            return Response({
                'auth': False,
                'description': 'this api is unauthenticated'
            })
    </code></pre>
    
    <pre><code>
    ### backend_app/urls.py:
    
    urlpatterns = [
        path('no-auth/', APIWithoutAuth.as_view(), name='no-auth-api'),
        path('auth/', APIWithAuth.as_view(), name='session-auth-api'),
    ]
    </code></pre>
11. [Proxy](https://create-react-app.dev/docs/proxying-api-requests-in-development/) the CRA API requests by
adding `"proxy": "http://localhost:8000"` to `package.json`.
12. Create some routed React components ([examples](https://github.com/ahmedaljawahiry/django_react_routing/tree/master/frontend/react/src)).
13. Dockerise everything if you wish. [These](https://github.com/ahmedaljawahiry/django_react_routing/tree/master/docker)
Dockerfiles, and [this](https://github.com/ahmedaljawahiry/django_react_routing/blob/master/docker-compose.yml) docker-compose
file should work nicely. You'll need to update the proxy above to `"http://django:8000"`.

## Final thoughts:
- When writing frontend code, go to port 3000 and you'll get hot-reloading; no need to worry about `collect_static`.
- If you're combining server-rendered pages with React ones, you'll probably want both templates to
extend some `base.html`. I'm yet to give this much thought, but worst-case scenario: you'll
have to add the `extend` tags after running `yarn build`.
- Something about the SPA path, `re_path('^ui.*', index, name='spa')`, doesn't fill me with 100% confidence.
I'm sure something smarter could be done here.
- I've worked on projects with completely decoupled front/backends, and Django projects with
[webpack](https://webpack.js.org/) setup to get React working. Both have different challenges. This setup <i>seems</i> 
to hit a sweet spot.
