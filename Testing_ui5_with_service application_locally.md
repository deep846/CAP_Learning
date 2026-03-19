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
                        - path: /api                         ----------------> 1st testing external service local
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


5. after that do 
            
                npm i

6. npm run start