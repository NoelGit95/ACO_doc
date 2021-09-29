.. raw:: pdf

    PageBreak

.. _additional:

.. note::
    **Source Code:** All source code described on this website can simply be copied into your *oTree* project
    from the code block at the bottom of this section.

Step 4: Adding additional code
=================================
Some additional code is needed to ensure that the email is sent at the right time, containing
the correct data. The easiest way to do this, is to send the email (e.g. calling the function) after
all participants have finished the experiment and the correct amount of saved carbon emission has
been calculated.

In order to monitor the status of each participant and make sure that all players have finished the
experiment, it is recommended to implement the following fields and functions in your models.py file.

Player is finished
----------------------
You should add a Boolean field ``is_finished`` in the Player class, that states whether or not
a player has finished the experiment. The initial value of this field should be set to ``False``,
and turn ``True`` once the player has completed the experiment. Add the following code to your Player class:

.. code-block:: python

    class Player(BasePlayer):
        #-------------------------
        #ALL YOUR OTHER CODE HERE
        #-------------------------

        is_finished = models.BooleanField(initial=False)




Subsession is finished
--------------------------
Secondly, a Boolean field ``all_players_finished`` should be added to your Subsession class that states whether
or not all players in the Subsession have finished the experiment. This field has to be initialised as
``False`` and be set to ``True`` once every player has finished the experiment. In addition to this field
a corresponding function ``set_all_players_finished`` must be added in the Subsession class.
This function counts the total number of players that have finished the experiment and sets the
``all_players_finished`` field to ``True`` once all players have finished. Add the following code to
your Subsession class:

.. code-block:: python

    class Subsession(BaseSubsession):
        #------------------------------------------------
        #ALL YOUR OTHER CODE HERE
        #def send_payment_mail(...) should be here too
        #------------------------------------------------

        all_players_finished = models.BooleanField(initial=False)

        def set_all_players_finished(self):
            sum_finished = 0
            for p in self.get_players():
                if p.is_finished:
                    sum_finished += 1

            if sum_finished == self.session.num_participants:
                self.all_players_finished = True

After this code is implemented in your models.py file, the function can be called at the correct time,
including the correct data.
