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

Easily deploy a new release on PyPI, for example by pushing a tag on master branch.

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

Example:

    !python
    import os
    from setuptools import setup

    setup(
        name='your-lovely-package',
        version='0.1',
        packages=['your_lovely_package'],
        include_package_data=True,
        license='BSD License',  # example license
        description='A simple lovely package.',
        long_description='You could read README file and put it there',
        url='https://github.com/romgar/your-lovely-package',
        author='Romain Garrigues',
        author_email='romain.garrigues.cs@gmail.com',
        classifiers=[
            'Framework :: Django',
        ],
    )

---

# Travis-CI : Create an account

- Continuous integration platform for GitHub projects @ [https://travis-ci.org/](https://travis-ci.org/)
- Trigger scripts on every commit on every branch of your GitHub projects

![travis-landing_page][travis_landing_page]
[travis_landing_page]: images/travis_landing_page.png

---

# Travis-CI: Link GitHub account

![github-login][github_login]
[github_login]: images/github_login.png

---

# Travis-CI: Activate GitHub repositories
[https://travis-ci.org/profile/romgar](https://travis-ci.org/profile/romgar)

![travis-activate_repo][travis_activate_repo]
[travis_activate_repo]: images/travis_activate_repo.png

---

# GitHub: configure travis

Create a .travis.xml file on your GitHub repository root:

    !shell
    language: python

    python:
      - "2.7"

    script:
      - touch foo

---

# GitHub: deploy section in travis config file

    !shell
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

# GitHub: deploy section credentials

user: your PyPI username

password: should be generated with:

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
- Every step explained in details @ [5minutes.youkidea.com](http://5minutes.youkidea.com/howto-deploy-python-package-on-pypi-with-github-and-travis.html)
