oData v4:  (use .then(async (result) => {}); to avoid any) backend error.
 
create->
		oNewEntry = <object>
		const oModel = this.getView().getModel();
		// /Insert_upskillsSRV --> service name
<CALL> oModel.bindList("/Insert_upskillsSRV").create(oNewEntry);
		--> to get the updated data
<CALL> const datacheck = await oModel.bindContext("/zupskills_empSRV").requestObject();
 
 
Read->
		--> to get all the data from the service
<CALL> const datacheck = await oModel.bindContext("/zupskills_empSRV").requestObject();
		--> to get only one data form the service of your data object.
<CALL> await oModel.bindContext("/zupskills_empSRV(id='1',<if any other key specified>)").requestObject()
 
updated->
		-->create the operation path with details
		const sPath = `/Update_upskillsSRV(id='${sId}',trainingid='${sTrainingId}')`;
		-->Bind the context
		const oContextBinding = oModel.bindContext(sPath);
		-->getting the context to add the updated property we will see below.
		const oContext = oContextBinding.getBoundContext();
		-->now like this below we need to set our property to the context with correct name in the db
		oContext.setProperty("trainingname", sap.ui.getCore().byId("trainingNameInput1").getValue());
		-->finally after this we need to do our update operation.
<CALL> oModel.submitBatch("$auto")
 
Delete->
		-->create the operation path with details
		const sPath = `/Delete_upskillsSRV(id='${sId}',trainingid='${sTrainingId}')`;
		-->Bind the context
		const oContextBinding = oModel.bindContext(sPath);
		-->getting the context to add the Delete property we will see below.
		const oContext = oContextBinding.getBoundContext();
		-->finally after this we need to do our Delete operation. but it should call after object fetched like below
		oContextBinding.requestObject().then(() => {
		oContext.delete("$auto")
		})
		===================================================================================
		there is a new way came 
		const oModel = this.getView().getModel();
		const sPath = oEvent.getSource().getBindingContext().sPath;
		await oModel.delete(sPath, "$auto", false); 


EXTRA->>>>>>>>>
 
		there is something extra ->   we can call this while data is refreshed
		oModel.refresh();
		
		
					--> for CRUD
				https://community.sap.com/t5/technology-blog-posts-by-members/sap-capm-full-stack-ui5-application-with-crud-operations/ba-p/13536813
				https://youtube.com/playlist?list=PL6aK0ufxLukLb0lSqxkR35PUDm4Z75ens&si=6WDqf101w2XdViGZ
				https://youtu.be/NcOpWjuIH6k?si=zp2h44iFJpN32nfR
				blogs:
				https://sapui5.hana.ondemand.com/#/api/sap.ui.model.odata.v4.Context%23methods
				https://community.sap.com/t5/technology-blog-posts-by-members/sap-capm-full-stack-ui5-application-with-crud-operations/ba-p/13536813
				
		
		
		
		
		
		
		
		================================================================================================
		                           For Deployment follow the below blog:
				https://community.sap.com/t5/technology-blog-posts-by-members/build-configure-and-deploy-sap-cap-projects-to-the-cloud/ba-p/14105562
				
			
				command:
				
				cds add hana --for production
				cds add hana --for hybrid
				cds add mta
				npm install
				<<<<<Adding a Managed Application Router (refer from the above blog link point no 7)>>>>>>
				<<<<<<<<<<after right click of MTA.yaml file
				<<<<<<<Right-click on the mta.yaml file and select Create MTA Module from Template.
				<<<<<<<<Choose the Approuter Configuration template.
				<<<<<<<<<Use a Managed Approuter.    (Indicate that a Fiori UI will be added later)
				cds add xsuaa --for production
				npm install
				<<<<<<<<<<<<<Right-click mta.yaml → Create MTA Module from Template.>>>>>>>>>>>>>>>>>>>>> now create the ui
				while creating the ui from the template add the following option on Cloud Foundry page
				Target: Cloud Foundry

						->Destination: Local CAP Project API

						->Semantic Object: bookshop

						->Action: show

						->Title: Books
						
				CTRL + Shift + P → CF: Login to Cloud Foundry
				<<<<<<<<Bind the HANA HDI Container -> Open the SAP HANA Projects panel in BAS. -> Bind to a HDI Container (follow blog , point no 10.2)
				<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<Build and Deploy the MTA Archive
				mbt build
				cf deploy mta_archives/<your mtar file>


			
			
								Authentication & Authorization -- Deployment.
								
					
					 
					https://youtu.be/nRew7KDwFj8?si=tlU4S0Bwe_ufRCbl --1
					https://youtu.be/Vigz5CIiQHU?si=ijyOWgLvAeRojVD_ -- 2
					https://youtu.be/GkgLLzKNszs?si=LzHGwOb7YQuY6eKi --3
					https://youtu.be/NxqE8pPzdZg?si=FlVkCs2lv9XtC5tZ -- 4
					 - YouTube
					Enjoy the videos and music you love, upload original content, and share it all with friends, family, and the world on YouTube.

					in package.json

					"cds": {
    "requires": {
      "[development]":{
        "auth":{
          "kind": "mocked",
          "users": {
            "deep@gmail.com":{
              "password": "12345",
              "roles": ["manager"]
            },
            "deep@sap.com":{
              "password": "12345",
              "roles": ["user"]
            }
          }
        }
      },
      "[production]": {
        "db": "hana",
        "auth": "xsuaa"
      },
      "[hybrid]": {
        "db": "hana"
      }
    }
  }







  CAP learning Playlist:

  https://www.youtube.com/@ABAPExplained/playlists
  https://www.youtube.com/@anybodycancode-b5v



	CAP learning 

	Please find the below link to access the training :
		https://learning.sap.com/learning-journeys/building-side-by-side-extensions-on-sap-btp


	Please find the below link to access the training :

	Building Side-by-Side Extensions on SAP BTP

	Please find the below link to access additional link:

	https://help.sap.com/docs/btp/sap-business-technology-platform/developing-with-sap-cloud-application-programming-model
	
	https://help.sap.com/docs/btp/btp-developers-guide/tutorials-for-sap-cloud-application-programming-model


	Best learning for CAP:

	https://www.youtube.com/playlist?list=PL3Q8uML_MhIg3x6lWB4KY_qaGQPaVFG_R

					 
 