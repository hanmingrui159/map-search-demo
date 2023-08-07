<template>
    <q-layout class="set-width">

        <q-page-container>
            <q-page class="items-center">
                <div class="title text-h2">Map Search</div>

                <q-card-section>
                    <q-banner v-if="error" inline-actions class="text-white bg-red">
                        {{ error }}
                    </q-banner>


                    <div>
                    </div>
                </q-card-section>

                <!-- New Card -->

                <q-card class="my-card" flat bordered>
                    <q-card-section horizontal>
                        <q-card-section class="q-pt-xs">
                            <div class="text-h5 q-mt-sm q-mb-xs">Your location</div>
                            <!-- <div class="text-caption text-grey">
                                Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut
                                labore et dolore magna aliqua.
                            </div> -->
                            <div class="text-caption text-grey"> {{ myAddress }}</div>

                        </q-card-section>
                    </q-card-section>

                    <q-separator />

                    <q-card-actions>
                        <div class="q-my-md text-center">
                            <q-btn class="btn-find-location" label="Find My Location" @click="getLocation" />
                        </div>
                    </q-card-actions>
                </q-card>

                <form @submit.prevent="submitTask">
                    <q-input ref="autocompleteInput" outlined type="text" v-model="searchAddress"
                        placeholder="Enter your address:" class="q-my-sm" id="autocomplete">
                        <!-- @keydown.enter="submitTask" -->

                        <template v-slot:append>
                        </template>
                    </q-input>
                    <button class="btn btn-warning rounded-0" @click="submitTask">
                        SUBMIT
                    </button>
                </form>

                <!-- Time Zone + Local Time -->
                <q-card-section>
                    <q-card class="my-card">

                        <div class="text-subtitle1 centers">{{ timezone }} {{ localTime }}</div>

                    </q-card>
                </q-card-section>
                <!-- <p class="mt-3"></p> -->
                <!-- Google Map -->
                <!-- <q-card-section> -->
                <div id="map"></div>
                <!-- </q-card-section> -->




                <q-btn class="flex-center" color="white" text-color="black" label="Delete Selected"
                    @click="deleteSelected" />

                <!-- Button is now right before the table -->

                <q-table v-model:selected="selected" row-key="name" :rows-per-page-options="[10]" :rows="searchLogs"
                    selection="multiple">
                </q-table>

            </q-page>
        </q-page-container>
    </q-layout>
</template>


<script setup>
import { onMounted, computed, ref } from 'vue'
import axios from "axios";

const lat = ref('')
const lng = ref('')
const myAddress = ref('')
const searchAddress = ref('')
const error = ref('');

const geocoder = ref();

const timezone = ref('');
const localTime = ref('');
const autocomplete = ref();

// currentPage = 1
// perPage = 10

const selected = ref([])

let searchLogs = ref([
    {
        name: "Sample Search Log 1",
    },
    {
        name: "Sample Search Log 2",
    },
    {
        name: "Sample Search Log 3",
    },
    {
        name: "Sample Search Log 4",
    },
    {
        name: "Sample Search Log 5",
    },
    {
        name: "Sample Search Log 6",
    },
    {
        name: "Sample Search Log 7",
    },
    {
        name: "Sample Search Log 8",
    },
    {
        name: "Sample Search Log 9",
    },
    {
        name: "Sample Search Log 10",
    },
    {
        name: "Sample Search Log 11",
    },
    {
        name: "Sample Search Log 12",
    },
    {
        name: "Sample Search Log 13",
    },
    {
        name: "Sample Search Log 14",
    },
    {
        name: "Sample Search Log 15",
    },
    {
        name: "Sample Search Log 16",
    },
    {
        name: "Sample Search Log 17",
    }
])


const pagesNumber = computed(() => {
    return Math.ceil(this.rows.length / this.pagination.rowsPerPage)
})

const numberOfPages = computed(() => {
    return Math.ceil(this.searchLogs.length / this.perPage);
})

const paginatedData = computed(() => {
    const start = (this.currentPage - 1) * this.perPage;
    const end = start + this.perPage;
    return this.searchLogs.slice(start, end);
})

onMounted(() => {
    setInterval(updateTime, 1000);
})

// onMounted(() => {
//     autocomplete.value.addListener('place_changed', () => {
//         let place = autocomplete.value.getPlace();

//         if (place.geometry) {
//             // Place details fetched successfully. Update your data property.
//             searchAddress.value = place.formatted_address;
//         } else {
//             // No details fetched for the selected place. Clear your data property.
//             searchAddress.value = '';
//         }
//     });
// })


// onMounted(() => {
//     this.$nextTick(() => {
//         let input = this.$refs.autocompleteInput.$el.getElementsByTagName('input')[0];

//         new google.maps.places.Autocomplete(input);
//     });
// })

const deleteSelected = () => {
    console.log("triggered")
    console.log(selected)
    searchLogs.value = searchLogs.value.filter(row => !selected.value.includes(row))
    console.log("end")

    Object.assign(selected, ref([]))
}

const capitalizeFirstChar = (str) => {
    return str.charAt(0).toUpperCase() + str.slice(1);
}

/**
 * Add / Update task
 */
