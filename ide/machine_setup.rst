Setup a DevOps Work Machine
===========================

Setup IDEA
----------

Install IDEA
````````````

Download the `latest version <https://www.jetbrains.com/idea/download/>`__ and extract it:

.. code-block:: bash

    cd ~/Download
    wget https://download-cf.jetbrains.com/idea/ideaIC-XXXX.X.X.tar.gz
    cd ~/install   # or wherever you want to install it
    tar xf ~/Download/ideaIC-XXXX.X.X.tar.gz
    ln -s ~/Download/ideaIC-XXXX.X.X idea
    mkdir -p ~/bin
    ln -s ~/install/idea/bin/idea.sh ~/bin/idea
    rm ideaIC-XXXX.X.X.tar.gz

If an update is released that forces you to download a new \*.tar.gz file, extract it and replace the link
in ``~/install/``using ``ln -sfn ideaIC-XXXX.X.X idea``.

.. hint::

   This assumes that ``~/bin`` is in your ``$PATH`` which is the case on most Linux-based systems. If you had
   to create ``~/bin``, you may have to close and reopen the terminal for it to be added to ``$PATH``.

   If necessary, add it manually to ``$PATH`` by adding this to ``~/.profile``::

       export PATH="$HOME/bin:$PATH"


Create Desktop Entry
````````````````````

open ``idea`` and find **Tools** → **Create Desktop Entry…** (or **Configure** → **Create Desktop Entry…** on
the welcome screen).


Generate GUI into Java Source Code
``````````````````````````````````

Find **File** → **Settings** (or **Configure** → **Settings** on welcome screen).

.. figure:: resources/gui_designer_settings.png

   Select "Generate GUI into: Java source code"


Increase Memory Available to IDEA
`````````````````````````````````

#. **Help** → **Edit Custom VM Options…**
#. Say **Yes** to create a config file
#. Set max. memory (-Xmx) to something sensible (e.g. -Xmx3096m)


Increase Memory IDEA Uses for Maven Import
``````````````````````````````````````````

In Idea's main window: **File** → **Settings** → **Build, Execution, Deployment** → **Build Tools** → **Maven** →
**Importing** → **VM options for Importer** set memory to **-Xmx1024m** (Increase this even further if Maven import fails due to
out of memory.)


Increase Watch Limit (Linux only)
`````````````````````````````````

.. code-block:: bash

    cat >>/etc/sysctl.d/50-idea.conf <<EOF
    sysctl fs.inotify.max_user_watches=524288
    EOF

    sysctl -p /etc/sysctl.d/50-idea.conf


Install Java
------------

Required Java versions:

    ============== ===============
     Java Version   Nice Versions
    ============== ===============
     8              2.9 - 2.19
     11             2.20 -
    ============== ===============

Debian-Based Systems
````````````````````

Install default JDK:

.. code-block:: bash

    apt install default-jdk

… or install a particular version:

.. code-block:: bash

    # install Java 8
    apt install openjdk-8-jdk

… or, if you need a newer version, get it from the backports repository (Debian only):

.. code-block:: bash

    cat >>/etc/apt/source.list.d/backports <<EOF
    deb http://ftp.debian.org/debian $(lsb_release -cs)-backports main
    EOF

    apt update

    # install Java 11
    apt install openjdk-11-jdk

… or, use the ``openjdk-r`` PPA repository (Ubuntu-based only)

   See `How To Install OpenJDK 11 In Ubuntu 18.04, 16.04 or 14.04 / Linux Mint 19, 18 or 17 <https://www.linuxuprising.com/2019/01/how-to-install-openjdk-11-in-ubuntu.html>`_

   .. hint::

       Use this PPA to install Java 11 on **Ubuntu 18.04** (codename bionic). Later version have Java 11 included in the official repository.


Setup Maven
-----------

Install Maven
`````````````

Debian-Based Systems
^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    apt install maven


Support Multiple Java Versions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Add aliases to ``~/.bash_aliases`` (adjust versions as needed)::

    alias mvn8="JAVA_HOME='/usr/lib/jvm/java-8-openjdk-amd64' mvn"
    alias mvn11="JAVA_HOME='/usr/lib/jvm/java-11-openjdk-amd64' mvn"

This will allow you to run ``mvn`` using an explicit java version. Examples::

    You may also want to use ``-DskipEirslett`` to speed up the build, see
    `maven-eirslett`_.

    # Java 8
    mvn8 -pl customer/test -am clean install -DskipTests

    # Java 11
    mvn11 -pl customer/test -am clean install -DskipTests

See :ref:`Maven_Eirslett` about Eirslett.

.. hint::

    Close and reopen the terminal for ``mvn8`` and ``mvn11`` to become available.

Configure Maven
```````````````

