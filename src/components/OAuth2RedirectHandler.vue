<template>
<div class="OAuth2RedirectHandler">
</div>
</template>

<script>
import {ACCESS_TOKEN} from "../constants/index.js";

export default { name: "o-auth2-redirect-handler",
mounted:function(){
   const token = this.getUrlParameter('token');
                //const error = this.getUrlParameter('error');
        
                if(token) {
                    localStorage.setItem(ACCESS_TOKEN, token);
                    this.$router.push("/dashboard/messages"); 
                } else {
                    this.$router.push("/error");
                }
            },

        methods:{
            getUrlParameter(name) {
        name = name.replace(/[[]/, '\\[').replace(/[\]]/, '\\]');
        var regex = new RegExp('[\\?&]' + name + '=([^&#]*)');

        var results = regex.exec(window.location.search);
        return results === null ? '' : decodeURIComponent(results[1].replace(/\+/g, ' '));
    }

        
}
}
</script>