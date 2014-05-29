:title: Unit testing
:data-transition-duration: 1000
:css: tutorial.css

----

Unit testing în Python
======================

Claudiu Popa (claudiu@ropython.org)
-----------------------------------

.. note::

    Welcome to the presenter console!

----

Problemă
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
       
----

Problemă
========

* utilizatorul nu are bani în cont -> primește un produs pe gratis

* plata este o valoare negativă -> primește bani de la sistemul nostru

* utilizatorul nu are suficienți bani în cont -> primește un produs pe gratis

* utilizatorul are contul blocat -> primește un produs pe gratis

* utilizatorul are suficienți bani în cont -> primește ceea ce a plătit

   
----

Dacă funcția nu a fost testată
==============================

* Astfel de probleme pot apărea în producție.

* Gradul de încredere în modificarea funcției este **0**.

----

:data-x: r0
:data-y: r500
:data-scale: 0.1

Soluția
=======

* testarea funcției în același timp cu scrierea ei

.. code:: python

    ...
    def process_payment(user, payment, product):
        if payment <= 0:
            raise ValueError('negative payment: {}'.format(payment))
        balance = user.account_balance()
        payment_gateway.get_money_from_account(
            user.account, balance - payment)
        shipment_gateway.ship_product(user, product)

    ...
    def test_negative_payment(self):
        self.assertRaises(ValueError, process_payment, user, 0, product)
        self.assertRaises(ValueError, process_payment, user, -1, product)
		
----


Unit testing
============

* ideal, fiecare test este independent de celelalte teste.

* testează doar cea mai mică unitate testabilă

* crează un API contract pe care o unitate trebuie să îl satisfacă

----

Beneficii
=========

* găsirea rapidă a problemelor în ciclul de dezvoltare a unui produs


* dovedește corectitudinea componentelor unui program.

----

unittest
========

* framework de testare inclus în librăria standard Python

* bazat pe familia xUnit

* folosit pentru testarea unei singure **unități**, funcție sau clasă.

        

