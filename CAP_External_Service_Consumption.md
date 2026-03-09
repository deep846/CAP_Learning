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






<!-- watch the following vide -->


https://www.youtube.com/watch?v=CNinDSPDUCM&list=PL3Q8uML_MhIg3x6lWB4KY_qaGQPaVFG_R&index=22

