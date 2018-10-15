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
open ``idea`` and find **Tools** → **Create Desktop Entry…**


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

    cat >>/etc/apt/source.list.d/backports <EOF
    deb http://ftp.debian.org/debian $(lsb_release -cs)-backports main
    EOF
    apt update

    # install Java 9
    apt install openjdk-9-jdk


Setup Maven
-----------

Install Maven
`````````````

Debian-Based Systems
^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    apt install maven


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

You also want to give the content of **~/.ssh/id_rsa.pub** to someone of operations if you want SSH access to any of the
servers (i.e. ask in the operations channel to have your key added).


Configure SSH
`````````````

.. code-block:: bash

    cd ~/src
    git clone https://github.com/tocco/tocco-dotfiles
    cat >>~/.ssh/config <<EOF
    # overwrite settings here, *before* the "Include"

    Host *
        Include ~/src/tocco-dotfiles/ssh/config
    EOF


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
    scp -p -P 29418 ${USERNAME}@git.tocco.ch:hooks/commit_msg hooks/  # replace ${USERNAME} with pgerber, …

Test if Maven build works:

.. code-block:: bash

    cd ~/src/nice2
    mvn -am install -T1.5C -DskipTests  # add "-Dmac" on Mac


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
    eval $(oc completion bash)
    EOF


Install Docker
--------------

Find your OS `here <https://docs.docker.com/install/#supported-platforms>`__ and follow the instructions.
