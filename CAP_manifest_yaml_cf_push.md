https://stackoverflow.com/questions/42010661/add-parameters-for-service-in-cloud-foundry-manifest

https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html




to know about build packs run the command in the terminal below


cf buildpacks



update your manifest.json file

        ---
        version: 1
        applications:
        - name: my-microservice-app
        memory: 512M
        instances: 2
        buildpacks:
        - nodejs_buildpack
        random-route: true



then from the root of your project run the below command

            cf push
            cf push <perticular module from manifest.yaml file>