:title: Unit testing
:data-transition-duration: 500
:css: tutorial.css

----

Unit testing în Python
======================

Claudiu Popa (claudiu@ropython.org)
-----------------------------------

.. note::

    Welcome to the presenter console!

----

Problema
========

* Ce ar putea să meargă prost?

.. code:: python

    def process_payment(user, payment, product):
        balance = user.account_balance()
        payment_gateway.get_money_from_account(
            user.account, balance - payment)
        shipment_gateway.ship_product(user, product)

.. note::

       # account frozen
       # negative payment, gets money from the user
       # positive payment, but the user doesn't have enough money
       # shipping without checking the results from the payment gateway


