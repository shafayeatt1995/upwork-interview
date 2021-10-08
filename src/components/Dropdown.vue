<template>
<div class="container" v-if="error.status">
    <h2 class="error my-15">{{error.message}}</h2>
    <button class="btn" type="button" @click="newToken">Fix & Retry</button>
</div>
<div class="container" v-else>
    <transition name="fade" mode="out-in">
        <div class="sk-circle" v-if="loading" key="1">
            <div class="sk-circle1 sk-child"></div>
            <div class="sk-circle2 sk-child"></div>
            <div class="sk-circle3 sk-child"></div>
            <div class="sk-circle4 sk-child"></div>
            <div class="sk-circle5 sk-child"></div>
            <div class="sk-circle6 sk-child"></div>
            <div class="sk-circle7 sk-child"></div>
            <div class="sk-circle8 sk-child"></div>
            <div class="sk-circle9 sk-child"></div>
            <div class="sk-circle10 sk-child"></div>
            <div class="sk-circle11 sk-child"></div>
            <div class="sk-circle12 sk-child"></div>
        </div>
        <div key="2" v-else>
            <div class="country my-15">
                <label for="country">Select a Country</label>
                <select v-model="form.country" @change="changeCountry(form.country)" id="country">
                    <option value="">Select a country</option>
                    <option v-for="(country, key) in countryList" :value="country.country_name" :key="key">{{country.country_name}}</option>
                </select>
            </div>

            <transition name="fade" mode="out-in">
                <div class="spinner" v-if="stateLoading" key="1">
                    <div class="bounce1"></div>
                    <div class="bounce2"></div>
                    <div class="bounce3"></div>
                </div>
                <div class="state my-15" key="2" v-else>
                    <label for="state">Select a State</label>
                    <select v-model="form.state" @change="changeState(form.state)" id="state">
                        <option value="">Select a state</option>
                        <option v-for="(state, key) in stateList" :value="state.state_name" :key="key">{{state.state_name}}</option>
                    </select>
                </div>
            </transition>
            <transition name="fade" mode="out-in">
                <div class="spinner" v-if="cityLoading" key="1">
                    <div class="bounce1"></div>
                    <div class="bounce2"></div>
                    <div class="bounce3"></div>
                </div>
                <div class="city my-15" v-else>
                    <label for="country">Select Cities <span class="small">(For multiple select hold CTRL and Click)</span></label>
                    <select v-model="form.city" multiple>
                        <option v-for="(city, key) in cityList" :value="city.city_name" :key="key">{{city.city_name}}</option>
                    </select>
                    <div class="select-btn my-15">
                        <button class="btn" type="button" @click="selectAllCity">Select all city</button>
                        <button class="btn" type="button" @click="form.city = []">Deselect all city</button>
                    </div>
                    <h3>Total Selected City {{form.city.length}}</h3>
                </div>
            </transition>
        </div>
    </transition>
</div>
</template>

