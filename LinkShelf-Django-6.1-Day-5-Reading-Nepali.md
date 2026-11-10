# LinkShelf · दिन ५ को पठन सामग्री

## Django 6.1 मा खाता, निजी बुकमार्क र सार्वजनिक Explore

**समय** · १२० मिनेट  
**आजको नतिजा** · प्रयोगकर्ताले खाता खोलेर आफ्ना Bookmark राख्न सक्छन्। सार्वजनिक Bookmark सबैले Explore मा देख्छन्। निजी Bookmark त्यसको मालिकले मात्र हेर्न सक्छ। LinkShelf बाट सम्पादन र मेटाउने अधिकार पनि मालिककै हुन्छ।

दिन ४ का `BookmarkForm`, सूची, विवरण र form template यसै पाठमा विस्तार गर्नुहोस्। दिन ४ मा छुटेको मेटाउने अभ्यास पनि यहाँ समेटिएको छ।

| पेज | कसले के देख्छ? |
|---|---|
| Explore · `/bookmarks/` | सबैले सार्वजनिक Bookmark र तिनका ट्याग देख्छन्। |
| My Bookmarks · `/bookmarks/mine/` | login भएको प्रयोगकर्ताले आफ्ना सार्वजनिक र निजी Bookmark देख्छ। |
| विवरण · `/bookmarks/<pk>/` | सार्वजनिक भए सबैले, निजी भए मालिकले हेर्न सक्छ। |
| सच्याउने र मेटाउने | login भएको मालिकले मात्र गर्न सक्छ। |

## १ · Bookmark मा मालिक र visibility राख्ने · २० मिनेट

