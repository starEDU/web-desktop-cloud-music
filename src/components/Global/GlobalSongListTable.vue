<template>
	<div class='GlobalSongListTable'>

		<a-table
			class="ant-table-striped"
			:columns="columns"
			:data-source="songList"
			bordered
			rowKey="id"
			:row-selection="{
        type: 'radio',
        selectedRowKeys: selectedRowKeys,
        onChange: onSelectChange
      }"
			:custom-row="customRow"
			:rowClassName="(record, index) => (index % 2 === 1 ? 'table-striped' : null)"
		>
			<template #footer>
				<span style="color: red;">双击表格播放</span>
			</template>
		</a-table>

	</div>
</template>

<script>
import {reactive, toRefs, watchEffect} from "vue"
import getSingerName from "@/utils/getSingerName"
import {timeFormat} from "@/utils/format"
import downloadMusic from "@/utils/downloadMusic"
import {useRouter} from "vue-router"
import {useStore} from "vuex"
import musicPlayTool from "@/utils/musicPlayTool"

export default {
	name: "GlobalSongListTable",
	props: ['songList'],
	setup(props) {
		const Router = useRouter()
		const {dispatch, commit,} = useStore()

		const songListTableState = reactive({
			columns: [
				{
					title: '序号',
					dataIndex: 'id',
					width: 60,
					customRender({index}) {
						return index + 1
					}
				},
				{
					title: '歌曲名称',
					dataIndex: 'name',
					customRender({index, text, record,}) {
						// jsx: 在js里面写xml(html) ，用react框架语法
						return (
							<a
								href="javascript:;"
								title={record.name}
								onClick={async () => {
									commit('setPlaySongList', props.songList)
									commit('setCurrentPlayIndex', index)
									await musicPlayTool(props.songList, index)
									await Router.push('/song?id=' + record.id)
								}}
							>
								<img src={(record['album'] ? record['album']['picUrl'] : record['al']['picUrl']) + '?param=50y50'}
								     alt=""/>
								&nbsp;
								&nbsp;
								{text}
							</a>
						)
					}
				},
				{
					title: '歌手',
					dataIndex: 'ar',
					customRender({index, text, record,}) {
						return text ? getSingerName(text) : getSingerName(record['artists'])
					}
				},
				{
					title: '专辑',
					dataIndex: 'al',
					customRender({index, text, record,}) {
						return text ? (text.name) : (record['album']['name'])
					}
				},
				{
					title: '时长',
					width: 80,
					dataIndex: 'dt',
					customRender({index, text, record,}) {
						return text ? timeFormat(text / 1000) : timeFormat(record['duration'] / 1000)
					}
				},
				{
					title: '下载',
					customRender({index, text, record,}) {
						const name = record.name
						return (
							<u
								style="cursor: pointer;"
								title="下载"
								onClick={() => {
									console.log(record.id)
									console.log(record.name)
									downloadMusic(record.id, record.name)
								}}
							>
								{name}
							</u>
						)
					}
				}
			],

			selectedRowKeys: [],
			onSelectChange(selectedRowKeys) {
				songListTableState.selectedRowKeys = selectedRowKeys;
			},
			customRow(record, index) {
				return {
					onDblclick: async (event) => {
						songListTableState.selectedRowKeys = [record.id];

						// 把当前的歌曲列表 挂载到store中，方便实现播放模式
						commit('setCurrentPlayIndex', index)
						commit('setPlaySongList', props.songList)

						//const r = await dispatch('getMusicUrl',{id:33894312})

						await musicPlayTool(props.songList, index)
						commit('setIsPlayState', true)
					},
				}
			}
		})


		return {
			...toRefs(songListTableState),
		}
	}
}
</script>

<style lang="less" scoped>
.ant-table-striped :deep(.table-striped) td {
	background-color: #fafafa !important;
}
</style>