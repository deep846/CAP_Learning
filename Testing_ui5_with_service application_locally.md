look into this : https://github.com/deep846/ui5_to_cap_local_testing for testing


1. in the ui5 project in the project root terminal run the below command
                    
                    npm i -D ui5-middleware-simpleproxy

2. then go to ui5.yaml for proxy

                    for service with authorization use 
                               ----> ui5-middleware-simpleproxy
                    for service with non authorization use
                               ----> fiori-tools-proxy


3. under the configuration for of 'fiori-tools-proxy' name add a ' backend: ' property

                - name: fiori-tools-proxy
                  afterMiddleware: compression
                  configuration:
                    backend:
                        - path: /api        # (name_used_in_regiser_service_in_manifestl.json)  ---------> 1st testing external service local
                          pathReplace: /odata/v4/student
                          url: http://localhost:4004

                        - path: /functiondd                  ----------------> 2nd testing external service local
                          pathReplace: /odata/v4/functionand-action
                          url: http://localhost:8070

4. under the configuration for auth service of 'ui5-middleware-simpleproxy'

                    # --- ONLY 8090: inject Basic Auth so the popup disappears ---

                            - name: ui5-middleware-  ----------> 1st 
                              afterMiddleware: compression
                              mountPath: /secureservice
                              configuration:
                                    baseUri: http://localhost:8090/odata/v4/employee/   # service root (note trailing /)
                                    username: "deep@gmail.com"                          # your ID
                                    password: "12345"                                   # your password
                                    # If 8090 runs HTTPS with self-signed certs in DEV:
                                    # baseUri: https://localhost:8090/odata/v4/employee/
                                    # strictSSL: false
                            
                            - name: ui5-middleware-  ----------> 2nd (It might work I didn't try)
                              afterMiddleware: compression
                              mountPath: /secureservice
                              configuration:
                                    baseUri: http://localhost:8090/odata/v4/employee/   # service root (note trailing /)
                                    username: "deep@gmail.com"                          # your ID
                                    password: "12345"                                   # your password
                                    # If 8090 runs HTTPS with self-signed certs in DEV:
                                    # baseUri: https://localhost:8090/odata/v4/employee/
                                    # strictSSL: false
                            

                    # --- Everything else stays on fiori-tools-proxy ---

5. configure your your manifest.json of the application.

            -----> first under your sap.app ------> under datasources ----> register your services defined in ui5.yaml

                  "sap.app": {
                        "dataSources": {
                            "mainService": {
                                "uri": "/api/",  --------> add the name defined in ui5.yaml /<service_name>/ (it could be path or mountPath (for auth))
                                "type": "OData",
                                "settings": {
                                "annotations": [],
                                "odataVersion": "4.01"
                                }
                            },
                            "secureservice": {
                                "uri": "/secureservice/",           -----------> add the name defined in ui5.yaml /<service_name>/
                                "type": "OData",
                                "settings": {
                                "annotations": [],
                                "odataVersion": "4.01"
                                }
                            },
                            "actionfuction": {
                                "uri": "/functiondd/",      -----------> add the name defined in ui5.yaml /<service_name>/
                                "type": "OData",
                                "settings": {
                                "annotations": [],
                                "odataVersion": "4.01"
                                }
                            },
                            "externalll": {
                                "uri": "/external/",        -----------> add the name defined in ui5.yaml /<service_name>/
                                "type": "OData",
                                "settings": {
                                "annotations": [],
                                "odataVersion": "4.01"
                                }
                            }
                        }

                    },

            ----> under sap.ui5 ---> under models ---> define your models to use it in your applicatioln

                    "sap.ui5": {
                        "models": {
                        "": {                       ---> your mail model name (anything you want to keep it)
                            "dataSource": "mainService",                      -------> add your services registered under sap.app -> datasource
                            "preload": true,
                            "settings": {
                            "operationMode": "Server",
                            "autoExpandSelect": true,
                            "earlyRequests": true
                            }
                        },
                        "secureservice":{
                            "dataSource": "secureservice",
                            "preload": true,
                            "settings": {
                            "operationMode": "Server",
                            "autoExpandSelect": true,
                            "earlyRequests": true
                            }
                        },
                        "actionfuction":{
                            "dataSource": "actionfuction",
                            "preload": true,
                            "settings": {
                            "operationMode": "Server",
                            "autoExpandSelect": true,
                            "earlyRequests": true
                            }
                        },
                        "externaltrio":{
                            "dataSource": "externalll",
                            "preload": true,
                            "settings": {
                            "operationMode": "Server",
                            "autoExpandSelect": true,
                            "earlyRequests": true
                            }
                        }
                    }
                }


6. after that do 
            
                npm i

7. npm run start