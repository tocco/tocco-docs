Customizing Docker Images
=========================

Docker Image Build - the Manual Way
-----------------------------------

#. Change working directory

    .. parsed-literal::

        cd **${ROOT_DIR_OF_NICE2_REPOSITORY}**

#. Assembly build

    .. parsed-literal::

        rm nice2-customer-\*-1.0-SNAPSHOT.tar.gz
        mvn -pl customer/**${CUSTOMER}** -am clean install -T1.5C -DskipTests -P assembly
        mv ./customer/tocco/target/nice2-customer-**${CUSTOMER}**-1.0-SNAPSHOT.tar.gz .

#. Building the Docker image

    .. parsed-literal::

        docker build -t **nice** --build-arg NICE2_VERSION=$(<current-version.txt) --build-arg NICE2_REVISION=$(git rev-parse HEAD) .

    The resulting image is called **nice**.

#. Deploy image

    .. parsed-literal::

        docker tag **nice** registry.appuio.ch/toco-nice-${INSTALLATION}/nice
        docker push registry.appuio.ch/toco-nice-${INSTALLATION}/nice

    Deployment is automatically started once the image is pushed.
