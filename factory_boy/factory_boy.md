# Django tests with factory_boy (@rgarrigues)

---

Current problem

How to create a complex context (objects in database) in tests ?
We would like to do something like:

    MyCentralComplexObject.create()


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

---

# Different field initialisation steps

On factory instantiation

    !python
    MyFactory.create(my_field='foo')

If not defined, on factory definition

    !python
    MyFactory.create()

    class MyFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = MyFactory
        my_field = factory.Sequence(lambda n: u'author#%s' % n)

If not defined, on Django models definition

    !python
    MyFactory.create()

    class MyFactory(models.Model):
        my_field = models.CharField(max_length=255, default='bar')

---

# Overview

Why factory_boy ?

- Simplify object creation for testing purpose
- Avoid painful test code refactoring if your models are changing
- Tests are more readable

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

        name = factory.Sequence(lambda n: u'name#%s' % n)

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

# Avoid test code refactoring

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

# Avoid test code refactoring

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

# Test readability

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

---

# Django relations (TODO)

ForeignKey

    class BookFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Book

        author = factory.SubFactory(AuthorFactory)

---

# Django relations (TODO)

OneToOne

    class BookFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Book

        author = factory.SubFactory(AuthorFactory)

---

# Django relations (TODO)

ManyToMany

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

    AuthorFactory.create(add_books_to_author=[book1, book2, book3])

---

# Django relations (TODO)

ManyToMany with intermediary table

    class BookFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Book

        author = factory.SubFactory(AuthorFactory)

---

# Django relations (TODO)

GenericForeignKey

    class BookFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = Book

        author = factory.SubFactory(AuthorFactory)

---

Data generation

**factory.fuzzy** module, to generate random datas for some defined types : string,
 integer, float, decimal, date, datetime, choiceField, ...

**faker**

---

Data generation on already existing data

django_get_or_create

---

Where should your factories live ?

    .
    +-- manage.py
    +-- settings
    +-- your_app
    |   +-- admin.py
    |   +-- factories.py  <---
    |   +-- models.py
    |   +-- urls.py
    |   +-- views.py

---

Not-so-great ideas

Create a factory per "context", like BookFactory, BookWithAuthorFactory, BookWithAuthorAndAddressFactory.
-> better to create utils functions
-> nice to initialise some states easily

Use it for non-test purposes: be really careful on random generated fields (!).

Only set mandatory fields:

If field not defined on factory instantiation, neither in factory definition, then it will take model default value.
Always better to set data if default is not wanted than unset in tests.

---

Other fancy stuff:

- Bulk creation
- Attributes defined from other attributes / other factories
