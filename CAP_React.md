https://signatov.com/react-frontend-for-the-cap-application/

https://community.sap.com/t5/technology-blog-posts-by-members/deploy-a-react-app-to-sap-btp-cloud-foundry/ba-p/14177342

https://community.sap.com/t5/technology-blog-posts-by-members/how-to-build-end-to-end-custom-applications-in-cloud-foundry/ba-p/13485823


https://community.sap.com/t5/technology-blog-posts-by-members/cf-create-app-create-end-to-end-apps-and-deploy-to-cloud-foundry-in-minutes/ba-p/13470128


create a CAP application using the above article ( https://signatov.com/react-frontend-for-the-cap-application/ )



First add approuter in your application in your cap project root run the below command

        cds add approuter

        ---> it will create a router file in your app 



create a folder under router folder called public ( we will maintain our hosted cloud application under public folder by creating and moving the build of react application over under public/<yourappname>/)

under router folder you will found a file called xs-app.json

            {
            "source": "^(.*)$",
            "localDir": "public"
            }

        -----> add this for local directory access in public folder under router folder



To create react app move to app directory and run the below command

        npx create-react-app <your app name>

        ---> then go to your package.json and add this in the root

          "homepage": "/<your_app_folder_name>",   
          ( what ever you want to give just make sure under public folder you need to create same folder name in which you will move your application build )

Then build your application

        npm run build

        ---> it will create a build folder under the application repo

move the repo for local dev

        cp -rf build/* ../<your app name>

                        <<<<<<<<<OR>>>>>>>>>
for deployment move to

        cp -rf build/* ../router/public/<your application name>


Then as well as for default application loading you need to put these below lines under your app/router/xs-app.json

      "welcomeFile": "<your app name under public folder>/index.html",
      "authenticationMethod": "none",
