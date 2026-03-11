Consumig external services


1.download edmx file.

2. upload it in project srv/external folder and run import command to create the cns file	

cds import srv/external/API_BUSINESS_PARTNER.edmx 

3. add credentials

  "cds": {
    "requires": {
      "API_BUSINESS_PARTNER": {
        "kind": "odata-v2",
        "model": "srv/external/API_BUSINESS_PARTNER",
        "credentials":{
          "url": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_BUSINESS_PARTNER",
          "headers":{
            "APIkey":"<your api key should be here>"
          }
        }
      }
    }
  }



  "API_BUSINESS_PARTNER": {
        "kind": "odata-v2",
        "model": "srv/external/API_BUSINESS_PARTNER",
        "credentials": {
          "url": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_BUSINESS_PARTNER",
          "headers": {
            "APIkey": "qG22EbL7qY0W6atnF9XVI1h3QsNOBHfY"
          }
        }
      },



      "northwind": {
        "kind": "odata",
        "model": "srv/external/northwind",
        "credentials": {
          "url": "https://services.odata.org/V4/Northwind/Northwind.svc/"
        }
      },



      "employeedata": {
        "kind": "odata",
        "model": "srv/external/employeedata",
        "credentials": {
          "path": "/c2995fa0-27fe-48a1-afb3-5bb26ebdd4fc.employeemanagement.employeemanagement-0.0.1/odata/v4/employee",
          "destination": "empmanagement-destination-dest",
          "requestTimeout": 30000000
        }
      },


      "student": {
        "kind": "odata",
        "credentials": {
          "path": "/odata/v4/student",
          "destination": "services-dst-Destination",
          "requestTimeout": 30000000
        }
      },
  
  
3.5. service defination like

using {API_BUSINESS_PARTNER as s4} from './external/API_BUSINESS_PARTNER';
using {northwind as nw} from './external/northwind';
using {employeedata as emp} from './external/employeedata';
using {student as s} from './external/student';

service BuisnessService {

    entity A_BuisnessPartner as projection on s4.A_BusinessPartner;
    entity customers as projection on nw.Customers;
    entity products as projection on nw.Products;
    entity localData as projection on emp.Employee;
    entity studentdata as projection on s.student;

}



4. in the js file we need to connect to the api and then we need to return the query
  
  const cds = require('@sap/cds');

class BuisnessService extends cds.ApplicationService {
    async init(){
        const bp = await cds.connect.to('API_BUSINESS_PARTNER');
        this.on('READ','A_BuisnessPartner',async req=>{
            return bp.run(req.query);
        })

        return super.init();

    }
}

module.exports = {BuisnessService}  






<!-- watch the following video -->


https://www.youtube.com/watch?v=CNinDSPDUCM&list=PL3Q8uML_MhIg3x6lWB4KY_qaGQPaVFG_R&index=22
https://www.youtube.com/watch?v=NvvUemu08CA



COMMANSA :-------->


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


