:title: Unit testing
:data-transition-duration: 1400
:css: css/tutorial.css

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

* utilizatorul nu are bani în cont -> primește un produs gratis

* plata este o valoare negativă -> primește bani de la sistemul nostru

* utilizatorul nu are suficienți bani în cont -> primește un produs gratis

* utilizatorul are contul blocat -> primește un produs gratis


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

----

Concepte
========

* test case - o unitate de test care verifică răspunsuri specifice pentru inputuri specifice

* test fixture - pregătirile necesare pentru unul sau mai multe teste.

* test suite - o colecție de test cases

* test runner - un executor al testelor respective

----

unittest
========

* Cea mai simplă formă a unui test

.. code-block:: python

    import unittest

    class TestDeque(unittest.TestCase):
        def test_popleft(self):
           d = deque([1, 2, 3])
           self.assertEqual(d.popleft(), 3)
           self.assertEqual(d, deque([1, 2])

    unittest.main()

----

unittest
========

* ``unittest.TestCase`` reprezintă o unitate de testare. Testele efective trebuie să înceapă cu ``test_``.

* pune la dispoziție o listă de aserțiuni, printre care:

.. image:: images/asserts.png

----

unittest
========

* outputul este intuitiv

.. code:: sh

    F.EF
    ======================================================================
    ERROR: test_raises_fails (__main__.TestCase)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
      File "a.py", line 17, in test_raises_fails
        zero_division()
      File "a.py", line 4, in zero_division
        return 1 / 0
    ZeroDivisionError: division by zero

    ======================================================================
    FAIL: test_equal_fails (__main__.TestCase)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
      File "a.py", line 10, in test_equal_fails
        self.assertEqual(1, 2)
    AssertionError: 1 != 2

    Ran 4 tests in 0.005s

    FAILED (failures=2, errors=1)

----

Test fixtures
=============

* ``setUp`` este rulat automat înainte de fiecare test. Poate fi folosit pentru pregătirea resurselor necesare pentru teste.

* ``tearDown`` este rulat după fiecare test. Poate fi folosit pentru închiderea și terminarea anumitor resurse.

.. code-block:: python

    class Test(unittest.TestCase):
        def setUp(self):
            self.database = create_connecton(host='localhost',
                                             user='postgres')
        def tearDown(self):
            self.database.close()

        def test_admin_is_created(self):
            users = self.database.select_users()
            self.assertIn('admin', users)

----

unittest
========

* codul de mai devreme poate deveni:

.. code:: python

    ...
    def process_payment(user, payment, product):
        if payment <= 0:
            raise ValueError('negative payment: {}'.format(payment))
        balance = user.account_balance()
        if balance <= 0:
            raise ValueError('invalid account balance: {}'.format(balance))
        if balance - payment < 0:
            raise ValueError('not enough money in account')
        payment_gateway.get_money_from_account(
            user.account, balance - payment)
        shipment_gateway.ship_product(user, product)

----

unittest
========

* În condițiile de față, cum testăm metoda ``account_balance`` în cadrul metodei ``process_payment``?

* Putem refactoriza astfel încât ``account_balance`` să fie primit ca argument sau ca metodă în cadrul unei clase.

* Sau putem folosi mocking

----

:data-x: r500
:data-y: r500
:data-rotate-x: 180
:data-scale: 0.1

mocking
=======

* concept avansat de testare, în care obiectele și resursele costisitoare pot fi înlocuite de obiecte false

* Python ne pune la dispoziție librăria ``mock``

.. code:: python

   from unittest import mock

   ...

   @mock.patch('User.account_balance',
               new=lambda: -1)
   def test_negative_balance(self):
       with self.assertRaisesRegex(ValueError, "negative payment"):
           process_payment(user, 100, product)

----

mocking
=======

* În exemplul de mai sus, înlocuim metoda ``account_balance`` din clasa ``User`` cu o funcție anonimă ce întoarce un număr negativ

* ``process_payment`` va utiliza noua funcție.

----

Mulțumesc!
==========

