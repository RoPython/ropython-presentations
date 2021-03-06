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

Utilizare
=========

1. Logging

.. code-block:: python

   @log("Calling func {}")
   def add_user(request):
      ...
	  
2. Sincronizare

.. code-block:: python

   @synchronize
   def do_atomic_stuff():
      ...   

3. Caching

.. code-block:: python

   @memoize
   def do_expensive_operation():
      ...
	  
----------

Utilizare
=========

4. Validarea argumentelor

.. code-block:: python

   @accepts(int, int)
   @returns(list)
   def do_operation(a, b):
      ...

5. Reexecuție în caz de eroare

.. code-block:: python

   @retry(FileNotFoundError)
   def read_data(file):
      ...
	  
6. Decoratori builtin

.. code-block:: python

   class Deque:
      @classmethod
      def from_list(cls, obj):
          ...
      @staticmethod
      def do_operation(deque):
         ... 	  

--------------
	  

Decoratori
==========

* Python ne pune la dispoziție conceptul de **decoratori** pentru a realiza aceste lucruri.

* Este parțial bazat pe șablonul **Decorator**, fiind mai degrabă similar cu macro-urile din C.

* Ca să îi înțelegem, trebuie să înțelegem funcțiile.

* Totul în Python este un obiect, în conformitate cu modelul Von Neumann-ian.

* Funcțiile sunt *first-class citizens*, sunt și ele obiecte. 

-------------

Funcții
=======

.. code-block:: python

   >>> def compute(a):
   ...    return (a ** a) + 42
   >>> list(map(compute, [1, 2, 3, 4, 5]))
   [43, 46, 69, 298, 3167]
   >>>
   >>> def compute_data(elements, compute_func):
   ...    return list(map(compute_func, elements))
   >>> compute_data([1, 2, 3, 4, 5], compute)
   [43, 46, 69, 298, 3167]
   >>> a = compute_data
   >>> a([1, 2, 3, 4, 5], compute)
   [43, 46, 69, 298, 3167]
   
---------------------------

Decoratori
==========

* cea mai simplă definiție a unui decorator: o funcție care întoarce o altă funcție.

.. code-block:: python

   >>> def decorator(func):
   ...   print("!!! I'm a decorator. !!!")
   ...   return func
   >>> def func(a):
   ...   return a + a
   >>> func = decorator(func)
   >>> func(2)
   !!! I'm a decorator. !!!
   4

------------

Decoratori
==========

* un alt feature care stă la baza conceptului de decorator este abilitatea de a defini o funcție într-o altă funcție.

.. code-block:: python

   >>> def decorator(func):
   ...    def wrapper(a):
   ...       print("!!! I'm another decorator !!!")
   ...       return func(a)
   ...    return wrapper
   >>> func = decorator(func)
   >>> func(2)
   !!! I'm a decorator. !!!
   4

---------------------

Decoratori
==========

.. code-block:: python

   def decorator(func):
      ...
   func = decorator(func)

* **func** este repetat de 3 ori.
* Nu este evident din prima faptul că funcția este decorată.
* De aceea, putem scrie:

.. code-block:: python

   @decorator
   def func(a):
       ...

* **@** este doar syntactic sugar pentru **func = decorator(func)**

---------------

Componența unui decorator
=========================

* funcția întoarsă din decorator

.. image:: decorator_replacement_func.png
   :scale: 100 %
   :align: center

-------------------

Componența unui decorator
=========================

* closure / lexical scope

.. image:: closure.png
   :scale: 100 %
   :align: center

-----------------

Closure
=======

* o funcție care ține minte valorile din mediul lexical înconjurător.
* în exemplul nostru, *cache* este ținut minte de *wrapper*, chiar și în afara funcției *decorator*.
* closure != funcții anonime.

-----------

Componența unui decorator
=========================

* adnotarea unei funcții cu un decorator

.. image:: decorator_annotation.png
   :scale: 100 %
   :align: center

---------------------

Componența unui decorator
=========================

* decoratorul complet

.. image:: complete_decorator.png
   :scale: 100 %
   :align: center

--------------------------------

Decoratori
==========

* ceva mai complicat:

.. code-block:: python

   def memoize(func):
       cache = {} # works due to lexical scoping
       def wrapper(*args):
           if args not in cache:
               cache[args] = func(*args)
           return cache[args]
       return wrapper

   @memoize
   def fibbonaci(n):
       if n == 0:
           return 0
       elif n == 1:
           return 1
       else:
           return fibbonaci(n-1) + fibbonaci(n-2)

-------------------

Decoratori
==========

* Folosind un decorator, putem optimiza o funcție, salvând rezultatele frecvente într-un cache.

.. code-block:: sh

   % timeit fibbonaci_simple(30)
   1 loops, best of 3: 1.48 s per loop

   % timeit fibbonaci_decorated(30)
   1 loops, best of 3: 1.9 µs per loop

--------------------

Decoratori
==========

* Pot primi argumente.

.. code-block:: python

   cache = {}
   
   @memoize(cache=cache)
   def fibbonaci(n):
       ...
	   
* Pentru asta, trebuie să modificăm decoratorul, astfel încât să adăugăm un nou nivel de scoping.

.. code-block:: python

   def memoize(cache=None):
      cache = cache or {}
      def wrapper(func):
          def wrapped_f(*args):
              if args not in cache:
                 cache[args] = func(*args)
              return cache[args]
          return wrapped_f
      return wrapper

* **memoize** este acum un **decorator factory**, un decorator ce întoarce alt decorator.	  

------------------

Decoratori
==========

* Pot fi aplicați și pe clase.
* Un decorator aplicat pe o clasă trebuie să întoarcă tot o clasă.

.. code-block:: python

   def memoize_methods(klass):
      ...
	  return klass
      
   @memoize_methods
   class MyClass:
       ...
	   
-------------------	   
	   

Decoratori
==========

* Sunt mult mai multe lucruri de povestit despre ei.
* Forma lor cea mai simplă nu este și cea corectă.
* Un viitor articol detaliat pe blog.ropython.org.

------------------

Mulțumesc!
==========