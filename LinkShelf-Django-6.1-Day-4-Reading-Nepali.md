# LinkShelf · दिन ४ को पठन सामग्री

## Django 6.1 मा ModelForm, validation र Bookmark थप्ने तथा सच्याउने

**समय** · ९० मिनेट  
**आजको नतिजा** · स्टाफले LinkShelf कै पेजबाट Bookmark थप्न, शीर्षक र URL सच्याउन, र ट्याग बदल्न सक्छन्। गलत विवरण पठाउँदा सोही form मा त्रुटि देखिन्छ।

[आधिकारिक Tutorial Part 4](https://docs.djangoproject.com/en/6.1/intro/tutorial04/) सँगै पढ्नुहोस्। Tutorial मा मत पठाउन प्रयोग भएको POST, CSRF token र redirect यहाँ Bookmark सेभ गर्न प्रयोग हुन्छन्।

## १ · Bookmark बाट form बनाउने · १५ मिनेट

**पढ्नुहोस्** · [Creating forms from models](https://docs.djangoproject.com/en/6.1/topics/forms/modelforms/) · [Selecting the fields to use](https://docs.djangoproject.com/en/6.1/topics/forms/modelforms/#selecting-the-fields-to-use)

**LinkShelf मा गर्नुहोस्** · दिन ३ को प्रोजेक्टको root फोल्डरमा जानुहोस्। त्यही SQLite database र virtual environment प्रयोग गर्नुहोस्।

```bash
source .venv/bin/activate
python -m django --version
python manage.py check
```

`bookmarks/forms.py` बनाउनुहोस्।

```python
from django import forms

from .models import Bookmark


class BookmarkForm(forms.ModelForm):
    class Meta:
        model = Bookmark
        fields = ("title", "url", "description", "tags")
        labels = {
            "title": "शीर्षक",
            "url": "लिंक (URL)",
            "description": "विवरण",
            "tags": "ट्यागहरू",
        }
        widgets = {"description": forms.Textarea(attrs={"rows": 4})}
```

`ModelForm` ले model का field बाट input र validation नियम तयार गर्छ। `fields` ले प्रयोगकर्ताले सम्पादन गर्न पाउने field तोक्छ। `tags` मा database मा भएका Tag हरू छान्न मिल्छन्। यो form मा परिवर्तन गर्दा database को संरचना उस्तै भएकाले migration चाहिँदैन।

**बुझाइ जाँच** · CodeIgniter 4 मा validation पछि model बाट insert वा update गरिन्छ। यहाँ form ले validation गर्छ र सफल भएपछि `save()` ले model सेभ गर्छ। Form मा कुन field देखाउने भन्ने निर्णय कुन लाइनले गर्छ?

## २ · नयाँ Bookmark सेभ गर्ने · २५ मिनेट

**पढ्नुहोस्** · [Working with forms](https://docs.djangoproject.com/en/6.1/topics/forms/#the-view) · [CSRF protection](https://docs.djangoproject.com/en/6.1/howto/csrf/) · [staff_member_required](https://docs.djangoproject.com/en/6.1/ref/contrib/admin/#the-staff-member-required-decorator)

**LinkShelf मा गर्नुहोस्** · `bookmarks/views.py` का imports मिलाउनुहोस्। दिन ३ का view functions राख्नुहोस्।

```python
from django.contrib.admin.views.decorators import staff_member_required
from django.shortcuts import get_object_or_404, redirect, render

from .forms import BookmarkForm
from .models import Bookmark, Tag
```

सोही फाइलमा यो function थप्नुहोस्।

```python
@staff_member_required
def bookmark_create(request):
    if request.method == "POST":
        form = BookmarkForm(request.POST)
        if form.is_valid():
            bookmark = form.save()
            return redirect("bookmark_detail", pk=bookmark.pk)
    else:
        form = BookmarkForm()

    return render(
        request,
        "bookmarks/bookmark_form.html",
        {"form": form, "heading": "नयाँ बुकमार्क थप्नुहोस्"},
    )
```

| Request | View ले गर्ने काम |
|---|---|
| GET | खाली form देखाउँछ। |
| POST, सही विवरण | Bookmark र छानिएका ट्याग सेभ गर्छ र विवरण पेजमा पठाउँछ। |
| POST, गलत विवरण | पठाएको विवरण र validation errors सहित त्यही form देखाउँछ। |

सफल POST पछि browser ले विवरण पेज GET बाट खोल्छ। त्यसैले त्यही पेज refresh गर्दा form फेरि submit हुँदैन। यसलाई **Post/Redirect/Get** भनिन्छ।

`bookmarks/urls.py` को `urlpatterns` मा यो route थप्नुहोस्।

```python
    path("bookmarks/new/", views.bookmark_create, name="bookmark_create"),
```

`bookmarks/templates/bookmarks/bookmark_form.html` बनाउनुहोस्। यही template सम्पादनका लागि पनि प्रयोग हुन्छ।

```html
{% extends "bookmarks/base.html" %}

{% block title %}{{ heading }} · LinkShelf{% endblock %}

{% block content %}
    <h1>{{ heading }}</h1>

    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">सेभ गर्नुहोस्</button>
    </form>

    <p><a href="{% url 'bookmark_list' %}">सूचीमा फर्कनुहोस्</a></p>
{% endblock %}
```

`{{ form.as_p }}` ले field, label र validation error देखाउँछ। `{% csrf_token %}` ले POST सँग token पठाउँछ। Django ले token जाँच्छ र जाँच असफल हुँदा सामान्यतः 403 फर्काउँछ। Form हाल खोलिएको URL मै submit हुन्छ।

`bookmark_list.html` मा `<h1>` पछि र ट्याग filter form अघि यो लिंक राख्नुहोस्।

```html
{% if user.is_active and user.is_staff %}
    <p><a href="{% url 'bookmark_create' %}">नयाँ बुकमार्क थप्नुहोस्</a></p>
{% endif %}
```

**जाँच** · `python manage.py runserver` चलाउनुहोस्। दिन २ को superuser बाट [Admin](http://127.0.0.1:8000/admin/) मा login गर्नुहोस् र [नयाँ Bookmark form](http://127.0.0.1:8000/bookmarks/new/) खोल्नुहोस्। शीर्षकमा `Django Forms`, URL मा `https://docs.djangoproject.com/en/6.1/topics/forms/` र ट्यागमा `django` राखेर सेभ गर्नुहोस्।

## ३ · भएको Bookmark सच्याउने · ३० मिनेट

**पढ्नुहोस्** · [ModelForm save()](https://docs.djangoproject.com/en/6.1/topics/forms/modelforms/#the-save-method) · [get_object_or_404()](https://docs.djangoproject.com/en/6.1/topics/http/shortcuts/#get-object-or-404)

**LinkShelf मा गर्नुहोस्** · `bookmarks/views.py` मा यो function थप्नुहोस्।

```python
@staff_member_required
def bookmark_edit(request, pk):
    bookmark = get_object_or_404(Bookmark, pk=pk)

    if request.method == "POST":
        form = BookmarkForm(request.POST, instance=bookmark)
        if form.is_valid():
            bookmark = form.save()
            return redirect("bookmark_detail", pk=bookmark.pk)
    else:
        form = BookmarkForm(instance=bookmark)

    return render(
        request,
        "bookmarks/bookmark_form.html",
        {"form": form, "heading": "बुकमार्क सच्याउनुहोस्"},
    )
```

GET मा `instance=bookmark` ले पहिले सेभ भएका मान र ट्याग form मा देखाउँछ। POST मा त्यही `instance` दिँदा `save()` ले उही Bookmark अपडेट गर्छ। यो argument छुटेमा Django ले नयाँ Bookmark बनाउन खोज्छ।

`bookmarks/urls.py` मा अर्को route थप्नुहोस्।

```python
    path("bookmarks/<int:pk>/edit/", views.bookmark_edit, name="bookmark_edit"),
```

`bookmark_detail.html` मा शीर्षकपछि यो लिंक थप्नुहोस्।

```html
{% if user.is_active and user.is_staff %}
    <p><a href="{% url 'bookmark_edit' bookmark.pk %}">सच्याउनुहोस्</a></p>
{% endif %}
```

Template को `if` ले लिंक देखाउने कि नदेखाउने भन्ने तय गर्छ। URL सिधै खोल्दा वा POST पठाउँदा `@staff_member_required` ले पहुँच जाँच्छ। आजको अभ्यासमा कुनै पनि सक्रिय स्टाफले यी दुई view प्रयोग गर्न सक्छ।

**बुझाइ जाँच** · `bookmark_edit()` मा GET र POST दुवैमा `instance=bookmark` राख्नुको कारण एक–एक वाक्यमा लेख्नुहोस्।

## ४ · परीक्षण र अभ्यास · २० मिनेट

| गर्ने काम | अपेक्षित नतिजा |
|---|---|
| नयाँ Bookmark सेभ गर्ने | विवरण पेज खुल्छ। दिन २ का पाँचवटा मात्र थिए भने सूचीमा अब छवटा हुन्छन्। |
| विवरण पेज refresh गर्ने | Bookmark को संख्या उस्तै रहन्छ। |
| सम्पादन form खोल्ने | हालको शीर्षक, URL, विवरण र ट्याग भरिएका देखिन्छन्। |
| शीर्षकलाई `Django Form Guide` बनाएर सेभ गर्ने | विवरण पेजमा नयाँ शीर्षक देखिन्छ। URL को ID र सूचीको संख्या उस्तै रहन्छन्। |
| ट्याग बदलेर दिन ३ को filter चलाउने | Bookmark नयाँ ट्यागको सूचीमा देखिन्छ। |
| शीर्षक खाली राखेर submit गर्ने | Browser ले रोक्न सक्छ। POST सर्भरसम्म पुगेमा Django ले पनि validation error देखाउँछ। |
| स्टाफ login गरेर `/bookmarks/999999/edit/` खोल्ने | त्यो ID नभए 404 आउँछ। |
| Private window मा `/bookmarks/new/` वा भएको ID को edit URL खोल्ने | Admin login मा पठाउँछ। सामान्य सूची भने खुल्छ। |

**Server-side validation आफैँ जाँच्नुहोस्** · स्थानीय browser को DevTools बाट form मा अस्थायी रूपमा `novalidate` attribute राख्नुहोस्। URL मा `https://` मात्र लेखेर submit गर्नुहोस्। Django ले URL field को त्रुटि देखाउँछ र विवरण पेजमा पठाउँदैन। सही URL राखेर फेरि सेभ गर्नुहोस्।

| समस्या | के जाँच्ने |
|---|---|
| `403 CSRF verification failed` | `{% csrf_token %}` POST form भित्र छ कि हेर्नुहोस्। पेज पुनः खोलेर submit गर्नुहोस्। |
| ट्याग छान्ने ठाउँ खाली छ | दिन २ को database प्रयोग भएको छ र Admin मा Tag छन् कि हेर्नुहोस्। |
| सम्पादन गर्दा अर्को Bookmark बन्यो | POST को `BookmarkForm(...)` मा `instance=bookmark` दिइएको छ कि हेर्नुहोस्। |
| नयाँ वा सच्याउने लिंक देखिएन | Admin मा login भएको र खातामा `is_active` तथा `is_staff` दुवै सक्रिय भएको जाँच्नुहोस्। |

**अन्तिम अभ्यास** · एउटा Bookmark थप्नुहोस्। त्यसमा दुई ट्याग छान्नुहोस्। पछि एउटा मात्र राखेर सेभ गर्नुहोस् र बाँकी ट्यागअनुसार फिल्टर गर्दा त्यो Bookmark देखिन्छ कि जाँच्नुहोस्।
