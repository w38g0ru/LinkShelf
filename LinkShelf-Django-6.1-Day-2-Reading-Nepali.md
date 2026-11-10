# LinkShelf · दिन २ को पठन सामग्री

## Django 6.1 मा SQLite, Models, Migrations र Admin

**समय** · ९० मिनेट  
**आजको नतिजा** · Django Admin बाट पाँचवटा उपयोगी लिंक र तीनवटा ट्याग सेभ गर्नुहुन्छ। त्यसपछि Django shell बाट एउटा ट्यागमा जोडिएका लिंक निकाल्नुहुन्छ।

[आधिकारिक Tutorial Part 2](https://docs.djangoproject.com/en/6.1/intro/tutorial02/) सँगै यो पाठ पढ्नुहोस्। आधिकारिक पाठमा `Question` र `Choice` बीचको सम्बन्ध देखाइएको छ। LinkShelf मा `Bookmark` र `Tag` जोड्न छुट्टै [many-to-many उदाहरण](https://docs.djangoproject.com/en/6.1/topics/db/examples/many_to_many/) हेर्नुहोस्। प्रत्येक खण्डमा पहिले **पढ्नुहोस्**, त्यसपछि **LinkShelf मा गर्नुहोस्** पछ्याउनुहोस्।

## १ · दिन १ को प्रोजेक्ट खोल्ने र SQLite बुझ्ने · १० मिनेट

**पढ्नुहोस्** · [Database setup](https://docs.djangoproject.com/en/6.1/intro/tutorial02/#database-setup) · [DATABASES setting](https://docs.djangoproject.com/en/6.1/ref/settings/#databases)

**पढ्दा ध्यान दिनुहोस्** · नयाँ Django project मा SQLite पहिले नै सेट गरिएको हुन्छ। LinkShelf को database `manage.py` भएको फोल्डरमा `db.sqlite3` फाइलका रूपमा बस्छ।

**LinkShelf मा गर्नुहोस्** · Terminal मा दिन १ को `linkshelf` फोल्डर खोल्नुहोस्। त्यही फोल्डरबाट चलाउनुहोस्।

```bash
source .venv/bin/activate
python -m django --version
python manage.py check
```

**जाँच** · Django `6.1.x` देखिन्छ र `check` मा error आउँदैन। `config/settings.py` मा `DATABASES` भित्र SQLite र `INSTALLED_APPS` भित्र `"bookmarks.apps.BookmarksConfig"` देखिन्छ।

## २ · Bookmark र Tag model लेख्ने · २५ मिनेट

**पढ्नुहोस्** · [Creating models](https://docs.djangoproject.com/en/6.1/intro/tutorial02/#creating-models) · [Model field reference](https://docs.djangoproject.com/en/6.1/ref/models/fields/) · [Many-to-many relationships](https://docs.djangoproject.com/en/6.1/topics/db/examples/many_to_many/)

**पढ्दा ध्यान दिनुहोस्** · एउटा लिंकमा `django` र `python` दुवै ट्याग हुन सक्छन्। त्यही `django` ट्याग अर्को लिंकमा पनि जोड्न सकिन्छ। PHP/MySQL मा `bookmark_id` र `tag_id` भएको बीचको तालिका बनाउने काम Django मा `ManyToManyField` ले सम्हाल्छ।

**LinkShelf मा गर्नुहोस्** · `bookmarks/models.py` मा तलको कोड राख्नुहोस्।

```python
from django.db import models


class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)

    def __str__(self):
        return self.name


class Bookmark(models.Model):
    title = models.CharField(max_length=200)
    url = models.URLField(max_length=500)
    description = models.TextField(blank=True)
    tags = models.ManyToManyField(
        Tag,
        related_name="bookmarks",
        blank=True,
    )
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

| कोड | LinkShelf मा काम |
|---|---|
| `unique=True` | ठ्याक्कै उही नामको अर्को Tag राख्न रोक्छ। अभ्यासमा ट्यागका नाम साना अक्षरमा लेख्नुहोस्। |
| `URLField` | Admin form मा URL को ढाँचा जाँच्न मद्दत गर्छ। |
| `blank=True` | विवरण र ट्याग form मा खाली छोड्न दिन्छ। |
| `related_name="bookmarks"` | कुनै ट्यागबाट त्यसमा जोडिएका लिंक `tag.bookmarks.all()` ले निकाल्न दिन्छ। |
| `auto_now_add=True` | Bookmark पहिलोपटक सेभ हुँदा समय आफैँ राख्छ। |
| `__str__()` | Admin मा record लाई नाम वा शीर्षकबाट चिन्न सजिलो बनाउँछ। |

**जाँच** · `bookmark.tags.all()` ले एउटा Bookmark का ट्याग र `tag.bookmarks.all()` ले एउटा Tag का Bookmark दिन्छन् भन्ने फरक छुट्याउन सक्नुहुन्छ। यी query पाँचौँ खण्डमा चलाउनुहुन्छ।

## ३ · Migration बनाउने र लागू गर्ने · १५ मिनेट

**पढ्नुहोस्** · [Activating models](https://docs.djangoproject.com/en/6.1/intro/tutorial02/#activating-models) · [Migrations workflow](https://docs.djangoproject.com/en/6.1/topics/migrations/#workflow) · [showmigrations](https://docs.djangoproject.com/en/6.1/ref/django-admin/#showmigrations)

**LinkShelf मा गर्नुहोस्**

```bash
python manage.py makemigrations bookmarks
python manage.py migrate
python manage.py showmigrations bookmarks
```

`makemigrations` ले model को परिवर्तन migration फाइलमा लेख्छ। `migrate` ले त्यो परिवर्तन database मा लागू गर्छ। पहिलोपटक चलाउँदा Admin, authentication र sessions का बाँकी migrations पनि लागू हुन्छन्।

**जाँच** · नयाँ दिन १ प्रोजेक्टमा `bookmarks/migrations/0001_initial.py` बन्छ र `showmigrations bookmarks` मा `[X] 0001_initial` देखिन्छ।

## ४ · Django Admin मा models दर्ता गर्ने · २० मिनेट

**पढ्नुहोस्** · [Introducing Django Admin](https://docs.djangoproject.com/en/6.1/intro/tutorial02/#introducing-the-django-admin) · [ModelAdmin options](https://docs.djangoproject.com/en/6.1/ref/contrib/admin/#modeladmin-options)

**LinkShelf मा गर्नुहोस्** · `bookmarks/admin.py` मा दुवै model दर्ता गर्नुहोस्। `list_display` ले Bookmark सूचीका column र `list_filter` ले ट्याग फिल्टर थप्छ।

```python
from django.contrib import admin

from .models import Bookmark, Tag


@admin.register(Tag)
class TagAdmin(admin.ModelAdmin):
    search_fields = ("name",)


@admin.register(Bookmark)
class BookmarkAdmin(admin.ModelAdmin):
    list_display = ("title", "url", "created_at")
    list_filter = ("tags",)
    search_fields = ("title", "url", "description")
```

```bash
python manage.py createsuperuser
python manage.py runserver
```

`createsuperuser` ले username, email र password माग्छ। Password टाइप गर्दा Terminal मा अक्षर देखिँदैन। [स्थानीय Admin](http://127.0.0.1:8000/admin/) खोलेर त्यही खाताबाट लगइन गर्नुहोस्।

**जाँच** · Admin मा **Bookmarks** र **Tags** देखिन्छन्। दिन १ को मुख्य पेज पनि खुल्छ।

## ५ · नमुना लिंक राख्ने र ORM बाट जाँच्ने · २० मिनेट

**पढ्नुहोस्** · [Playing with the API](https://docs.djangoproject.com/en/6.1/intro/tutorial02/#playing-with-the-api) · [Querying related objects](https://docs.djangoproject.com/en/6.1/topics/db/queries/#related-objects)

**LinkShelf मा गर्नुहोस्** · Admin मा पहिले `django`, `python` र `database` ट्याग थप्नुहोस्। त्यसपछि यी पाँच लिंक राख्नुहोस्। कुनै दुई लिंकमा आफूले बुझ्ने गरी छोटो विवरण लेख्नुहोस्।

| शीर्षक | URL | ट्याग |
|---|---|---|
| Django Tutorial | https://docs.djangoproject.com/en/6.1/intro/tutorial01/ | `django`, `python` |
| Python Tutorial | https://docs.python.org/3/tutorial/ | `python` |
| Django Model Fields | https://docs.djangoproject.com/en/6.1/ref/models/fields/ | `django`, `database` |
| Django Migrations | https://docs.djangoproject.com/en/6.1/topics/migrations/ | `django`, `database` |
| SQLite Documentation | https://www.sqlite.org/docs.html | `database` |

**आफैँ अभ्यास गर्नुहोस्** · Admin मा `django` ट्यागबाट सूची फिल्टर गर्नुहोस्। Python Tutorial को विवरण सच्याउनुहोस्। त्यसपछि छुट्टै एउटा अस्थायी Bookmark थपेर मेटाउनुहोस्। यसरी दिन ३ का लागि पाँचवटै नमुना लिंक उपलब्ध हुन्छन्।

अर्को Terminal मा उही प्रोजेक्ट फोल्डर खोलेर चलाउनुहोस्।

```bash
source .venv/bin/activate
python manage.py shell
```

```python
from bookmarks.models import Bookmark, Tag

Bookmark.objects.count()

tag = Tag.objects.get(name="django")
list(tag.bookmarks.order_by("title").values_list("title", flat=True))

bookmark = Bookmark.objects.get(title="Django Tutorial")
list(bookmark.tags.order_by("name").values_list("name", flat=True))
```

**जाँच** · खाली database बाट माथिको नमुना मात्र राख्नुभएको छ भने क्रमशः `5`, `['Django Migrations', 'Django Model Fields', 'Django Tutorial']` र `['django', 'python']` आउँछन्। `exit()` ले shell बन्द गर्छ।

| समस्या | के जाँच्ने |
|---|---|
| Admin मा Bookmark वा Tag देखिएन | `bookmarks/admin.py` मा दर्ता, `INSTALLED_APPS` मा app र सही खाताबाट लगइन भएको जाँच्नुहोस्। |
| `no such table` | `python manage.py showmigrations bookmarks` चलाउनुहोस्। migration बनेको छ तर `[ ]` छ भने `python manage.py migrate` चलाउनुहोस्। |
| `Tag.DoesNotExist` | Admin मा ट्यागको नाम ठ्याक्कै `django` राखिएको छ कि हेर्नुहोस्। |

## दिन ३ का लागि तयार

आजका पाँच लिंकलाई अर्को दिन Admin बाहिर, LinkShelf को आफ्नै पेजमा देखाइनेछ। त्यसका लागि [आधिकारिक Tutorial Part 3 को Views र Templates](https://docs.djangoproject.com/en/6.1/intro/tutorial03/) पढ्नुहोस्।
