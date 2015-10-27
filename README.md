Affiliate system for django
===========================

Affiliate system to reward partners with their visitors payments.
Contains abstract models with basic functionality, so it is easy
to subclass them and add your custom logic and relations.

**Note**: version 0.2.1 is backwards incompatible with previous versions.


Example
-------

Visitor goes to site with affiliate code:

    http://site.com/?aid=12345

This code is saved in his cookies (also another way is supported: keep aid=12345 at every url).
Now each request object has property `request.affiliate`, so you can access to affiliate, that attract current visitor, and therefore reward him at needed event.

You can find example project in this repository.

Used by projects
----------------
- [builds.io](http://builds.io/)
- [wirelayer.net](http://www.wirelayer.net/)


Requirements
-----------

- python (2.7, 3.4)
- django (1.6, 1.7, 1.8)


Quick start
-----------

1. Install this package to your python distribution

        pip install django-affiliate

2. Add 'affiliate' to INSTALLED_APP:

        INSTALLED_APPS = [
            # ...
            'affiliate',
        ]

3. Add 'affiliate.middleware.AffiliateMiddleware' to MIDDLEWARE_CLASSES:

        MIDDLEWARE_CLASSES = (
            # ...
            'affiliate.middleware.AffiliateMiddleware',
        )

4. [Optional] Define your custom affiliate model (similar to custom user model):

        # our_app/models.py
        from django.db import models
        from affiliate.models import AbstractAffiliate


        class Affiliate(AbstractAffiliate):
            some_field = models.CharField()

        # settngs.py
        AFFILIATE_AFFILIATE_MODEL = 'our_app.Affiliate'

5. Create tables

        python manage.py syncdb

6. Finally, reward affiliate

        from django.views.generic import FormView
        from affiliate.tools import get_affiliate_model

        Affiliate = get_affiliate_model()

        class SomeView(FormView):
            # ...

            def form_valid(self.form):
                product = self.get_product()
                if self.request.affiliate.exist() and self.request.affiliate.is_active:
                    # reward affiliate here, your custom logic is here
                    Transaction.objects.create(
                        user=self.affiliate.user,
                        amount=Affiliate.calc_affiliate_reward(product.price))
                    return super(SomeView, self).form_valid(form)

#### Optional

To always keep the aid GET parameter (maybe you don't trust the cookies or you want to reward affiliate only if his visitor make payment at current link access, and not tomorrow)

1. Load 'affiliate_urls' tags:

        {% load affiliate_urls %}

2. Use 'url_aff' instead of 'url' template tag:

        <a href="{% url_aff 'home' %}">Home</a>

Configuration
-------------

Define in settings.py

- AFFILIATE_AFFILIATE_MODEL - the model to use to represent an Affiliate, similar to [AUTH_USER_MODEL](https://docs.djangoproject.com/en/dev/ref/settings/#std:setting-AUTH_USER_MODEL). Default `'affiliate.Affiliate'`.
- AFFILIATE_PARAM_NAME - name of affiliate GET parameter in url. Default `'aid'`.
- AFFILIATE_REWARD_AMOUNT - default affiliate reward amount. Can be set as string (`'5.55'`) or as int (`10`). Default `10`.
- AFFILIATE_REWARD_PERCENTAGE - if True, that `AFFILIATE_REWARD_AMOUNT` is treated as percentage. Otherwise as exact amount of money. Default `True`.
- AFFILIATE_SAVE_IN_SESSION - save affiliate id in session or not. Default `True`.
- AFFILIATE_SESSION_AGE - how long keep affiliate id in session, in seconds. Default `5 * 24 * 60 * 60` seconds (5 days).
- AFFILIATE_DEFAULT_LINK - default link, that will be used by Affiliate.build_absolute_affiliate_uri. Default `'/'`.