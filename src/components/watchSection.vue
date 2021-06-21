<template>
    <!-- ======= About Section ======= -->
    <div>
    <section id="about" class="about">
        <div style="background-color: rgba(255,255,255,0.8)" class="container" data-aos="fade-up">
            <div class="row p-5">
                <div class="col-lg-6 video-box align-self-baseline" data-aos="zoom-in" data-aos-delay="0">
                    <div class="ratio ratio-16x9">
                        <video id="myVideo"
                               class="video-js vjs-default-skin vjs-big-play-centered"
                               title="Test video"
                               preload="metadata"
                               :poster="videoInfo.poster"
                               controls
                        ></video>
                    </div>
                </div>
                <div class="col-lg-6 p-3 pt-lg-0 content">
                    <h3>{{videoInfo.title}}</h3>
                    <p class="font-italic">
                        {{videoInfo.description}}<br><br>
                        server: {{host}}
                    </p>
                </div>
            </div>
            <div class="row justify-content-end">
                <div class="col-lg-11">
                    <div class="row justify-content-end">

                        <div class="col-lg-3 col-md-5 col-6 d-md-flex align-items-md-stretch">
                            <div class="count-box">
                                <span data-toggle="counter-up">65</span>
                                <p>Happy Clients</p>
                            </div>
                        </div>

                        <div class="col-lg-3 col-md-5 col-6 d-md-flex align-items-md-stretch">
                            <div class="count-box">
                                <span data-toggle="counter-up">85</span>
                                <p>Projects</p>
                            </div>
                        </div>

                        <div class="col-lg-3 col-md-5 col-6 d-md-flex align-items-md-stretch">
                            <div class="count-box">
                                <span data-toggle="counter-up">12</span>
                                <p>Years of experience</p>
                            </div>
                        </div>

                        <div class="col-lg-3 col-md-5 col-6 d-md-flex align-items-md-stretch">
                            <div class="count-box">
                                <span data-toggle="counter-up">16</span>
                                <p>Awards</p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section><!-- End About Section -->
    </div>
</template>

<script>
    import videojs from "video.js";
    import 'dashjs' ;
    import 'videojs-contrib-dash';
    import 'video.js/dist/video-js.css';
    import axios from "axios";

    let lastLearnTime = null;
    export default {
        name: "playerSection",
        props:["videoInfo"],
        mounted() {
            this.player();
        },
        activated() {
            this.player();
        },
        data(){
            return{
                host: "172.23.187.202"
            }
        },
        watch: {
            videoInfo: function () {
                // this.getVideoInfo();
            },
        },
        methods:{
            player(){
                let flag=0;
                let player=videojs('myVideo');
                let videoUrl = "http://" + this.host + ":1935" + this.videoInfo.Url;
                let that = this;
                
                player.ready(function(){
                    player.src({
                        src:videoUrl,
                        type:"application/dash+xml"
                    })

                    player.play();

                    player.on('loadstart', function() {
                        console.log('loadstart')
                    });

                    player.on('loadedmetadata', function() {
                        console.log('loadedmetadata-載入完成')              
                        player.currentTime(lastLearnTime);
                        console.log(`現在時間${player.currentTime()}`)
                    });

                    player.on('loadeddata', function() {
                        console.log('loadeddata-渲染畫面'); 
                    });

                    player.on('error', function() {
                        console.log('error');
                    });

                    player.on('progress', function() {
                        console.log('progress');
                        console.log(videoUrl);

                        player.on('waiting', function() {
                            console.log('waiting');
                            flag++;
                            console.log(flag);
                            if(flag<5){
                                setTimeout(() => {
                                    that.getVideoInfo();
                                }, 1000);
                            }
                        });
                    });
                
                    let totalTime = 0;
                    player.on('timeupdate', function() {
                        let currentTime = Math.floor(player.currentTime());
                        if (currentTime > 0 && currentTime > totalTime && (currentTime % 5 === 0)) {
                            lastLearnTime=currentTime;                           
                        }
                        totalTime = currentTime;
                    });
                
                    player.on('ended', function() {
                        console.log('播放結束');
                    });
                })
            },
            getVideoInfo(){
                console.log("redirect");

                axios({
                    method: 'get',
                    url: '/api/media-host'
                })
                    .then((response) => {
                        this.host = response.data.host;
                    }).then(
                        this.player()
                )
            }
        }
    }

</script>

<style scoped>
.video-js{
    position: absolute!important;
}
</style>