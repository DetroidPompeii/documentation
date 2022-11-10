=============
Pay by checks
=============

Once you decide to pay a supplier bill, you can select to pay by check. You can then print all the
the registered payment by checks. Finally, the bank reconciliation process will match the checks you
sent to suppliers with actual bank statements.

Configuration
=============

Activate checks payment methods
-------------------------------

The first necessary step to pay by checks is to activate the setting. To do so go to
:menuselection:`Accounting app --> Configuration --> Checks`, there you can activate the payment
method as well as setup the :guilabel:`Check Layout`.
Once you have activated the setting, you must also activate the payment method on related bank
journals. Go to :menuselection:`Accounting dashboard --> Bank Journal --> drop-down menu (⋮) -->
Configuration`.
In the :guilabel:`Outgoing Payments` tab, add :guilabel:`Checks` as a payment method.

.. note::
   Some countries require specific modules to print checks, such modules may be installed by
   default. For instance the :guilabel:`U.S. Check Printing` module is required to print U.S.
   checks.

Compatible check stationery for printing checks
===============================================

United States
-------------

For the United States, Odoo supports by default the check formats of:

- **Quickbooks & Quicken**: check on top, stubs in the middle and bottom
- **Peachtree**: check in the middle, stubs on top and bottom
- **ADP**: check in the bottom, and stubs on the top.

Pay a supplier bill with a check
================================

Paying a supplier with a check is done in three steps:

1. registering a payment
2. printing checks in batch for all registered payments
3. reconcile bank statements

Register a payment by check
---------------------------

To register a payment, open any supplier bill from the menu :menuselection:`Purchases --> Vendor
Bills`.
Once the supplier bill is validated, you can register a payment. Set the :guilabel:`Payment Method`
to :guilabel:`Checks` and validate the payment.

Print checks
------------

On your :guilabel:`Accounting Dashboard` in the :guilabel:`Bank Journal`, you are able to see the
number of checks registered. By clicking on :guilabel:`Checks to print` you have get the possibility
to print the checks reconciled.

To print all checks in batch, select all payments from the list view and click on :guilabel:`Print`.