Memory Settings
^^^^^^^^^^^^^^^

By default, the memory limit is based on available memory but that might not always be enough.

.. code-block:: bash

    echo 'export MAVEN_OPTS="-Xmx1024m"' >>~/.profile

.. important::

    Close and reopen your terminal for the changes to take effect.


Repository Access
`````````````````

.. code-block:: bash

    mkdir -p ~/.m2
    cat >~/.m2/settings.xml <<EOF
        THE ACTUAL CONTENT THAT GOES HERE CONTAINS A PASSWORD, GET IT AT:
        https://wiki.tocco.ch/wiki/index.php/Maven_2#settings.xml
    EOF

Increase Max. Number of Open Files
``````````````````````````````````

Create ``/etc/security/limits.d/open_file_limit.conf`` with this content::

        *                -       nofile          1000000

.. hint::

    Only effective once you **logged out and in** again.

If you encounter too-many-files-open errors during ``mvn build``, check out
:ref:`too-many-open-files-maven`.


Install Dependencies
--------------------

On **Ubuntu 18.04 and newer** install the following dependencies::

    apt install libjpeg62 libpng16-16

(libraries dynamically loaded by :term:`wkhtmltopdf`)


Setup SSH
---------

Create a Key
````````````

.. code-block:: bash

    ssh-keygen -t rsa -b 4096
    cat ~/.ssh/id_rsa.pub # copy key and paste in the next step


Distribute Key
``````````````

Add key on https://git.tocco.ch (**Your name** (top right corner) → **Settings** → **SSH Public Keys** → **Add Key** …)

You also want to give the content of **~/.ssh/id_rsa.pub** to someone of operations. To that regard, you can
post the **public** key in the :term:`Operations Public channel` and ask for it to be granted access.

.. tip::

    For admins: How to allow access is documented in :doc:`/devops/server_access`


Configure SSH
`````````````

#. Clone the *toco-dotfiles* repository::

    cd ~/src
    git clone https://github.com/tocco/tocco-dotfiles

#. Link ``authorized_keys_tocco`` into SSH config directory::

    mkdir -p ~/.ssh
    ln -s ~/src/tocco-dotfiles/ssh/known_hosts_tocco ~/.ssh/

#. Include config and set user name::

    cat >>~/.ssh/config <<EOF
    # First entry wins. So, override settings here, at the top.

    Host *.tocco.cust.vshn.net
        User \${FIRST_NAME}.\${LAST_NAME}

    # Comment in if you want to login as 'tadm' by default instead as 'tocco' (root permissions required).
    # Host *.tocco.ch
    #     User tadm

    Host *
        Include ~/src/tocco-dotfiles/ssh/config
    EOF

Replace **${FIRST_NAME}**.**${LAST_NAME}** like this: **peter.gerber**.

Setup Git
---------

Install Git
```````````

Debian-Based Systems
^^^^^^^^^^^^^^^^^^^^

.. code::

   apt install git


Configure Git
`````````````

.. code-block:: bash

    git config --global user.email "${USERNAME}@tocco.ch"  # replace ${USERNAME} with pgerber, …
    git config --global user.name "${YOUR_NAME}"  # replace ${YOUR_NAME} with "Peter Gerber", …

Ignore some files in all repository by default:

.. code-block:: bash

    cat >>~/.config/git/ignore <<EOF
    .idea/
    *~
    .*.swp
    EOF


Clone and Build Nice2
---------------------

.. code-block:: bash

    mkdir ~/src
    cd ~/src
    git clone ssh://${USERNAME}@git.tocco.ch:29418/nice2  # replace ${USERNAME} with pgerber, …
    cd nice2
    scp -p -P 29418 ${USERNAME}@git.tocco.ch:hooks/commit-msg .git/hooks/  # replace ${USERNAME} with pgerber, …

Test if Maven build works:

.. code-block:: bash

    cd ~/src/nice2
    # Depending on the Java version required, use ``mvn8``, etc.
    mvn11 -am install -T1.5C -DskipTests  # add "-Dmac" on Mac

.. _setup-openshift-client:

Setup OpenShift Client
----------------------

Install Client
``````````````

Download the client from the `OpenShift download page <https://www.okd.io/download.html>`__, extract it and
move the ``oc`` binary into ``$PATH``:

