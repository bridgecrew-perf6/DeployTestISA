<template>
    <div>
        <NavbarUser v-if="isOwner"></NavbarUser>
        <NavbarClient v-else-if="isClient"></NavbarClient>
        <NavbarAdmin v-else-if="isAdmin"></NavbarAdmin>
        <NavbarGuest v-else></NavbarGuest>
        <div v-if="ship" style="margin: 100px">
            <div class="d-flex flex-row" style="margin: 50px">
                <div class="d-flex flex-column" style="width: 50%">
                    <div class="slideshow-container">
                        <div class="mySlides" style="display: block;">
                            <div class="numbertext">{{this.currentPicture + 1}} / {{ship.pictures.length}}</div>
                            <img :src="require(`../assets/${this.ship.pictures[this.currentPicture]}`)" style="width:100%; height:400px; border-radius:25px">
                        </div>
                        <a class="prev" @click="changePicture(-1)">&#10094;</a>
                        <a class="next" @click="changePicture(1)">&#10095;</a>
                    </div>
                    <div class="d-flex flex-column" style="border-radius: 25px; margin: 5px; border: 1px solid #323539">
                        <div>
                            <h5 style="margin: 5px">Additional services</h5>
                            <ul>
                                <template v-for="service in ship.additionalServices">
                                    <li style="margin: 5px" :key="service">{{service}}</li>
                                </template>
                            </ul>
                        </div>
                        <div>
                            <h5 style="margin: 5px">Rules of conduct</h5>
                            <ul>
                                <template v-for="rule in ship.rulesOfConduct">
                                    <li style="margin: 5px" :key="rule">{{rule}}</li>
                                </template>
                            </ul>
                        </div>
                        <div>
                            <h5 style="margin: 5px">Fishing equipment</h5>
                            <ul>
                                <template v-for="equipment in ship.fishingEquipment">
                                    <li style="margin: 5px" :key="equipment">{{equipment}}</li>
                                </template>
                            </ul>
                        </div>
                        <div>
                            <h5 style="margin: 5px">Navigation equipment</h5>
                            <ul>
                                <template v-for="equipment in ship.navigationEquipment">
                                    <li style="margin: 5px" :key="equipment">{{equipment}}</li>
                                </template>
                            </ul>
                        </div>
                    </div>
                </div>
                <div class="d-flex flex-column" style="width: 50%; margin: 5px;">
                    <div style="height: 10%; margin: 5px; align-self: center;">
                        <h1>
                            <span>{{ship.name}}</span>
                        </h1>
                    </div>
                    <div style="height: 80%; margin: 5px; border: 1px solid #323539; border-radius: 25px; ">
                        <div>
                            <h5 style="margin: 5px">Description</h5>
                            <p style="margin: 5px">{{ship.description}}</p>
                        </div>
                        <div class="d-flex flex-row">
                            <h5 style="margin: 5px">Type: {{ship.type}}</h5>
                        </div>
                        <div class="d-flex flex-row">
                            <h5 style="margin: 5px">Length: {{ship.length}}</h5>
                        </div>
                        <div class="d-flex flex-row">
                            <h5 style="margin: 5px">Engine number: {{ship.engineNum}}</h5>
                        </div>
                        <div class="d-flex flex-row">
                            <h5 style="margin: 5px">Engine power: {{ship.enginePower}} kW</h5>
                        </div>
                        <div class="d-flex flex-row">
                            <h5 style="margin: 5px">Max speed: {{ship.maxSpeed}}</h5>
                        </div>
                        <div class="d-flex flex-row">
                            <h5 style="margin: 5px">Capacity: {{ship.capacity}}</h5>
                        </div>
                        <div class="d-flex flex-row">
                            <h5 style="margin: 5px">Price: {{ship.price}} €</h5>
                        </div>
                        <div>
                            <h5 style="margin: 5px">Reservation cancellation conditions</h5>
                            <p style="margin: 5px">{{ship.reservationCancellationConditions}}</p>
                        </div>
                        <div>
                            <h5 style="margin: 5px">Address</h5>
                            <p style="margin: 5px">{{ship.country}}, {{ship.city}}, {{ship.street}}</p>
                        </div>
                        <div v-if="ship.averageRating > 0">
                            <h5 style="margin: 5px">Average rating</h5>
                            <p style="margin: 5px"><StarRating :show-rating="false" :increment="0.01" :star-size="24" :inline="true" 
                                :rating="ship.averageRating" :read-only="true"></StarRating> {{ship.averageRating}}/5 </p>
                        </div>
                        <div v-else>
                            <h5 style="margin: 5px">Average rating</h5>
                            <p style="margin: 5px">Ship has no rating yet.</p>
                        </div>
                        <div>
                            <iframe :src="mapSrc" style="margin: 15px; border-radius: 25px; border: 1px solid #323539"></iframe>
                        </div>
                    </div>
                    <div v-if="isClient || isOwner" class="d-flex flex-row" style="height: 10%; margin: 5px; border: 1px solid #323539">
                        <div v-if="isClient" class="d-flex flex-column" style="margin: 10px; align-self: left;">
                            <button @click="subscribe" v-if="isClient && !isSubscribed" class="btn btn-primary" value="Subscribe">Subscribe</button>
                            <button @click="unsubscribe" v-if="isClient && isSubscribed" class="btn btn-primary" value="Unsubscribe">Unsubscribe</button>
                        </div>
                        <div v-if="isClient" class="d-flex flex-column" style="margin: 10px; align-self: right;">
                            <button @click="goToActions" class="btn btn-primary" value="Actions">See actions</button>
                        </div>
                        <div style="margin: 10px; align-self: center;" v-if="isOwner">
                            <button class="btn btn-primary" @click="reserveForClient" value="Reserve">Reserve for client</button>
                        </div>
                        <div style="margin: 10px; align-self: center;" v-if="isOwner">
                            <button class="btn btn-primary" @click="showCalendar" value="Show calendar">Show calendar for ship</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import Vue from 'vue'
    import axios from 'axios'
    import VueAxios from 'vue-axios'
    import StarRating from 'vue-star-rating'
    import NavbarClient from '@/components/NavbarClient.vue'
    import NavbarGuest from '@/components/NavbarGuest.vue'
    import NavbarUser from '@/components/NavbarUser.vue'
    import NavbarAdmin from '@/components/NavbarAdmin.vue'

    Vue.use(VueAxios, axios)

    export default {
        name: 'ShipView',
        components: {
            StarRating,
            NavbarGuest,
            NavbarClient,
            NavbarUser,
            NavbarAdmin
        },
        data() {
            return {
                ship: null,
				currentPicture: 0,
				mapSrc: "",
                client: null,
            }
        },
        computed: {
            isClient() {
                return window.sessionStorage.getItem("role") === "ROLE_client";
            },
            isOwner() {
                return window.sessionStorage.getItem("role") === "ROLE_shipOwner";
            },
            isSubscribed() {
                if (this.client != null) {
                    return this.client.subscriptions.includes(parseInt(this.$route.params.id));
                }
                return false;
            },
            isAdmin() {
                return window.sessionStorage.getItem('role') === "ROLE_admin" || window.sessionStorage.getItem('role') === "ROLE_mainAdmin";
            }
        },
        methods: {
			changePicture(n) {
				if (this.currentPicture + n < 0) {
					this.currentPicture = this.ship.pictures.length - 1;
				}else if (this.currentPicture + n > this.ship.pictures.length - 1) {
					this.currentPicture = 0;
				}else {
					this.currentPicture += n;
				}
			},
            subscribe() {
                axios.put("https://isa-projekat-tim-3.herokuapp.com/clients/subscribe/" + this.$route.params.id, {
                },
                {
                    headers:{
                        Authorization: 'Bearer ' + window.sessionStorage.getItem("accessToken")
                    }
                }).then((response) => {
                    alert("You have subscribed.")
                    this.client = response.data
                })
            },
            unsubscribe() {
                axios.put("https://isa-projekat-tim-3.herokuapp.com/clients/unsubscribe/" + this.$route.params.id, {
                },
                {
                    headers:{
                        Authorization: 'Bearer ' + window.sessionStorage.getItem("accessToken")
                    }
                }).then((response) => {
                    alert("You have unsubscribed.")
                    this.client = response.data
                })
            },
            goToActions() {
                this.$router.push("/fast-reservations/" + this.$route.params.id)
            },
            reserveForClient() {
                this.$router.push("/owner-reserve/" + this.$route.params.id);
            },
            showCalendar() {
                this.$router.push("/service-calendar/" + this.$route.params.id);
            }
        },
        mounted() {
            axios.get("https://isa-projekat-tim-3.herokuapp.com/ships/get/" + this.$route.params.id, 
				{
					headers: {
						Authorization: 'Bearer ' + window.sessionStorage.getItem("accessToken")
					}
				}).then((response) => {
					this.ship = response.data;
					this.mapSrc = "https://maps.google.com/maps?q=" + response.data.country + "," + response.data.city + "," + response.data.street + "&t=&z=13&ie=UTF8&iwloc=&output=embed"
				}
			);
            if (window.sessionStorage.getItem("role") === "ROLE_client") {
                axios.get("https://isa-projekat-tim-3.herokuapp.com/clients/getLoggedClient", {
                    headers:{
                        Authorization: 'Bearer ' + window.sessionStorage.getItem("accessToken")
                    }
                }).then((response) => {
                    this.client = response.data
                })
            }
        },
    }
</script>