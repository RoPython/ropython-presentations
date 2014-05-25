Decoratori
==========

----------


.. image:: decorators.gif
   :scale: 100 %
   :alt: alternate text
   :align: center

---------

Exemplu
=======

.. code-block:: python

    def delete_user(request, user_id):
        logging.info("Deleting user {}.".format(user_id))
        try:
            user = User.objects.get(id=user_id)
        except User.UserDoesNotExist:
            logging.error("User does not exist")
            return False
        
        if user != request.user:
            logging.error("Can't delete yourself.")
            return False
        
        # delete the user and user's settings
        user.delete()
        user.settings.delete()
        
        logging.info("User {} deleted.".format(user_id))
        return True
        
------

Exemplu
=======

* Logica pentru logging este amestecată cu logica ștergerii unui user.
* Avem precondiții (obținerea unui user din baza de date, verificarea faptului că nu este același user ca cel autentificat).
* Precondițiile pot fi mutate în altă funcție.
* Codul care șterge efectiv userul este ascuns prin cod adițional, nu iese în evidență.

------

Rescris cu decoratori
=====================

.. code-block:: python

   @login_required
   @verify_user
   def delete_user(request, user):
      user.delete()
      user.settings.delete()
	  
Acum, codul are următoarele avantaje:

* Logica verificării unui user se face înainte de funcția noastră.
* Codul ce șterge un user este mult mai ușor de citit și de modificat.
* Codul este succint, clar și ușor de înțeles.
	  
-------------	  


Problemă
========

* Modificarea comportamentului unei funcții

* Separarea intereselor unui program în grupuri izolate

---------

Decoratori
==========

* Python ne pune la dispoziție conceptul de ``decoratori`` pentru a realiza aceste lucruri.

*