<template>
    <div class="container">
        <div class="row">
            <div class="d-flex m-4">
                <form class="form-floating flex-grow-1">
                    <input class="form-control" type="text" placeholder="Enter a link to shorten" v-model="link">
                    <label>Shorten your link!</label>
                </form>
            </div>
        </div>
        <div class="row">
            <button  @click.prevent="shorten" class="btn btn-primary my-3">Shorten!</button>
            <div class="d-flex justify-content-center">
                <p v-if="error">Link you entered is invalid</p>
                <a v-if="shorten_link" :href="shorten_link" target="_blank" class="link-success">{{ shorten_link }}</a>
            </div>
        </div>
    </div>
    <div class="container mt-4">
        <h1>List of links shorten</h1>
        <div v-for="(short, long) in link_map">
            <p>{{ long }} -> <a :href="short" target="_blank">{{ short }}</a></p>
        </div>
    </div>
</template>

<script>
import { handleError } from 'vue';

export default {
    data() {
        return {
            token: import.meta.env.VITE_SHORTENER,
            link: '',
            shorten_link: '',
            error: false,
            link_map: {},
        }
    },

    methods: {
        async getGroupID() {
            // Turn out groupID is not needed for bit.ly API?
            let response = await fetch('https://api-ssl.bitly.com/v4/groups', {
                headers: {
                    'Authorization': 'Bearer ' + this.token,
                }
            })

            if (!response.ok) {
                console.log("Getting groupID error");
                return;
            }

            let json = await response.json();

            return json['groups'][0]['guid'];
        },

        async shorten() {
            if (!this.link) return;

            let response = await fetch('https://api-ssl.bitly.com/v4/shorten', {
                method: 'post',
                headers: {
                    'Authorization': 'Bearer ' + this.token,
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    'long_url': this.link,
                })
            })
            
            if (!response.ok) {
                this.error = true;
                return;
            }
            
            let json = await response.json();

            this.shorten_link = json['link'];
            this.error = false;
            
            console.log(this.link_map);
            // Add the url mapping to link_map, which will be set into local storage
            this.addNewLink(this.link, this.shorten_link);
        },
        
        addNewLink(url, shortenUrl) {
            this.link_map[url] = shortenUrl;
        }
    },
    
    created() {
        this.link_map = JSON.parse(localStorage.getItem('linkMap')) || {};
    },
    
    watch: {
        link_map: {
            handler(newVal, oldVal) {
                localStorage.setItem('linkMap', JSON.stringify(newVal));
            },
            deep: true,
        }
    }
}
</script>