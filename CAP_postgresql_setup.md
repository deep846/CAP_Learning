1. Create an instance in the of postgresql hyperscaler from the btp service instance then go to the instance and create a service key.

2. go to your project

        cds bind --to <serviceName>:<servicekeyname>

3. then run the following command below in the root directory project

        cds add mta
        cds add postgres
        npm i
        cds build
        cds deploy
        cds deploy --to postgres

then change the below in the mta file

          - name: postgresql-inst
            type: org.cloudfoundry.managed-service
            parameters:
                service: postgresql-db
                service-plan: trial

under module under service requires add a -name: < your instance name >

          modules:
            - name: CAP_postgres-srv
                type: nodejs
                path: gen/srv
                requires:
                - name: CAP_postgres-auth
                - name: postgresql-inst          <--- add here your instance name

under module under < your app name >-postgres-deployer and under requires add your instacename;

              - name: CAP_postgres-postgres-deployer
                type: nodejs
                path: gen/pg
                parameters:
                buildpack: nodejs_buildpack
                no-route: true
                no-start: true
                tasks:
                    - name: deploy-to-postgresql
                    command: npm start
                requires:
                - name: postgresql-inst       <----- add here your instance name

4. then run the following command

                cf login
                mbt build
                cf deploy <.mtar file>
