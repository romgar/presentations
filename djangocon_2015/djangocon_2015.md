# The Best of DjangoCon 2015

---

# Overall feeling

- Focused on well being (and burnout/depression witness)
- Accessible community (lot of first-time speakers)
- Need to go outside (meet people, see real talks)

---

# Architecture / Separating concerns

*(@HannaKollo, @codeinthehole)*

- Avoid big views.py files.
- Use service layers to move business logic from views.

This:

    !python
    def my_view(request):
        params = request.GET
        # Lot of code here before returning HttpResponse
        # ...
        return HttpResponse("This is a too big view")

could be:

    !python
    from my_project.services import deal_with_data

    def my_view(request):
        params = request.GET
        deal_with_data(params)
        return HttpResponse("We have separated business from view logic")

---

# Testing

*(@magopian, @RaeKnowler)*

## py.test

More pythonic

    !python
    assert 3 == 4 # instead of self.assertEqual(3, 4)

**pytest-django** with really interesting runner options:

- reuse database: *--reuse-db* and *--create-db*
- launch only last failed tests

## Hypothesis

Use randomised data and find edge cases for you

    !python
    @given(floats())
    def test_negation_is_self_inverse(x):
        assert x == -(-x)

---

# Django admin

*(@olasitarska)*

- Use for: small tricks to improve UI
- **Don't** use for: end-users UI, big customisations
- Other themes: django-flat-theme, django-suit (commercial)

![django admin dashboard with django-flat-theme][admin_dashboard]

[admin_dashboard]: images/admin_dashboard.png

---

# Security

*(@erikpub, @ubernostrum)*

- Test 12 basic security issues: [ponycheckup.com](https://www.ponycheckup.com/)
- Examples of Django protections:
    * Timing attack on password

---

# Rabbit hole

*(@asendecka, @thomaswturner)*

# Definition
- Trying to resolve a problem that takes ages at the end
- Hard to diagnose
- Ask for help, do some breaks

# Example (Django on Desktop)
Unbelievable stack (Django, C++, Firebird, CEF, Sikuli)

---

# Performance issues

*(@piquadrat_ch)*

## Steps
- Step 1: reduce SQL queries (select_related, prefetch_related on reverse FKey/M2M)
- Step 2: do less work
    * move work out of request/response cycle (celery)
    * only fetch what you need (QuerySet.defer())
    * do calculations on the db (annotate, aggregate)
- Step 3: caching (hard to invalidate, johnny-cache or django-cache-machine)

## Tools
- django-debug-toolbar
- django-devserver

---

# Real-time

- Swamp dragon (Django + tornado + redis, or Django + Pusher)
- Custom run_server command to test without tornado

---

# ORM

- Lookup, transforms, expressions
- StoreField : key -> value (PostGre, Django 1.8)
- ArrayField : [ ] (PostGre, Django 1.8)

---

# Useful tools

- git-crypt: encrypt/decrypt files on github
- cookiecutter: django project templating

---

# Iwoca

- Lend money to small/medium businesses,
- Non middle-age technologies (Django, Twisted, Crossbar.io, AngularJS)

---

# Conclusion

- REALLY friendly
- Not so technical -> djangounderthehood.com
- Pies in pieminister and welsh cakes were ... waouh.