.. code-block:: bash

    cd ~/Download
    wget https://github.com/openshift/origin/releases/download/vX.X.X/openshift-origin-client-tools-vX.X.X-XXXXXXX-linux-64bit.tar.gz
    tar xf https://github.com/openshift/origin/releases/download/vX.X.X/openshift-origin-client-tools-vX.X.X-XXXXXXX-linux-64bit.tar.gz
    mkdir -p ~/bin
    cp openshift-origin-client-tools-vX.X.X-XXXXXXX-linux-64bit/oc ~/bin/oc
    rm openshift-origin-client-tools-vX.X.X-XXXXXXX-linux-64bit.tar.gz


Enable Autocompletion
`````````````````````

.. code-block:: bash

    cat >>~/.bashrc <<EOF
    eval \$(oc completion bash)
    EOF


Install Docker
--------------

Find your OS `here <https://docs.docker.com/install/#supported-platforms>`__ and follow the instructions.


S3 Access Key
-------------

Ask operations (e.g. via :term:`Operations Public channel`) to issue you an access key for our S3 storage.
Once you retrieved the key, add it to ``~/.aws/credentials`` like this::

     [nice2]
     aws_access_key_id=XXXXXXXXXXXXXXXXXXXX
     aws_secret_access_key=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

The section must be called *nice2*, the name is hardcoded in Nice2.

.. tip::

    For admins: how to issue a key is described in :ref:`s3-user-creation`.


S3 Access via ``s3cmd``
-----------------------

Install s3cmd::

    apt install s3cmd

Configure s3cmd::

    s3cmd --configure

Select the default for all options.

Adjust the following configuration entries in ``~/.s3cfg``::

    [default]
    access_key = XXXXXXXXXXXXXXXXXXXX
    secret_key = XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    host_base = objects.rma.cloudscale.ch
    host_bucket = %(bucket)s.rma.objects.cloudscale.ch

The access and secret keys are the keys you obtained in the previous step.

Test access::

    s3cmd info s3://tocco-dev-nice-overlay >/dev/null
    # no output expected


.. _setup-ansible:

Setup Ansible
-------------

.. hint::

    The following instruction have been tested on Debian, for other
    operating systems make sure you have this installed:

    * Python3
    * Ansible >= 2.7

    And these Python packages:

    * dnspython
    * boto3
    * openshift
    * psycopg2

.. hint::

    **Ubuntu 18.04 is not supported!** Upgrade to 19.10 or newer.

    Ansible shipped with Ubuntu 18.04 as well as the official PPAs ship with
    Python2. Our Ansible does not support this in any way.

#. Install Ansible and dependencies::

    apt install ansible python3-pip python3-boto3 python3-dnspython python3-psycopg2
    pip3 install openshift

#. Clone the repository:

   .. parsed-literal::

       mkdir -p ~/src
       cd ~/src
       git clone "ssh://\ **pgerber**\ @git.tocco.ch:29418/ansible"
       cd ansible
       scp -p -P 29418 **pgerber**\ @git.tocco.ch:hooks/commit-msg ".git/hooks/"

   Replace **pgerber** with your user name.

#. Get Ansible Vault password(s)

   Ansible Vault is used to store sensitive data (password, API keys) encrypted. You
   need a password to access them.

   There is two Vaults:

   a) One Vault that expects a file containing the password at ``~/.ansible-password``. This
      Vault is needed for server management and you generally **only need it if you're
      a part of the operations team**.

   b) The second Vault expects a file containing the password at ``~/.ansible-tocco-password``.
      This Vault is need to manage our application and is generally needed by devs and ops.

   Ask one of your colleagues to get the Vault passwords and store in said files. Be sure
   to set proper permission on the files::

       (umask 0077; echo ${PASSWORD_GOES_HERE} >~/.ansible-password)
       (umask 0077; echo ${PASSWORD_GOES_HERE} >~/.ansible-tocco-password)


.. _setup-postgres:

Setup Postgres (Optional)
-------------------------

If you wish to have Postgres running locally for development, you can setup Postgres
like this.

#. Install Postgres::

       sudo apt install postgresql

#. Setup Postgres for use with Ansible::

        sudo apt install zstd
        sudo -u postgres psql -c "CREATE ROLE \"$(id -un)\" WITH LOGIN SUPERUSER"
        psql -c "CREATE DATABASE \"$(id -un)\"" postgres

   The above is required to make sure :ref:`Ansible can be used to copy databases
   <ansible-copy-db>`. This creates a Postgres user with admin rights and with the
   same name as the Linux user. This allows to login via Unix socket without providing
   credentials.
