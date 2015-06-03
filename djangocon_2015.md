# The Best of DjangoCon 2015

---

# Overall feeling

- Focused on well being (and burnout/depression witness)
- Accessible community (lot of first-time speakers)
- Need to go outside (meet people, see real talks)

---

# Architecture / Separating concerns

- Avoid big views.py files
- Use service layers to move business logic from views.

---

# Testing

- Use py.test : more pythonic, reuse database options, ...


---

# Django admin

- Other themes: django-flat-theme, django-suit (commercial)
- Use for: small tricks to improve UI
- Don't use for: end-users UI, big customisations

---

# Security

- Test 12 basic security issues: ponycheckup.com

---

# Rabbit hole

- Trying to resolve a problem that takes ages at the end
- Hard to diagnostise
- Ask for help, do some breaks.

---

# Performance issues

- Tools: django-debug-toolbad, django-devserver
- Step 1: reduce SQL queries (select_related, prefetch_related on reverse FKey/M2M)
- Step 2: do less work
   - move work out of request/response cycle (celery)
   - only fetch what you need (QuerySet.defer())
   - do calculations on the db (annotate, aggregate)
- Step 3: caching (hard to invalidate, johnny-cache or django-cache-machine)
