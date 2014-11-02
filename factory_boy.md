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

Specify nested objects values

    !python
    class Author(factory.django.DjangoModelFactory):
        name = models.CharField(max_length=666)

    class Book(models.Model):
        author = models.ForeignKey(Author)
        title = models.CharField(max_length=255)


    class TestObjectCreation(TestCase):

        def test_without_factory_boy(self):
            author = Author.objects.create(name="George RR Martin")
            Book.objects.create(author=author, title="Game of Scones")

        def test_with_factory_boy(self):
            BookFactory.create(
                title="Game of Scones",
                author__name="George RR Martin")


---

# Test readability

Only initialise data that are really important.

- Only focus on what is useful for the test
- Tests are more understandable, as we directly see what is needed
- Don't forget that other people will read your tests for code comprehension

---

# Test readability

Example with models that contains more fields

    !python
    class Author(factory.django.DjangoModelFactory):
        name = models.CharField(max_length=666)
        birth_date = models.DateTimeField()
        favorite_color = models.CharField(max_length=50)
        favorite_expression = models.CharField(max_length=1050)
        favorite_breakfast_cereals = models.CharField(max_length=111)

    class Book(models.Model):
        author = models.ForeignKey(Author)
        title = models.CharField(max_length=255)
        publication_date = models.DateTimeField()
        category = models.CharField(max_length=255)

---

# Test readability

Too many useless data impact test readability, and are... useless !

    !python
    class TestBookAuthorFavoriteCereals(TestCase):

        def test_without_factory_boy(self):
            author = Author.objects.create(
                name="George RR Martin",
                birth_date=datetime(1948, 9, 20),
                favorite_color="black",
                favorite_expression="Breakfast is coming !",
                favorite_breakfast_cereals="Honey smacks"
            )
            book = Book.objects.create(
                author = author,
                title="Game of scones",
                publication_date=datetime(2014, 9, 9),
                category="cooking"
            )

            query = Book.objects.filter(
                category="cooking",
                author__favorite_breakfast_cereals="Honey smacks")
            self.assertEquals(query.count(), 1)

---

# Test readability

Less infos, and you focus on what is really important for your test.

    !python
    class TestBookAuthorFavoriteCereals(TestCase):

        def test_with_factory_boy(self):
            BookFactory.create(
                category="cooking",
                author__favorite_breakfast_cereals="Honey smacks")

            query = Book.objects.filter(
                category="cooking",
                author__favorite_breakfast_cereals="Honey smacks")
            self.assertEquals(query.count(), 1)