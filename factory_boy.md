# Django tests with factory_boy

---

# Overview

Why factory_boy ?

- Reason 1
- Reason 2
- Reason 3

---

# Code Sample

Object creation simplification !!

    !python
    class FooTests(unittest.TestCase):

        def test_with_factory_boy(self):
            # We need a 200€, paid order, shipping to australia, for a VIP
            OrderFactory(amount=200, status='PAID', 
            customer__is_vip=True, address__country='AU')
            # Run the tests here

        def test_without_factory_boy(self):
            address = Address(street="42 fubar street", zipcode="42Z42",
                city="Sydney", country="AU",
            )
            customer = Customer(first_name="John", last_name="Doe",
                phone="+1234", email="john.doe@example.org",
                active=True, is_vip=True, address=address,
            )
            # etc.