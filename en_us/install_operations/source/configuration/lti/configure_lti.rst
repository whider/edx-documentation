.. _Configuring Credentials for a Tool Consumer:

###############################################################
Configuring Credentials for a Tool Consumer
###############################################################

For each installation that you want to allow access to your edX instances as an
LTI tool consumer, you create an OAuth key and secret pair, and then configure
your edX instance to allow access.

While each installation that you configure as a tool consumer must have
separate credentials, you can also shoose to create and configure more than one
set of credentials for each installation.

For example, you can configure every course that embeds edX content on a remote
learning system to have unique credentials. This approach lessens the
disruption and reconfiguration that might result if you have to revoke access
for one course at a later time.

.. contents::
   :local:
   :depth: 1

After you complete the configuration of a tool consumer, you can add the
consumer credentials to your learning system. For more information, see
:ref:``.

.. TBD

**********************************************
Generate the Key and Secret
**********************************************

To generate a secure key and secret pair, follow these steps.

.. placeholder: used saml command and just put consumer in instead, TBD

#. On your local computer or on the server, open Terminal or a Command Prompt
   and run the following command.
   
   ``openssl req -new -x509 -days 3652 -nodes -out consumer.crt -keyout consumer.key``

#. Provide information at each prompt. 
   
Two text files, ``consumer.crt`` and ``consumer.key``, are created in the
directory where you ran the command.

**************************************************
Configure the Tool Consumer
**************************************************

To configure an LTI tool consumer to have access to your Open edX installation,
follow these steps.

.. placeholder. need access to Django admin. TBD

#. Sign in to the Django administration console for your base URL. For example,
   ``http://{your_URL}/admin``.

#. In the **LTI** section, next to **Tool Configuration** select
   **Add**.

#. Select **Enabled**.

#. Enter the following information.

  - **Consumer Name**: An identifying name for the tool consumer.

  - **Consumer Key**: Use a text editor to open the ``consumer.key`` file, and
    then copy the private key into this field.
  
  - **Consumer Secret**: Use a text editor to open the ``consumer.crt`` file,
    and then copy the secret into this field.

  - **instance_guid**: A system generated, globally unique identifier. Do not supply a value for this field.  
  
.. more to come TBD

5. Select **Save** at the bottom of the page.



