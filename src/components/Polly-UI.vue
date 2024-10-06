<template>
    <img src="../assets/polly_logo.png">
    <p>Text to speak:</p>
    <textarea v-model="textToSpeech" rows="7" placeholder="Enter text..."></textarea><br>
    <button class="speakBtn" @click="synthesizeSpeech"><b>Speak</b></button><br><br>
     <audio ref="audioPlayer" controls></audio>
</template>

<script>
import AWS from 'aws-sdk';
export default {
    name: 'PollyUI',
    data() {
        return {
            textToSpeech: ''
        }
    },
    mounted() {
        AWS.config.region = 'REGION'; // enter region of Identity Pool
        AWS.config.credentials = new AWS.CognitoIdentityCredentials({
            IdentityPoolId: 'IDENTITY_POOL_ID' // enter Identity Pool ID
        });
        AWS.config.credentials.get((err) => {
            if (err) {
                console.error('Error retreiving Cognito credentials:', err);
            } else {
                console.log('Cognito credentials retrieved successfully.');
            }
        });
    },
    methods: {
        synthesizeSpeech() {
            if (this.textToSpeech.trim() === '') {
                alert('Please enter some text.');
                return;
            }
            // initialize Polly API
            const polly = new AWS.Polly();
            const params = {
                Text: this.textToSpeech,
                OutputFormat: 'mp3',
                VoiceId: 'Ruth',
                Engine: 'long-form'
            };
            // Polly API call
            polly.synthesizeSpeech(params, (err, data) => {
                if (err) {
                    console.error('Error calling Polly:', err);
                    alert('Error with Polly, check console.');
                } else {
                    const audioBlob = new Blob([data.AudioStream], { type: 'audio/mp3'});
                    const audioUrl = URL.createObjectURL(audioBlob);
                    this.playAudio(audioUrl);
                } 
            });
        },
        playAudio(audioUrl) {
            const audioPlayer = this.$refs.audioPlayer;
            audioPlayer.src = audioUrl;
            audioPlayer.play();
        }
    }
}
</script>
<style>
img {
    width: 250px;
    height: 250px;
}
p {
    padding-right: 300px;
    margin: auto;
}
textarea {
    width:400px;
    border: 1px solid;
}
.speakBtn {
    width: 410px;
    color: white;
    background-color: #527fff;
    cursor: pointer;
}
</style>