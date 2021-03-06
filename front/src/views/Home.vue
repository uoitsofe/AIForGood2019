<template>
    <v-app dark id="app-home">
        <v-toolbar color="primary" dark fixed app>
            <v-icon>public</v-icon>
            <v-toolbar-title>TrashSort</v-toolbar-title>
            <v-spacer></v-spacer>
            <v-toolbar-items>
                <v-btn flat fab primary @click="openDialogLeaderboard">
                    <v-icon>poll</v-icon>
                </v-btn>
            </v-toolbar-items>
        </v-toolbar>
        <v-content>
            <v-dialog v-model="confirm_dialog" persistent max-width="290">
                <v-card>
                    <v-card-title class="headline">What is it?</v-card-title>
                    <v-card-text>{{ guess_value }}</v-card-text>
                    <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn color="white" flat @click="closeSecondConfirm">Okay!</v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>
            <v-dialog v-model="dialog" fullscreen hide-overlay transition="dialog-bottom-transition">
                <v-card>
                    <v-toolbar dark color="primary">
                        <v-btn icon dark @click="closeDialog">
                            <v-icon>close</v-icon>
                        </v-btn>
                        <v-toolbar-title>Confirm Image</v-toolbar-title>
                        <v-spacer></v-spacer>
                        <v-toolbar-items>
                            <v-btn flat primary @click="clickConfirm">
                                Confirm
                            </v-btn>
                        </v-toolbar-items>
                    </v-toolbar>
                    <canvas id="camera--sensor"></canvas>
                    <div id="float-over">
                        <v-progress-circular
                                v-if="p"
                                :size="200"
                                :width="20"
                                color="primary"
                                indeterminate
                        ></v-progress-circular>
                    </div>
                </v-card>
            </v-dialog>
            <v-dialog v-model="dialog_leaderboard" fullscreen hide-overlay transition="dialog-bottom-transition">
                <v-card>
                    <v-toolbar dark color="primary">
                        <v-btn icon dark @click="closeDialogLeaderboard">
                            <v-icon>close</v-icon>
                        </v-btn>
                        <v-toolbar-title>Leaderboard</v-toolbar-title>
                    </v-toolbar>
                    <v-data-table v-model="selected" :headers="headers" :items="cities" select-all item-key="name" class="elevation-1" hide-actions>
                        <template v-slot:headers="props">
                            <tr>
                                <th v-for="header in props.headers" :key="header.text">
                                    {{ header.text }}
                                </th>
                            </tr>
                        </template>
                        <template v-slot:items="props">
                            <tr :active="props.selected" @click="props.selected = !props.selected">
                                <td>{{ props.item.name }}</td>
                                <td class="text-xs-right">{{ props.item.count }}</td>
                            </tr>
                        </template>
                    </v-data-table>
                </v-card>
            </v-dialog>
            <div id="camera">
                <video id="camera--view" autoplay playsinline></video>
                <button id="camera--trigger" @click="clickTrigger">Take a picture</button>
            </div>
        </v-content>
    </v-app>
</template>

<script>
    import axios from 'axios'

    export default {
        name: "Home",
        data: () => ({
            dialog: null,
            confirm_dialog: null,
            dialog_leaderboard: null,
            cameraView: null,
            cameraOutput: null,
            cameraSensor: null,
            cameraTrigger: null,
            p: null,
            guess_value: "",
            pagination: {
                sortBy: 'name'
            },
            selected: [],
            headers: [
                { text: 'City', align: 'left', value: 'name' },
                { text: 'Count', value: 'count' }
            ],
            cities: [
                {
                    name: 'Whitby',
                    count: 10,
                }
            ]
        }),
        mounted() {
            this.cameraView = document.querySelector("#camera--view")
            this.cameraSensor = document.querySelector("#camera--sensor")
            this.cameraTrigger = document.querySelector("#camera--trigger")
            window.addEventListener("load", () => {
                navigator.mediaDevices.getUserMedia({video: {facingMode: "environment"}}).then(mediaStream => { this.cameraView.srcObject = mediaStream; })
            }, false)
        },
        methods: {
            clickTrigger() {
                this.cameraSensor.width = this.cameraView.videoWidth
                this.cameraSensor.height = this.cameraView.videoHeight
                this.cameraSensor.getContext("2d").drawImage(this.cameraView, 0, 0)
                this.dialog = true
                this.p = false
            },
            closeDialog() {
                this.dialog = false
                this.p = false
            },
            openDialogLeaderboard() {
                this.dialog_leaderboard = true
            },
            closeDialogLeaderboard() {
                this.dialog_leaderboard = false
            },
            clickConfirm() {
                this.p = true
                let formData = new FormData()
                formData.set('image', this.cameraSensor.toDataURL("image/jpeg"))
                formData.set('can_id', 0)
                axios({
                    method: 'post',
                    url: 'http://localhost:5000/vision',
                    data: formData,
                    headers: {
                        'Content-Type': 'multipart/form-data'
                    }
                }).then((response) => {
                    this.confirm_dialog = true
                    this.guess_value = `I think this is ${response.data.trash_item}.`
                    this.dialog = false
                    this.p = false
                    console.log(response.data)
                }).catch((error) => {
                    this.dialog = true
                    this.p = false
                    console.log("ERROR IN HTTP REQUEST")
                    console.error(error)
                })
            },
            closeSecondConfirm() {
                this.confirm_dialog = false
                this.guess_value = ""
            }
        }
    }
</script>

<style>
    #camera, #camera--view, #camera--sensor{
        position: fixed;
        height: 100%;
        width: 100%;
        object-fit: cover;
    }

    #camera--view, #camera--sensor {
        /*transform: scaleX(-1);*/
    }

    #camera--trigger{
        width: 200px;
        background-color: black;
        color: white;
        font-size: 16px;
        border-radius: 30px;
        border: none;
        padding: 15px 20px;
        text-align: center;
        box-shadow: 0 0 10px 2px rgba(0,0,0,0.5);
        position: fixed;
        bottom: 30px;
        left: calc(50% - 100px);
    }

    #float-over {
        width: 200px;
        text-align: center;
        position: fixed;
        bottom: calc(50vh - 100px);
        left: calc(50vw - 100px);
    }
</style>