<script>
import axios from 'axios'
export default {
    data() {
        return {
            loading: true,
            stateLoading: false,
            cityLoading: false,
            error: {
                status: false,
                message: "",
            },
            countryList: [],
            stateList: [],
            cityList: [],
            form: {
                country: "",
                state: "",
                city: [],
            }
        }
    },

    methods: {
        async getData() {
            let find = localStorage.getItem('token');
            if (find === null) {
                // Api Required Header Info
                let config = {
                    headers: {
                        "Accept": "application/json",
                        "api-token": "IioWNjZZz8WEbUNlIlwQjmLQSWX7R9ryx5uVBeG1qoVy2bPlMol5rfTfdhni-VWkLDo",
                        "user-email": "shafayetalanik@gmail.com"
                    }
                };

                // Get Authorization Token
                await axios.get("https://www.universal-tutorial.com/api/getaccesstoken", config).then(
                    (response) => {
                        // Set Token Local Storage
                        localStorage.setItem('token', response.data.auth_token);

                        // Get Country List
                        let getDataConfig = {
                            headers: {
                                "Accept": "application/json",
                                "Authorization": `Bearer ${response.data.auth_token}`,
                            }
                        }
                        axios.get("https://www.universal-tutorial.com/api/countries/", getDataConfig).then(
                            (res) => {
                                this.countryList = res.data
                                this.loading = false;
                            },
                            (e) => {
                                this.error.status = true;
                                this.error.message = e.response.data.error.name;
                                this.loading = false;
                            }
                        )
                    },
                    (e) => {
                        this.error.status = true;
                        this.error.message = e.response.data.error;
                        this.loading = false;
                    }
                )
            } else {
                // Get Country List
                let getDataConfig = {
                    headers: {
                        "Accept": "application/json",
                        "Authorization": `Bearer ${find}`,
                    }
                }
                axios.get("https://www.universal-tutorial.com/api/countries/", getDataConfig).then(
                    (res) => {
                        this.countryList = res.data
                        this.loading = false;
                    },
                    (e) => {
                        this.error.status = true;
                        this.error.message = e.response.data.error.name;
                        this.loading = false;
                    }
                )
            }
        },

        // Change country & load new states
        changeCountry(country) {
            this.stateLoading = true;
            this.cityLoading = true;
            this.cityList = [];
            this.form.state = "";
            this.form.city = [];
            let getDataConfig = {
                headers: {
                    "Accept": "application/json",
                    "Authorization": `Bearer ${localStorage.getItem('token')}`,
                }
            }
            axios.get(`https://www.universal-tutorial.com/api/states/${country}`, getDataConfig).then(
                (res) => {
                    this.stateList = res.data
                    this.stateLoading = false;
                    this.cityLoading = false;
                },
                () => {
                    this.stateLoading = false;
                    this.cityLoading = false;
                },
            )
        },

        // Change state & load new cities
        changeState(state) {
            this.cityLoading = true;
            this.cityList = [];
            this.form.city = [];
            let getDataConfig = {
                headers: {
                    "Accept": "application/json",
                    "Authorization": `Bearer ${localStorage.getItem('token')}`,
                }
            }
            axios.get(`https://www.universal-tutorial.com/api/cities/${state}`, getDataConfig).then(
                (res) => {
                    this.cityList = res.data
                    this.cityLoading = false;
                },
                () => {
                    this.cityLoading = false;
                },
            )
        },

        // Select All City
        selectAllCity() {
            this.cityList.forEach((city) => { this.form.city.push(city.city_name) })
        },

        newToken() {
            this.error.status = false
            this.loading = true;
            // Api Required Header Info
            let config = {
                headers: {
                    "Accept": "application/json",
                    "api-token": "IioWNjZZz8WEbUNlIlwQjmLQSWX7R9ryx5uVBeG1qoVy2bPlMol5rfTfdhni-VWkLDo",
                    "user-email": "shafayetalanik@gmail.com"
                }
            };

            // Get Authorization Token
            axios.get("https://www.universal-tutorial.com/api/getaccesstoken", config).then(
                (response) => {
                    // Set Token Local Storage
                    localStorage.setItem('token', response.data.auth_token);
                    this.$emit('getAppData')
                },
                (e) => {
                    this.error.status = true;
                    this.error.message = e.response.data.error;
                    this.loading = false;
                }
            )
        },
    },

    created() {
        this.getData();
        this.$on("getAppData", () => {
            this.getData();
        })
    },
}
</script>

<style>
* {
    margin: 0;
    padding: 0;
}

.container {
    width: 550px;
    margin: 0 auto;
}

.my-15 {
    margin-top: 15px;
    margin-bottom: 15px;
}

select {
    background-color: #41B883;
    border-radius: 2px;
    border: none;
    color: #35495E;
    padding: 10px 30px 10px 10px;
    outline: none;
    width: 100%;
    font-weight: bold;
    font-size: 20px;
}

label {
    display: block;
    font-size: 25px;
}

h3 {
    text-align: center;
}

.btn {
    outline: none;
    border: none;
    background: #41B883;
    color: #35495E;
    padding: 15px 30px;
    font-size: 18px;
    font-weight: bold;
    cursor: pointer;
}

.select-btn {
    display: flex;
    justify-content: space-around;
}

.error {
    color: #f44;
}

.city select {
    height: 150px;
}

.small {
    font-size: 80%;
}

.strong {
    font-weight: bold;
}

.fade-enter-active,
.fade-leave-active {
    transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
    opacity: 0;
}

.sk-circle {
    margin: 150px auto;
    width: 100px;
    height: 100px;
    position: relative;
}

.sk-circle .sk-child {
    width: 100%;
    height: 100%;
    position: absolute;
    left: 0;
    top: 0;
}

.sk-circle .sk-child:before {
    content: '';
    display: block;
    margin: 0 auto;
    width: 15%;
    height: 15%;
    background-color: #35495E;
    border-radius: 100%;
    -webkit-animation: sk-circleBounceDelay 1.2s infinite ease-in-out both;
    animation: sk-circleBounceDelay 1.2s infinite ease-in-out both;
}

.sk-circle .sk-circle2 {
    -webkit-transform: rotate(30deg);
    -ms-transform: rotate(30deg);
    transform: rotate(30deg);
}

