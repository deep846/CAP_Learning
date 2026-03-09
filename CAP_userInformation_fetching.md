<!-- Everything will happen in backend -->



<!-- 1. IN Services declare your function -->
    function me() returns {};
<!-- 2. add the below methode in the releted JS files -->
    this.on('me',customlogic.getUserData);
<!-- 3. In the custom logic write the methode-->

    const getUserData = async function(req){
  
        const u = req.user;

        return {
            id: u.id,
            email: u.attr.email,
            roles: Object.keys(u.roles)
        };

        }