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

# Avoid test code refactoring

We have just added a "new_field" in Author model that is **required**

    !python
    class Author(models.Model):
        name = models.CharField(max_length=666)
        new_field = models.CharField(max_length=666)

    class Book(models.Model):
        author = models.ForeignKey(Author)
        title = models.CharField(max_length=255)

---

# Security
- Test 12 basic security issues: ponycheckup.com

---

