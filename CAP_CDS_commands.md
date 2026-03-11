Commands:



 npm install @cap-js/hana --save

        cds init
        cds add data
        cds add xsuaa
        cds add hana --for production
        cds add xsuaa --for production
        cds add mta
        cds build
        cds deploy
        cds deploy --to hana
        cds watch --profile hana
        cds watch
        mbt build
        cf deploy mta_archives/externalServicesConnections_1.0.0.mtar




        cf cs destination lite externalservice-destination
        cf cs connectivity lite externalservice-connectivity
        cf cs hana hdi-shared externalservice-db


        cds bind --to externalservice-connectivity
        cds bind --to externalservice-db
        cds bind --to externalservice-destination



        cds bind --to services-db:studentService