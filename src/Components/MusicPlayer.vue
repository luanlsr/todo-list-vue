<template>
    <WDropfiles
        class="player"
        :borderRadius="0"
        :borderWidth="0"
        :changeItemsAudio="changeItemsAudio"
        @get-files="dropFiles"
        @drop-error="dropError"
    >

        <template v-if="list.length===0">

            <div style="display:flex; align-items:center; justify-content:center; height:100%;">
                <div :style="`text-align:center; width:300px; color:${dropTextColor};`">
                    <div>{{textDrop}}</div>
                    <div style="margin-top:5px; font-size:0.8rem; opacity:0.75;">{{textDropMsg}}</div>
                    <div style="margin-top:5px; font-size:0.8rem; color:#f26;" v-if="textDropError">{{textDropError}}</div>
                </div>
            </div>

        </template>

        <template v-else>

            <div
                ref="menuPanel"
                :class="`${isScrollTop?'':'menuShadow'}`"
            >

                <div :style="`font-size:0.8rem; display:flex; align-items:center; padding:10px; color:${menuTextColor};`">
                    <div style="white-space:nowrap; padding-right:5px; opacity:0.5;">{{textPlayItem}}:</div>
                    <input style="background:transparent; border:0px; width:100%; color:inherit; font-family:inherit; outline:none;" type="text" spellcheck="false" :value="getPlayingItemName">
                </div>

                <div style="padding:0px 10px 14px 10px;">
                    <div style="display:flex; align-items:center;">

                        <div ref="barPanel" :style="`background:${barBackgroundColor}; padding:0px 0px; width:100%; height:${barHeight}px; border-radius:10px; cursor:pointer;`" @click="adSeek">
                            <div :style="`background:${barColor}; width:${barWidth}%; height:${barHeight}px; border-radius:10px;`"></div>
                        </div>

                        <div style="padding-left:10px; text-align:center; font-size:0.7rem; letter-spacing:1px; min-height:16px;">
                            <span :style="`color:${menuTextColor};`">{{cPlayTime}}</span>
                        </div>

                    </div>
                </div>

                <div style="padding:0px 12px 15px 12px;">
                    <div style="display:flex; align-items:center; justify-content:space-between;">
                        <div style="margin-right:5px; cursor:pointer;">
                            <WIconSvg
                                :title="textTitlePause"
                                :path="mdiPauseCircleOutline"
                                :size="40"
                                :color="menuIconColorActive"
                                :colorHover="menuIconColorActive"
                                v-if="cPlayMode==='play'"
                                @click.native="adPause"
                            ></WIconSvg>
                            <WIconSvg
                                :title="textTitlePlay"
                                :path="mdiPlayCircleOutline"
                                :size="40"
                                :color="menuIconColorActive"
                                :colorHover="menuIconColorActive"
                                v-if="cPlayMode==='pause' || cPlayMode==='stop'"
                                @click.native="if(cPlayMode==='pause'){adResume()}else{adPlay(0)}"
                            ></WIconSvg>
                        </div>
                        <div :style="`margin-right:5px; ${cPlayNextMode==='loop'?'':'cursor:pointer'}`" :title="textTitleLoop">
                            <WIconSvg
                                :path="mdiSortDescending"
                                :size="22"
                                :color="cPlayNextMode==='loop'?menuIconColorActive:menuIconColor"
                                :colorHover="menuIconColorActive"
                                @click.native="cPlayNextMode='loop'"
                            ></WIconSvg>
                        </div>
                        <div :style="`margin-right:5px; ${cPlayNextMode==='repeat'?'':'cursor:pointer'}`" :title="textTitleRepeat">
                            <WIconSvg
                                :path="mdiReplay"
                                :size="22"
                                :color="cPlayNextMode==='repeat'?menuIconColorActive:menuIconColor"
                                :colorHover="menuIconColorActive"
                                @click.native="cPlayNextMode='repeat'"
                            ></WIconSvg>
                        </div>
                        <div :style="`margin-right:5px; ${cPlayNextMode==='random'?'':'cursor:pointer'}`" :title="textTitleRandom">
                            <WIconSvg
                                :path="mdiShoePrint"
                                :size="22"
                                :color="cPlayNextMode==='random'?menuIconColorActive:menuIconColor"
                                :colorHover="menuIconColorActive"
                                @click.native="cPlayNextMode='random'"
                            ></WIconSvg>
                        </div>
                        <div style="margin-right:5px; cursor:pointer;" :title="textTitleDeleteAll">
                            <WIconSvg
                                :path="mdiDelete"
                                :size="22"
                                :color="menuIconColor"
                                :colorHover="menuIconColorActive"
                                @click.native="deleteAllItems"
                            ></WIconSvg>
                        </div>
                    </div>
                </div>

            </div>

            <WPanelScrolly
                class="drop"
                :barColor="scrollBarColor"
                :barColorHover="scrollBarColorHover"
                @scroll="scrollList"
            >
                <table style="width:100%; border-collapse:collapse;">
                    <tbody>
                        <tr
                            :style="`transition:all 0.25s linear; font-size:0.8rem; height:${itemHeightMin}px; ${iPlayItem===kitem?'':'cursor:pointer;'} color:${ iPlayItem===kitem?itemTextColorActive:item.mouseIn?itemTextColorHover:itemTextColor }`"
                            :key="kitem"
                            v-for="(item,kitem) in list"
                            @mouseenter="item.mouseIn=true"
                            @mouseleave="item.mouseIn=false"
                        >
                            <td
                                :style="`border-bottom:1px solid ${itemSeplineColor}; padding-left:15px; padding-right:15px; text-align:left;`"
                                @click="adPlay(kitem)"
                            >
                                {{kitem+1}}.
                            </td>
                            <td
                                :style="`border-bottom:1px solid ${itemSeplineColor}; padding:5px 0px; width:100%; text-align:left;`"
                                @click="adPlay(kitem)"
                            >
                                {{item.name}}
                            </td>
                            <td
                                :style="`border-bottom:1px solid ${itemSeplineColor}; padding-left:13px; padding-right:19px; user-select:none;`"
                            >
                                <WIconSvg
                                    style="cursor:pointer;"
                                    :path="mdiTrashCanOutline"
                                    :size="17"
                                    :color="`${iPlayItem===kitem?itemTextColorActive:itemTextColor}`"
                                    :colorHover="`${iPlayItem===kitem?itemTextColorActive:itemTextColorHover}`"
                                    @click.native="deleteItem(kitem)"
                                ></WIconSvg>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </WPanelScrolly>

        </template>

    </WDropfiles>
