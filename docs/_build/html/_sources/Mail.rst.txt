.. raw:: pdf

    PageBreak

.. _mail:

.. note::
    **Source Code:** All source code described on this website can simply be copied into your *oTree* project
    from the code block at the bottom of this section.

Step 3: Embedding the function
=================================
Next, the ``send_payment_mail()`` function has to be embedded in the Subsession class of the models.py file.
The following section explains how the function works, how it can be modified and how it has to be integrated
in the Subsession class of your *oTree* app.

Initial set up
------------------
First, enter your SMTP account credentials in the `#CONSTANTS` section of the function.

 - **MAIL_USER & MAIL_PASS:** There are two ways to enter your SMTP account info. Either enter a API key and
   secret key combination or enter your SMTP account email and your password.
 - **MAIL_SERVER:** Enter the the name of the SMTP server of your provider. You can find the server's
   name in the SMTP configuration settings of your SMTP account. The *Mailjet* server is called "in-v3.mailjet.com",
   the *SendinBlue* server is called "smtp-relay.sendinblue.com".
 - **MAIL_SENDER:** Enter your SMTP validated email address here. It is possible to enter the same email address
   for MAIL_USER and MAIL_SENDER.
 - **MAIL_PORT:** Enter the port through which a connection to the server is established. Various ports are
   possible, click `here <https://kinsta.com/blog/smtp-port/>`_ for an overview. We found that port 465 works best for
   us. If you have troubles with port 465, try port 587.
 - **DONATION_MINIMUM:** The minimal possible donation to make is 1 cent. This value must not be changed.

Parameters
----------------
The function requires the following six parameters:

 - ``self``: self is required by all functions in the oTree framework. It has no explicit use within this particular function.
 - ``weight_to_donate``: A float value used to pass the amount of carbon emission that is saved by the experimental participants.
 - ``unit``: A string value that defines the unit of the saved carbon emission. The following values are accepted: ``["mg", "g", "kg", "t", "oz", "lbs", "st"]``
 - ``experiment_name``: A string value that specifies the name of the experiment (e.g. "Carbon Emission Task").
 - ``payment_e_mail_name``: A string that specifies the name of the person or team that receives the mail.
 - ``payment_e_mail_to``: A list containing the mail addresses of all recipients . If the mail is only to be sent to one address then a single string can be passed to the function.

Sequence of events
---------------------
 1. The ``weight_to_donate`` value is converted to metric tons. The conversion is based on the ``unit`` value.
 2. The current CO2 price per ton for emission certificates is fetched from a price endpoint that is provided by
    `Compensators <https://www.compensators.org/>`_.
 3. The price of the carbon-emission certificate is calculated.
 4. The contents of the email are defined.

     - The mail subject includes the ``experiment_name`` parameter.
     - The mail body includes the ``payment_e_mail_name`` parameter as an initial greeting. Furthermore,
       the body includes the total weight of carbon-emission saved, the current price per ton for
       carbon-emission certificates, as well as the link to Compensators
       `donation form <https://www.spendenformular-direkt.org/forms/6944d11a-60d9-48a2-803f-b4b0c7797cb9>`_
       with the correct price to make the carbon-emission certificate purchase. These contents can be
       changed at will.
 5. A connection to the SMTP server is established and the email is sent to all recipients specified in
    the ``payment_e_mail_to`` list.

Add the function to your Subsession class
---------------------------------------------
Simply insert the function into the Subsession Class of your models.py file.
The Subsession class should look something like this:

.. code-block:: python

    class Subsession(BaseSubsession):

        #-------------------------
        #ALL YOUR OTHER CODE HERE
        #-------------------------

        def send_payment_mail(self,
                              weight_to_donate: float,
                              unit: str = "t",
                              experiment_name: str = "Experiment Name",
                              payment_e_mail_name: str = "John Doe",
                              payment_e_mail_to: list = ["john.doe@gmail.com"]):

            #CONSTANTS:
            MAIL_USER = "API key or SMTP account email"
            MAIL_PASS = "API secret key or SMTP account password"
            MAIL_SERVER = "SMTP Mail server here e.g.: `in-v3.mailjet.com`"
            MAIL_SENDER = "validated.email@gmail.com"
            MAIL_PORT = 465
            DONATION_MINIMUM = 1

            #UNIT CHECK:
            unit_list = ["mg", "g", "kg", "t", "oz", "lbs", "st"]
            if unit not in unit_list:
                raise Exception("unit parameter ", unit, "not recognised. Unit has to be in ", unit_list)

            #CONVERT UNIT TO METRIC TONS:
            if unit == "mg":
                weight_in_tons = weight_to_donate / 1000000000
            if unit == "g":
                weight_in_tons = weight_to_donate / 1000000
            if unit == "kg":
                weight_in_tons = weight_to_donate / 1000
            if unit == "t":
                weight_in_tons = weight_to_donate
            if unit == "oz":
                weight_in_tons = weight_to_donate / 35273.96198069
            if unit == "lbs":
                weight_in_tons = weight_to_donate / 2204.62262185
            if unit == "st":
                weight_in_tons = weight_to_donate / 157.47304442

            #GETTING THE CURRENT CO2 PRICE:
            price = 0
            try:
                price = requests.get("http://compensate.compensators.org/price.php").json()
                if 'price_per_ton' not in price:
                    raise Exception("Price not found in data")
                price_per_ton = float(price['price_per_ton'])
            except:
                pass
            donation_in_cents = weight_in_tons * price_per_ton

            # CHECK DONATION MINIMUM
            if donation_in_cents < DONATION_MINIMUM:
                print("The donation is less than 1 cent, therefore too small. No Mail was sent.")

            #SENDING THE PAYMENT MAIL
            else:

                #Define the body of the mail
                body = f"""Hello {payment_e_mail_name},

    The participants in your experiment: "{experiment_name}" donated {weight_to_donate:.3f} {unit} of CO2 Emission.
    This equals to {weight_in_tons:.3f} tons of CO2. At the current price of {(price_per_ton / 100):.2f} € per ton this sums up to a total donation of {(donation_in_cents / 100):.2f} €.

    To authorize the payment, please click here:
    https://www.spendenformular-direkt.org/forms/6944d11a-60d9-48a2-803f-b4b0c7797cb9?default_amount_1_in_cents={donation_in_cents}

    Best Regards
    The Automated Donation system :)
              """

                #DEFINE MAIL SUBJECT ADD MAIL BODY:
                email_text = f"Subject: [{experiment_name}] Please confirm the donation for the experiment\n\n{body}"

                try:
                    #CONNECT TO THE SMTP SERVER:
                    server = smtplib.SMTP_SSL(MAIL_SERVER, MAIL_PORT)

                    #LOGIN TO THE SMTP SERVER
                    server.login(MAIL_USER, MAIL_PASS)

                    #SEND THE EMAIL
                    server.sendmail(MAIL_SENDER, payment_e_mail_to, email_text.encode('utf8', 'ignore'))
                    server.close()
                    print("Your mail has been sent successfully")

                except:
                    print("Unable to send mail")
                    traceback.print_exc()





In order to call the function some additional set up in your code is needed.