<template>
  <div class='SearchBox header-search'>
    <input v-model="keyword" type="text" placeholder="搜索音乐...追梦vue3实战"/>
    <SearchOutlined />
    <CloseCircleOutlined @click.native="clearKeyword()" v-if="keyword"/>

    <div class="search-list" v-if="keyword">
      <a-menu>
        <a-menu-item
          v-for="(item,index) in songList"
          :key="item.id"
          @click="handlePlay(songList,index)"
        >{{index+1}}: {{item['name']}} - {{item['alias'].length?item['alias'][0]:item['album']['name']}}</a-menu-item>
        <a-menu-divider />

      </a-menu>
    </div>
  </div>
</template>

<script setup>
import {reactive, toRefs, watchEffect} from "vue"
import api from '@/common/baseProxy'
import musicPlayTool from '@/utils/musicPlayTool'

const searchState = reactive({
  keyword: '',
	songList: [],
  clearKeyword(){
    this.keyword = ''
  }
})

const searchMusic = async (keyword)=>{
	try {
		const res = await $axios.get(api + '/search?keywords='+keyword)

		searchState.songList = res['data']['result']['songs']
	}catch (e) {
		console.log(e.message)
	}
}

const handlePlay = (songList,index) => {
	musicPlayTool(songList,index)
	searchState.keyword = ''
}


watchEffect(()=>{
	if (searchState.keyword !== ''){
		searchMusic(searchState.keyword)
	}
})


const {keyword,clearKeyword,songList} = toRefs(searchState)

</script>

<style lang="less" scoped>

.header-search{
  height: 26px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: relative;

  .anticon-search{
    position: absolute;
    right: 10px;
    opacity: 0.7;
    cursor: pointer;

  }
  .anticon-close-circle{
    position: absolute;
    right: 30px;
    color: rgba(0,0,0,.5);
    cursor: pointer;
  }
  input{
    width: 240px;
    height: 26px;
    border: 0;
    outline: none;
    border-radius: 30px;
    background: rgba(0,0,0,0.2);
    display: block;
    color: #eee;
    text-indent: 1em;
    font-size: 12px;
  }
  input::-webkit-input-placeholder {
    color: #ccc;
    font-size: 12px;
  }

  .search-list{
    position: absolute;
    top: 40px;
    left: 0;
    width: 300px;
    max-height: 350px;
	  overflow-x: hidden;
	  overflow-y: auto;
    background-color: #fff;
    box-shadow: 0 0 20px 0px rgba(0,0,0,0.2);
  }
}
</style>