# LinkShelf का लागि २४ Django प्रो टिप्स

LinkShelf ट्युटोरियलका पाँचै दिनका अभ्याससँग मिल्ने टिप्स। प्रत्येक टिपसँग सम्बन्धित Django 6.1 डकुमेन्टेसनको लिंक छ।

## दिन १ · सेटअप र URL

1. **Django को version पहिले जाँच्नुहोस्।** Virtual environment सक्रिय गरेपछि `python -m django --version` चलाउनुहोस्। आफ्नो version सँग मिल्ने डकुमेन्टेसन पढ्दा command र उदाहरण पछ्याउन सजिलो हुन्छ। [Django Tutorial 1][tutorial1]
2. **ठूलो परिवर्तनपछि `python manage.py check` चलाउनुहोस्।** नयाँ app, model वा route जोड्दा configuration मा देखिने समस्या browser खोल्नुअघि भेट्न सकिन्छ। [System check command][check]
3. **URL लाई नाम दिनुहोस्।** Template मा `/bookmarks/` लेख्नुको सट्टा `{% url 'bookmark_list' %}` प्रयोग गर्नुहोस्। पछि Explore को path बदले पनि template को link चल्छ। [URL dispatcher][urls]
4. **प्रोजेक्ट र app का routes छुट्याउनुहोस्।** `config/urls.py` बाट `bookmarks/urls.py` लाई `include()` गर्नुहोस्। Bookmark का सबै routes एउटै ठाउँमा खोज्न सकिन्छ। [Django Tutorial 1][tutorial1]

## दिन २ · Model, migration र database

5. **Model र migration एउटै Git commit मा राख्नुहोस्।** `owner` र `is_public` थप्दा `models.py` सँगै Django ले बनाएको migration file पनि commit गर्नुहोस्। [Migrations][migrations]
6. **`makemigrations` र `migrate` को काम छुट्याएर सम्झनुहोस्।** पहिलोले database परिवर्तनको निर्देशन बनाउँछ। दोस्रोले त्यो निर्देशन लागू गर्छ। कुन migration चलिसक्यो भनेर `python manage.py showmigrations bookmarks` बाट हेर्नुहोस्। [Migrations][migrations]
7. **पुराना record को व्यवहार पहिले तय गर्नुहोस्।** LinkShelf मा `is_public=False` ले अघिल्ला Bookmark निजी राख्छ। Migration चलाएपछि Admin बाट मालिक तोक्नुहोस् र साझा गर्नुपर्ने Bookmark मात्र सार्वजनिक गर्नुहोस्। [Migrations][migrations]
8. **Query पहिला Django shell मा चलाएर हेर्नुहोस्।** `python manage.py shell` खोलेर `Bookmark.objects.filter(tags__name="django").count()` चलाउनुहोस्। नतिजा गलत छ भने template छुनुअघि data र query जाँच्न सकिन्छ। [Making queries][queries]

## दिन ३ · सूची, template र ट्याग फिल्टर

9. **सबै पेजको साझा भाग `base.html` मा राख्नुहोस्।** Explore, My Bookmarks र logout बटन त्यहीँ राख्दा navigation बदल्न एउटा template सम्पादन गरे पुग्छ। [Template language][templates]
10. **खाली सूचीको सन्देश राख्नुहोस्।** `{% for %}` सँग `{% empty %}` प्रयोग गर्नुहोस्। नयाँ खाताको My Bookmarks र नतिजा नभएको ट्याग फिल्टर दुवैमा स्पष्ट सन्देश दिन सकिन्छ। [Template tags][tags]
11. **फिल्टरका लागि `GET` प्रयोग गर्नुहोस्।** `/bookmarks/?tag=django` कपी गरेर पठाउँदा अर्को व्यक्तिले पनि उही सार्वजनिक ट्यागको सूची खोल्न सक्छ। [HttpRequest.GET][get]
12. **सूचीमा ट्याग देखाउँदा `prefetch_related("tags")` प्रयोग गर्नुहोस्।** यसले हरेक Bookmark का ट्यागका लागि छुट्टाछुट्टै query चल्ने अवस्था घटाउँछ। [QuerySet API][queryset]
13. **सूचीको क्रम स्थिर राख्नुहोस्।** `order_by("-created_at", "-pk")` प्रयोग गर्दा दुई Bookmark को सिर्जना समय उस्तै भए पनि `pk` ले तिनको क्रम निर्धारण गर्छ। [QuerySet API][queryset]

## दिन ४ · Form, validation र सेभ

