<template>
    <div class="singer">
        <ListView @select="selectSinger" :data="singers"></ListView>
        <router-view></router-view>
    </div>
</template>

<script>
    import { getSingertList } from 'api/singer'
    import { ERR_OK } from 'api/config'
    import Singer from 'common/js/singer'
    import ListView from 'base/listview/listview'
    import { mapMutations } from 'vuex'

    const HOT_NAME = '热门'
    const HOT_SINGER_LEN = 10

    export default {
        data() {
            return {
                singers: []
            }
        },
        created() {
            this._singersList()
        },
        methods: { 
            selectSinger(singer) {
                this.$router.push({
                    path: `/singer/${singer.id}`
                })
                this.setSinger(singer)
            },
            _singersList() {
                getSingertList().then(res => {
                    if( res.code === ERR_OK ) {
                        this.singers = this._normalizeSinger(res.data.list)
                    }
                })
            },
            _normalizeSinger(list){
                let map = {
                    hot: {
                        title: HOT_NAME,
                        items: []
                    }
                }

                list.forEach((item, index) => {
                    if ( index < HOT_SINGER_LEN ) {
                        map.hot.items.push(new Singer({
                            id: item.Fsinger_mid,
                            name: item.Fsinger_name
                        }))
                    }

                    const key = item.Findex
                    if (!map[key]) {
                        map[key] = {
                            title: key,
                            items: []
                        }
                    }
                    map[key].items.push(new Singer({
                        id: item.Fsinger_mid,
                        name: item.Fsinger_name
                    }))
                })

                //为了得到有序列表，我们需要处理 map
                let  hot = []
                let  ret = []
                for(let key in map) {
                    let val = map[key]
                    if (val.title.match(/[a-zA-z]/)) {
                        ret.push(val)
                    } else if (val.title === HOT_NAME) {
                        hot.push(val)
                    }
                }
                ret.sort((a, b) => {
                    return a.title.charCodeAt(0) - b.title.charCodeAt(0)
                })
                // console.log(hot.concat(ret))
                return hot.concat(ret)
            },
            ...mapMutations({
                setSinger: 'SET_SINGER'
            })
        },
        components: {
            ListView
        }
    }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
     .singer
        position: fixed
        top: 88px
        bottom: 0
        width: 100%
</style>