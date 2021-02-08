RabbitMQ
========

You have two options to run RabbitMQ. Either run it locally on your device or reuse our test queue in the VSHN cloud.

Option 1: Use a local RabbitMQ
------------------------------

Pull the docker image and start a container::

    docker pull rabbitmq:3-management
    docker run -d -p 15672:15672 -p 5672:5672 rabbitmq:3-management

Set the following application properties::

    nice2.optional.rabbitmq.host=localhost
    nice2.optional.rabbitmq.port=5672
    nice2.optional.rabbitmq.virtualHost=/
    nice2.optional.rabbitmq.username=guest
    nice2.optional.rabbitmq.password=guest
    nice2.optional.rabbitmq.entity.exchangeName=tocco.test

The RabbitMQ management page is available under ``http://localhost:15672/``

Option 2: Use RabbitMQ at VSHN
------------------------------

Set the following application properties::

    nice2.optional.rabbitmq.host=185.72.22.67
    nice2.optional.rabbitmq.port=5672
    nice2.optional.rabbitmq.virtualHost=/
    nice2.optional.rabbitmq.username=XXXXXXX
    nice2.optional.rabbitmq.password=XXXXXXX
    nice2.optional.rabbitmq.entity.exchangeName=tocco.test

``185.72.22.67`` is the external ip address of the rabbitmq-loadbalancer service
For ``username`` and ``password`` see ``customer/integration/etc/application.properties``

The RabbitMQ management page is available under ``https://rabbmitmq-ui-toco-nice-integration.appuioapp.ch/``

Setup
^^^^^

First check if RabbitMQ is already configured.
If you see a login page under ``https://rabbmitmq-ui-toco-nice-integration.appuioapp.ch/`` skip the setup.

* Open project ``toco-nice-integration``
* Add to project -> Deploy Image -> ``rabbitmq:3-management`` as image name
* Set ``RABBITMQ_DEFAULT_USER`` and ``RABBITMQ_DEFAULT_PASS`` as environment variable (for values see ``customer/integration/etc/application.properties``)
* Add to project -> Import YAML / JSON (add access to RabbitMQ port)

.. code-block:: yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: rabbitmq-loadbalancer
    spec:
      ports:
      - name: rabbitmq
        port: 5672
      type: LoadBalancer
      selector:
        app: rabbitmq

* Add to project -> Import YAML / JSON (add access to RabbitMQ management web interface)

.. code-block:: yaml

    apiVersion: v1
    kind: Route
    metadata:
      name: rabbmitmq-ui
    spec:
      host: rabbmitmq-ui-toco-nice-integration.appuioapp.ch
      port:
        targetPort: 15672-tcp
      tls:
        termination: edge
      to:
        kind: Service
        name: rabbitmq
        weight: 100
      wildcardPolicy: None

Configure RabbitMQ
------------------

* Open the management page
* Create an exchange with the name ``tocco.test`` and the type ``topic``
* Create a queue with the name ``tocco``
* Open the queue and add a binding between the exchange and queue (A routing key is required)

Here are some routing key examples:

* ``#`` Forward all messages to the queue
* ``user.#`` only forward messages of user entities to the queue
* ``user.update`` only forward messages of user entities which are updated to the queue (alternatives are ``insert`` or ``delete``)

Under Queues -> ``tocco`` -> Get messages you see the current messages in the queue.
So you can easily check if the values are correct passed to the queue.

Configure installation
----------------------

Add the application properties (see above) and configure the RabbitMQ listener (change the ``filter`` property):

.. code-block:: xml

    <contribution configuration-id="nice2.persist.core.EntityListeners">
        <listener listener="service:nice2.optional.rabbitmq.RabbitMqAfterCommitListener" filter="User"/>
    </contribution>