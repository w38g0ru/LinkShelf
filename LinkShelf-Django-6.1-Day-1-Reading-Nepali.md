# LinkShelf · दिन १ को पठन सामग्री

## Django 6.1 सेटअप, पहिलो View र URL

**समय** · ९० मिनेट  
**आजको नतिजा** · ब्राउजरमा `/` खोल्दा **Welcome to LinkShelf** देखिन्छ। आफैँ अर्को URL बनाएर request कुन function मा पुग्छ भनेर बुझ्न सकिन्छ।

यो नोटसँगै [Django 6.1 को आधिकारिक Tutorial Part 1](https://docs.djangoproject.com/en/6.1/intro/tutorial01/) खोल्नुहोस्। हरेक खण्डमा पहिले **पढ्नुहोस्** अन्तर्गतको लिंक खोल्नुहोस्, त्यसपछि **LinkShelf मा गर्नुहोस्** पछ्याउनुहोस्। आधिकारिक उदाहरणमा `mysite` र `polls` छन्। यहाँ ती नामको ठाउँमा क्रमशः `config` र `bookmarks` प्रयोग हुन्छन्।

## १ · Python र Django तयार गर्ने · १५ मिनेट

**पढ्नुहोस्** · [Django 6.1 को Python compatibility](https://docs.djangoproject.com/en/6.1/releases/6.1/#python-compatibility) · [Python को virtual environment बनाउने खण्ड](https://docs.python.org/3/tutorial/venv.html#creating-virtual-environments) · [Django लाई pip बाट स्थापना गर्ने](https://docs.djangoproject.com/en/6.1/topics/install/#installing-an-official-release-with-pip)

**पढ्दा ध्यान दिनुहोस्** · `pip` ले Python package स्थापना गर्छ। `.venv` ले यो project का packages छुट्टै राख्छ। यो योजना Python 3.13 को पछिल्लो patch र Django 6.1.x मा आधारित छ। अघिल्लो Day 1 नोटमा Django 5.2 प्रयोग भएको थियो। त्यो अभ्यास गरिसक्नुभएको छ भने तलको नयाँ सेटअप अलग फोल्डरमा गर्नुहोस्।

**LinkShelf मा गर्नुहोस्** · macOS Terminal मा `python3 --version` चलाउनुहोस्। `python3` ले Python 3.13 देखाएपछि, project राख्ने फोल्डरबाट यी command चलाउनुहोस्।

```bash
mkdir linkshelf
cd linkshelf
python3 -m venv .venv
source .venv/bin/activate
python -m pip install "Django>=6.1,<6.2"
python -m django --version
```

**जाँच** · `which python` को नतिजा `.venv/bin/python` मा टुङ्गिन्छ र Django को संस्करण `6.1.x` देखिन्छ।

## २ · Project बनाउने र server चलाउने · २० मिनेट

**पढ्नुहोस्** · [Creating a project](https://docs.djangoproject.com/en/6.1/intro/tutorial01/#creating-a-project) · [The development server](https://docs.djangoproject.com/en/6.1/intro/tutorial01/#the-development-server)

**पढ्दा ध्यान दिनुहोस्** · `manage.py` बाट project का command चलाइन्छन्। `config/settings.py` मा settings र `config/urls.py` मा मुख्य routes रहन्छन्।

**LinkShelf मा गर्नुहोस्** · `linkshelf` भित्र यी command चलाउनुहोस्। `startproject` को अन्तिम `.` ले अहिलेको फोल्डरमै `manage.py` राख्छ।

```bash
python -m django startproject config .
python manage.py runserver
```

**जाँच** · [स्थानीय साइट](http://127.0.0.1:8000/) खोल्दा Django को प्रारम्भिक पेज देखिन्छ। Migration बाँकी भएको सन्देश आए database को काम दिन २ मा हुनेछ। Terminal मा `Ctrl+C` ले server रोक्नुहोस्। `runserver` स्थानीय अभ्यासका लागि हो।

## ३ · App बनाउने र Django को संरचना बुझ्ने · १५ मिनेट

**पढ्नुहोस्** · [Creating the Polls app](https://docs.djangoproject.com/en/6.1/intro/tutorial01/#creating-the-polls-app) · [Django को Model र Template को परिचय](https://docs.djangoproject.com/en/6.1/intro/overview/) · [INSTALLED_APPS](https://docs.djangoproject.com/en/6.1/ref/settings/#installed-apps)

**LinkShelf मा गर्नुहोस्**

```bash
python manage.py startapp bookmarks
```

`config/settings.py` मा `INSTALLED_APPS` खोज्नुहोस्। सूचीका अरू entry सँगै `"bookmarks.apps.BookmarksConfig",` थप्नुहोस्। `config` ले पूरै साइटको settings सम्हाल्छ। `bookmarks` ले लिंकसम्बन्धी सुविधा सम्हाल्छ।

| Django | काम | CodeIgniter 4 सँग तुलना |
|---|---|---|
| Model | Data को संरचना र database सँग काम | Model र database तह |
| View | HTTP request लिएर response फर्काउने | Controller method |
| Template | प्रयोगकर्ताले देख्ने HTML को ढाँचा | `app/Views/` |

आज View बाट सीधै पाठ फर्काउँछौँ। दिन २ मा Model र दिन ३ मा Template प्रयोग गर्छौँ।

## ४ · पहिलो View लेख्ने र URL जोड्ने · ३० मिनेट

**पढ्नुहोस्** · [Write your first view](https://docs.djangoproject.com/en/6.1/intro/tutorial01/#write-your-first-view) · [Django ले request कसरी सम्हाल्छ](https://docs.djangoproject.com/en/6.1/topics/http/urls/#how-django-processes-a-request) · [HttpResponse](https://docs.djangoproject.com/en/6.1/ref/request-response/#httpresponse-objects)

**LinkShelf मा गर्नुहोस्** · `bookmarks/views.py` मा पहिलो function लेख्नुहोस्।

```python
from django.http import HttpResponse


def home(request):
    return HttpResponse("Welcome to LinkShelf")
```

`bookmarks/urls.py` नामको नयाँ फाइल बनाएर app को route राख्नुहोस्।

```python
from django.urls import path

from . import views

urlpatterns = [
    path("", views.home, name="home"),
]
```

`config/urls.py` मा app को URL सूची जोड्नुहोस्।

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("admin/", admin.site.urls),
    path("", include("bookmarks.urls")),
]
```

ब्राउजरले `GET /` पठाउँदा `config/urls.py` को `include()` ले बाँकी path `bookmarks/urls.py` मा पठाउँछ। बाँकी path खाली भएकाले `path("", views.home)` मिल्छ। Django ले `home(request)` बोलाउँछ र त्यसले फर्काएको `HttpResponse` ब्राउजरमा देखिन्छ। `name="home"` ले route लाई पछि template बाट उल्लेख गर्न मिल्ने नाम दिन्छ।

```bash
python manage.py check
python manage.py runserver
```

**जाँच** · [मुख्य पेज](http://127.0.0.1:8000/) मा **Welcome to LinkShelf** देखिन्छ।

## ५ · आफैँ गर्ने अभ्यास · १० मिनेट

1. `bookmarks/views.py` मा `about(request)` बनाउनुहोस्। यसले `Save, organize, and share useful links.` फर्काओस्।
2. `bookmarks/urls.py` मा दोस्रो route थप्नुहोस्। संकेत · `path("about/", views.about, name="about")`।
3. [मुख्य पेज](http://127.0.0.1:8000/), [About पेज](http://127.0.0.1:8000/about/) र [नबनाएको पेज](http://127.0.0.1:8000/does-not-exist/) खोल्नुहोस्। पहिलो दुईमा फरक सन्देश र अन्तिममा 404 आउनुपर्छ।

**बुझाइ जाँच** · `startproject` र `startapp` को फरक के हो? `/about/` खोल्दा कुन दुई URL फाइल हुँदै कुन function चल्छ? उत्तर सम्झन गाह्रो भए [request सम्हाल्ने क्रम](https://docs.djangoproject.com/en/6.1/topics/http/urls/#how-django-processes-a-request) फेरि हेर्नुहोस्।

| समस्या | छिटो जाँच |
|---|---|
| `No module named django` | `source .venv/bin/activate` पछि `python -m django --version` चलाउनुहोस्। |
| `manage.py` भेटिएन | `pwd` र `ls` चलाएर `manage.py` भएको फोल्डरमा जानुहोस्। |
| `No module named 'bookmarks.urls'` | `bookmarks/urls.py` बनाएको र नाम ठीक भएको हेर्नुहोस्। |
| Port 8000 प्रयोगमा छ | `python manage.py runserver 8001` चलाउनुहोस् र ब्राउजरमा port 8001 प्रयोग गर्नुहोस्। |

## दिन २ का लागि तयार

अर्को दिन यही `linkshelf` फोल्डर र `.venv` प्रयोग गर्नुहोस्। `bookmarks/models.py` मा `Bookmark` र `Tag` बनाउने काम [आधिकारिक Tutorial Part 2 को Database setup](https://docs.djangoproject.com/en/6.1/intro/tutorial02/#database-setup) बाट सुरु हुन्छ।
