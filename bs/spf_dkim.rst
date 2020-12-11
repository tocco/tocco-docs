Handling of new Domains (DKIM/SPF Records, Etc.)
################################################

Why SPF / DKIM
==============

Defense against spam:

* Make it harder to abuse domain for phishing
* Stop spam

Customers perspective:

* Don't allow sending mails in our name
* Ensure legitimate mails aren't classified spam or rejected


What's Needed for DKIM
======================

For DKIM one DNS records is required that identical for all customers, installations and
domains.

See :ref:`dkim-record`


What's Needed for SPF
=====================

For SPF a DNS records is needed that includes our SPF policy (``include:spf.tocco.ch``).

See :ref:`spf-record`

In the wild you'll encounter …:

a) … domains that already have a SPF record.

   In which case you insert ``include:spf.tocco.ch``.

   So, this::

       v=spf1 ip4:193.246.64.0/19 redirect=spf.mail.hostpoint.ch -all

   becomes::

       v=spf1 ip4:193.246.64.0/19 include:spf.tocco.ch redirect=spf.mail.hostpoint.ch -all

   (``include:spf.tocco.ch`` can appear anywhere between ``v=spf1`` and ``-all``.)

b) … domains that have no SPF records yet.

   In this case we'll have to find out what other providers send mails using the domain.

   Common examples for other providers:

   ================= ================== ========================================
    Name              Type               SPF Policy
   ================= ================== ========================================
    Abacus                               ? (required record varies)
    Atlassian                            ``include:_spf.atlassian.net``
    HostPoint         Hosting            ``include:spf.mail.hostpoint.ch``
    Google            Office             ``include:_spf.google.com``
    IWay              Hosting            ``include:spf.iway.ch``
    MailGun           Marketing mails    ``include:mailgun.org``
    MailJet           Marketing mails    ``include:spf.mailjet.com``
    MS Office365      Office             ``include:spf.protection.outlook.com``
   ================= ================== ========================================

   It's also common to send mails from ones own machines:

   ========================== ==========================================
    What                       Example policy
   ========================== ==========================================
    specific IP                ``ip4:1.2.3.4`` or ``ip6::2000:4e8::1``
    specific IP range          ``ip4:1.2.3.4/24`` or
                               ``ip6::2000:4e8::/48``
    specific host              ``a:mailhost.example.com``
   ========================== ==========================================

   Essentially we'll have to create a SPF record like this::

       v=spf1 include:spf.tocco.ch [OTHER_PROVIDERS]... -all


Who has to Update the Records
=============================

In most cases DNS is managed by the customer or a third party.

Some domains are managed by us. In this case we have to make DNS
adjustments ourselves.

See also :ref:`dns-who-updates-record`.


How to Collect the Required Information
=======================================

Phase 1: Collect Domains Customer wants to Use
----------------------------------------------

**When is the information collected and who collects it?**

a) New customers / initial domains:

   Information is collected during preliminary project phase.

b) Existing customer / new domains:

   Project manager, BS or sales receive request to add or remove domain(s).

**What information is needed?**

* What domains will be used to access Tocco via browser (webpage hosted in Tocco, intranet and backoffice)?
  
  (Existing customers: what domains need to added or removed.)

  Examples:

  * www.example.com
  * example.com
  * intranet.example.com

* What domain will be used in email sender addresses?

  (Existing customers: what domains need to be added or removed.)

  **One domain is required** to be able to send basic system mails. For instance,
  to be able to reset ones password.

  Examples:

  * example.com (includes peter\@example.com)
  * student.example.com (includes peter\@student.example.com)

**What to do with the collected information?**

Create a ticket for BS describing what domains need to be added or removed. From that point on BS will
handle all that's required. This includes further inquiries, communicating the required DNS changes, and
issuing TLS certificates.


Phase 2: Perform Required Changes / Contact Customer (BS)
---------------------------------------------------------

**Who?**

This is done by BS after receiving a ticket.

**What needs to be done?**

* Check if we manage DNS for that domain. See :ref:`dns-who-updates-record`.
* If we don't manage the domain, fetch registration details via https://www.nic.ch/whois/ (.ch domains)
  or https://whois.domaintools.com. Keep the information handy in case the customer does not know
  who manages DNS for it.

SPF:

* Check current SPF record. Online tool: `SPF validation <https://www.kitterman.com/spf/validate.html>`_.
* If none exists, ask the customer what other services send mails for that domain.  Then construct a
  new SPF record. See also `What's Needed for SPF`_ above.
* If one exists, have ``include:spf.tocco.ch`` inserted into the record

DKIM:

* Check current DKIM record. Online tool: `dnslookup <https://dnslookup.org/default._domainkey.tocco.ch/TXT/#delegation>`_
  (replace 'tocco.ch' with actual domain)
* If it doesn't exists yet, have it added. See :ref:`dkim-record`.
   

See Also
========

* `spread sheet with SPF/DKIM validation status <https://tocco.sharepoint.com/:x:/r/sites/Technik/_layouts/15/Doc.aspx?sourcedoc=%7BE3A9535A-64B0-4ECE-86F7-4E358FBDE07F%7D&file=spf_and_dkim_validity.ods&action=default&mobileredirect=true&DefaultItemOpen=1>`__