const submitTask = () => {
    if (searchAddress.value.length === 0) return;

    // geocode location and show on map
    myGeocode(searchAddress.value)

    /* We need to add new task */
    searchLogs.value.push({
        name: searchAddress.value,
        selected: false
    });

    searchAddress.value = "";

}

const getLocation = () => {
    if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
            (position) => {
                getAddressFrom(
                    position.coords.latitude,
                    position.coords.longitude
                );

                showUserLocationOnTheMap(
                    position.coords.latitude,
                    position.coords.longitude
                );
            },
            (error) => {
                console.log("An error happened!");
                console.log(error.message);
                error.value = error.message;
            }
        );
    } else {
        console.log("Your browser does not support geolocation API");
        error.value = "Your browser does not support geolocation API";
    }
}

const updateTime = () => {
    if (lat.value && lng.value) {
        getLocalTime(lat.value, lng.value)
    }
}

const getLocalTime = (lat, lng) => {
    axios.get(`https://maps.googleapis.com/maps/api/timezone/json?location=${lat},${lng}&timestamp=${Math.floor(Date.now() / 1000)}&key=AIzaSyB3pkQRsODKcVU2tSjlL2Yw5lrQicvReMY`)
        .then(result => {
            const timezoneData = result.data

            const date = new Date();
            const localT = date.getTime();
            const currentTimeZoneOffsetInMS = date.getTimezoneOffset() * 60000;

            const timeZoneOffset = timezoneData.dstOffset * 1000 + timezoneData.rawOffset * 1000;
            const localDateTime = new Date(localT + timeZoneOffset + currentTimeZoneOffsetInMS);

            timezone.value = result.data.timeZoneName
            localTime.value = localDateTime.toLocaleDateString(undefined, {
                year: '2-digit',
                month: '2-digit',
                day: '2-digit'
            }).replace(/(\d{2})\/(\d{2})\/(\d{2})/, "$3/$1/$2") + ' ' + localDateTime.toLocaleTimeString(undefined, {
                hour: '2-digit',
                minute: '2-digit',
                second: '2-digit',
                hour12: false
            });

            // alert(this.localTime)
            // alert(result.data.timeZoneName)

        })
        .catch((error) => {
            console.log(error.message);
            error.value = error.message;
        });
}

const getAddressFrom = (lat, long) => {
    axios
        .get(
            "https://maps.googleapis.com/maps/api/geocode/json?latlng=" +
            lat +
            "," +
            long +
            "&key=" +
            "AIzaSyB3pkQRsODKcVU2tSjlL2Yw5lrQicvReMY"
        )
        .then((response) => {
            if (response.data.error_message) {
                console.log(response.data.error_message);
                error.value = response.data.error_message;
            } else {
                console.log(response.data.results[0].formatted_address);
                myAddress.value = response.data.results[0].formatted_address;
            }
        })
        .catch((err) => {
            console.log(err.message);
            error.value = err.message;
        });
}

const showUserLocationOnTheMap = (latitude, longitude) => {
    // Create A Map Object
    console.log('showUserLocationOnTheMap invoked')
    console.log(google)
    let map = new google.maps.Map(document.getElementById("map"), {
        zoom: 12,
        center: new google.maps.LatLng(latitude, longitude),
        mapTypeId: google.maps.MapTypeId.ROADMAP,
    });
    // Add Marker
    new google.maps.Marker({
        position: new google.maps.LatLng(latitude, longitude),
        map: map,
        title: "You are here!",
    });
    console.log('showUserLocationOnTheMap finished')

}

const myGeocode = (query) => {
    geocoder.value = new google.maps.Geocoder();
    // geocode the search query, then show it on map and add to search logs
    geocoder.value.geocode({ address: query })
        .then((result) => {
            const { results } = result;
            // show on map
            lat.value = results[0].geometry.location.lat()
            lng.value = results[0].geometry.location.lng()
            showUserLocationOnTheMap(results[0].geometry.location.lat(), results[0].geometry.location.lng())
            getLocalTime(results[0].geometry.location.lat(), results[0].geometry.location.lng())
            // add to search history
            // this.searchLogs.push()
        })
        .catch((e) => {
            alert("Geocode was not successful for the following reason: " + e);
        });
}


</script>


<style scoped>
.pointer {
    cursor: pointer;
}

.noselect {
    -webkit-touch-callout: none;
    /* iOS Safari */
    -webkit-user-select: none;
    /* Safari */
    -khtml-user-select: none;
    /* Konqueror HTML */
    -moz-user-select: none;
    /* Old versions of Firefox */
    -ms-user-select: none;
    /* Internet Explorer/Edge */
    user-select: none;
    /* Non-prefixed version, currently
                                  supported by Chrome, Edge, Opera and Firefox */
}

.line-through {
    text-decoration: line-through;
}

#map {
    position: block;
    margin-bottom: 2rem;
    bottom: 0;
    background: white;
    z-index: -1;
    width: 100%;
    height: 15rem;
    text-align: center;
}

.title {
    margin-top: 2rem;
    font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif
}

.set-width {
    position: relative;
    left: 17%;
    width: 70%;
    text-align: center;
}

.btn-find-location {
    margin-left: 1rem;
}
</style>