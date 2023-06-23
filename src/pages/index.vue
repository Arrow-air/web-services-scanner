<template>
    <div class="container">
        <button v-if="location != null">
            <nuxt-link
                class="link"
                :to="{
                    path: 'scan',
                    query: {
                        longitude: location.coords.longitude,
                        latitude: location.coords.latitude,
                    },
                }"
                >Scan</nuxt-link
            >
        </button>
        <p v-else-if="gettingLocation">Getting location...</p>
        <p v-else-if="errorStr">Something went wrong: {{ errorStr }}</p>
    </div>
</template>

<script setup>
    import { onMounted, ref, onUnmounted } from "vue";

    const location = ref(null);
    const errorStr = ref("");
    const gettingLocation = ref(false);
    let watcher = null;

    onMounted(() => {
        if (!navigator.geolocation) {
            errorStr = "Geolocation is not available.";
            return;
        }
        gettingLocation.value = true;

        watcher = navigator.geolocation.watchPosition(
            (pos) => {
                location.value = pos;
                gettingLocation.value = false;
            },
            (err) => {
                errorStr.value = err.message;
                gettingLocation.value = false;
            }
        );
    });

    onUnmounted(() => {
        if (watcher) navigator.geolocation.clearWatch(watcher);
    });
</script>

<style scoped>
    .container {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
    }
    button {
        padding: 1rem 2rem;
        border-radius: 0.5rem;
        border: none;
        background-color: #f1f1f1;
        font-size: 1.5rem;
        font-weight: 600;
        cursor: pointer;
        transition: all 0.3s ease-in-out;
    }
    button:hover {
        background-color: #e1e1e1;
    }
    .link {
        text-decoration: none;
        color: #000;
    }
</style>