**पढ्नुहोस्** · [Referencing the User model](https://docs.djangoproject.com/en/6.1/topics/auth/customizing/#referencing-the-user-model) · [ForeignKey](https://docs.djangoproject.com/en/6.1/ref/models/fields/#foreignkey) · [Migrations](https://docs.djangoproject.com/en/6.1/topics/migrations/)

दिन ४ कै प्रोजेक्ट र virtual environment खोल्नुहोस्। `bookmarks/models.py` को imports मा `from django.conf import settings` थप्नुहोस्। विद्यमान `Bookmark` class भित्र यी दुई field थप्नुहोस्।

```python
    owner = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name="bookmarks",
        null=True,
        blank=True,
    )
    is_public = models.BooleanField(default=False)
```

```bash
source .venv/bin/activate
python manage.py makemigrations bookmarks
python manage.py migrate
```

अघिल्ला Bookmark सुरक्षित राख्न `owner` खाली हुन मिल्ने बनाइएको छ। ती पुराना Bookmark सुरुमा निजी हुन्छन् र Explore मा देखिँदैनन्। दिन २ को superuser बाट Admin मा गएर चाहेको पुरानो Bookmark को `owner` तोक्नुहोस्। सार्वजनिक गर्नुपर्ने लिंकमा मात्र `is_public` छान्नुहोस्। नयाँ LinkShelf form ले login भएको प्रयोगकर्तालाई मालिक बनाउँछ। `CASCADE` का कारण खाता मेट्दा त्यस खाताका Bookmark पनि मेटिन्छन्।

## २ · खाता खोल्ने, login र logout · २५ मिनेट

**पढ्नुहोस्** · [UserCreationForm](https://docs.djangoproject.com/en/6.1/topics/auth/default/#django.contrib.auth.forms.UserCreationForm) · [LoginView](https://docs.djangoproject.com/en/6.1/topics/auth/default/#django.contrib.auth.views.LoginView) · [LogoutView](https://docs.djangoproject.com/en/6.1/topics/auth/default/#django.contrib.auth.views.LogoutView)

`config/settings.py` मा यी settings थप्नुहोस्।

```python
LOGIN_URL = "login"
LOGIN_REDIRECT_URL = "bookmark_mine"
LOGOUT_REDIRECT_URL = "bookmark_list"
```

`bookmarks/views.py` को imports मा `from django.contrib.auth.forms import UserCreationForm` थप्नुहोस्। अनि यो view थप्नुहोस्।

```python
def register(request):
    if request.method == "POST":
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect("login")
    else:
        form = UserCreationForm()

    return render(request, "bookmarks/account_form.html", {
        "form": form,
        "heading": "खाता खोल्नुहोस्",
    })
```

`bookmarks/urls.py` मा `from django.contrib.auth import views as auth_views` थप्नुहोस्। विद्यमान `urlpatterns` मा यी तीन route थप्नुहोस्।

```python
    path("accounts/register/", views.register, name="register"),
    path(
        "accounts/login/",
        auth_views.LoginView.as_view(
            template_name="bookmarks/account_form.html",
            extra_context={"heading": "लगइन गर्नुहोस्"},
        ),
        name="login",
    ),
    path("accounts/logout/", auth_views.LogoutView.as_view(), name="logout"),
```

`bookmarks/templates/bookmarks/account_form.html` बनाउनुहोस्।

```html
{% extends "bookmarks/base.html" %}

{% block title %}{{ heading }} · LinkShelf{% endblock %}

{% block content %}
    <h1>{{ heading }}</h1>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        {% if next %}
            <input type="hidden" name="next" value="{{ next }}">
        {% endif %}
        <button type="submit">{{ heading }}</button>
    </form>
{% endblock %}
```

`next` ले login पछि सुरुमा खोल्न खोजेको पेजमा फर्कन मद्दत गर्छ। पेज आफैँ सुरक्षित छ कि छैन भन्ने जाँच view मा हुन्छ।

## ३ · Explore, My Bookmarks र निजी विवरण · २५ मिनेट

**पढ्नुहोस्** · [login_required](https://docs.djangoproject.com/en/6.1/topics/auth/default/#the-login-required-decorator) · [QuerySet filter](https://docs.djangoproject.com/en/6.1/ref/models/querysets/#filter) · [get_object_or_404](https://docs.djangoproject.com/en/6.1/topics/http/shortcuts/#get-object-or-404)

`bookmarks/views.py` का imports मा `from django.contrib.auth.decorators import login_required` र `from django.db.models import Q` थप्नुहोस्। `bookmark_list()` र `bookmark_detail()` बदल्नुहोस्। `bookmark_mine()` र सूची बनाउने साझा function थप्नुहोस्।

```python
def render_bookmark_list(request, bookmarks, heading, route_name):
    selected_tag = request.GET.get("tag", "").strip()
    tags = Tag.objects.filter(
        pk__in=bookmarks.values("tags")
    ).order_by("name")

    if selected_tag:
        bookmarks = bookmarks.filter(tags__name=selected_tag)

    return render(request, "bookmarks/bookmark_list.html", {
        "bookmarks": bookmarks.order_by("-created_at", "-pk").prefetch_related("tags"),
        "tags": tags,
        "selected_tag": selected_tag,
        "heading": heading,
        "list_route": route_name,
    })


def bookmark_list(request):
    bookmarks = Bookmark.objects.filter(is_public=True)
    return render_bookmark_list(request, bookmarks, "Explore", "bookmark_list")


@login_required
def bookmark_mine(request):
    bookmarks = Bookmark.objects.filter(owner=request.user)
    return render_bookmark_list(request, bookmarks, "My Bookmarks", "bookmark_mine")


def bookmark_detail(request, pk):
    visible = Q(is_public=True)
    if request.user.is_authenticated:
        visible |= Q(owner=request.user)

    bookmark = get_object_or_404(Bookmark.objects.filter(visible), pk=pk)
    return render(request, "bookmarks/bookmark_detail.html", {"bookmark": bookmark})
```

पहिले कुन Bookmark देखाउन मिल्छ भन्ने QuerySet तयार हुन्छ। त्यसपछि मात्र ट्याग फिल्टर वा ID खोजी हुन्छ। त्यसैले निजी Bookmark मा मात्र प्रयोग भएको ट्याग Explore को dropdown मा आउँदैन। निजी Bookmark को ID थाहा पाएर URL खोल्दा पनि अरूलाई 404 आउँछ।

`bookmarks/urls.py` मा यो route थप्नुहोस्।

```python
    path("bookmarks/mine/", views.bookmark_mine, name="bookmark_mine"),
```

`bookmark_list.html` को title block मा `{{ heading }} · LinkShelf` र `<h1>` मा `{{ heading }}` राख्नुहोस्। फिल्टर form को `action` र «फिल्टर हटाउनुहोस्» लिंक दुवैमा `{% url 'bookmark_list' %}` लाई `{% url list_route %}` ले बदल्नुहोस्। «नयाँ बुकमार्क थप्नुहोस्» लिंक वरिपरिको स्टाफ जाँचलाई `{% if user.is_authenticated %}` बनाउनुहोस्।

`base.html` को `<header>` लाई यसले बदल्नुहोस्।

```html
<header>
    <nav aria-label="मुख्य नेभिगेसन">
        <a href="{% url 'bookmark_list' %}">Explore</a>
        {% if user.is_authenticated %}
            <a href="{% url 'bookmark_mine' %}">My Bookmarks</a>
            <form method="post" action="{% url 'logout' %}">
                {% csrf_token %}
                <button type="submit">लगआउट</button>
            </form>
        {% else %}
            <a href="{% url 'login' %}">लगइन</a>
            <a href="{% url 'register' %}">खाता खोल्नुहोस्</a>
        {% endif %}
    </nav>
</header>
```

## ४ · मालिकले थप्ने, सच्याउने र मेटाउने · ३५ मिनेट

**पढ्नुहोस्** · [ModelForm save() र save_m2m()](https://docs.djangoproject.com/en/6.1/topics/forms/modelforms/#the-save-method) · [Database transactions](https://docs.djangoproject.com/en/6.1/topics/db/transactions/#controlling-transactions-explicitly) · [HTTP method decorators](https://docs.djangoproject.com/en/6.1/topics/http/decorators/#allowed-http-methods) · [CSRF](https://docs.djangoproject.com/en/6.1/howto/csrf/)

`bookmarks/forms.py` मा `fields` लाई `("title", "url", "description", "tags", "is_public")` बनाउनुहोस्। `labels` मा `"is_public": "Explore मा सार्वजनिक देखाउनुहोस्",` थप्नुहोस्। मालिक form बाट छानिँदैन। View ले login भएको खाताबाट मालिक तोक्छ।

`bookmarks/views.py` बाट `staff_member_required` को import हटाउनुहोस्। `from django.db import transaction` र `from django.views.decorators.http import require_http_methods` थप्नुहोस्। दिन ४ का create र edit functions लाई decorators सहित यसले बदल्नुहोस् र delete view थप्नुहोस्।

```python
@login_required
def bookmark_create(request):
    if request.method == "POST":
        form = BookmarkForm(request.POST)
        if form.is_valid():
            with transaction.atomic():
                bookmark = form.save(commit=False)
                bookmark.owner = request.user
                bookmark.save()
                form.save_m2m()

            return redirect("bookmark_detail", pk=bookmark.pk)
    else:
        form = BookmarkForm()

    return render(request, "bookmarks/bookmark_form.html", {
        "form": form,
        "heading": "नयाँ बुकमार्क थप्नुहोस्",
    })


@login_required
def bookmark_edit(request, pk):
    bookmark = get_object_or_404(Bookmark, pk=pk, owner=request.user)

    if request.method == "POST":
        form = BookmarkForm(request.POST, instance=bookmark)
        if form.is_valid():
            with transaction.atomic():
                form.save()

            return redirect("bookmark_detail", pk=bookmark.pk)
    else:
        form = BookmarkForm(instance=bookmark)

    return render(request, "bookmarks/bookmark_form.html", {
        "form": form,
        "heading": "बुकमार्क सच्याउनुहोस्",
    })


@login_required
@require_http_methods(["GET", "POST"])
def bookmark_delete(request, pk):
    bookmark = get_object_or_404(Bookmark, pk=pk, owner=request.user)

    if request.method == "POST":
        bookmark.delete()
        return redirect("bookmark_mine")

    return render(request, "bookmarks/bookmark_confirm_delete.html", {
        "bookmark": bookmark,
    })
```

`commit=False` ले मालिक तोक्ने मौका दिन्छ। Bookmark को ID बनेपछि `save_m2m()` ले ट्याग सम्बन्ध सेभ गर्छ। `transaction.atomic()` भित्र भएकाले बीचमा database त्रुटि आए Bookmark र ट्यागको परिवर्तन सँगै फिर्ता हुन्छ। Edit र delete दुवैमा `owner=request.user` ले GET र POST मा मालिक जाँच्छ।

`bookmarks/urls.py` मा यो route थप्नुहोस्।

```python
    path("bookmarks/<int:pk>/delete/", views.bookmark_delete, name="bookmark_delete"),
```

`bookmark_detail.html` मा दिन ४ को स्टाफ जाँचसहितको «सच्याउनुहोस्» खण्डलाई यसले बदल्नुहोस्।

```html
{% if user.is_authenticated and bookmark.owner_id == user.pk %}
    <p>
        <a href="{% url 'bookmark_edit' bookmark.pk %}">सच्याउनुहोस्</a>
        <a href="{% url 'bookmark_delete' bookmark.pk %}">मेटाउनुहोस्</a>
    </p>
{% endif %}
```

`bookmarks/templates/bookmarks/bookmark_confirm_delete.html` बनाउनुहोस्।

```html
{% extends "bookmarks/base.html" %}

{% block title %}बुकमार्क मेटाउनुहोस् · LinkShelf{% endblock %}

{% block content %}
    <h1>बुकमार्क मेटाउनुहोस्</h1>
    <p>«{{ bookmark.title }}» मेटाउने हो?</p>
    <form method="post">
        {% csrf_token %}
        <button type="submit">हो, मेटाउनुहोस्</button>
    </form>
    <p><a href="{% url 'bookmark_detail' bookmark.pk %}">रद्द गर्नुहोस्</a></p>
{% endblock %}
```

GET ले पुष्टि पेज खोल्छ। पुष्टि बटनको POST आएपछि मात्र Bookmark मेटिन्छ। दिन ४ को साझा `bookmark_form.html` मा «सूचीमा फर्कनुहोस्» लिंकलाई `{% url 'bookmark_mine' %}` मा पठाउनुहोस्।

## ५ · दुई खाताबाट अन्तिम अभ्यास · १५ मिनेट

```bash
python manage.py check
python manage.py runserver
```

[खाता खोल्ने पेज](http://127.0.0.1:8000/accounts/register/) बाट दुई सामान्य खाता बनाउनुहोस्। पहिलो खातालाई सामान्य browser window मा र अर्कोलाई private window वा अर्को browser profile मा login गर्नुहोस्। पहिलो खाताबाट दुई Bookmark बनाउनुहोस्। एउटामा सार्वजनिक विकल्प छान्नुहोस् र अर्को निजी राख्नुहोस्। दुवैको विवरण URL टिप्नुहोस्।

| गर्ने काम | अपेक्षित नतिजा |
|---|---|
| पहिलो खाताको My Bookmarks खोल्ने | दुवै Bookmark देखिन्छन्। |
| दोस्रो खाताको Explore खोल्ने | पहिलो खाताको सार्वजनिक Bookmark देखिन्छ। |
| दोस्रो खाताबाट निजी Bookmark को विवरण URL खोल्ने | 404 आउँछ। |
| दोस्रो खाताबाट सार्वजनिक Bookmark को edit वा delete URL सिधै खोल्ने | 404 आउँछ। |
| पहिलो खाताले निजी Bookmark सार्वजनिक बनाएर दोस्रो खाताबाट त्यसको ट्याग छान्ने | Bookmark Explore मा देखिन्छ। निजी बनाउँदा फेरि हट्छ। |
| पहिलो खाताले delete URL खोल्ने र रद्द गर्ने | Bookmark विवरण खुल्छ र Bookmark रहन्छ। |
| पहिलो खाताले पुष्टि गरेर मेटाउने | My Bookmarks खुल्छ। मेटिएको Bookmark को पुरानो विवरण URL मा 404 आउँछ। |
| logout गरेर `/bookmarks/mine/` खोल्ने | Login पेज खुल्छ। login पछि My Bookmarks मा फर्किन्छ। |

**बुझाइ जाँच** · सार्वजनिक Bookmark को विवरण सबैलाई खुल्छ। त्यही Bookmark को edit URL अर्को खातालाई किन खुल्दैन? यसको उत्तर view को एउटा लाइन देखाएर दिनुहोस्।

| समस्या | के जाँच्ने |
|---|---|
| `no such column: bookmarks_bookmark.owner_id` | नयाँ migration बनाएर त्यही database मा चलाएको जाँच्नुहोस्। |
| My Bookmarks मा पुरानो लिंक छैन | Admin मा त्यसको मालिक कुन खाता छ हेर्नुहोस्। |
| नयाँ Bookmark का ट्याग सेभ भएनन् | Create view मा `bookmark.save()` पछि `form.save_m2m()` छ कि हेर्नुहोस्। |
| Logout मा 405 वा 403 | Logout को HTML form मा `method="post"` र भित्र `{% csrf_token %}` दुवै जाँच्नुहोस्। |
| My Bookmarks को फिल्टरले Explore खोल्छ | form को action र फिल्टर हटाउने लिंकमा `{% url list_route %}` प्रयोग भएको जाँच्नुहोस्। |
