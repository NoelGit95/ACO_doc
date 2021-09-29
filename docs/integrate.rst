.. raw:: pdf

    PageBreak

.. _integrate:

Step 2: Importing Modules
============================
In order to integrate the software module in your *oTree* project the following three
modules have to be imported at the top of the models.py file of your oTree app.

 - ``smtblib``: Used to send the automated email.
 - ``requests``: Used to obtain the current CO2 price.
 - ``traceback``: Used for error handling purposes.

The top of your models.py file should look like this:

.. code-block:: python

    from otree.api import (
        models,
        widgets,
        BaseConstants,
        BaseSubsession,
        BaseGroup,
        BasePlayer,
        Currency as c,
        currency_range,
    )
    import smtplib
    import requests
    import traceback
    #import everything else you need here



