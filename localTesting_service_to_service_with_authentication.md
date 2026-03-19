1. create a file in the project root called 

    default-env.json

2. and create a destination

        {
            "destinations":[
            {
                "name": "empmanagementsrvdest",
                "url": "http://localhost:8090/odata/v4/employee",
                "username":"deep@gmail.com",     ----> mockuser id
                "password":"12345"              -----> mock password
            }
        ]
        }

3. then in the package.json create the service file

        	"cds": {
		"server": {
			"index": true
		},
		"requires": {
			"auth": {
				"kind": "dummy"
			},
			"[production]": {
				"db": "hana"
			},
            //// this highlited part/////////////



                                                                "empmanagement": {
                                                                    "kind": "odata",
                                                                    "model": "srv/external/empmanagement",
                                                                    "credentials": {
                                                                        "destination":"empmanagementsrvdest"
                                                                    }
                                                                }


            /////////////////////////////////////////
		}
	}
}


now you can test it locallly no need to deploy first. just develop first.
