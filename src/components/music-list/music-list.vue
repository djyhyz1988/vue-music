<template>
    <div class="music-list">
        <div class="back" @click="back">
            <i class="icon-back"></i>
        </div>
        <h1 class="title" v-html="title"></h1>
        <div class="bg-image" :style="bgStyle" ref="bgImage">
            <div class="play-wrapper">
                <div ref="playBtn" v-show="songs.length>0" class="play">
                    <i class="icon-play"></i>
                    <span class="text">随机播放全部</span>
                </div>
            </div>
            <div class="filter" ref="filter"></div>
        </div>
        <div class="bg-layer" ref="layer"></div>
        <scroll @scroll="scroll" :probe-type="probeType" :listen-scroll="listenScroll" :data="songs" class="list" ref="list">
            <div class="song-list-wrapper">
                <song-list @select="selectItem" :songs="songs"></song-list>
            </div>
            <div v-show="!songs.length" class="loading-container">
                <loading></loading>
            </div>
        </scroll>
    </div>
</template>
<script>
import Scroll from '../../base/scroll/scroll'
import SongList from '../../base/song-list/song-list'
import { prefixStyle } from '../../assets/js/dom'
import Loading from '../../base/loading/loading'
import { mapActions} from 'vuex'

const transform = prefixStyle('transform')
const backdrop = prefixStyle('backdrop-filter')

const RESERVED_HEIGHT = 40

export default {
    props: {
        bgImage: {
            type: String,
            default: ''
        },
        songs: {
            type: Array,
            default: []
        },
        title: {
            type: String,
            default: ''
        }
    },
    data() {
        return {
            scrollY: 0
        }
    },
    computed: {
        bgStyle() {
            return `background-image:url(${this.bgImage})`
        }
    },
    created() {
        this.probeType = 3
        this.listenScroll = true
    },
    mounted() {
        this.imageHeight = this.$refs.bgImage.clientHeight
        this.minTransalteY = -this.imageHeight + RESERVED_HEIGHT
        this.$refs.list.$el.style.top = `${this.imageHeight}px`
    },
    methods: {
        scroll(pos) {
            this.scrollY = pos.y
        },
        back() {
            this.$router.back()
        },
        selectItem(item, index) {
            this.selectPlay({
                list: this.songs,
                index
            })
        },
        ...mapActions([
            'selectPlay'
        ])
    },
    components: {
        Scroll,
        Loading,
        SongList
    },
    watch: {
        scrollY(newVal) {
            let translateY = Math.max(this.minTransalteY, newVal)
            let scale = 1
            let zIndex = 0
            let blur = 0
            const percent = Math.abs(newVal / this.imageHeight)
            if (newVal > 0) {
                scale = 1 + percent
                zIndex = 10
            } else {
                blur = Math.min(20, percent * 20)
            }

            this.$refs.layer.style[transform] = `translate3d(0,${translateY}px,0)`
            this.$refs.filter.style[backdrop] = `blur(${blur}px)`
            if (newVal < this.minTransalteY) {
                zIndex = 10
                this.$refs.bgImage.style.paddingTop = 0
                this.$refs.bgImage.style.height = `${RESERVED_HEIGHT}px`
                this.$refs.playBtn.style.display = 'none'
            } else {
                this.$refs.bgImage.style.paddingTop = '70%'
                this.$refs.bgImage.style.height = 0
                this.$refs.playBtn.style.display = ''
            }
            this.$refs.bgImage.style[transform] = `scale(${scale})`
            this.$refs.bgImage.style.zIndex = zIndex
        }
    }
}
</script>
<style lang="less">
.music-list {
    position: fixed;
    z-index: 100;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: #222;
    .back {
        position: absolute;
        top: 0;
        left: 6px;
        z-index: 50;
        .icon-back {
            display: block;
            padding: 10px;
            font-size: 13px;
            color: #ffcd32;
        }
    }
    .title {
        position: absolute;
        top: 0;
        left: 10%;
        z-index: 40;
        width: 80%;
        white-space: nowrap;
        text-align: center;
        line-height: 40px;
        font-size: 12px;
        color: #fff;
    }
    .bg-image {
        position: relative;
        width: 100%;
        height: 0;
        padding-top: 70%;
        transform-origin: top;
        background-size: cover;
        .filter {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(7, 17, 27, 0.4);
        }
        .play-wrapper {
            position: absolute;
            bottom: 20px;
            z-index: 50;
            width: 100%;
            .play {
                box-sizing: border-box;
                width: 135px;
                padding: 7px 0;
                margin: 0 auto;
                text-align: center;
                border: 1px solid #ffcd32;
                color: #ffcd32;
                border-radius: 100px;
                font-size: 0;
                .icon-play {
                    display: inline-block;
                    vertical-align: middle;
                    margin-right: 6px;
                    font-size: 13px;
                }
                .text {
                    display: inline-block;
                    vertical-align: middle;
                    font-size: 14px;
                }
            }
        }
    }
    .bg-layer {
        position: relative;
        height: 100%;
        background: #222;
    }
    .list {
        position: fixed;
        top: 0;
        bottom: 0;
        width: 100%;
        background: #222; // overflow: hidden;
        .song-list-wrapper {
            padding: 20px 30px;
        }
    }
    .loading-container {
        position: absolute;
        width: 100%;
        top: 50%;
        transform: translateY(-50%);
    }
}
</style>