.sk-circle .sk-circle3 {
    -webkit-transform: rotate(60deg);
    -ms-transform: rotate(60deg);
    transform: rotate(60deg);
}

.sk-circle .sk-circle4 {
    -webkit-transform: rotate(90deg);
    -ms-transform: rotate(90deg);
    transform: rotate(90deg);
}

.sk-circle .sk-circle5 {
    -webkit-transform: rotate(120deg);
    -ms-transform: rotate(120deg);
    transform: rotate(120deg);
}

.sk-circle .sk-circle6 {
    -webkit-transform: rotate(150deg);
    -ms-transform: rotate(150deg);
    transform: rotate(150deg);
}

.sk-circle .sk-circle7 {
    -webkit-transform: rotate(180deg);
    -ms-transform: rotate(180deg);
    transform: rotate(180deg);
}

.sk-circle .sk-circle8 {
    -webkit-transform: rotate(210deg);
    -ms-transform: rotate(210deg);
    transform: rotate(210deg);
}

.sk-circle .sk-circle9 {
    -webkit-transform: rotate(240deg);
    -ms-transform: rotate(240deg);
    transform: rotate(240deg);
}

.sk-circle .sk-circle10 {
    -webkit-transform: rotate(270deg);
    -ms-transform: rotate(270deg);
    transform: rotate(270deg);
}

.sk-circle .sk-circle11 {
    -webkit-transform: rotate(300deg);
    -ms-transform: rotate(300deg);
    transform: rotate(300deg);
}

.sk-circle .sk-circle12 {
    -webkit-transform: rotate(330deg);
    -ms-transform: rotate(330deg);
    transform: rotate(330deg);
}

.sk-circle .sk-circle2:before {
    -webkit-animation-delay: -1.1s;
    animation-delay: -1.1s;
}

.sk-circle .sk-circle3:before {
    -webkit-animation-delay: -1s;
    animation-delay: -1s;
}

.sk-circle .sk-circle4:before {
    -webkit-animation-delay: -0.9s;
    animation-delay: -0.9s;
}

.sk-circle .sk-circle5:before {
    -webkit-animation-delay: -0.8s;
    animation-delay: -0.8s;
}

.sk-circle .sk-circle6:before {
    -webkit-animation-delay: -0.7s;
    animation-delay: -0.7s;
}

.sk-circle .sk-circle7:before {
    -webkit-animation-delay: -0.6s;
    animation-delay: -0.6s;
}

.sk-circle .sk-circle8:before {
    -webkit-animation-delay: -0.5s;
    animation-delay: -0.5s;
}

.sk-circle .sk-circle9:before {
    -webkit-animation-delay: -0.4s;
    animation-delay: -0.4s;
}

.sk-circle .sk-circle10:before {
    -webkit-animation-delay: -0.3s;
    animation-delay: -0.3s;
}

.sk-circle .sk-circle11:before {
    -webkit-animation-delay: -0.2s;
    animation-delay: -0.2s;
}

.sk-circle .sk-circle12:before {
    -webkit-animation-delay: -0.1s;
    animation-delay: -0.1s;
}

@-webkit-keyframes sk-circleBounceDelay {

    0%,
    80%,
    100% {
        -webkit-transform: scale(0);
        transform: scale(0);
    }

    40% {
        -webkit-transform: scale(1);
        transform: scale(1);
    }
}

@keyframes sk-circleBounceDelay {

    0%,
    80%,
    100% {
        -webkit-transform: scale(0);
        transform: scale(0);
    }

    40% {
        -webkit-transform: scale(1);
        transform: scale(1);
    }
}

.spinner {
    margin: 22px auto;
    width: 70px;
    text-align: center;
}

.spinner>div {
    width: 18px;
    height: 18px;
    background-color: #35495E;

    border-radius: 100%;
    display: inline-block;
    -webkit-animation: sk-bouncedelay 1.4s infinite ease-in-out both;
    animation: sk-bouncedelay 1.4s infinite ease-in-out both;
}

.spinner .bounce1 {
    -webkit-animation-delay: -0.32s;
    animation-delay: -0.32s;
}

.spinner .bounce2 {
    -webkit-animation-delay: -0.16s;
    animation-delay: -0.16s;
}

@-webkit-keyframes sk-bouncedelay {

    0%,
    80%,
    100% {
        -webkit-transform: scale(0)
    }

    40% {
        -webkit-transform: scale(1.0)
    }
}

@keyframes sk-bouncedelay {

    0%,
    80%,
    100% {
        -webkit-transform: scale(0);
        transform: scale(0);
    }

    40% {
        -webkit-transform: scale(1.0);
        transform: scale(1.0);
    }
}
</style>
