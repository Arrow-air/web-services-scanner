<template>
    <div v-if="location == null">
        <div class="container">
            <h2 class="fancy-title">
                You must enable location services to use this app.
            </h2>
            <button class="fancy-button">
                <nuxt-link to="/" class="fancy-link">Go Back</nuxt-link>
            </button>
        </div>
    </div>
    <div v-else>
        <div v-if="result != null">
            <div class="container">
                <div class="result">
                    <h1 class="fancy-title">Success!</h1>
                    <p class="fancy-text">Parcel Id: {{ result }}</p>
                    <p class="fancy-text">
                        Latitude: {{ location.coords.latitude }}
                    </p>
                    <p class="fancy-text">
                        Longitude: {{ location.coords.longitude }}
                    </p>
                    <div>
                        <p v-if="uploadError" class="error-text">
                            {{ uploadError }}
                        </p>
                        <button
                            class="fancy-button"
                            @click="uploadParcelData"
                            :disabled="uploading || uploaded"
                        >
                            <span v-if="uploading">Uploading...</span>
                            <span v-else-if="uploaded">Uploaded!</span>
                            <span v-else>Upload Parcel Data</span>
                        </button>
                    </div>
                    <button class="fancy-button" @click="result = null">
                        Next Package
                    </button>
                </div>
            </div>
        </div>
        <div class="fullscreen" v-else>
            <qrcode-stream
                :key="_uid"
                :track="track"
                @init="onInit"
                @decode="onDecode"
            >
                <div class="loading-indicator" v-if="loading">Loading...</div>
            </qrcode-stream>
        </div>
    </div>
</template>

<script>
    import { DEVICE_UUID } from "../data/data";

    export default {
        name: "Scan",
        data() {
            return {
                track: this.paintOutline,
                result: null,
                loading: false,
                location: null,
                uploading: false,
                uploaded: false,
                uploadError: null,
            };
        },
        mounted() {
            if (this.$route.query.longitude && this.$route.query.latitude) {
                this.location = {
                    coords: {
                        longitude: this.$route.query.longitude,
                        latitude: this.$route.query.latitude,
                    },
                };
            }
        },
        methods: {
            async uploadParcelData() {
                this.uploading = true;
                await fetch(
                    // TODO: replace with actual endpoint
                    "http://localhost:8002/cargo/scan",
                    {
                        method: "PUT",
                        headers: {
                            "Content-Type": "application/json",
                        },
                        body: JSON.stringify({
                            scanner_id: DEVICE_UUID,
                            parcel_id: String(this.result),
                            longitude: Number(this.location.coords.longitude),
                            latitude: Number(this.location.coords.latitude),
                        }),
                    }
                )
                    .then((response) => {
                        if (response.ok) {
                            console.log("success");
                            this.uploaded = true;
                        } else {
                            console.log("error");
                            this.uploadError = text;
                        }
                        this.uploading = false;
                    })
                    .catch((error) => {
                        this.uploading = false;
                        this.uploadError = error;
                        console.log(error);
                    });
            },

            onDecode(result) {
                this.result = result;
            },

            paintOutline(detectedCodes, ctx) {
                for (const detectedCode of detectedCodes) {
                    const [firstPoint, ...otherPoints] =
                        detectedCode.cornerPoints;

                    ctx.strokeStyle = "green";
                    ctx.lineWidth = 8;

                    ctx.beginPath();
                    ctx.moveTo(firstPoint.x, firstPoint.y);
                    for (const { x, y } of otherPoints) {
                        ctx.lineTo(x, y);
                    }
                    ctx.lineTo(firstPoint.x, firstPoint.y);
                    ctx.closePath();
                    ctx.stroke();
                }
            },

            async onInit(promise) {
                this.loading = true;

                try {
                    await promise;
                } catch (error) {
                    console.error(error);
                } finally {
                    this.loading = false;
                }
            },
        },
    };
</script>

<style scoped>
    .fancy-title {
        font-family: "Helvetica Neue", Arial, sans-serif;
        font-size: 2.5rem;
        font-weight: bold;
        color: #333333;
        margin-bottom: 1rem;
    }

    .fancy-text {
        font-family: "Helvetica Neue", Arial, sans-serif;
        font-size: 1.5rem;
        color: #555555;
    }

    .error-text {
        font-family: "Helvetica Neue", Arial, sans-serif;
        font-size: 1.5rem;
        color: #ff0000;
    }

    .fancy-button {
        margin-top: 1rem;
        margin-bottom: 1rem;
        width: 100%;
        padding: 1rem 2rem;
        border-radius: 0.5rem;
        border: none;
        background-color: #f1f1f1;
        font-family: "Helvetica Neue", Arial, sans-serif;
        font-size: 1.5rem;
        font-weight: bold;
        color: #333333;
        cursor: pointer;
        transition: all 0.3s ease-in-out;
    }

    .fancy-button:disabled {
        cursor: not-allowed;
        background-color: #787878;
    }

    .fancy-button:hover {
        background-color: #e1e1e1;
    }

    .fancy-link {
        text-decoration: none;
        color: #333333;
    }

    .container {
        margin: 0 auto;
        max-width: 960px;
        padding: 2rem;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .fullscreen {
        position: fixed;
        z-index: 1000;
        top: 0;
        bottom: 0;
        right: 0;
        left: 0;
    }

    .loading-indicator {
        font-family: "Helvetica Neue", Arial, sans-serif;
        font-size: 1.5rem;
        color: #555555;
        margin-top: 1rem;
        display: flex;
        justify-content: center;
        align-items: center;
    }
</style>