</template>

<script>
import { mdiPlayCircleOutline, mdiPauseCircleOutline, mdiStopCircleOutline, mdiTrashCanOutline, mdiDelete, mdiShoePrint, mdiSortDescending, mdiReplay } from '@mdi/js'
import each from 'lodash/each'
import size from 'lodash/size'
import isEqual from 'lodash/isEqual'
import concat from 'lodash/concat'
import cloneDeep from 'lodash/cloneDeep'
import pullAt from 'lodash/pullAt'
import random from 'lodash/random'
import split from 'lodash/split'
import last from 'lodash/last'
import arrHas from 'wsemi/src/arrHas.mjs'
import WHowler from 'w-howler'
import WDropfiles from 'w-component-vue/src/components/WDropfiles.vue'
import WIconSvg from 'w-component-vue/src/components/WIconSvg.vue'
import WPanelScrolly from 'w-component-vue/src/components/WPanelScrolly.vue'
function getFileName(str) {
    return str.split('\\').pop().split('/').pop()
}

export default {
    components: {
        WDropfiles,
        WIconSvg,
        WPanelScrolly,
    },
    props: {
        items: {
            type: Array,
            default: () => [],
        },
        dropTextColor: {
            type: String,
            default: '#aaa',
        },
        backgroundColor: {
            type: String,
            default: '#323232',
        },
        barHeight: {
            type: Number,
            default: 6,
        },
        barColor: {
            type: String,
            default: '#f1d895',
        },
        barBackgroundColor: {
            type: String,
            default: '#222',
        },
        menuTextColor: {
            type: String,
            default: '#aaa',
        },
        menuIconColor: {
            type: String,
            default: '#aaa',
        },
        menuIconColorActive: {
            type: String,
            default: '#f1d895',
        },
        itemTextColor: {
            type: String,
            default: '#aaa',
        },
        itemTextColorActive: {
            type: String,
            default: '#f1d895',
        },
        itemTextColorHover: {
            type: String,
            default: '#ccc',
        },
        itemHeightMin: {
            type: Number,
            default: 48,
        },
        itemSeplineColor: {
            type: String,
            default: '#444',
        },
        scrollBarColor: {
            type: String,
            default: 'rgba(200,200,200,0.2)',
        },
        scrollBarColorHover: {
            type: String,
            default: 'rgba(200,200,200,0.3)',
        },
        textDrop: {
            type: String,
            default: 'Vue Player',
        },
        textDropMsg: {
            type: String,
            default: 'Arraste seu arquivo de música para cá',
        },
        textPlayItem: {
            type: String,
            default: 'Now',
        },
        textWaitUserPlay: {
            type: String,
            default: `Hora de ouvir uma música`,
        },
        textTitlePlay: {
            type: String,
            default: 'Play',
        },
        textTitlePause: {
            type: String,
            default: 'Pause',
        },
        textTitleLoop: {
            type: String,
            default: 'Loop play',
        },
        textTitleRepeat: {
            type: String,
            default: 'Repeat play',
        },
        textTitleRandom: {
            type: String,
            default: 'Random play',
        },
        textTitleDeleteAll: {
            type: String,
            default: 'Delete all',
        },
    },
    data: function() {
        return {
            mdiPlayCircleOutline,
            mdiPauseCircleOutline,
            mdiStopCircleOutline,
            mdiTrashCanOutline,
            mdiDelete,
            mdiShoePrint,
            mdiSortDescending,
            mdiReplay,
            wh: null,
            itemsTemp: [],
            list: [],
            iPlayItem: null,
            cPlayItem: '',
            cPlayMode: 'stop',
            cPlayNextMode: 'loop', //loop, repeat, random
            cPlayTime: '',
            barWidth: 0,
            isScrollTop: true,
            codecs: ['mp3', 'mpeg', 'opus', 'ogg', 'oga', 'wav', 'aac', 'caf', 'm4a', 'mp4', 'weba', 'webm', 'dolby', 'flac'],
            textDropError: null,
        }
    },
    mounted: function() {
        let vo = this
        //WHowler
        vo.wh = new WHowler()
        vo.wh.on('refresh', (s) => {
            vo.adRefreshBar(s)
        })
        vo.wh.on('end', () => {
            vo.adPlayNext()
        })
    },
    beforeDestroy: function() {
        let vo = this
        //stop
        vo.wh.stop()

        vo.deleteAllItems()
    },
    computed: {
        changeItemsAudio: function() {
            let vo = this
            let items = cloneDeep(vo.items)
            if (size(vo.items) === 0) {
                return ''
            }
            if (!isEqual(vo.items, vo.itemsTemp)) {
                vo.addItems(items, 'urls')
                vo.itemsTemp = cloneDeep(vo.items)
            }
            return ''
        },
        getPlayingItemName: function() {
            let vo = this
            let r = vo.textWaitUserPlay
            if (vo.cPlayItem !== '') {
                r = vo.cPlayItem
            }
            return r
        },
    },
    methods: {
        tempAsync: async function() {
        },
        adRefreshBar: function(s) {
            let vo = this
            vo.barWidth = s.prog
            vo.cPlayTime = s.timeShow
        },
        adSeek: function(e) {
            let vo = this
            let x = e.offsetX
            let w = vo.$refs.barPanel.clientWidth
            let r = x / w
            vo.wh.seek(r)
        },
        adPlayNext: function(bDelete = false) {
            let vo = this
            let n = size(vo.list)
            let kitem
            if (!bDelete) {
                if (vo.cPlayNextMode === 'random') {
                    kitem = vo.randomItemEx(vo.iPlayItem, n)
                }
                else if (vo.cPlayNextMode === 'repeat') {
                    kitem = vo.iPlayItem
                }
                else {
                    kitem = vo.iPlayItem + 1
                    if (kitem > size(vo.list) - 1) {
                        kitem = 0
                    }
                }
            }
            else {
                if (vo.cPlayNextMode === 'random') {
                    kitem = vo.randomItem(n)
                }
                else {
                    kitem = vo.iPlayItem
                    if (kitem > n - 1) {
                        kitem = 0
                    }
                }
            }
            //adPlay force
            vo.adPlay(kitem, true)
        },
        adPause: function() {
            let vo = this
            //pause
            vo.wh.pause()
            vo.cPlayMode = 'pause'
        },
        adResume: function() {
            let vo = this
            //resume
            vo.wh.resume()
            vo.cPlayMode = 'play'
        },
        adReset: function() {
            let vo = this
            //use pause to stop
            vo.wh.pause()
            vo.cPlayMode = 'stop'
            vo.barWidth = 0
        },
        adPlay: function(kitem, bForce = false) {
            let vo = this
            //check
            if (vo.iPlayItem !== kitem || bForce) {
                let item = vo.list[kitem]
                vo.iPlayItem = kitem
                vo.cPlayItem = item.name
                //reset
                vo.barWidth = 0
                vo.cPlayTime = ''
                //play
                vo.wh.play(item.src, item.ext)
                vo.cPlayMode = 'play'
            }
        },
        deleteAllItems: function() {
            let vo = this
            vo.adReset()
            vo.iPlayItem = null
            vo.cPlayItem = ''
            vo.list = []
        },
        deleteItem: function(kitem) {
            let vo = this
            if (size(vo.list) > 1) {
                let list = cloneDeep(vo.list)
                pullAt(list, kitem)
                vo.list = list
                if (vo.iPlayItem === kitem) {
                    vo.adPlayNext(true)
                }
                else if (kitem > vo.iPlayItem) {
                    //PlayItem
                }
                else {
                    vo.iPlayItem -= 1
                }
            }
            else {
                vo.deleteAllItems()
            }
        },
        randomItem: function(n) {
            let r = random(n - 1)
            return r
        },
        randomItemEx: function(i, n) {
            let r
            if (n === 1) {
                r = 0
            }
            else if (n === 2) {
                if (i === 0) {
                    r = 1
                }
                else {
                    r = 0
                }
            }
            else if (n > 2) {
                r = random(n - 1)
                while (r === i) {
                    r = random(n - 1)
                }
            }
            else {
                r = 0
            }
            return r
        },
        scrollList: function(e) {
            let vo = this
            vo.isScrollTop = e.r === 0
        },
        addFiles: function (items) {
            let vo = this
            let list = []
            each(items, function(item) {
                let name = item.name
                let s = split(name, '.')
                let ext = last(s)
                let b = arrHas(ext, vo.codecs)
                if (b) {
                    let src = URL.createObjectURL(item.file)
                    list.push({
                        name,
                        ext,
                        src,
                        mouseIn: false,
                    })
                }
            })
            return list
        },
        addUrls: function (items) {
            let vo = this
            let list = []
            each(items, function(item) {
                let name = getFileName(item.name)
                let s = split(name, '.')
                let ext = last(s)
                let b = arrHas(ext, vo.codecs)
                if (b) {
                    list.push({
                        name,
                        ext,
                        src: item.url,
                        mouseIn: false,
                    })
                }
            })
            return list
        },
        addItems: function (items, mode = 'files', autoPlay = false) {
            let vo = this
            setTimeout(() => {
                //list
                let list = cloneDeep(vo.list)
                if (mode === 'files') {
                    list = concat(list, vo.addFiles(items))
                }
                else {
                    list = concat(list, vo.addUrls(items))
                }
                vo.list = list
                //check
                if (autoPlay) {
                    if (vo.iPlayItem === null) {
                        vo.adPlay(0)
                    }
                }
            }, 1)
        },
        dropFiles: function({ filesTree }) {
            // console.log('methods dropFiles', filesTree)
            let vo = this
            //textDropError
            vo.textDropError = null
            //addItems, 改用timer會直接往後呼叫cb
            vo.addItems(filesTree, 'files', true)
        },
        dropError: function(err) {
            let vo = this
            //textDropError
            vo.textDropError = err
        },
    },
}
</script>

<style scoped>
    .menuShadow {
        box-shadow: 0px 10px 5px -3px rgba(0, 0, 0, 0.2), 0px 0px 0px 0px #000, 0px 0px 0px 0px #000;
    }

    .player {
        /* position: fixed; */
        background:#323232;
        display: flex;
        flex-direction: column;
        max-width: 500px;
        margin-top: 100px;
        margin-bottom: 50px;
    }

    .drop {
        height: inherit;
    }

    .player:hover {
        background-color: transparent !important;
        cursor: pointer;
    }

    @media only screen and (max-width: 600px) {
        .player {
            max-width: 350px;
        }
}
</style>
