# LinkShelf · दिन ३ को पठन सामग्री

## Django 6.1 मा Views, Templates र ट्याग फिल्टर

**समय** · ९० मिनेट  
**आजको नतिजा** · `/bookmarks/` मा दिन २ का पाँच लिंक देखिन्छन्। प्रत्येकको विवरण खोल्न, ट्यागबाट सूची छान्न र नभएको ID मा 404 जाँच्न सक्नुहुन्छ।

[Django 6.1 को आधिकारिक Tutorial Part 3](https://docs.djangoproject.com/en/6.1/intro/tutorial03/) सँगै पढ्नुहोस्। त्यहाँको प्रश्न सूची र प्रश्न विवरणले यहाँको Bookmark सूची र Bookmark विवरण बुझ्न मद्दत गर्छ। हरेक खण्डमा पहिले **पढ्नुहोस्**, त्यसपछि **LinkShelf मा गर्नुहोस्** पछ्याउनुहोस्।

## १ · ORM बाट data ल्याएर template मा पठाउने · १५ मिनेट

**पढ्नुहोस्** · [Retrieving objects](https://docs.djangoproject.com/en/6.1/topics/db/queries/#retrieving-objects) · [Use the template system](https://docs.djangoproject.com/en/6.1/intro/tutorial03/#use-the-template-system) · [render()](https://docs.djangoproject.com/en/6.1/topics/http/shortcuts/#render)

**LinkShelf मा गर्नुहोस्** · दिन २ को `linkshelf` फोल्डर खोल्नुहोस्। त्यही SQLite database प्रयोग गर्नुहोस्।

```bash
source .venv/bin/activate
python -m django --version
python manage.py check
```

`bookmarks/views.py` मा भएका `home` र `about` functions राख्नुहोस्। माथिका imports मा `from django.shortcuts import render` र `from .models import Bookmark` थप्नुहोस्। त्यसपछि यो function थप्नुहोस्।

```python
def bookmark_list(request):
    bookmarks = Bookmark.objects.order_by("-created_at", "-pk").prefetch_related("tags")
    return render(
        request,
        "bookmarks/bookmark_list.html",
        {"bookmarks": bookmarks},
    )
```

`order_by("-created_at", "-pk")` ले नयाँ Bookmark पहिले राख्छ। `prefetch_related("tags")` ले सूचीका ट्यागहरू छुट्टै query मा सँगै ल्याउँछ। `render()` को तेस्रो argument **context** हो। त्यसमा राखेको `bookmarks` नाम HTML template बाट प्रयोग गर्न मिल्छ।

**बुझाइ जाँच** · CodeIgniter मा view लाई data पठाउँदा प्रयोग हुने array सँग यहाँको context dictionary तुलना गर्नुहोस्। HTML मा देखाउने नाम `bookmarks` कहाँ तोकियो?

## २ · Base template र सूची बनाउने · ३० मिनेट

**पढ्नुहोस्** · [Template inheritance](https://docs.djangoproject.com/en/6.1/ref/templates/language/#template-inheritance) · [for … empty](https://docs.djangoproject.com/en/6.1/ref/templates/builtins/#for-empty) · [URL tag](https://docs.djangoproject.com/en/6.1/ref/templates/builtins/#url)

**LinkShelf मा गर्नुहोस्** · प्रोजेक्टको root बाट template फोल्डर बनाउनुहोस्।

```bash
mkdir -p bookmarks/templates/bookmarks
```

`bookmarks/templates/bookmarks/base.html` मा साझा HTML राख्नुहोस्।

```html
<!doctype html>
<html lang="ne">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block title %}LinkShelf{% endblock %}</title>
</head>
<body>
    <header>
        <a href="{% url 'bookmark_list' %}">LinkShelf</a>
    </header>
    <main>
        {% block content %}{% endblock %}
    </main>
</body>
</html>
```

`bookmarks/templates/bookmarks/bookmark_list.html` मा सूची राख्नुहोस्।

```html
{% extends "bookmarks/base.html" %}

{% block title %}बुकमार्कहरू · LinkShelf{% endblock %}

{% block content %}
    <h1>बुकमार्कहरू</h1>

    <ul>
        {% for bookmark in bookmarks %}
            <li>
                <strong>{{ bookmark.title }}</strong>
                <div>
                    {% for tag in bookmark.tags.all %}
                        <span>{{ tag.name }}</span>
                    {% empty %}
                        <span>ट्याग छैन</span>
                    {% endfor %}
                </div>
            </li>
        {% empty %}
            <li>कुनै बुकमार्क भेटिएन।</li>
        {% endfor %}
    </ul>
{% endblock %}
```

`bookmarks/urls.py` को `urlpatterns` मा भएका routes सँगै यो लाइन थप्नुहोस्। दिन १ को `config/urls.py` ले यो फाइल पहिल्यै include गर्छ।

```python
    path("bookmarks/", views.bookmark_list, name="bookmark_list"),
```

```bash
python manage.py runserver
```

**जाँच** · [Bookmark सूची](http://127.0.0.1:8000/bookmarks/) खोल्नुहोस्। दिन २ का पाँच शीर्षक र तिनका ट्याग देखिन्छन्। पहिले सेभ गरेकोभन्दा पछि सेभ गरेको लिंक माथि देखिन्छ।

## ३ · विवरण पेज र 404 बनाउने · २५ मिनेट

**पढ्नुहोस्** · [Writing more views](https://docs.djangoproject.com/en/6.1/intro/tutorial03/#writing-more-views) · [Raising a 404 error](https://docs.djangoproject.com/en/6.1/intro/tutorial03/#raising-a-404-error) · [Removing hardcoded URLs](https://docs.djangoproject.com/en/6.1/intro/tutorial03/#removing-hardcoded-urls-in-templates)

**LinkShelf मा गर्नुहोस्** · `bookmarks/views.py` को shortcuts import लाई `from django.shortcuts import get_object_or_404, render` बनाउनुहोस् र यो function थप्नुहोस्।

```python
def bookmark_detail(request, pk):
    bookmark = get_object_or_404(Bookmark, pk=pk)
    return render(
        request,
        "bookmarks/bookmark_detail.html",
        {"bookmark": bookmark},
    )
```

`bookmarks/urls.py` को `urlpatterns` मा विवरण route थप्नुहोस्।

```python
    path("bookmarks/<int:pk>/", views.bookmark_detail, name="bookmark_detail"),
```

`<int:pk>` ले URL बाट अंक लिएर view मा `pk` का रूपमा पठाउँछ। `get_object_or_404()` ले त्यो primary key भएको record खोज्छ। Record नभेटिए Django ले 404 response दिन्छ।

`bookmarks/templates/bookmarks/bookmark_detail.html` बनाउनुहोस्।

```html
{% extends "bookmarks/base.html" %}

{% block title %}{{ bookmark.title }} · LinkShelf{% endblock %}

{% block content %}
    <h1>{{ bookmark.title }}</h1>
    <p><a href="{{ bookmark.url }}">मूल लिंक खोल्नुहोस्</a></p>
    <p>{{ bookmark.description|default:"विवरण राखिएको छैन।" }}</p>

    <h2>ट्यागहरू</h2>
    <ul>
        {% for tag in bookmark.tags.all %}
            <li>{{ tag.name }}</li>
        {% empty %}
            <li>ट्याग छैन।</li>
        {% endfor %}
    </ul>

    <p><a href="{% url 'bookmark_list' %}">सूचीमा फर्कनुहोस्</a></p>
{% endblock %}
```

सूची template मा भएको `<strong>{{ bookmark.title }}</strong>` लाई यसले बदल्नुहोस्।

```html
<a href="{% url 'bookmark_detail' bookmark.pk %}">{{ bookmark.title }}</a>
```

**जाँच** · सूचीबाट कुनै लिंकको शीर्षक थिच्दा त्यसको विवरण खुल्छ। Database मा नभएको ID राखेर हेर्नुहोस्, जस्तै [बुकमार्क ९९९९९९](http://127.0.0.1:8000/bookmarks/999999/)। 404 आउनुपर्छ।

## ४ · ट्यागअनुसार सूची छान्ने · २० मिनेट

**पढ्नुहोस्** · [HttpRequest.GET](https://docs.djangoproject.com/en/6.1/ref/request-response/#django.http.HttpRequest.GET) · [Lookups that span relationships](https://docs.djangoproject.com/en/6.1/topics/db/queries/#lookups-that-span-relationships)

**LinkShelf मा गर्नुहोस्** · `bookmarks/views.py` को model import लाई `from .models import Bookmark, Tag` बनाउनुहोस्। अहिलेको `bookmark_list()` लाई यसले बदल्नुहोस्।

```python
def bookmark_list(request):
    selected_tag = request.GET.get("tag", "").strip()
    bookmarks = Bookmark.objects.order_by("-created_at", "-pk").prefetch_related("tags")

    if selected_tag:
        bookmarks = bookmarks.filter(tags__name=selected_tag)

    return render(
        request,
        "bookmarks/bookmark_list.html",
        {
            "bookmarks": bookmarks,
            "tags": Tag.objects.order_by("name"),
            "selected_tag": selected_tag,
        },
    )
```

`?tag=django` बाट आएको `django` लाई `request.GET` ले पढ्छ। `tags__name` मा दुई underscore ले Bookmark को `tags` सम्बन्ध हुँदै Tag को `name` सम्म पुग्ने बाटो जनाउँछ।

`bookmark_list.html` मा `<h1>` पछि र `<ul>` अघि यो form राख्नुहोस्।

```html
<form method="get" action="{% url 'bookmark_list' %}">
    <label for="tag">ट्याग छान्नुहोस्</label>
    <select name="tag" id="tag">
        <option value="">सबै ट्याग</option>
        {% for tag in tags %}
            <option value="{{ tag.name }}"{% if tag.name == selected_tag %} selected{% endif %}>
                {{ tag.name }}
            </option>
        {% endfor %}
    </select>
    <button type="submit">देखाउनुहोस्</button>
    {% if selected_tag %}
        <a href="{% url 'bookmark_list' %}">फिल्टर हटाउनुहोस्</a>
    {% endif %}
</form>
```

यो form ले सूची छान्ने मात्र भएकाले `GET` प्रयोग गरिएको छ। छानिएको ट्याग URL मा देखिन्छ र त्यो URL फेरि खोल्दा उही फिल्टर लाग्छ। दिन ४ मा database मा लेख्ने form का लागि `POST` प्रयोग गर्नुहुन्छ।

## अन्तिम जाँच

दिन २ को नमुना data मात्र छ भने यी नतिजा देखिनुपर्छ।

| खोल्ने URL वा गर्ने काम | अपेक्षित नतिजा |
|---|---|
| [सबै Bookmark](http://127.0.0.1:8000/bookmarks/) | पाँच लिंक देखिन्छन्। |
| [django ट्याग](http://127.0.0.1:8000/bookmarks/?tag=django) | Django Tutorial, Django Model Fields र Django Migrations देखिन्छन्। |
| [नभएको ट्याग](http://127.0.0.1:8000/bookmarks/?tag=does-not-exist) | «कुनै बुकमार्क भेटिएन» देखिन्छ। |
| «फिल्टर हटाउनुहोस्» थिच्ने | फेरि पाँचै लिंक देखिन्छन्। |
| सूचीबाट शीर्षक थिच्ने | विवरण, मूल URL र ट्याग देखिन्छन्। |
| [नभएको ID](http://127.0.0.1:8000/bookmarks/999999/) | 404 response आउँछ। |

**आफैँ अभ्यास गर्नुहोस्** · Admin बाट एउटा Bookmark को ट्याग परिवर्तन गर्नुहोस्। सूचीमा फर्केर पुरानो र नयाँ ट्यागको फिल्टरमा त्यो Bookmark कहाँ देखिन्छ जाँच्नुहोस्।

| समस्या | के जाँच्ने |
|---|---|
| `TemplateDoesNotExist` | फाइल `bookmarks/templates/bookmarks/` भित्र छ र view ले दिएको नाम मिल्छ कि हेर्नुहोस्। |
| `NoReverseMatch` | `{% url %}` को नाम `urls.py` मा तोकिएको नामसँग मिल्छ कि जाँच्नुहोस्। विवरण route लाई `bookmark.pk` दिनुपर्छ। |
| सूची खाली छ | फिल्टर हटाउनुहोस् र दिन २ कै database मा पाँच Bookmark छन् कि Admin बाट हेर्नुहोस्। |

## दिन ४ का लागि तयार

अर्को दिन स्टाफले LinkShelf बाटै Bookmark थप्न र सच्याउन मिल्ने form बनाउनुहुन्छ। त्यसका लागि [आधिकारिक Tutorial Part 4](https://docs.djangoproject.com/en/6.1/intro/tutorial04/) र [ModelForm reference](https://docs.djangoproject.com/en/6.1/topics/forms/modelforms/) प्रयोग हुन्छन्।
