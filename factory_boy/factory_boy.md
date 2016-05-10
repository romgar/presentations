# Django tests with factory_boy (@rgarrigues - ![iwoca-logo][iwoca_logo])

[iwoca_logo]: images/logo_iwoca.png

---

# Current problem

## Create a complex set of data in test environment that is:

### - Simple to use

### - Easy to read

### - Fast to learn

### - Be used on any project

---

# Several solutions

### - "Direct" Django ORM

### - Django built-in fixtures (xml/yaml/json)

### - Some python packages created for that purpose: factory_boy, model_mummy


---

# Overview

## Why factory_boy ?

### - Simplify object creation for testing purpose
### - Avoid painful test code refactoring if your models are changing
### - Tests are more readable
### - Well adapted to Django, with Django-like syntax, managing all relations

---

# How it works

---

# How it works: create your factory

    !python
    # Django "classic" model
    class Author(models.Model):
        name = models.CharField(max_length=255)
        name2 = models.CharField(max_length=255, default='we_dont_care')

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
            self.assertEqual(author.name, 'George RR Martin')

        def test_with_factory_boy(self):
            author = AuthorFactory(name='George RR Martin')
            self.assertEqual(author.name, 'George RR Martin')

        def test_with_factory_boy_auto_creation(self):
            author = AuthorFactory()
            self.assertEqual(author.name, 'name#1')

        def test_with_factory_boy_auto_creation_and_undefined_field(self):
            author = AuthorFactory()
            self.assertEqual(author.name2, 'we_dont_care')

---

# Simplify nested object creation

---

# Simplify nested object creation

    !python
    class Author(models.Model):
        name = models.CharField(max_length=255)

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

    !python
    class Author(models.Model):
        name = models.CharField(max_length=255)

    class Book(models.Model):
        author = models.ForeignKey(Author)
        title = models.CharField(max_length=255)

    # Factory definition
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

# Simplify nested object creation (3)

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
            # Specify nested objects values
            BookFactory(
                title="Game of Scones",
                author__name="George RR Martin"
            )

---

# Avoid test code refactoring

---

# Avoid test code refactoring

We have just added a **new_field** in Author model that is **required**

    !python
    class Author(models.Model):
        name = models.CharField(max_length=255)
        new_field = models.CharField(max_length=255)

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
**new_field** on new tests.

---

# Improve test readability

---

# Test readability

### - Only focus on what is useful for the test
### - Tests are more understandable, as we directly see what is needed
### - Don't forget that other people will read your tests for code comprehension

---

# Test readability (2)

Example with models that contain more fields

    !python
    class Author(models.Model):
        name = models.CharField(max_length=255)
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
            self.assertEqual(query.count(), 1)

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
            self.assertEqual(query.count(), 1)

---

# Manage all Django ORM relations

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

# Extras

---

# Extra: fancy data generation

factory.fuzzy module to generate random data for some defined types:

    !python
    class FuzzyFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = FuzzyModel

        random_int = factory.fuzzy.FuzzyInteger(30, 99)
        random_char = factory.fuzzy.FuzzyChoice(['Low', 'Medium', 'High'])
        [...]


---

# Extra: "real" data generation

Faker python module can be used to generate "real" data

    !python
    class AuthorFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Author

        name = factory.LazyAttribute(lambda x: faker.name())
        phone_number = factory.LazyAttribute(lambda x: faker.phone_number())

Faker can generate addresses, datetimes, ...


---

# Extra: LazyAttribute

We can generate some fields depending on other fields

    !python
    class AuthorFactory(factory.Factory):
        class Meta:
            model = User
        name = factory.Sequence(lambda n: 'name%s' % n)
        email = factory.LazyAttribute(lambda o: '%s@example.com' % o.name)

    class TestLazyAttributes(TestCase):

        def test_lazy(self):
            author = AuthorFactory()
            self.assertEqual(author.name, 'name1')
            self.assertEqual(author.email, 'name1@example.com')

---

# Extra: build vs create

Build: create python object

Create: build + database saving

    !python
    class TestAuthorCreation(TestCase):

        def test_build(self):
            AuthorFactory.build()
            self.assertEqual(Author.objects.count(), 0)

        def test_creation(self):
            # Equivalent to AuthorFactory()
            AuthorFactory.create()
            self.assertEqual(Author.objects.count(), 1)

---

# Extra: batch build/create

    !python
    class TestBatchAuthorOperations(TestCase):

        def test_build(self):
            authors = AuthorFactory.build_batch(20)
            self.assertEqual(len(authors), 20)
            self.assertFalse(Author.objects.exists())

        def test_create(self):
            AuthorFactory.create_batch(20)
            self.assertEqual(Author.objects.count(), 20)

---

# Tips

---

# Tips: data generation conflicts

    !python
    class AuthorFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Author
        name = factory.Sequence(lambda n: u'author#%s' % n)

    # Imagine you have created an author in an initial data migration
    # with name='author#1'

    class TestAuthorCreation(TestCase):

        def test_author(self):
            # IntegrityError because the factory will try to create
            # an author with name='author#1'
            AuthorFactory()

---

# Tips: data generation conflicts (2)

Use django_get_or_create

    !python
    class AuthorFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Author
            django_get_or_create = ('name', )
        name = factory.Sequence(lambda n: u'author#%s' % n)

    class TestAuthorCreation(TestCase):

        def test_author(self):
            query = Author.objects.filter(name='author#1')
            self.assertEqual(query.count(), 1)

            # No more IntegrityError
            AuthorFactory()
            self.assertEqual(query.count(), 1)

---

# Tips: Anti-patterns

**DON'T** create a factory per "context", like BookFactory, BookWithAuthorFactory, BookWithAuthorAndAddressFactory, ...

    -> create util functions that uses different factories, depending
    on the context.

**DON'T** use it outside of a test environment:

    -> be really careful on random generated fields (!).

**DON'T** define ALL model fields in your factories, just mandatory ones and without default values:

    -> Always better to set data (if default is not wanted) than unset.

---

# Tips: How to switch to factory_boy ?

### Not switch directly all your tests

### Begin to create new factories on your new tests, in a separate file for example

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

### When you modify/fix/add tests in an app, update them with factories instead of direct orm calls

---

# ![iwoca-logo][iwoca_logo] is looking for factory boys !!

[iwoca_logo]: images/logo_iwoca.png

