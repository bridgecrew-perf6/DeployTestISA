<template>
    <div class="d-flex flex-row">
            <NavbarUser></NavbarUser>
        <div class="mx-auto my-4">
            <PersonalInfoUser v-if="user" :user="user"></PersonalInfoUser>
        </div>
    </div>
</template>

<script>
    import NavbarUser from '@/components/NavbarUser.vue'
    import PersonalInfoUser from '@/components/PersonalInfoUser.vue'
    import Vue from 'vue'
    import axios from 'axios'
    import VueAxios from 'vue-axios'
import router from '@/router'

    Vue.use(VueAxios, axios)

    export default {
        name: 'ProfilePageFishingInstructor',
        components: {
            NavbarUser,
            PersonalInfoUser
        },
        data() {
            return {
                user: "",
            }
        },
        mounted(){
            if (window.sessionStorage.getItem('role') === "ROLE_fishingInstructor") {
                axios.get("https://isa-projekat-tim-3.herokuapp.com/users/getLoggedUser", {
                    headers: {
                        Authorization: 'Bearer ' + window.sessionStorage.getItem("accessToken")
                    }
                }).then((response) =>{
                    this.user = response.data
                })
            }
            else {
                router.push("/");
            }
        }
    }
</script>