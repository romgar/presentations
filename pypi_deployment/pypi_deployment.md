# PyPI deployment in 5 minutes with GitHub + Travis CI

---

# Me (Romain Garrigues @rgarrigues)

- Web Developer @ ![iwoca-logo][iwoca_logo]
- Python/Django developer for 5 years
- Maintainer of some little django apps ([django-dirtyfields](https://github.com/smn/django-dirtyfields), [django-deep-collector](https://github.com/iwoca/django-deep-collector/))
- Presentation available @ [https://github.com/romgar/presentations/](https://github.com/romgar/presentations/)

[iwoca_logo]: images/logo_iwoca.png

---

# The goal

    !python
    $ pip install your_lovely_package

- Needs to be available on PyPI

---

# Your project on GitHub

![github-defaultproject][default_structure]
[default_structure]: images/default_structure.png

You only need:

- Your django app (python package)
- README to describe your project
- setup.py to describe your python package

---

# Setup file

- Reusable apps tutorial @ [https://docs.djangoproject.com/en/1.8/intro/reusable-apps/](https://docs.djangoproject.com/en/1.8/intro/reusable-apps/)

    !python
    import os
    from setuptools import setup

    with open(os.path.join(os.path.dirname(__file__), 'README.md')) as readme:
        README = readme.read()

    # allow setup.py to be run from any path
    os.chdir(os.path.normpath(os.path.join(os.path.abspath(__file__), os.pardir)))

    setup(
        name='your-lovely-package',
        version='0.1',
        packages=['your_lovely_package'],
        include_package_data=True,
        license='BSD License',  # example license
        description='A simple lovely package.',
        long_description=README,
        url='https://github.com/romgar/your-lovely-package',
        author='Romain Garrigues',
        author_email='romain.garrigues.cs@gmail.com',
        classifiers=[
            'Environment :: Web Environment',
            'Framework :: Django',
            'Intended Audience :: Developers',
            'License :: OSI Approved :: BSD License', # example license
            'Operating System :: OS Independent',
            'Programming Language :: Python',
            # Replace these appropriately if you are stuck on Python 2.
            'Programming Language :: Python :: 3',
            'Programming Language :: Python :: 3.2',
            'Programming Language :: Python :: 3.3',
            'Topic :: Internet :: WWW/HTTP',
            'Topic :: Internet :: WWW/HTTP :: Dynamic Content',
        ],
    )

---

# Travis-CI

- Continuous integration platform for GitHub project
- Trigger scripts on every commit on every branch of your GitHub projects

First step: create an account on TravisCI by linking your GitHub account

![travis-landing_page][travis_landing_page]

Second step: activate GitHub repositories you want to use with Travis.

![github-login][github_login]



[travis_landing_page]: images/travis_landing_page.png
[github_login]: images/github_login.png
