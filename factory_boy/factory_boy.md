# Django tests with factory_boy (@rgarrigues)

---

# Current problem

Find a way to create a complex set of data in test environment that is:

    - Fast
    - Simple
    - Readable
    - DRY

---

# Several solutions

- Create your own code
- Use Django built-in fixtures (xml/yaml/json)
- Use some python packages created for that purpose: factory_boy, model_mummy


---

# Overview

Why factory_boy ?

- Simplify object creation for testing purpose
- Avoid painful test code refactoring if your models are changing
- Tests are more readable
- Well adapted to Django, with Django-like syntax, managing all relations


---

# How it works: create your factory

    !python
    # Django "classic" model
    class Author(models.Model):
        name = models.CharField(max_length=666)
        name2 = models.CharField(default='we_dont_care')

    # factory_boy factory definition for that model
    class AuthorFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Author
        name = factory.Sequence(lambda n: u'name#%s' % n)

---

# Use it

    !python
    class TestObjectCreation(TestCase):

        def test_without_factory_boy(self):
            author = Author.objects.create(name='George RR Martin')
            self.assertEquals(author.name, 'George RR Martin')

        def test_with_factory_boy(self):
            author = AuthorFactory(name='George RR Martin')
            self.assertEquals(author.name, 'George RR Martin')

        def test_with_factory_boy_auto_creation(self):
            author = AuthorFactory()
            self.assertEquals(author.name, 'name#1')

        def test_with_factory_boy_auto_creation_and_undefined_field(self):
            author = AuthorFactory()
            self.assertEquals(author.name2, 'we_dont_care')

---

# Simplify nested object creation

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
            BookFactory()

---

# Simplify nested object creation (2)

Factory definition

    !python
    class Author(models.Model):
        name = models.CharField(max_length=666)

    class Book(models.Model):
        author = models.ForeignKey(Author)
        title = models.CharField(max_length=255)


    class AuthorFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Author

        name = factory.Sequence(lambda n: u'name#%s' % n)

    class BookFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Book

        author = factory.SubFactory(AuthorFactory)
        title = factory.Sequence(lambda n: u'title#%s' % n)

---

# Simplify nested object creation (2)

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
            BookFactory(
                title="Game of Scones",
                author__name="George RR Martin"
            )

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

# Avoid test code refactoring (2)

Without factory_boy, we have to update each test that is using Author model

    !python
    class TestAuthor(TestCase):
    
        def test_author_1(self):
            Author.objects.create(
                name = "George RR Martin",
                new_field = "new value 1")

        def test_author_2(self):
            Author.objects.create(
                name = "George RR Martin",
                new_field = "new value 2")
        ...

Of course, you can create a setUp class to resolve this problem, but how to
deal with several TestCase classes that need Author objects ?

A function that factorise Author creation ? Yep. factory_boy already 
does the job :-)

---

# Avoid test code refactoring (3)

With factory_boy, we just have to update the factory.

    !python
    class AuthorFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Author

        name = factory.Sequence(lambda n: u'author#%s' % n)
        new_field = factory.Sequence(lambda n: u'new_field#%s' % n)

Each old test is not impacted by the newly added field, and we can use the 
"new_field" on new tests.

---

# Test readability

Only initialise data that are really important.

- Only focus on what is useful for the test
- Tests are more understandable, as we directly see what is needed
- Don't forget that other people will read your tests for code comprehension

---

# Test readability (2)

Example with models that contain more fields

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

# Test readability (3)

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

# Test readability (4)

Less infos, and you focus on what is really important for your test.

    !python
    class TestBookAuthorFavoriteCereals(TestCase):

        def test_with_factory_boy(self):
            BookFactory(
                category="cooking",
                author__favorite_breakfast_cereals="Honey smacks"
            )

            query = Book.objects.filter(
                category="cooking",
                author__favorite_breakfast_cereals="Honey smacks")
            self.assertEquals(query.count(), 1)

---

# Django relations: ForeignKey

    !python
    class Book(models.Model):
        author = models.ForeignKey(Author)

    class BookFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Book
        author = factory.SubFactory(AuthorFactory)

---

# Django relations: OneToOneField

Similar to ForeignKey

    !python
    class Book(models.Model):
        author = models.OneToOneField(Author)

    class BookFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Book
        author = factory.SubFactory(AuthorFactory)

---

# Django relations: ManyToManyField

    !python
    class Author(models.Model):
        books = models.ManyToManyField(Book)

    class AuthorFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Book

        @factory.post_generation
        def add_books_to_author(self, create, extracted, **kwargs):
            if not create:
                return
            if extracted:
                for book in extracted:
                    self.books.add(book)

    # In a test
    AuthorFactory(add_books_to_author=[book1, book2, book3])

Also possible to manage ManyToManyFields with intermediary tables, GenericForeignKeys, ...

---

Extra: fancy data generation

factory.fuzzy module to generate random datas for some defined types:

    !python
    class SomethingFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Something
        random_int = factory.fuzzy.FuzzyInteger(30, 99)
        randow_values_from_list = factory.fuzzy.FuzzyChoice(['Low', 'Medium', 'High'])
        [...]


---

Extra: "real" data generation

Faker python module can be used to generate "real" data

    !python
    class AuthorFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Author
        name = factory.LazyAttribute(lambda x: faker.name())
        phone_number = factory.LazyAttribute(lambda x: faker.phone_number())

Faker can generate addresses, datetimes, ...

---

Extra: batch creation

    !python
    class AuthorFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Author
        name = factory.LazyAttribute(lambda x: faker.name())

    !python
    class TestBatchAuthorCreation(TestCase):

        def test_creation(self):
            AuthorFactory.create_batch(20)
            self.assertEquals(Author.objects.count(), 20)

---

Extra: relations between several fields

TODO: give an example


---

Tips: data generation conflicts

    !python
    class AuthorFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Author
        name = factory.Sequence(lambda n: u'author#%s' % n)

    # Imagine you have created an author in an initial data migration with name='author#1'

    class TestAuthorCreation(TestCase):

        def test_author(self):
            # IntegrityError because the factory will try to create an author with name='author#1'
            AuthorFactory()

---

Tips: data generation conflicts (2)

Use django_get_or_create

    !python
    class AuthorFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Author
            django_get_or_create = ('name', )

        name = factory.Sequence(lambda n: u'author#%s' % n)

    class TestAuthorCreation(TestCase):

        def test_author(self):
            # No more IntegrityError
            AuthorFactory()

            self.assertEquals(Author.objects.filter(name='author#1).count(), 1)

---

Tips: "not-so-good" practices

*Don't* create a factory per "context", like BookFactory, BookWithAuthorFactory, BookWithAuthorAndAddressFactory, ...
    - better to create util functions that uses different factories, depending on the context.

*Don't* use it outside of a test environment:
    - be really careful on random generated fields (!).

*Don't* define ALL model fields in your factories, just mandatory ones and without default values:
    - Always better to set data (if default is not wanted) than unset.

---

# How to switch to factory_boy ?

Not switch directly all your tests
Begin to create new factories on your new tests, in a separate file for example

    !python
    .
    +-- manage.py
    +-- settings
    +-- your_app
    |   +-- admin.py
    |   +-- factories.py  <---
    |   +-- models.py
    |   +-- urls.py
    |   +-- views.py

When you modify/fix/add tests in an app, update them with factories instead of direct orm calls
