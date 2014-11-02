# Django tests with factory_boy

---

# Overview

Why factory_boy ?

- It's easier to create objects for testing
- It makes your tests more readable
- It can avoid painful test code refactoring if your models are changing

---

# Simplify nested object creation

Object creation with/without factory_boy

    !python
    class Author(models.Model):
        name = models.CharField(max_length=666)

    class Book(models.Model):
        author = models.ForeignKey(Author)
        title = models.CharField(max_length=255)


    class TestObjectCreation(TestCase):

        def test_without_factory_boy(self):
            author = Author.objects.create(name="George RR Martin")
            Book.objects.create(author=author, title="Game of Scones")

        def test_with_factory_boy(self):
            BookFactory.create()            

---

# Simplify nested object creation

Factory definition (django model like)

    !python
    class Author(models.Model):
        name = models.CharField(max_length=666)

    class Book(models.Model):
        author = models.ForeignKey(Author)
        title = models.CharField(max_length=255)


    class AuthorFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Author

        name = factory.Sequence(lambda n: u'author#%s' % n)

    class BookFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Book

        author = factory.SubFactory(AuthorFactory)
        title = factory.Sequence(lambda n: u'title#%s' % n)

---

# Simplify nested object creation

Specify nested object values

    !python
    class Author(factory.django.DjangoModelFactory):
        name = models.CharField(max_length=666)

    class Book(models.Model):
        author = models.ForeignKey(Author)
        title = models.CharField(max_length=255)


    class TestCreateObject(TestCase):

        def test_without_factory_boy(self):
            author = Author.objects.create(name="George RR Martin")
            Book.objects.create(author=author, title="Game of Scones")

        def test_with_factory_boy(self):
            BookFactory.create(
                title="Game of Scones",
                author__name="George RR Martin")