14. **`ModelForm.Meta.fields` मा सम्पादन गर्न मिल्ने field स्पष्ट लेख्नुहोस्।** `title`, `url`, `description`, `tags` र `is_public` राख्नुहोस्। `owner` भने view बाट `request.user` अनुसार तोक्नुहोस्। [ModelForm][forms]
15. **गलत POST आएपछि पठाइएको form नै फेरि देखाउनुहोस्।** `BookmarkForm(request.POST)` लाई errors सहित render गर्दा प्रयोगकर्ताले भरिसकेका field फेरि टाइप गर्नुपर्दैन। [ModelForm][forms]
16. **Edit गर्दा POST मा पनि `instance=bookmark` दिनुहोस्।** `BookmarkForm(request.POST, instance=bookmark)` ले त्यही record अपडेट गर्छ। `instance` छुटेमा form ले नयाँ record बनाउन खोज्छ। [ModelForm][forms]
17. **सफल POST पछि redirect गर्नुहोस्।** Create वा edit सेभ भएपछि विवरण पेजमा पठाउनुहोस्। त्यसपछि refresh गर्दा browser ले विवरण पेजको GET दोहोर्‍याउँछ। [Django Tutorial 4][tutorial4]
18. **आफ्नै साइटमा पठाउने हरेक POST form भित्र `{% csrf_token %}` राख्नुहोस्।** Bookmark थप्ने, सच्याउने, मेटाउने र logout गर्ने form सबैमा यो नियम लागू हुन्छ। [CSRF protection][csrf]
19. **`commit=False` पछि ट्याग पनि सेभ गर्नुहोस्।** पहिले `bookmark.owner` तोक्नुहोस्, त्यसपछि `bookmark.save()` र `form.save_m2m()` चलाउनुहोस्। दुवै save एउटै `transaction.atomic()` भित्र राख्दा ट्याग सेभ गर्ने क्रममा database त्रुटि आए Bookmark को save पनि rollback हुन्छ। [ModelForm][forms] · [Transactions][transactions]

## दिन ५ · खाता, गोपनीयता र परीक्षण

20. **Login र ownership छुट्टाछुट्टै जाँच्नुहोस्।** `@login_required` ले लगइन गरेको प्रयोगकर्तालाई view खोल्न दिन्छ। Edit र delete मा `get_object_or_404(Bookmark, pk=pk, owner=request.user)` ले त्यो Bookmark उसैको हो कि भनेर जाँच्छ। [Authentication][auth] · [Django shortcuts][shortcuts]
21. **पहिले visibility छान्नुहोस्, त्यसपछि ट्याग फिल्टर लगाउनुहोस्।** Explore मा `is_public=True` बाट सुरु गर्नुहोस्। Dropdown का ट्याग पनि त्यही सार्वजनिक QuerySet बाट निकाल्दा निजी Bookmark मा मात्र प्रयोग भएको ट्याग बाहिर देखिँदैन। [Making queries][queries]
22. **मेटाउने GET ले पुष्टि पेज मात्र खोलोस्।** CSRF सहितको POST आएपछि मात्र record मेटाउनुहोस्। अभ्यासमा पहिले पुष्टि पेज खोलेर रद्द गर्नुहोस्, अनि Bookmark अझै छ कि जाँच्नुहोस्। [HTTP decorators][decorators] · [CSRF protection][csrf]
23. **दुई खाताको परीक्षणमा सीधै URL खोल्नुहोस्।** दोस्रो खाताबाट पहिलोको सार्वजनिक Bookmark हेर्न मिल्नुपर्छ। निजी विवरण र सार्वजनिक Bookmark कै edit वा delete URL मा 404 आउनुपर्छ। पछि यही परीक्षण Django test client को `force_login()` ले दोहोर्‍याउन सकिन्छ। [Testing tools][testing]
24. **साइट सार्वजनिक गर्नुअघि production settings मा `python manage.py check --deploy` चलाउनुहोस्।** `DEBUG=False`, `ALLOWED_HOSTS`, `SECRET_KEY` र HTTPS सम्बन्धी settings Django को deployment checklist सँग मिलाएर हेर्नुहोस्। [Deployment checklist][deploy]

[tutorial1]: https://docs.djangoproject.com/en/6.1/intro/tutorial01/
[check]: https://docs.djangoproject.com/en/6.1/ref/django-admin/#check
[urls]: https://docs.djangoproject.com/en/6.1/topics/http/urls/
[migrations]: https://docs.djangoproject.com/en/6.1/topics/migrations/
[queries]: https://docs.djangoproject.com/en/6.1/topics/db/queries/
[templates]: https://docs.djangoproject.com/en/6.1/ref/templates/language/
[tags]: https://docs.djangoproject.com/en/6.1/ref/templates/builtins/#for
[get]: https://docs.djangoproject.com/en/6.1/ref/request-response/#django.http.HttpRequest.GET
[queryset]: https://docs.djangoproject.com/en/6.1/ref/models/querysets/
[forms]: https://docs.djangoproject.com/en/6.1/topics/forms/modelforms/
[tutorial4]: https://docs.djangoproject.com/en/6.1/intro/tutorial04/
[csrf]: https://docs.djangoproject.com/en/6.1/howto/csrf/
[transactions]: https://docs.djangoproject.com/en/6.1/topics/db/transactions/
[auth]: https://docs.djangoproject.com/en/6.1/topics/auth/default/
[shortcuts]: https://docs.djangoproject.com/en/6.1/topics/http/shortcuts/#get-object-or-404
[decorators]: https://docs.djangoproject.com/en/6.1/topics/http/decorators/
[testing]: https://docs.djangoproject.com/en/6.1/topics/testing/tools/
[deploy]: https://docs.djangoproject.com/en/6.1/howto/deployment/checklist/
