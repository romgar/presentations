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

Be able to install your package with pip:

    !bash
    $ pip install your_lovely_package

- Needs to be available on PyPI

---

# GitHub: Your project

![github-defaultproject][default_structure]
[default_structure]: images/default_structure.png

You only need:

- Your django app (python package)
- README to describe your project
- setup.py to describe your python package

---

# GitHub: Setup file

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
            'Framework :: Django',
        ],
    )

---

# Travis-CI

- Continuous integration platform for GitHub project @ [https://travis-ci.org/](https://travis-ci.org/)

- Trigger scripts on every commit on every branch of your GitHub projects

---

# Travis-CI: Create an account

![travis-landing_page][travis_landing_page]
![github-login][github_login]

[travis_landing_page]: images/travis_landing_page.png
[github_login]: images/github_login.png

---

# Travis-CI: Activate GitHub repositories
[https://travis-ci.org/profile/romgar](https://travis-ci.org/profile/romgar)

![travis-activate_repo][travis_activate_repo]
[travis_activate_repo]: images/travis_activate_repo.png

---

# GitHub: configure travis

- Create a .travis.xml file on your GitHub repository root.

    !python
    language: python

    python:
      - "2.7"

    script:
      - touch foo

---

# GitHub: deploy section in travis config file

    !python
    language: python

    python:
        - "2.7"

    script:
        - touch foo

    deploy:
        provider: pypi
        user: romgar
        password:
            secure: my_secure_password
        on:
            tags: true
            branch: master

---

# Generate your secure password

    !shell
    $ gem install travis
    $ travis encrypt --add deploy.password

The generated password will be automatically added to your .travis.yml config file.

---

# Summary

- Create a GitHub repository with your package, a README and a setup.py file,
- Activate Continuous Integration of this repository on TravisCI,
- Create a .travis.yml, with a deploy section.
- Generate secure password.
- Well done !!!
- Every steps explained in details @ [5minutes.youkidea.com](http://5minutes.youkidea.com/howto-deploy-python-package-on-pypi-with-github-and-travis.html)
